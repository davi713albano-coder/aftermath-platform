# 05 — Arquitetura proposta

## 5.1 Visão geral

```
                        ┌─────────────────────────────┐
   Navegador ──────────▶│  apps/web (Next.js 16)      │
   Jogadores/Admins     │  Site + Painel + API /api/v1│──────┐
                        └───────────────┬─────────────┘      │
                                        │ REST (service      │
                                        │ token + ator)      │
                        ┌───────────────▼─────────────┐      │
   Discord ──gateway───▶│  apps/bot (discord.js v14)  │      │
                        │  sem banco; só consome API  │      │
                        └───────────────┬─────────────┘      │
                                        │                    │
                        ┌───────────────▼─────────────┐      ▼
                        │  apps/worker (BullMQ)       │   PostgreSQL 16
                        │  outbox Discord, checagens  │   (Prisma 6)
                        │  Roblox, expirações, cron   │
                        └───────┬───────────────┬─────┘      ▲
                                │               │            │
                            Redis 7      APIs externas──────┘
                            (fila+lock)  (Discord REST, Roblox)
```

## 5.2 Componentes e responsabilidades

| Componente | Tecnologia | Responsabilidade | Proibido |
|---|---|---|---|
| `apps/web` | Next.js 16 (App Router), React 19 | Site público, área do jogador, painel admin, **API REST `/api/v1`** (route handlers), SSE, autenticação Auth.js | Regra de negócio inline; acesso direto do cliente ao banco |
| `apps/bot` | discord.js 14 | Interações (comandos, selects, botões), envio de DMs, painéis V2 | Ler/escrever banco; guardar resultados; decidir permissão |
| `apps/worker` | Node + BullMQ | Outbox→Discord, checagens Roblox (orçamento de rate), expiração de convites, reconciliação, cron de manutenção | Regra de negócio (apenas executa tarefas e grava resultados) |
| `packages/core` | TS puro, sem IO | Máquinas de estado, motor de chaveamento, validação de config, política de permissões | Imports de Prisma/Discord/Next |
| `packages/schemas` | Zod | Contratos de entrada/saída compartilhados web/bot | — |
| `packages/db` | Prisma 6 + Postgres | Modelos, migrações, repositories com transações | Lógica de domínio |
| `packages/integrations` | fetch | Clientes Roblox (OAuth2, inventário, thumbnails) e Discord REST, com cache/rate-limit/backoff (padrão do repo antigo) | Conhecer o domínio |
| `packages/shared` | pino, ioredis | env tipada, logger, lock, idempotência, erros de domínio | — |
| `packages/ui` | discord.js builders | Painéis Components V2, custom-ids tipados | Chamadas de rede de negócio |

## 5.3 Decisões arquiteturais e justificativas

**D1 — API dentro do Next (BFF único), não API separada.**
O requisito "site e bot usam a mesma API e a mesma fonte de dados" é atendido por `/api/v1` (route handlers Node). Uma API separada (Fastify/Nest) duplicaria auth, deploy e tipos na v1. O core fica em `packages/core`, então extrair uma API dedicada depois é mecânico. *Trade-off*: route handlers compartilham o processo do site (escala conjunta) — aceitável para o porte do produto; documentado como ponto de extração futura.

**D2 — Bot sem banco.**
Diferente do repo antigo (bot → core/DB direto), aqui o bot chama `/api/v1` com `SERVICE_TOKEN` + ator (`X-Acting-Discord-Id`). Garante fonte única, permite rate limit/auditoria centralizados e mantém o bot stateless (facilita deploy). *Trade-off*: latência extra + o bot depende da API no ar — mitigado com mensagens de erro claras e healthcheck.

**D3 — PostgreSQL + Prisma como fonte de verdade; Redis apenas para fila/lock/rate-limit.**
Invariantes críticos (uma partida uma vez, convite único, uma equipe por jogador) vivem em **constraints do Postgres + transações**, nunca só no Redis (o lock é otimização de contenção — padrão já provado no repo antigo).

**D4 — Outbox transacional para Discord.**
Toda mudança que deve gerar log público grava um `outbox_event` **na mesma transação** do domínio; o worker publica no Discord com retries (BullMQ, backoff exponencial + jitter). Falha no Discord nunca perde o fato — o feed do site lê o banco, não o Discord. Atende exatamente o requisito "se a publicação falhar, a partida continua salva e a notificação pode ser reenviada".

**D5 — Fila BullMQ + Redis 7 (AOF).**
Já é o padrão do repositório atual (ioredis no lock), o worker existe de qualquer forma (Roblox + Discord). Alternativa avaliada: `pg-boss` (fila no Postgres, um serviço a menos) — mantida como alternativa B em 19, decisão D-04 para aprovação.

**D6 — Realtime com SSE + fallback de polling.**
SSE (route handler Node, `runtime='nodejs'`, heartbeat 25 s, `Last-Event-ID`) para chaveamento/feed; o cliente troca para polling de 10 s automaticamente se o stream falhar. WebSockets rejeitados na v1: só leitura servidor→cliente é necessária.

**D7 — Monorepo pnpm sem Turborepo na v1.** Scripts `pnpm -r` bastam neste porte (padrão do repo atual). Turbo é adição trivial depois.

**D8 — Deploy alvo: Docker Compose em VPS.** O bot exige conexão de gateway persistente; manter web+bot+worker+pg+redis no mesmo host simplifica operação da v1. Alternativas (Cloud Run para web + VPS para bot; tudo na Oracle/AWS) em 19/D-01.

## 5.4 Fluxo de dados canônico (exemplo: finalizar partida)

1. Admin clica "Finalizar" no site (ou `/admin finalizar`) → `POST /api/v1/matches/:id/finish {winnerTeamId}` (site: cookie; bot: service token + ator).
2. Route handler valida Zod + permissão (banco) → chama `core.match.finish()` dentro de transação Prisma:
   - `UPDATE matches SET status='COMPLETED', winner_team_id=..., ended_at=now(), duration_seconds=... WHERE id=? AND status='IN_PROGRESS'` → `rowCount===1` obrigatório;
   - avança vencedora no confronto seguinte; grava `match_events` (histórico) e `audit_logs`;
   - insere `outbox_events` (match_finished) — mesma transação.
3. Commit. Resposta 200 com o estado novo.
4. Worker consome outbox → publica no canal do Discord → marca `DELIVERED` + `discord_message_id`. Falha → retry com backoff; `DEAD` após N tentativas → alarme.
5. SSE empurra snapshot do chaveamento; clientes atualizam.

Nenhum componente mantém cópia: site e bot leem o mesmo banco via a mesma API.
