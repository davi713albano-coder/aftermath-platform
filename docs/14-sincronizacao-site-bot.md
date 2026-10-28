# 14 — Sincronização site–bot (fonte única)

## 14.1 Princípio

**Site e bot não trocam dados entre si.** Ambos consomem `/api/v1` (mesma validação, mesma autorização, mesmo banco). O bot é um *thin client* Discord: recebe interação → chama API → renderiza resposta em componentes. Estado efêmero (ex.: página de select) fica no `custom_id`/payload da interação.

## 14.2 Autenticação bot→API

- `Authorization: Bearer SERVICE_TOKEN` (segredo forte, env) — identifica o **processo bot**.
- `X-Acting-Discord-Id: <snowflake>` — identifica o **ator humano**; o backend resolve permissões do ator no banco. Ações sem ator (ex.: DM de convite) usam ator `SYSTEM`.
- Nunca o inverso (API não chama o bot): API grava eventos; **worker** entrega ao Discord (outbox). O bot só é acionado por interações.

## 14.3 Contrato de responsabilidades

| Tema | Regra |
|---|---|
| Resultados | Só API grava; bot não calcula nada |
| Permissões | API decide; bot apenas oculta opções (UX) |
| Listas p/ selects | API fornece páginas de ≤25 itens (`/internal/bot/...`); paginação via botões |
| Erros | API devolve `{error{code,message}}`; bot mapeia códigos → mensagens amigáveis em pt-BR (sem stack) |
| Idempotência | Ações de escrita do bot enviam `Idempotency-Key` (hash de `interactionId`); API deduplica (padrão `packages/shared/idempotency` do repo atual) |
| Timeout | Interações Discord expiram em 3 s [CONFIRMADO] → bot **defer** imediato (ephemeral) e edita a resposta quando a API responde |

## 14.4 Outbox (notificações site↔Discord)

Já especificado em 05 (D4) e 07: transação de domínio insere `outbox_events`; worker entrega; dedup por `dedup_key`; retries com backoff; `DEAD` com alarme; cron de reconciliação reenvia `PENDING` com mais de N minutos. O **feed público do site lê `public_events` do banco** — o Discord é espelho, nunca fonte.

## 14.5 Consistência em tempo real

- Chaveamento/feed: SSE por tópico `tournament:{id}` ( eventos `bracket.updated`, `match.started/finished`, `feed.new`), heartbeat 25 s, reconexão com `Last-Event-ID`.
- Fallback: se `EventSource` falhar 2× (proxy/CDN bloqueando), cliente alterna polling SWR 10 s automaticamente.
- O bot consulta a API por demanda (interação) — não mantém cache de resultados; painéis V2 publicados no Discord são atualizados pelo worker (edit da mensagem) quando o evento correspondente é entregue.

## 14.6 Semântica de falha externa (requisito explícito)

| Falha | Efeito no domínio | Recuperação |
|---|---|---|
| Discord REST fora / 429 | partida/resultados **já salvos**; mensagem fica PENDING | worker reenvia com backoff; reconciliação |
| Roblox inventário fora | estado `INCONCLUSIVE` | re-tentativa agendada + manual |
| API fora (bot sem resposta) | bot responde "serviço indisponível, tente em instantes" | nenhuma ação perdida: usuário re-tenta |
| Redis fora | API continua (leitura/escrita sem fila); jobs enfileirados falham e ficam `PENDING` no outbox | fila reprocessa ao voltar; alarme |
