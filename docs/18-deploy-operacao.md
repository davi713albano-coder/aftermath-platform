# 18 — Deploy, variáveis de ambiente, backups e operação

## 18.1 Topologia alvo (v1)

Docker Compose em VPS (4 vCPU/8 GB mínimo): `web` (Next.js standalone), `bot`, `worker`, `postgres:16`, `redis:7` (AOF). Alternativas (Cloud Run p/ web + VPS p/ bot) em 19/D-01.

```
compose/
  docker-compose.yml      # prod
  docker-compose.dev.yml  # dev (só pg+redis)
services:
  web:    build apps/web,  healthcheck /internal/health
  bot:    build apps/bot,  restart unless-stopped
  worker: build apps/worker
  postgres: volume pgdata + backup sidecar (pg_dump cron)
  redis:    --appendonly yes
```

Deploy: CI (GitHub Actions) roda typecheck/lint/testes → build images → push → `docker compose pull && up -d` → `prisma migrate deploy` (passo explícito, nunca no boot) → smoke tests. Rollback: tag anterior das images + migrate não-destrutivo.

## 18.2 Variáveis de ambiente (`.env.example` completo)

```bash
# Ambiente
NODE_ENV=production
LOG_LEVEL=info
APP_PUBLIC_URL=https://seu-dominio

# Discord (bot + OAuth2)
DISCORD_TOKEN=              # bot token
DISCORD_CLIENT_ID=
DISCORD_CLIENT_SECRET=
DISCORD_GUILD_ID=           # guild única v1
DISCORD_INVITE_URL=         # link de convite p/ o site
OWNER_DISCORD_ID=1102745175098404935

# Roblox (OAuth2 Open Cloud + jogo)
ROBLOX_OAUTH_CLIENT_ID=
ROBLOX_OAUTH_CLIENT_SECRET=
ROBLOX_PLACE_ID=15327728308
ROBLOX_UNIVERSE_ID=5286749994
ROBLOX_CHECK_BUDGET_PER_MINUTE=30

# Banco / fila
DATABASE_URL=postgresql://user:pass@postgres:5432/aftermath?schema=public
REDIS_URL=redis://redis:6379

# Segredos internos
SERVICE_TOKEN=              # bot/worker → API
SESSION_SECRET=             # Auth.js
STATE_SIGNING_SECRET=       # HMAC dos states OAuth Roblox
```

Validação: env schema (Zod) no boot de cada app — faltou chave, não sobe. `OWNER_DISCORD_ID`, `ROBLOX_PLACE_ID`, `ROBLOX_UNIVERSE_ID` são **configuração inicial validada**, nunca substitutos de autenticação.

## 18.3 Backups e recuperação

- Postgres: `pg_dump` diário (retenção 30 dias) + WAL archiving se volume gerenciado; testes de restauração **mensais** (script `ops:restore-drill`).
- Redis: AOF + snapshot 6 h (fila é reprocessável; dados críticos estão no Postgres).
- Offsite: upload do dump para storage externo (S3/B2) com criptografia.
- RTO alvo: 1 h; RPO: 24 h (WAL reduz p/ minutos).

## 18.4 Monitoramento e operação

- Logs: pino JSON com correlationId; agregados via `docker logs` → Loki/Grafana (opcional) ou logfile rotativo.
- Health: `/internal/health` (web), worker heartbeat em Redis (TTL 60 s), bot: `ready` + presença.
- Alarmes mínimos (UptimeRobot/healthchecks.io + webhook no Discord de ops): web down, worker heartbeat expirado, bot desconectado >5 min, outbox DEAD >0, taxa 5xx >2%, circuit breaker Roblox aberto.
- Rotinas: reconciliação outbox (cron 5 min), expiração de convites (cron 15 min), watchdog partidas (cron 30 min), digest de auditoria (diário, opcional).
- Runbooks: token vazado (17.6), Roblox bloqueando IP (trocar egress/proxy de saída + abrir circuit), Discord rate 429 em massa (throttle do outbox).

## 18.5 Migrações e seeds

- `prisma migrate dev` local; `migrate deploy` em prod via CI; migrações destrutivas em duas etapas (expand/contract) com revisão.
- Seed de produção: **nenhum dado fake**; apenas criação do torneio raiz pelo painel. Seed de dev separado.
