# 16 — Plano de testes

Stack: **vitest** (unit/integração — padrão do repo atual), **Playwright** (E2E web), **testcontainers** ou `docker compose` com banco/redis efêmeros para integração. Meta de cobertura do `packages/core`: ≥ 90% (lógica crítica).

## 16.1 Unitários — packages/core (puros)

- **Chaveamento**: N=2..33,64,127,128; ímpares/BYEs; BYE nunca encara BYE; rounds=ceil(log2(size)); gabarito de next_match/next_slot; determinismo (mesma seed ⇒ mesmo chaveamento); propriedade: todo time aparece exatamente 1× na rodada 1 (fast-check).
- **Config impact**: reduzir max_teams abaixo das equipes confirmadas → INCOMPATIBLE; mudar players_per_team com equipes existentes → lista impactada; formato após bracket → bloqueado.
- **Permissões**: matriz 9.3 inteira como tabela de casos; admin não concede; owner imune; revogação surte efeito imediato.
- **Máquina de convites**: transições válidas/inválidas; expiração; duplicados.
- **Correção de partida**: dry-run de impactos; casos com/sem dependentes iniciados.

## 16.2 Integração — API + Postgres/Redis reais

- Migrações aplicam limpas; seed de teste fixo.
- Auth: login Discord simulado (mock do provider), sessão cookie, logout, callback com state inválido → 401.
- Roblox client: **mock provider** (padrão do repo atual) + fixtures gravadas dos status 200/403(privacy)/403(not owned)/429 → asserções da máquina de estados 10.4.
- Endpoints críticos: equipe/convite/partida/sorteio com permissões corretas e erradas (403 em cada).
- Idempotência: mesma `Idempotency-Key` → mesma resposta, sem duplicar efeito.
- Outbox: evento inserido na tx; worker entrega (Discord mock); falha → retry com backoff; dedup_key evita duplicata.

## 16.3 Ponta a ponta — Playwright (site)

1. Fluxo feliz do jogador: login (provider mock) → vincular Roblox (mock OAuth) → criar equipe → convidar (2º usuário) → aceitar → equipe READY.
2. Admin: configurar torneio → fechar inscrições → prévia do sorteio → confirmar → iniciar partida → finalizar com vencedora → chaveamento avançou + feed público atualizou (SSE) + outbox entregue (Discord mock).
3. Correção com impacto: correção bloqueada por partida dependente iniciada → cancelar dependente → corrigir com confirmação.
4. Realtime fallback: matar SSE (interceptar) → polling assume em ≤10 s.

## 16.4 Concorrência (dedicadas — requisito do pedido)

- **Convite**: 10 accepts paralelos do mesmo convite → exatamente 1 aceita; 9 `CONFLICT_INVITE_STATE`.
- **Vaga final**: 2 accepts paralelos para a última vaga → 1 vence, outro recebe conflito de vaga; sem ghost member.
- **Partida**: 2 starts paralelos → 1 IN_PROGRESS; 2 finishes com vencedoras diferentes → 1 COMPLETED, segundo 409; resultado ≠ vencedor lançado.
- **Sorteio**: 2 confirms paralelos → 1 bracket; re-executar preview depois não altera nada.
- Implementação: `Promise.all` de chamadas reais contra o servidor de teste (sem mocks de tx).

## 16.5 Bot

- Handlers unitários com interações fake (payloads v14): router, guards (ator/permissão via API mock), mapeamento de erros API→mensagem.
- Teste de contratos: schemas Zod da API validam payloads que o bot monta (evita drift site↔bot).
- Staging manual: servidor Discord de testes (guild própria), checklist por comando antes de cada release.

## 16.6 Externos (probe controlado)

- Probe Roblox (03 §3.4) executado em CI com flag manual (não automático) — documenta contrato vigente; se contrato mudar, teste falha explicitamente e bloqueia release da verificação.
- Discord: ambiente de staging com app/bot de teste (tokens de teste), guild dedicada.

## 16.7 Critérios de aceite por fase

Cada fase do roadmap (20) lista seus testes; nada avança de fase com suíte vermelha. Testes de concorrência rodam sempre no CI (não são "skip em flake": flake = bug a investigar).
