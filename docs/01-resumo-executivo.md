# 01 — Resumo executivo

## O produto

Plataforma web + bot Discord para organizar torneios do jogo Roblox **Aftermath** (eliminação por sobrevivência: **morreu, morreu** — sem placar). O sistema gerencia inscrições, equipes, convites, vínculo com contas Roblox, sorteio, chaveamento visual, partidas com horários confiáveis, logs públicos no Discord e auditoria privada — tudo sobre **uma única API e um único banco**, usados tanto pelo site quanto pelo bot.

## Princípios confirmados (do pedido)

1. **Fonte única de verdade**: site e bot consomem a mesma API REST sobre o mesmo banco. Nenhum dos dois mantém cópia própria de resultados ou permissões.
2. **Sem placar**: partidas registram equipes, estado, timestamps de início/fim, duração calculada, vencedora e responsável. Nenhum campo de gols/rounds/points.
3. **Configuração flexível**: nº de equipes e jogadores por equipe são padrões configuráveis, nunca limites codificados (os 12×4 citados são apenas defaults).
4. **Autorização no backend**: esconder botão na interface não é proteção; toda operação valida permissão no servidor.
5. **Fraude ≠ falha**: verificação de acesso ao jogo que falha/timeout/inconclusiva **não** marca o jogador como fraudulento; registra estado e permite nova tentativa controlada.
6. **Resultado oficial só por administrador autorizado**: não existem `/resultado enviar` nem `/resultado contestar` para jogadores.
7. **Duas camadas de log**: feed público (Discord + site) e auditoria privada append-only no banco.
8. **Falha externa ≠ falha do domínio**: se a publicação no Discord falhar, a partida continua salva no banco e a notificação é reenviada (outbox).

## Arquitetura em uma frase

Monorepo TypeScript (pnpm workspaces, padrão já usado no repositório atual) com **Next.js 16** (site + painel + API `/api/v1`), **discord.js v14** (bot, sem acesso ao banco — consome a API), **worker BullMQ** (notificações Discord via outbox, checagens Roblox, expirações), **PostgreSQL 16 + Prisma**, **Redis 7** (fila e locks), tudo em Docker Compose num VPS.

## Escopo da primeira versão (faseável)

- Login por Discord OAuth2; vínculo de conta Roblox por OAuth2 do Open Cloud; verificação de acesso ao jogo pelo endpoint público de inventário informado pelo dono do projeto.
- Torneio com formato **eliminação simples** (double elimination fica para fase posterior; o modelo já suporta).
- Registro de partidas iniciado/finalizado por admins, com prévia de impacto nas correções.
- Log público espelhado em canal do Discord configurável + feed no site.

## Principais riscos (detalhes em 19)

| Risco | Severidade | Mitigação |
|---|---|---|
| OAuth2 do Roblox está **beta**; app fica em modo privado com **limite de 10 usuários** até aprovação para produção | Alta | Registrar o app já na fase P1 e solicitar revisão para produção imediatamente; fallback documentado |
| Endpoint `is-owned` do inventário depende da privacidade do jogador e pode retornar 403/429 | Alta | Máquina de estados `HAS_ACCESS / NO_ACCESS / INCONCLUSIVE` + `can-view-inventory` para discriminar + retry com backoff |
| Bloqueio anti-bot do Roblox contra IPs de datacenter (o teste desta sessão retornou 403) | Média | Probe controlado de IP residencial antes de definir política de retry; webhook do jogo como alternativa futura |
| Webhooks SSE atrás de proxy/CDN | Baixa | SSE com heartbeat + fallback automático de polling |
