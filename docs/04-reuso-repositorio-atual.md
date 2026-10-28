# 04 — Inspeção do repositório atual e plano de reúso

Repositório inspecionado (nesta sessão): `C:\Users\davia\Downloads\ap de campeonato`
Nome interno: **elite-clubs** — plataforma de torneios **EA FC Pro Clubs** (bot Discord + Prisma + BullMQ + pagamentos Mercado Pago + integração Pro Clubs da EA). Monorepo pnpm, Node ≥22, TypeScript 5.7, vitest, tsup/tsx.

> A pasta `Downloads/aftermath` contém apenas assets (ícones de armas do jogo em `.webp`) — úteis para o design do site, sem código.

## 4.1 Estrutura existente

```
ap de campeonato/
├─ apps/bot            discord.js 14.27 — router de interações, handlers, guards, container DI
├─ packages/core       serviços de domínio (bracket, match, authorization, audit, championship, registration, finance, conflict, payment)
├─ packages/db         Prisma 6.19 + repositories + schema (747 linhas, alta qualidade)
├─ packages/shared     env, logger (pino), lock Redis, cache, idempotência, rate-limit, errors, ids, result
├─ packages/ui         painéis Discord Components V2 (theme, primitives, custom-id, view-models)
├─ packages/integrations  http client + providers (proclubs mock/ea/http, pagamentos mock/mercadopago)
└─ docker-compose.yml  postgres:16 + redis:7 (AOF) — infra:up/down/reset
```

## 4.2 Veredito por área

| Área | Veredito | Por quê |
|---|---|---|
| Layout de monorepo pnpm + apps/packages | ✅ **Reutilizar o padrão** | Comprovado, com scripts de build/typecheck/test coeso |
| `packages/shared` (lock Redis com token, idempotência, rate-limit, logger pino, errors, env) | ✅ **Portar quase intacto** | Exatamente os primitivos de que o novo projeto precisa |
| `packages/db` — schema Prisma (AuditLog append-only, transitions, unique constraints como invariante de concorrência, convenções UUID/timestamptz/snowflake-as-string) | 🟡 **Portar convenções e padrões**, não o schema em si | Domínio diferente (clubes/pagamentos ≠ equipes Aftermath); os padrões de constraint e auditoria valem ouro |
| `packages/core` — bracket.service, match.service | 🟡 **Referência de projeto** | Lógica de eliminação simples, seeds, `@@unique([championshipId, round, position])` e seedHash determinístico serão adaptados |
| `apps/bot` — router de interações, guards, container, deploy-commands separado | ✅ **Portar arquitetura** | Só trocar handlers; `interactionCreate` roteado por registry, erros globais sem derrubar processo |
| `packages/ui` (painéis Components V2 com custom-id tipado) | ✅ **Portar** | Acelera `/painel`, selects de confronto, botões confirmar |
| Pagamentos / finance / refunds / Mercado Pago | ❌ **Fora de escopo** | O Aftermath tournament não cobra na v1; arquivos ficam no repo antigo, sem portar |
| Integração EA Pro Clubs (mock/ea/http) | 🔄 **Substituir** por `integrations/roblox` (OAuth2 + inventário + thumbnails) com o mesmo padrão mock/real + cache + rate limit + backoff |
| `PlayerConflict` (jogador em clubes diferentes) | 🔄 **Substituir** por elegibilidade de convite (uma equipe ativa por torneio) |
| Testes (vitest, teste de mapping externo, cache/rate-limit) | ✅ **Reutilizar o padrão** de fixtures + mock provider |
| `apps/api` referenciado nos scripts mas **inexistente** | ⚠️ Lacuna do repo antigo | O novo projeto resolve com a API REST dentro do app Next (08), que é o ponto central da arquitetura nova |

## 4.3 O que o novo repositório adota

1. Monorepo pnpm idêntico em espírito: `apps/web`, `apps/bot`, `apps/worker`, `packages/{core,db,schemas,shared,ui,integrations}`.
2. Convenções do schema antigo: UUID PK, `timestamptz`, snowflakes como `String`, enums nomeados, `@@unique` como garantia de concorrência, `AuditLog` com before/after, tabelas de transição para histórico.
3. Bot **sem regra de negócio**: handlers → chamadas à API → core. (No repo antigo o bot acessava core/DB diretamente; aqui a exigência de "fonte única" muda isso deliberadamente.)
4. Nada do repo antigo é deletado ou migrado; o novo repositório nasce limpo, copiando os arquivos-padrão listados acima na fase P0.
