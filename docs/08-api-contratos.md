# 08 — API: endpoints e contratos (referência canônica)

Base: **`/api/v1`** (route handlers do `apps/web`). Este documento é a **única fonte de contratos** para site, bot, worker e a UI track do AI Studio (21). Schemas Zod em `packages/schemas`, compartilhados por web e bot. **Nenhum outro documento pode definir endpoint, nome de evento ou formato de payload divergente.**

## 8.0 Convenções canônicas

| Tema | Regra |
|---|---|
| JSON | campos em `camelCase` |
| IDs | sempre **string** (UUID interno, snowflake do Discord, ID Roblox) — nunca `number` (perda de precisão em snowflakes) |
| Datas | ISO-8601 UTC com sufixo `Z` (`2026-10-28T18:00:00Z`); durações em segundos inteiros (`durationSeconds`) |
| Enums | `UPPER_SNAKE_CASE` (`IN_PROGRESS`, `SINGLE_ELIMINATION`, `HAS_ACCESS`) |
| Paginação | query `?page=1&limit=25` (máx. 100) → resposta `{ "items": [...], "page": 1, "limit": 25, "total": 42, "hasMore": true }` |
| Erros | envelope único em **todos** os erros: `{ "error": { "code": "…", "message": "…", "details": { … } } }` (`details` opcional) |
| Eventos | catálogo canônico §8.10 — nomes `dotted lowercase` em `public_events`, `outbox_events` e SSE; nenhuma variante (`match_started`, `MATCH_STARTED`) é válida |
| Segredos | nunca em query string; `SERVICE_TOKEN` e cookies só em headers |

**Classes de autenticação** (coluna "Auth" das tabelas abaixo):

- `PUBLIC` — sem auth; somente dados públicos conforme `visibility` do torneio;
- `SESSION` — cookie Auth.js (navegador);
- `ADMIN` — SESSION + permissão granular **recarregada do banco a cada chamada** (09) — esconder botão na UI não é autorização;
- `INTERNAL` — `Authorization: Bearer SERVICE_TOKEN` (bot/worker); quando houver ator humano: header `X-Acting-Discord-Id` (resolvido e autorizado no banco).

**Idempotência**: mutações originadas no bot enviam header `Idempotency-Key` (derivada do `interactionId`); repetição devolve a resposta original sem novo efeito.

**Códigos de erro**: `VALIDATION_ERROR` (400) · `UNAUTHENTICATED` (401) · `FORBIDDEN` (403) · `NOT_FOUND` (404) · `CONFLICT_*` (409: `_MATCH_STATUS`, `_INVITE_STATE`, `_TEAM_MEMBER`, `_CONFIG_INCOMPATIBLE`, `_BRACKET_EXISTS`, `_BRACKET_EDIT`, `_ROBLOX_LINKED`, `_CONFIRM_TOKEN`) · `RATE_LIMITED` (429 + `Retry-After`) · `UPSTREAM_*` (502/504 p/ Discord/Roblox) · `INTERNAL` (500).

## 8.1 Auth (Auth.js v5) — fora de `/api/v1`

| Método | Rota | Descrição |
|---|---|---|
| GET | `/api/auth/signin/discord` | inicia login Discord (scope `identify`) |
| GET | `/api/auth/callback/discord` | callback OAuth2 (valida `state` anti-CSRF) |
| POST | `/api/auth/signout` | logout (limpa cookie) — **é o único logout; não existe `/me/logout`** |

## 8.2 Sessão / perfil

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/me` | SESSION | user + roblox (nome/ID/avatar/estado de acesso) + equipe ativa + permissões efetivas |
| GET | `/me/invites` | SESSION | convites recebidos/enviados |
| GET | `/me/permissions` | SESSION | o que o próprio usuário pode |

## 8.3 Roblox

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/roblox/link/start` | SESSION | gera `state` assinado (HMAC, 10 min, uso único) e responde **302** para o authorize do Roblox (`openid profile`) — navegação sempre via GET |
| GET | `/roblox/callback` | — | valida state/nonce → token → userinfo → upsert `roblox_accounts`; 409 `_ROBLOX_LINKED` se o robloxId já pertence a outro usuário |
| DELETE | `/roblox/link` | SESSION | desvincula (audit) |
| GET | `/roblox/status` | SESSION | estado `HAS_ACCESS/NO_ACCESS/INCONCLUSIVE/UNKNOWN` + `nextAttemptAt` |
| POST | `/roblox/access-check` | SESSION | força nova checagem (rate 5/min; fila do worker) |

## 8.4 Torneio e configuração

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments` | PUBLIC | lista (respeita `visibility`) |
| GET | `/tournaments/:id` | PUBLIC | detalhes + config pública |
| POST | `/admin/tournaments` | ADMIN `tournaments.config` | cria |
| PATCH | `/admin/tournaments/:id/config` | ADMIN `tournaments.config` | sem `confirm`: 200 com `impacts` + `confirmToken`; com `confirm=true&token=…`: aplica (audita) |
| POST | `/admin/tournaments/:id/status` | ADMIN `tournaments.config` | transições de status com guardas |

**Impacts** (exemplos): `max_teams` abaixo das equipes confirmadas → `INCOMPATIBLE` (bloqueia); `players_per_team`/`substitutes_per_team` mudos → lista equipes fora do novo tamanho (exige confirm; equipes incompatíveis perdem READY); formato após chaveamento → bloqueado; datas → warnings.

## 8.5 Equipes e convites

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/teams` | PUBLIC* | equipes + composição + prontidão (*conforme `visibility`) |
| GET | `/admin/tournaments/:id/teams` | ADMIN `teams.manage` | todas as equipes (todos os estados) + filtros — alimenta T12 |
| POST | `/teams` | SESSION | cria equipe (janela de inscrição aberta) |
| GET | `/teams/:id` | PUBLIC/membro | detalhes |
| PATCH | `/teams/:id` | owner ou ADMIN `teams.manage` | renomear etc. |
| POST | `/teams/:id/leave` | membro | sair (owner transfere ou bloqueia) |
| POST | `/teams/:id/invites` | owner | cria convite (`dm_state` pode sair FAILED sem perder o convite) |
| POST | `/invites/:id/accept` | convidado | transação com advisory lock + partial unique index (07/11) |
| POST | `/invites/:id/decline` | convidado | transição com guarda |
| DELETE | `/invites/:id` | owner ou ADMIN `teams.manage` | cancela pendente |

## 8.6 Sorteio e chaveamento

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/bracket` | PUBLIC | chaveamento (matches, slots, estados, vencedores) |
| POST | `/admin/tournaments/:id/draw/preview` | ADMIN `draws.run` | prévia determinística (seed provisório) — **NÃO persiste** |
| POST | `/admin/tournaments/:id/draw/confirm` | ADMIN `draws.run` | seed novo no servidor → grava `draws`+`brackets`+`matches`+locks em transação; 409 `_BRACKET_EXISTS` se já há bracket com partida iniciada |
| POST | `/admin/tournaments/:id/draw/redraw` | ADMIN `bracket.manage` + dono | só com todas as partidas PENDING/CANCELLED; bracket antigo → SUPERSEDED (evento `draw.superseded`) |
| GET | `/admin/tournaments/:id/draws` | ADMIN `draws.run` | histórico de sorteios |
| POST | `/admin/brackets/:id/edits/preview` | ADMIN `bracket.manage` + dono | prévia de **edição manual** (mutações declarativas) + `confirmToken`; NÃO persiste (12, §12.6) |
| POST | `/admin/brackets/:id/edits/confirm` | ADMIN `bracket.manage` + dono | aplica; `version++`; audit; outbox `bracket.updated`; 409 `_BRACKET_EDIT` se há partida IN_PROGRESS/COMPLETED |

## 8.7 Partidas

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/admin/tournaments/:id/matches?status=` | ADMIN `matches.view` | lista p/ seleção (alimenta autocomplete/select do bot via §8.9) |
| POST | `/admin/matches/:id/start` | ADMIN `matches.manage` | PENDING→IN_PROGRESS; timestamp `now()` do Postgres; 409 `_MATCH_STATUS` |
| POST | `/admin/matches/:id/finish` | ADMIN `matches.manage` | `{winnerTeamId}`; valida ∈ {A,B}; duração calculada; avança bracket; **última rodada → torneio FINISHED** (12, §12.3) |
| POST | `/admin/matches/:id/correct` | ADMIN `matches.manage` + dono | dry-run com `impacts` + `confirmToken`; confirm aplica invalidação controlada (13, §13.5); 409 se dependentes IN_PROGRESS/COMPLETED |
| POST | `/admin/matches/:id/cancel` | ADMIN `matches.manage` | razão obrigatória |
| POST | `/admin/matches/:id/walkover` | ADMIN `matches.walkover` | W.O. com razão; `wo_grace_minutes` respeitado |
| GET | `/admin/matches/:id/history` | ADMIN `audit.view` | `match_events` (histórico da partida) |

## 8.8 Administração, logs e realtime

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/admin/audit?target=&actor=&action=&page=` | ADMIN `audit.view` | paginado, append-only |
| GET · POST · DELETE | `/admin/admins`, `/admin/admins/:discordId` | OWNER | concessão/revogação granular; admin não pode conceder (403) |
| GET | `/tournaments/:id/events?page=` | PUBLIC | feed público paginado (`public_events`) |
| GET | `/stream/tournaments/:id` | SESSION ou PUBLIC conforme visibility | **SSE** — caminho completo: `/api/v1/stream/tournaments/:id`; eventos do catálogo §8.10; heartbeat 25 s; reconexão com `Last-Event-ID` |
| POST | `/admin/tournaments/:id/announcements` | ADMIN `announcements.publish` | anúncio + outbox |

## 8.9 Internos (bot/worker)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/internal/health` | INTERNAL | liveness web/api |
| POST | `/internal/outbox/ack` | INTERNAL (worker) | marca DELIVERED/DEAD na reconciliação manual |
| GET | `/internal/bot/select/matches?status=&page=&q=` | INTERNAL + ator | páginas de ≤25 confrontos p/ autocomplete/string select do bot |
| GET | `/internal/bot/select/teams?status=&page=&q=` | INTERNAL + ator | páginas de ≤25 equipes |

## 8.10 Catálogo canônico de eventos

**Única fonte de nomes** para `public_events`, `outbox_events` e tópicos SSE. Nomes `dotted lowercase`; payloads `camelCase`. Payloads públicos não contêm IDs de admins nem dados privados. Eventos privados (convites, Roblox) vão para `audit_logs`/outbox dirigido — nunca ao feed público.

| Evento | Origem | Payload (essência) |
|---|---|---|
| `tournament.created` | criação | `{tournamentId, name}` |
| `tournament.config_changed` | config aplicada (pós-confirmação) | `{version, changedFields[]}` |
| `tournament.status_changed` | transição de status | `{from, to}` (inclui FINISHED com campeão) |
| `tournament.announcement_published` | anúncio | `{announcementId, title, excerpt}` |
| `team.created` | criação de equipe | `{teamId, name}` |
| `team.member_joined` | convite aceito | `{teamId, userId}` |
| `team.member_left` | saída/remoção | `{teamId, userId}` |
| `team.disqualified` | desqualificação | `{teamId, reason}` |
| `invite.accepted` / `invite.declined` / `invite.expired` / `invite.cancelled` | convites (privados) | `{inviteId, teamId, userId}` |
| `match.started` | admin | `{matchId, tournamentId, round, teams{a,b}}` |
| `match.finished` | admin | `{matchId, winnerTeamId, durationSeconds}` |
| `match.walkover` | admin | `{matchId, winnerTeamId, reason}` |
| `match.corrected` | admin (pós-confirmação) | `{matchId, oldWinnerTeamId, newWinnerTeamId, reason}` |
| `match.cancelled` | admin | `{matchId, reason}` |
| `draw.confirmed` | sorteio confirmado | `{drawId, size, teamsCount}` |
| `draw.superseded` | redraw | `{oldBracketId, newBracketId}` |
| `bracket.updated` | avanço automático ou edição manual | `{version, reason}` |
| `roblox.linked` / `roblox.unlinked` | vínculo (privado → audit) | `{userId}` |
| `roblox.access_state_changed` | máquina de acesso (privado → audit) | `{robloxId, state}` |

Dedup key do outbox: `outbox:<evento>:<id da entidade>` (ex.: `outbox:match.finished:<matchId>`). Heartbeat do SSE é comentário (`: ping`), não evento.
