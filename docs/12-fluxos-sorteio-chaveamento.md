# 12 — Fluxos: sorteio e chaveamento

## 12.1 Motor de chaveamento (packages/core — funções puras, testadas)

`buildBracket(eligibleTeams, config)`:
1. `N` = nº de equipes elegíveis; `size` = próxima potência de 2 (≥ 2). Se `N < 2` → erro de domínio.
2. `rounds = log2(size)`; rodada 1 tem `size/2` confrontos.
3. Embaralhamento determinístico dado `(seed, orderedTeamIds)` (PRNG cripto-seeded, ex.: `@noble/hashes` sha256-stream ou `crypto.randomBytes` registrado). BYEs distribuídos de forma espalhada (padrão de seed de torneio) garantindo: BYE ≤ size/2 e nenhum confronto BYE×BYE.
4. Confronto `match_kind=BYE` auto-resolve: vencedora avança imediatamente (sem início/fim, sem timestamps).
5. Cada match recebe `next_match_id` + `next_slot` (A/B) conforme o gabarito.

Suportado na v1: **eliminação simples**. O gabarito (`structure jsonb`) é format-agnostic — double elimination entra como novo builder na fase posterior sem mudar o modelo.

## 12.2 Fluxo do sorteio

**Prévia (site ou `/admin sorteio`):**
1. `POST /draw/preview` → backend valida: estado `REG_CLOSED`, ≥2 equipes `READY`, sem bracket ativo; retorna prévia (rodadas + confrontos) com seed provisório **não persistido**.
2. Interface mostra: nº de equipes, BYEs, confrontos da rodada 1. Bot: painel V2 com botões `Confirmar sorteio` / `Cancelar`.

**Confirmação:**
3. `POST /draw/confirm` → gera **seed novo** no servidor (nunca do cliente), `buildBracket`, e grava em UMA transação: `draws` (imutável: autor, fonte WEB|DISCORD, seed, config_snapshot, eligible_teams, resultado), `brackets`, `matches` (todas as rodadas, `@@unique(bracket_id, round, slot)` garante idempotência), lock de equipes (`LOCKED`), `public_events` + `outbox` (`draw.confirmed`).
4. **Reexecutar a página não re-sortear**: resultado só vem de `draws`. `seed_hash` permite reproduzir/auditar o embaralhamento.

**Guards:**
- Já existe bracket ativo com partida `IN_PROGRESS`/`COMPLETED` → 409 `CONFLICT_BRACKET_EXISTS`.
- Redraw (`bracket.manage` + dono): só quando todas as partidas `PENDING`/`CANCELLED`; bracket antigo é arquivado (status `SUPERSEDED`, histórico preservado) — nunca sobrescrito silenciosamente.
- Duplicidade de equipes: valida por `team_id` (não por nome).

## 12.3 Avanço automático

Ao finalizar partida `M` (13, §13.3): transação marca `M` COMPLETED e:
- resolve `M.next_match`: `UPDATE matches SET team_a_id=? WHERE id=? AND team_a_id IS NULL` (slot conforme `next_slot`) — idempotente;
- se o confronto seguinte tiver ambos os lados preenchidos por BYEs encadeados, continua avançando BYEs pendentes;
- grava `public_events`/`outbox` `bracket.updated` (throttle: 1 por transação).

## 12.4 Chaveamento visual (site)

- Render server-side do `bracket` + hidratação; colunas por rodada; cada card mostra equipes (avatar), estado (pendente/em andamento/concluída/BYE/W.O.), vencedora destacada, duração quando concluída.
- Responsivo: desktop colunas lado a lado; mobile em abas por rodada.
- Atualização: SSE (`bracket.updated` → refetch do snapshot) com fallback polling 10 s.
- Padrões visuais: fonte forte, cartões com borda, estados por cor (pendente cinza, andamento âmbar, concluída verde, W.O. roxo) — detalhes em 15.

## 12.5 Histórico de sorteios

`GET /admin/tournaments/:id/draws` — lista com autor, data, seed (visível só p/ `audit.view`), nº de equipes, config. Cada sorteio tem link para o bracket que gerou. Auditoria `DRAW_CONFIRMED` contém antes/depois (bracket arquivado, se houve).
