# 06 — Diagramas

## 6.1 Diagrama de componentes

```mermaid
flowchart LR
    subgraph Clientes
        B[Navegador - jogador/admin]
        D[Discord]
    end

    subgraph Plataforma
        W[apps/web - Next.js 16<br/>site + painel + API /api/v1 + SSE]
        BOT[apps/bot - discord.js<br/>stateless]
        WRK[apps/worker - BullMQ<br/>outbox / roblox / cron]
    end

    subgraph Infra
        PG[(PostgreSQL 16<br/>fonte única de verdade)]
        RD[(Redis 7<br/>filas + locks + rate limit)]
    end

    subgraph Externos
        DAPI[Discord REST/Gateway]
        ROB[Roblox APIs<br/>OAuth2 / inventário / thumbnails]
    end

    B -->|HTTPS + cookie sessão| W
    D <-->|gateway intents| BOT
    BOT -->|REST + service token + ator| W
    W -->|Prisma| PG
    W -->|BullMQ producer| RD
    WRK <-->|BullMQ consumer| RD
    WRK -->|Prisma| PG
    WRK -->|REST outbox| DAPI
    WRK -->|checagens c/ orçamento| ROB
    W -->|OAuth callbacks| ROB
    W -->|login OAuth2| DAPI
```

## 6.2 Modelo entidade-relacionamento (simplificado)

```mermaid
erDiagram
    USER ||--o| ROBLOX_ACCOUNT : "1:1 vinculado"
    USER ||--o{ TEAM_MEMBER : "participa"
    USER ||--o{ INVITE_RECEIVED : "recebe"
    USER ||--o{ ADMIN_GRANT : "tem"
    TOURNAMENT ||--o{ TEAM : "tem"
    TOURNAMENT ||--o{ BRACKET : "1 versão ativa"
    TOURNAMENT ||--o{ CONFIG_VERSION : "versionado"
    TEAM ||--o{ TEAM_MEMBER : "composta"
    TEAM ||--o{ INVITE : "recebe"
    BRACKET ||--o{ MATCH : "contém"
    MATCH }o--|| TEAM : "teamA/teamB/winner"
    MATCH ||--o{ MATCH_EVENT : "histórico"
    TOURNAMENT ||--o{ DRAW : "sorteios"
    TOURNAMENT ||--o{ PUBLIC_EVENT : "feed público"
    TOURNAMENT ||--o{ OUTBOX_EVENT : "entregas Discord"
    USER ||--o{ AUDIT_LOG : "ator"
```

## 6.3 Sequência — login Discord + bootstrap do proprietário

```mermaid
sequenceDiagram
    actor U as Usuário
    participant W as apps/web
    participant D as Discord OAuth2
    participant PG as PostgreSQL

    U->>W: GET /api/auth/signin/discord
    W->>D: authorize (scope=identify, state anti-CSRF)
    D-->>U: tela de consentimento
    U->>D: autoriza
    D->>W: callback ?code&state
    W->>D: POST token (code)
    W->>D: GET users/@me (id, username, avatar)
    W->>PG: upsert user (por discordId) + sessão
    alt discordId == OWNER_DISCORD_ID (env)
        W->>PG: garante papel OWNER (idempotente) + audit OWNER_BOOTSTRAP
    end
    W-->>U: cookie HttpOnly (JWT) + redirect
```

## 6.4 Sequência — vínculo Roblox e verificação de acesso

```mermaid
sequenceDiagram
    actor U as Jogador
    participant W as apps/web
    participant R as Roblox
    participant Q as Fila (BullMQ)
    participant WK as Worker

    U->>W: GET /api/v1/roblox/link/start (302)
    W-->>U: redirect apis.roblox.com/oauth/v1/authorize (openid profile, state assinado 10 min)
    U->>R: consentimento
    R->>W: callback ?code&state
    W->>R: POST /oauth/v1/token → userinfo
    W->>W: upsert roblox_accounts (robloxId único; conflito → 409)
    W->>Q: enfileira game-access-check (dedup por robloxId)
    WK->>R: GET inventory .../is-owned (+ can-view-inventory se 403)
    alt 200 possui
        WK->>W: state=HAS_ACCESS
    else 403 "don't own"
        WK->>W: state=NO_ACCESS
    else 403 privacidade / 429 / timeout
        WK->>W: state=INCONCLUSIVE + nextAttemptAt (backoff)
    end
    U->>W: GET /api/v1/roblox/status (estado + botão "tentar novamente")
```

## 6.5 Sequência — início e finalização de partida (com concorrência)

```mermaid
sequenceDiagram
    actor A as Admin
    participant C as Cliente (site/bot)
    participant API as /api/v1
    participant PG as PostgreSQL (transação)
    participant O as Outbox→Worker→Discord

    A->>C: /admin start (autocomplete de confrontos pendentes)
    C->>API: POST /admin/matches/:id/start
    API->>PG: BEGIN; UPDATE matches SET status='IN_PROGRESS', started_at=now() WHERE id=? AND status='PENDING'
    alt rowCount = 1
        PG->>PG: match_events(iniciar) + audit + outbox(match.started); COMMIT
        API-->>C: 200 {match}
        O->>O: publica log público (retry com backoff)
    else rowCount = 0
        PG-->>API: ROLLBACK
        API-->>C: 409 CONFLICT_STATUS (alguém iniciou/estado mudou)
    end

    A->>C: /admin finalizar (select entre teamA/teamB)
    C->>API: POST /admin/matches/:id/finish {winnerTeamId}
    API->>PG: valida winner ∈ {A,B}; UPDATE ... WHERE status='IN_PROGRESS'
    PG->>PG: avança chaveamento + match_events + audit + outbox(match.finished); COMMIT
    API-->>C: 200 {match, nextMatch}
    O->>O: log "Equipe X venceu em 14m32s"
```

## 6.6 Fluxo do outbox (requisito de sincronização)

```mermaid
flowchart TB
    TX[Transação de domínio] -->|INSERT outbox_events na mesma tx| PG
    PG -->|job enfileirado pós-commit| RD[(Redis/BullMQ)]
    RD --> WK[Worker]
    WK -->|POST /channels/:id/messages| DC[Discord]
    DC -->|200| WK -->|status=DELIVERED + message_id| PG
    DC -->|429/5xx| WK -->|attempts++, next_attempt_at backoff| PG
    WK -->|N falhas| PG[status=DEAD → alarme]
    CRON[Cron de reconciliação] -->|reenvia PENDING antigos| RD
    SITE[Feed do site] -->|lê public_events| PG
```
