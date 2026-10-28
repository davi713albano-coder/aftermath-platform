# 19 — Riscos, decisões pendentes e alternativas

## 19.1 Riscos

| # | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| R-1 | OAuth2 Roblox **beta**; app em modo privado **máx. 10 usuários** até aprovação de produção [CONFIRMADO] | Média | Alto — trava vinculação em massa | Registrar app na P1 e pedir aprovação imediatamente; contingência: torneio fechado ≤10 até aprovação; monitorar docs |
| R-2 | `is-owned` depende de privacidade do jogador; respostas 403 ambíguas [TESTAR] | Alta | Médio — falsos "sem acesso" | Máquina de estados com INCONCLUSIVE + `can-view-inventory` + mensagens orientando o jogador; probe na P2 |
| R-3 | Bloqueio anti-bot Roblox a IPs de datacenter [INCERTO — 403 observado nesta sessão] | Média | Alto se produção em datacenter | Probe residencial; se confirmado, saída por IP residencial/proxy confiável ou alternativa com API key do dono (cloud v2 inventory-items) |
| R-4 | Intents privilegiadas do Discord exigem revisão se app crescer (>10k usuários) [CONFIRMADO] | Baixa | Baixo/Médio | Solicitar só `GUILD_MEMBERS`; sem `MESSAGE_CONTENT` |
| R-5 | Rate limits Discord/Roblox em pico (anúncios + logs simultâneos) | Média | Baixo | Outbox com bucket + throttle; orçamento global Roblox |
| R-6 | SSE bloqueado por proxy/CDN do ambiente de deploy | Média | Baixo | Fallback polling automático (10 s) já previsto |
| R-7 | Relógios divergentes (duração de partida) | Baixa | Baixo | Timestamps via `now()` do Postgres em transação |
| R-8 | Abuso de re-tentativa de verificação | Média | Baixo | Rate limit 5/min + orçamento global + circuit breaker |
| R-9 | Correções de resultado em cadeia afetando partidas futuras | Média | Médio | Dry-run de impactos + bloqueio + invalidação explícita + histórico |
| R-10 | OAuth2 beta do Roblox mudar escopos/contratos | Média | Médio | Camada `RobloxIdentityProvider` isolada; fixtures + probe contratual |

## 19.2 Alternativas avaliadas

| Tema | Escolhida | Alternativa | Por que a escolhida |
|---|---|---|---|
| API | Route handlers Next (`/api/v1`) | API Fastify/Nest separada | Menos superfície p/ v1; extração futura barata (core isolado) |
| Fila | BullMQ + Redis 7 | pg-boss (Postgres) | Redis já necessário p/ lock/rate-limit (padrão do repo atual); pg-boss é plano B de 1 dependência a menos |
| Sessão | Auth.js v5 JWT | Database sessions (adapter) | JWT simplifica scale-out; permissões ficam no banco de qualquer forma |
| Realtime | SSE + polling fallback | WebSocket | Só precisa server→client; SSE é HTTP puro, simples atrás de proxies |
| Host | VPS + Compose | Cloud Run (web) + VPS (bot); Render/Railway all-in-one | Bot requer processo persistente; Compose dá controle e custo previsível |
| Verificação Roblox | `is-owned` público | cloud v2 inventory-items (API key do dono) | Endpoint indicado pelo dono; alternativa robusta pronta se R-2/R-3 confirmarem |
| Vínculo Roblox | OAuth2 Open Cloud | none (só username digitado) | Username digitado não é prova; OAuth dá ID estável + avatar oficial |

## 19.3 DECISÕES QUE EXIGEM APROVAÇÃO

| # | Decisão | Recomendação | Alternativa |
|---|---|---|---|
| D-01 | Hosting da v1 | VPS + Docker Compose (um provedor, ex.: Hetzner/Contabo/Scaleway) | Cloud Run p/ web + VPS p/ bot |
| D-02 | API dentro do Next vs API separada | Dentro do Next | Fastify separado |
| D-03 | Sessão JWT (Auth.js) vs sessão em banco | JWT + permissões do banco | Database sessions |
| D-04 | Fila BullMQ+Redis vs pg-boss | BullMQ+Redis | pg-boss |
| D-05 | Não armazenar tokens OAuth Roblox | Confirmar (re-vincula quando necessário) | Armazenar refresh token criptografado |
| D-06 | Verificação primária `is-owned` + fallback `can-view` + alternativa cloud v2 | Confirmar ordem | Somente cloud v2 c/ API key |
| D-07 | Uso do Google AI Studio Build | Apenas protótipo de telas; produção no repo | Nenhum uso |
| D-08 | Formato v1: eliminação simples | Confirmar | incluir round-robin de grupos na v1 |
| D-09 | Cadeia de hash no AuditLog (tamper-evidence) | Fase P5 (opcional) | sem cadeia |
| D-10 | Domínio/e-mail p/ OAuth apps e redirects | Definir antes da P1 | — |
| D-11 | Política de suplentes default (quantos, contam p/ prontidão?) | 2 suplentes, não contam p/ prontidão mínima | configurável por torneio (já é) |
| D-12 | Aprovar plano de fases (20) e começar P0 | Sim | reordenar prioridades |

## 19.4 Incertezas que não bloqueiam o começo

- Contrato exato do `is-owned` (probe P2 — a máquina de estados tolera qualquer resposta).
- Prazo da aprovação do app OAuth Roblox (R-1) — o desenvolvimento continua com mock provider.
- Comportamento exato de DMs bloqueadas (teste de staging P5).
