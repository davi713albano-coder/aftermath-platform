# 20 — Roadmap (fases pequenas, ordenadas, verificáveis)

Regra: cada fase tem **critérios de aceite testáveis** e nada de fase posterior começa sem os anteriores verdes. Estimativas em dias de trabalho concentrado (não calendário).

## P0 — Bootstrap do repositório (~1–2 d)
- Monorepo pnpm (apps: web/bot/worker; packages: core/db/schemas/shared/ui/integrations) — copiando padrões do repo atual (04).
- TS strict, eslint, prettier, vitest, CI (typecheck+lint+test), docker-compose dev (pg+redis), `.env.example`, README.
- **Aceite**: `pnpm test` verde no CI; containers sobem.

## P1 — Identidade: Discord OAuth2 + dono + permissões (~3–4 d)
- Auth.js provider Discord; upsert users; sessão JWT; logout.
- Bootstrap do OWNER por env (10.2) + `admin_grants` + policy service (matriz 9.3) + audit append-only.
- `/me`, `/me/permissions`, `/admin/admins` (conceder/revogar, só dono).
- **Aceite**: login E2E; matriz de permissões coberta por testes; bootstrap idempotente auditado. *Paralelo: registrar app OAuth Roblox + pedir produção (R-1) e definir domínio (D-10).*

## P2 — Torneio + config + Roblox (~4–5 d)
- Modelo tournaments/config_versions + wizard T11 com análise de impacto e confirmação (8.4).
- `packages/integrations/roblox`: OAuth2 link (openid profile), userinfo, thumbnails; **probe do `is-owned`** (03 §3.4); máquina de estados de acesso com INCONCLUSIVE/backoff; fila de checagens com orçamento.
- **Aceite**: config salva com trilha; mudança incompatível bloqueada com impactos; vínculo Roblox funciona com mock + real (se app aprovado); estados de acesso conforme tabela 10.4.

## P3 — Equipes, convites + sorteio/chaveamento (~5–6 d)
- Teams/members/invites com invariantes (07); fluxos site (T7/T8) e `/equipe`.
- Motor de chaveamento puro + testes de propriedade; sorteio com prévia/confirmar/redraw guards (12); render do bracket (T3) com SSE + fallback polling.
- **Aceite**: testes de concorrência 16.4 (convites/vaga/sorteio) verdes; bracket renderiza 12 e 128 equipes; reload não altera resultado.

## P4 — Partidas (~3–4 d)
- start/finish/cancel/walkover/correct com guards de status + avanço automático + match_events; modais T13; outbox `match.*` (worker entrega no canal configurado).
- **Aceite**: 16.4 (partida) verde; correção com impacto; falha do Discord simulada não perde o resultado e reconcilia.

## P5 — Bot completo + logs públicos + anúncios (~4–5 d)
- Comandos RF-26 com autocomplete/selects/paginação; painéis V2; defer+edit (3 s); idempotency keys; mapeamento de erros.
- Feed público no site (T5) lendo `public_events`; anúncios (T17); digest opcional.
- **Aceite**: todos os comandos do checklist em guild de staging; outbox DEAD alarmado; nenhuma ação de resultado disponível a jogadores.

## P6 — Endurecimento + E2E completos (~3–4 d)
- Rate limits finais, CSRF/Origin checks, redação de logs, auditoria com filtros (T16), backups automáticos + drill de restauração, alarmes, runbooks.
- **Aceite**: suíte completa verde; restauração de backup testada; alarmes disparando em staging.

## P7 — Deploy de produção + go-live (~2–3 d)
- VPS + compose prod; DNS/TLS; migrate deploy via CI; seeds zerados; monitoramento externo; plano de rollback.
- **Aceite**: smoke tests em produção; dono ativa OWNER com login; primeiro torneio configurado pela UI.

## Pós-v1 (backlog ordenado)
Double elimination · round-robin de grupos · estatísticas de equipes · página de histórico de torneios · webhook nativo do jogo p/ verificação (proposta ao dev do Aftermath) · melhorias visuais com assets do jogo · AI Studio Build para experimentos de UI (D-07).

## Dependências externas (paralelas ao código)
1. Registro + aprovação do app OAuth2 Roblox (P1) — caminho crítico do R-1.
2. Criação do app/bot Discord + intents (`GUILD_MEMBERS`).
3. Definição de domínio + canal de logs no Discord.
