# 08 — API: endpoints e contratos

Base: `/api/v1`. Contratos com schemas Zod em `packages/schemas` (mesma lib usada por web e bot). Erro padrão:

```json
{ "error": { "code": "CONFLICT_MATCH_STATUS", "message": "…", "details": { } } }
```

Códigos: `VALIDATION_ERROR` (400) · `UNAUTHENTICATED` (401) · `FORBIDDEN` (403) · `NOT_FOUND` (404) · `CONFLICT_*` (409: `_MATCH_STATUS`, `_INVITE_STATE`, `_TEAM_MEMBER`, `_CONFIG_INCOMPATIBLE`, `_BRACKET_EXISTS`) · `RATE_LIMITED` (429 + `Retry-After`) · `UPSTREAM_*` (502/504 p/ Discord/Roblox) · `INTERNAL` (500).

Autenticação: cookie de sessão (Auth.js) para navegadores; bot usa `Authorization: Bearer <SERVICE_TOKEN>` + `X-Acting-Discord-Id: <id>` (ator real, validado no banco). Sem credenciais no query string.

## 8.1 Auth (Auth.js v5)

| Método | Rota | Descrição |
|---|---|---|
| GET/POST | `/api/auth/[...nextauth]` | provider Discord (scope `identify`), callback, signout |

## 8.2 Sessão / perfil

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/me` | autenticado | user + roblox (nome/ID/avatar/estado acesso) + equipe atual + permissões efetivas |
| POST | `/me/logout` | autenticado | encerra sessão |

## 8.3 Roblox

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| POST | `/roblox/link/start` | autenticado | gera state assinado (10 min) → redirect OAuth2 Roblox (`openid profile`) |
| GET | `/roblox/callback` | — | valida state → token → userinfo → upsert `roblox_accounts`; 409 se robloxId já vinculado a outro usuário |
| DELETE | `/roblox/link` | autenticado | desvincula (audit) |
| GET | `/roblox/status` | autenticado | estado de acesso (HAS_ACCESS/NO_ACCESS/INCONCLUSIVE/UNKNOWN) + `nextAttemptAt` |
| POST | `/roblox/access-check` | autenticado | força nova checagem (rate limit 5/min; fila) |

## 8.4 Torneio e configuração

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/tournaments` | público | lista (respeita `visibility`) |
| GET | `/tournaments/:id` | público | detalhes + config pública |
| POST | `/admin/tournaments` | `tournaments.config` | cria (dono/admin) |
| PATCH | `/admin/tournaments/:id/config` | `tournaments.config` | valida impacto; `?confirm=false` → 200 com `impacts` + `confirmToken`; `?confirm=true&token=…` → aplica |
| POST | `/admin/tournaments/:id/status` | `tournaments.config` | transições de status com guardas |

**Impacts** (exemplos): `max_teams` abaixo das equipes confirmadas → `INCOMPATIBLE` (bloqueia); `players_per_team` mudou → lista equipes fora do novo tamanho (exige confirm; equipes incompatíveis ficam bloqueadas de READY); formato após chaveamento → bloqueado; datas — warnings.

## 8.5 Equipes e convites

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/teams` | público (conforme visibility) | equipes + composição + prontas |
| POST | `/teams` | autenticado (janela de inscrição) | cria equipe do jogador |
| GET | `/teams/:id` | público/membro | detalhes |
| PATCH | `/teams/:id` | owner da equipe ou `teams.manage` | renomear, etc. |
| POST | `/teams/:id/leave` | membro | sair (owner transfere ou bloqueia) |
| POST | `/teams/:id/invites` | owner da equipe | cria convite (discordId validado; `dm_state` pode sair FAILED sem perder convite) |
| GET | `/me/invites` | autenticado | recebidos/enviados |
| POST | `/invites/:id/accept` | convidado | transação: convite→ACCEPTED + team_member (checa elegibilidade + vagas) |
| POST | `/invites/:id/decline` | convidado | |
| DELETE | `/invites/:id` | owner/`teams.manage` | cancela pendente |

## 8.6 Sorteio e chaveamento

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/bracket` | público | chaveamento (matches, slots, estados, vencedores) |
| POST | `/admin/tournaments/:id/draw/preview` | `draws.run` | prévia determinística (seed provisório) — NÃO persiste |
| POST | `/admin/tournaments/:id/draw/confirm` | `draws.run` | seed novo cripto → grava `draws` + `bracket` + matches em transação; 409 se já existe bracket ativo com partidas iniciadas |
| POST | `/admin/tournaments/:id/draw/redraw` | `bracket.manage` + dono | só se nenhuma partida IN_PROGRESS/COMPLETED; invalida bracket antigo (histórico mantido) |
| GET | `/admin/tournaments/:id/draws` | `draws.run` | histórico |

## 8.7 Partidas

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/admin/tournaments/:id/matches?status=pending` | `matches.view` | lista p/ seleção (autocomplete/select do bot usa isto) |
| POST | `/admin/matches/:id/start` | `matches.manage` | PENDING→IN_PROGRESS; timestamp do servidor; 409 se não PENDING |
| POST | `/admin/matches/:id/finish` | `matches.manage` | `{winnerTeamId}`; valida ∈ {A,B}; calcula duração; avança chaveamento; outbox |
| POST | `/admin/matches/:id/correct` | `matches.manage` + dono (se impacto) | novo vencedor; resposta inclui impactos; `confirm=true` aplica invalidação controlada dos confrontos dependentes (nunca os já iniciados) |
| POST | `/admin/matches/:id/cancel` | `matches.manage` | cancela (razão obrigatória) |
| POST | `/admin/matches/:id/walkover` | `matches.walkover` | W.O. com razão; `wo_grace_minutes` respeitado |
| GET | `/admin/matches/:id/history` | `audit.view` ou responsável | `match_events` |

## 8.8 Administração e logs

| Método | Rota | Permissão | Descrição |
|---|---|---|---|
| GET | `/admin/audit?target=&actor=&action=` | `audit.view` | paginado, append-only |
| GET | `/admin/admins` / POST `/admin/admins` / DELETE `/admin/admins/:discordId` | OWNER (gerenciar admins) | concessão/revogação granular; admin não pode conceder (403) |
| GET | `/me/permissions` | admin | o que o próprio admin pode |
| GET | `/tournaments/:id/events` | público | feed público paginado |
| GET | `/stream/tournaments/:id` | público | **SSE**: eventos `bracket`, `match`, `feed`; heartbeat 25 s; `Last-Event-ID` |
| POST | `/admin/tournaments/:id/announcements` | `announcements.publish` | anúncio + outbox |

## 8.9 Internos (worker/bot)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/internal/health` | service token | liveness web/api |
| POST | `/internal/outbox/ack` | service token (worker) | marca DELIVERED/DEAD (worker já grava via Prisma; rota p/ reconciliação manual) |
| GET | `/internal/bot/matches/select-list` | service token + ator | lista p/ string select (≤25) paginada |

## 8.10 Eventos internos (payloads de outbox/public_events)

`match.started` `{matchId, tournamentId, teams, round}` · `match.finished` `{matchId, winner, durationSeconds}` · `draw.confirmed` `{drawId, size, teamsCount}` · `bracket.updated` `{version, reason}` · `announcement.published` `{title, excerpt}`. Payloads públicos não contêm IDs de admins nem dados privados.
