# 13 — Fluxos: partidas

Domínio: **eliminação por sobrevivência** — sem placar. Campos: equipes, rodada/slot, estado, início, fim, duração (calculada), vencedora, responsável, histórico.

## 13.1 Iniciar

**Site**: painel admin → lista de confrontos pendentes → botão Iniciar. **Discord**: `/admin start` → autocomplete de `matches.select` (busca por "R2 — EquipeA vs EquipeB", ≤25 resultados por tecla) ou string select paginado; confirmação por botão (2 cliques, zero digitação de nomes).

`POST /admin/matches/:id/start` — transação:
```sql
UPDATE matches
   SET status='IN_PROGRESS', started_at=now(), recorded_by_user_id=:actor
 WHERE id=:id AND status='PENDING'
   AND team_a_id IS NOT NULL AND team_b_id IS NOT NULL;
```
`rowCount == 1` → grava `match_events(STARTED)` + `audit_logs` + `outbox(match_started)`; COMMIT; 200.
`rowCount == 0` → ROLLBACK → **409 `CONFLICT_MATCH_STATUS`** (já iniciada/concluída/cancelada — mensagem diz qual). Concorrência duplo-clique: a segunda chamada perde a corrida e recebe 409 — correto por design. Guardas adicionais: torneio `RUNNING`, admin com `matches.manage` (checado antes da tx).

**Timestamp confiável**: relógio do servidor de aplicação (UTC, NTP). Se vários servidores: relógio do Postgres `now()` dentro da tx — fonte única e consistente [decisão: usar `now()` do banco].

## 13.2 Durante

- `GET /matches/:id` mostra "em andamento há mm:ss" (agora − started_at), atualizado no cliente; estado `IN_PROGRESS` persiste no banco (reinício de processo não perde o fato).
- Watchdog (worker, opcional P5): partidas `IN_PROGRESS` > X horas ganham badge "atenção" na lista admin (nenhuma ação automática).

## 13.3 Finalizar

**Site**: botão Finalizar → modal com as duas equipes (radio). **Discord**: `/admin finalizar` → autocomplete da partida `IN_PROGRESS` → string select com `teamA`/`teamB` → confirmar.

`POST /admin/matches/:id/finish {winnerTeamId}` — transação:
1. valida `winnerTeamId ∈ {team_a_id, team_b_id}` (senão 400/409);
2. `UPDATE matches SET status='COMPLETED', ended_at=now(), winner_team_id=..., duration_seconds=EXTRACT(EPOCH FROM (now()-started_at)), recorded_by_user_id=:actor WHERE id=? AND status='IN_PROGRESS'` → rowCount 1 obrigatório;
3. avança chaveamento (12.3) dentro da mesma tx;
4. `match_events(FINISHED, before/after)`, `audit_logs`, `outbox(match_finished)`; COMMIT.

Iniciar sem finalizar é permitido (estado persiste); finalizar sem iniciar → 409. Cancelamento (`/cancel`, razão obrigatória): partida volta ao fluxo admin — não avança chaveamento.

## 13.4 W.O. e desistência

`POST /admin/matches/:id/walkover {winnerTeamId, reason}`: `match_kind=WALKOVER`, gravação idêntica à finalização com `started_at=ended_at=now()`, `duration_seconds=0`; audit + público ("W.O."). Guarda `wo_grace_minutes` quando aplicável (agendamento futuro). Desistência de equipe (`teams.manage`) → marca equipe DISQUALIFIED + partidas pendentes dela viram WALKOVER para a adversária (transação com prévia de impacto).

## 13.5 Correção de resultado (com invalidação controlada)

`POST /admin/matches/:id/correct {winnerTeamId, reason}`:

1. **Análise de impacto (dry-run)**: desce o grafo a partir da partida, listando confrontos dependentes já propagados com o vencedor antigo.
2. Se nenhum confronto dependente está `IN_PROGRESS`/`COMPLETED` → resposta 200 com `impacts` + `confirmToken`; confirmação (`confirm=true`) aplica em transação:
   - regrava `winner_team_id`, recalcula `duration_seconds` (preservado em `match_events`);
   - limpa e repropaga slots dependentes (UPDATE idempotente);
   - `match_events(CORRECTED, before/after)` — histórico íntegro.
3. Se algum confronto dependente já começou/terminou → **409** com a lista; fluxo obrigatório: cancelar/inverter os confrontos afetados explicitamente (cada um auditado) antes da correção subir. Nunca invalidação automática silenciosa.
4. Toda correção exige `reason` e vai ao audit; evento público `bracket.updated` (sem expor "corrigido" além do que a visibilidade permitir).

## 13.6 Proteções resumo (requisito: nada de resultado incompatível)

| Tentativa | Resultado |
|---|---|
| Iniciar partida já IN_PROGRESS/COMPLETED | 409 |
| Finalizar partida não iniciada | 409 |
| Vencedora fora das duas equipes | 400 |
| Finalizar com chaveamento inconsistente (slot destino ocupado por outra equipe) | 409 + revisão manual |
| Dois admins iniciando/finalizando simultaneamente | exatamente 1 vence a condicional; outro recebe 409 |
| Reinício do processo durante IN_PROGRESS | nada muda: estado no banco |

Todos os testes de concorrência em 16, §16.4.
