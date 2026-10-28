# 07 — Modelo de dados

Convenções herdadas do repositório atual [CONFIRMADO como boas práticas]: PK UUID gerada pelo banco; `timestamptz`; snowflakes do Discord como `String`; enums nomeados; constraints UNIQUE como garantia definitiva de concorrência; histórico via tabelas de eventos + `AuditLog` append-only. **Sem campos de placar.**

## 7.1 Entidades e relações

### users — identidade de login
| Campo | Tipo | Notas |
|---|---|---|
| id | uuid PK | |
| discord_user_id | String unique | identidade estável |
| username / global_name / avatar_hash | String? | cache de exibição, sincronizado no login |
| status | enum ACTIVE/BANNED | ban bloqueia novas ações, não histórico |
| last_login_at | timestamptz? | |

### roblox_accounts — vínculo 1:1 com Roblox (OAuth2)
| Campo | Tipo | Notas |
|---|---|---|
| user_id | uuid FK unique | 1 usuário = 1 conta Roblox |
| roblox_id | String unique | `sub` do userinfo — 1 conta Roblox = 1 usuário |
| roblox_username / display_name | String | snapshot (mudam; nunca chave) |
| avatar_url | String? | `picture` do userinfo ou thumbnails API |
| linked_at / unlinked_at | timestamptz | desvinculo mantém histórico |
| **Sem tokens Roblox armazenados** | — | decisão D-05 (19): só identidade; re-verificação usa endpoints públicos |

### game_access_checks — estado de acesso ao jogo (separado do vínculo)
| Campo | Tipo | Notas |
|---|---|---|
| roblox_id | String | |
| place_id | int | de `ROBLOX_PLACE_ID` |
| state | enum HAS_ACCESS / NO_ACCESS / INCONCLUSIVE / UNKNOWN | INCONCLUSIVE ≠ fraude (timeout/403 privacidade/429) |
| reason_code | String? | `HTTP_403_PRIVACY`, `HTTP_429`, `TIMEOUT`, `HTTP_403_NOT_OWNED`… |
| checked_at / next_attempt_at | timestamptz | backoff exponencial p/ inconclusivos |
| attempts | int | |

### tournaments — torneio + config corrente
Campos fortes: `name`, `description`, `rules_md`, `status` (DRAFT/REG_OPEN/REG_CLOSED/RUNNING/FINISHED/CANCELLED), `max_teams`, `players_per_team`, `substitutes_per_team` (0 = desativado), `format` (SINGLE_ELIMINATION; modelo pronto p/ DOUBLE), `registration_opens_at/closes_at`, `starts_at`, `visibility` (jsonb: o que é público), `discord_log_channel_id`, `wo_grace_minutes`, `incomplete_team_policy` (ALLOW/BLOCK_AT_DRAW), `created_by_user_id`. Default: 12 equipes × 4 jogadores — **editáveis, nada fixo**.

### tournament_config_versions — trilha de configuração
`config jsonb` (snapshot completo), `changed_by`, `changed_at`, `impacts jsonb` (relatório confirmado), `request_ip_hash`. Uma linha por mudança aceita; a config corrente é a última.

### teams
| Campo | Notas |
|---|---|
| tournament_id FK, name, tag? | `@@unique([tournament_id, lower(name)])` |
| owner_user_id | criador/capitão |
| status | DRAFT / READY / LOCKED (pós-sorteio) / DISQUALIFIED |
| locked_at | congelada no sorteio |

### team_members
`team_id`, `user_id`, `role` (OWNER/MEMBER/SUBSTITUTE), `joined_at`, `left_at?`.
**Invariantes**:
- `@@unique([team_id, user_id])` — um vínculo por equipe;
- partial unique: um usuário só tem **uma** participação ativa (`left_at IS NULL`) por torneio → implementado como unique index em `(user_id) WHERE left_at IS NULL AND role != 'SUBSTITUTE'` + `@@unique([tournament_id?])` via coluna derivada `tournament_id` na linha do membro (denormalizada p/ indexar) — alternativamente enforced transacionalmente + constraint excluída por política. Decisão final na implementação (detalhe em 12).

### invites
| Campo | Notas |
|---|---|
| team_id, tournament_id, invitee_user_id, invited_by_user_id | |
| status | PENDING / ACCEPTED / DECLINED / CANCELLED / EXPIRED |
| dm_state | SENT / FAILED / UNKNOWN (falha de DM não perde convite) |
| expires_at | padrão 48 h configurável |
| responded_at | |

**Invariantes**: `@@unique([invitee_user_id, tournament_id, status])` não funciona p/ multi-status ⇒ partial unique index: `(invitee_user_id, tournament_id) WHERE status='PENDING'` — no máx. 1 convite pendente por usuário por torneio; aceitação em transação com `UPDATE invites SET status='ACCEPTED' WHERE id=? AND status='PENDING'`.

### brackets
`tournament_id @unique` (v1 ativa), `format`, `size` (potência de 2), `rounds`, `seed_hash` (determinístico p/ auditoria do sorteio), `structure jsonb` (slots + links), `draw_id`, `status` (ACTIVE/FINALIZED), `version`. Chaveamento **nunca** é regerado implicitamente.

### matches
| Campo | Notas |
|---|---|
| bracket_id, round, slot | `@@unique([bracket_id, round, slot])` — idempotência da geração |
| team_a_id?, team_b_id? | `NULL` = TBD (aguardando confrontos anteriores/BYE resolvido) |
| status | PENDING / IN_PROGRESS / COMPLETED / CANCELLED (sem DISPUTED: sem contestação de jogador) |
| match_kind | NORMAL / BYE / WALKOVER |
| started_at, ended_at | **server clock** UTC |
| duration_seconds | calculado no UPDATE (`EXTRACT(EPOCH FROM ended_at-started_at)`) |
| winner_team_id? | ∈ {team_a, team_b} validado |
| recorded_by_user_id | admin responsável |
| next_match_id?, next_slot? ('A'|'B') | para avanço automático |

### match_events — histórico por partida
`match_id`, `type` (STARTED/FINISHED/CORRECTED/CANCELLED/WALKOVER/TEAM_SWAPPED), `actor_user_id?`, `before jsonb`, `after jsonb`, `reason?`, `created_at`. **Append-only.** Correções nunca UPDATE em silêncio.

### draws — sorteios imutáveis
`tournament_id`, `performed_by_user_id`, `source` (WEB/DISCORD), `seed` (random cripto do servidor), `config_snapshot jsonb`, `eligible_teams jsonb` (ids no momento), `draw_result jsonb` (ordem final), `created_at`. Reexecutar a página não re-sortear: resultado vem daqui.

### config_change_requests (opcional P2+)
`proposed jsonb`, `impacts jsonb`, `status` (PENDING/APPLIED/DISCARDED), `requested_by`. Suporta o modal "revise os impactos e confirme".

### public_events — feed público (fonte de verdade do log)
`tournament_id`, `type` (MATCH_STARTED/MATCH_FINISHED/DRAW_CONFIRMED/BRACKET_UPDATED/ANNOUNCEMENT), `payload jsonb` (dados já **públicos**), `created_at`, `visibility`. O site lê daqui; o Discord é espelho.

### outbox_events — entrega Discord
`dedup_key @unique` (ex.: `outbox:match_finished:<matchId>`), `type`, `target` (`channel_id` | `user_id`), `payload jsonb`, `status` (PENDING/DELIVERED/FAILED/DEAD), `attempts`, `next_attempt_at`, `discord_message_id?`, `created_at`. **Insert na mesma transação do domínio.**

### admin_grants — permissões granulares
`discord_user_id` (chave por identidade Discord — grant existe antes do primeiro login), `permissions String[]` (enum fechado), `granted_by_discord_user_id`, `granted_at`, `revoked_at?`, `note?`. **Sem linha OWNER aqui**: dono é derivado de `OWNER_DISCORD_ID` + tabela `owner_bootstrap` (ver 10).

### audit_logs — append-only
`actor_user_id?`, `actor_discord_id?`, `actor_type` (USER/ADMIN/SYSTEM/BOT), `action`, `target_type`, `target_id`, `before jsonb?`, `after jsonb?`, `reason?`, `ip_hash?`, `correlation_id?`, `created_at`. Índices por target/ator/data. **Sem endpoint de edição/exclusão**; opcional P4: cadeia de hash (`prev_hash`) p/ tamper-evidence (decisão D-09).

### announcements
`tournament_id`, `title`, `body`, `published_by`, `published_at`, `discord_message_id?` (pós-outbox).

## 7.2 Diagrama

Ver `06-diagramas.md` §6.2.

## 7.3 Invariantes globais (resumo para os testes)

1. Partida só transitada uma vez: `UPDATE ... WHERE status=<esperado>` + rowCount check.
2. Um convite pendente por usuário/torneio; aceitação atômica com transição de status.
3. Uma equipe ativa por usuário por torneio; uma equipe por torneio por nome.
4. Vencedora ∈ {teamA, teamB}; partida finalizada só com started_at válido; duração ≥ 0.
5. Avanço de chaveamento apenas se partida COMPLETED e slot destino ainda vazio ou idêntico (idempotente).
6. Config incompatível com estado atual → bloqueada (409 + impactos) até confirmação explícita.
7. `outbox_events.dedup_key` único ⇒ reexecução nunca duplica mensagem.
8. AuditLog só INSERT (sem UPDATE/DELETE em API; revoke via permission no banco, não em app).
