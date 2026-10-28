# 03 — Relatório de pesquisa web

**Data de consulta de todas as fontes: 28/10/2026.**
Classificação: ✅ CONFIRMADO (lido na fonte oficial nesta sessão) · 🧪 TESTAR (contrato plausível, requer teste controlado) · ⚠️ INCERTO (contingência definida).

## 3.1 Tabela-resumo

| Área | Fonte oficial | Status | Conclusão principal |
|---|---|---|---|
| Discord OAuth2 | https://docs.discord.com/developers/topics/oauth2 (e espelho `discord/discord-api-docs` no GitHub) | ✅ | Authorization code grant; `state` recomendado e validado contra CSRF; scopes declarados no portal |
| Discord rate limits | https://docs.discord.com/developers/topics/rate-limits | ✅ | Global 50 req/s por bot; per-route por buckets (`X-RateLimit-Bucket`); 429 com `retry_after`; 10.000 respostas inválidas (401/403/429) em 10 min por IP → ban temporário |
| Discord intents privilegiadas | https://docs.discord.com/developers/events/gateway · https://support-dev.discord.com/hc/en-us/articles/6207308062871 | ✅ | `GUILD_MEMBERS` e `MESSAGE_CONTENT` são privilegiadas; revisão obrigatória acima de ~10.000 usuários alcançáveis |
| Componentes/selects | https://docs.discord.com/developers/components/reference · discordjs.guide | ✅ | String select: máx. 25 opções; autocomplete de comando: máx. 25 escolhas dinâmicas |
| Roblox OAuth2 | https://create.roblox.com/docs/en-us/cloud/auth/oauth2-reference.md · `/oauth2-overview` · `/oauth2-registration` · `/oauth2-develop` | ✅ | **Beta**; code flow com PKCE opcional; código de autorização vive **1 minuto**; access token **15 min**; refresh **90 dias** uso único; userinfo claims (`sub`, `name`, `preferred_username`, `picture`…); redirect deve ser HTTPS/localhost; app não aprovado fica em **modo privado, máx. 10 usuários** |
| Roblox Users API | https://create.roblox.com/docs/en-us/cloud/reference/features/users.md (base `apis.roblox.com`; legado `users.roblox.com/v1`) | ✅ | Busca por username/ID; matriz de auth (API key / OAuth2 / cookie) varia por endpoint |
| Roblox Thumbnails | https://create.roblox.com/docs/en-us/cloud/reference/domains/thumbnails.md · `/features/thumbnails` | ✅ | `/v1/users/avatar-headshot` (headshots CSV); limite exemplar 100/min por autorização OAuth2 |
| Roblox Inventários | https://create.roblox.com/docs/en-us/cloud/reference/features/inventories.md · `inventory.roblox.com/docs/json/v1` | ✅/🧪 | `is-owned` existe e é público (sem `.ROBLOSECURITY`); visibilidade depende de privacidade do usuário; `can-view-inventory` existe como alternativa; **o corpo exato da resposta do `is-owned` NÃO está documentado publicamente** (o Swagger expõe apenas o sumário) → probe obrigatório (§3.4); thread oficial de 2020 relata que o `is-owned` historicamente **ignorava** configurações de privacidade — comportamento atual a validar |
| Endpoint informado pelo dono | `GET https://inventory.roblox.com/v1/users/{robloxId}/items/Place/15327728308/is-owned` | 🧪 | Testado desta sessão: **403** para user 1 (pode ser "não possui o item", privacidade OU bloqueio anti-bot de IP de datacenter — discriminação no probe §3.4) |
| Google AI Studio Build + GitHub | https://ai.google.dev/gemini-api/docs/aistudio-build-mode (publicado 26/10/2026) · https://aistudio.google.com/learn/sync-your-ai-studio-apps-with-github · `/docs/aistudio-deploying` | ✅ | Sincronização bidirecional com GitHub (push com commits gerados por IA, pull de IDE); import de repositório; deploy para Cloud Run (tier inicial: 2 apps grátis) |
| Next.js | https://nextjs.org/blog/next-16 · `/blog/next-16-4` (06/10/2026) | ✅ | Next 16 estável (Turbopack default, Cache Components, React 19.2); 16.4 recomenda Cache Components para todo app novo; `middleware.ts` renomeado para `proxy.ts` |
| Auth.js | https://authjs.dev/getting-started/installation · `/concepts/session-strategies` · https://next-auth.js.org/providers/discord | ✅ | Provider Discord oficial; estratégia JWT (default) ou database; cookie HttpOnly; `proxy.ts` no Next 16 |
| BullMQ | https://docs.bullmq.io/ · `/guide/retrying-failing-jobs` · `/guide/going-to-production` | ✅ | Fila persistente sobre Redis 6.2+; retries com backoff fixed/exponential + jitter; persistência do Redis deve ser habilitada (AOF); dead-letter via conjunto failed |
| SSE no Next.js | https://nextjs.org/docs/app/guides/streaming | ✅ | Route handler pode streamar com Web Streams; `runtime='nodejs'` + `dynamic='force-dynamic'`; padrão híbrido SSE+polling documentado pela comunidade |

## 3.2 Detalhes que mudam o design

### Discord
- **Limites que o bot respeita por design** [CONFIRMADO]: 25 opções em string select ⇒ listas de equipes/confrontos são **paginadas ou usam autocomplete**; 50 req/s global ⇒ worker de outbox com bucket próprio.
- **Intents mínimos**: `GUILD_MEMBERS` (resolução de membros/convites) é privilegiada — habilitar no portal; `MESSAGE_CONTENT` só se necessário (componentes V2 e interações não exigem) [CONFIRMADO].
- **DMs**: envio para usuário do servidor pode falhar por privacidade (erro 50007) — tratado como `dmState=FAILED`, convite sobrevive [TESTAR comportamento exato].
- Comandos de guilda propagam imediatamente; globais demoram — deploy de comandos é script separado (padrão já usado no repositório atual) [CONFIRMADO].

### Roblox
- **OAuth2 (status)** [CONFIRMADO]: a página oficial de referência exibida nesta consulta (28/10/2026) ainda traz aviso de **beta**; porém o anúncio oficial de **GA (disponibilidade geral) do Open Cloud OAuth 2.0** é de 28/09/2023 (DevForum). Tratar como estável com cautela: monitorar o aviso da página a cada release.
- **OAuth2 (registro e produção)** [CONFIRMADO — `oauth2-registration`]: registrar app em `create.roblox.com/dashboard/credentials`; redirect HTTPS (fora de localhost); até 10 redirect URLs. **App recém-criado fica em modo privado com limite de 10 usuários únicos.** Para produção é obrigatório **submeter à revisão** ("Edit and Publish → Submit for Review"): a submissão exige **vídeo de demonstração e justificativa** (usados apenas na revisão, não públicos); solicitar **apenas os escopos mínimos** (Roblox não aprova escopos para uso futuro); durante a revisão o app não pode ser editado nem nova submissão enviada; aprovado, o app vai para modo público e **não pode voltar ao privado**. Requisito prévio: a conta que registra/publica o app precisa estar com **ID verificado** (account verification). `sub` do userinfo é o ID estável; username/display name mudam ⇒ nunca chave.
- **Users API via OAuth2** [CONFIRMADO — anúncio oficial]: endpoints de User sob token OAuth2 têm limite exemplar de **10 req/min por token** (1.000/min por API key) — reforça o orçamento global do worker.
- **Verificação de acesso ao jogo** [TESTAR → probe]:
  - `GET /v1/users/{id}/items/Place/{placeId}/is-owned` — público, sem auth; hipóteses de contrato: 200 `{"robloxId": <id>}` quando possui; 403 quando não possui ou quando inventário é privado (mensagens distintas); 429 em excesso.
  - `GET /v1/users/{id}/can-view-inventory` → `{"canView": bool}` usado para distinguir "privacidade" de "não possui".
  - Nunca automatizar acima do orçamento definido (30 req/min global, fila no worker).
- **Avatares**: `thumbnails.roblox.com/v1/users/avatar-headshot?userIds=...&size=150x150&format=Png` (público) [CONFIRMADO]; OAuth `picture` do userinfo como alternativa [CONFIRMADO].
- **Anti-bot** [INCERTO]: IPs de datacenter podem receber 403 bloqueado pelo Roblox. Probe §3.4 define a política.

### Google AI Studio Build [CONFIRMADO]
- Build mode permite sincronizar bidirecionalmente com GitHub (import repo, push com commit message gerado, pull de IDE/teammates) e publicar em Cloud Run.
- **Uso no projeto**: aceitável para prototipar **telas** do site; o código de produção vive no repositório (a API/bot/worker não são adequados ao fluxo do AI Studio). Integração real fica como opção do time (decisão D-07).

### Stack
- **Next.js 16** [CONFIRMADO]: App Router + Turbopack; Cache Components habilitado por padrão em apps novos a partir de 16.4; `proxy.ts` substitui `middleware.ts`.
- **Auth.js v5** [CONFIRMADO]: provider Discord; JWT em cookie HttpOnly; callbacks `jwt`/`session` para injetar `discordId`/`userId`; permissões sempre recarregadas do banco (nunca confiadas ao token além da identidade).
- **BullMQ** [CONFIRMADO]: exigir Redis com AOF; retries exponential + jitter; tratar `failed` como dead-letter com alarme.
- **SSE** [CONFIRMADO]: route handler Node com `ReadableStream`; cabeçalhos `text/event-stream`, `Cache-Control: no-cache`, `X-Accel-Buffering: no`; heartbeat ~25 s; fallback automático de polling (10 s) no cliente.

## 3.3 O que permanece incerto (com plano)

1. Corpo exato do `is-owned` e mensagens dos 403 → probe §3.4 na fase P2, antes da verificação entrar no ar.
2. Aprovação de produção do app OAuth2 Roblox (prazo/taxa) → iniciar cedo; contingência: torneio fechado ≤10 contas até aprovação.
3. Comportamento de DM bloqueada (código exato) → teste controlado com conta de testes.
4. Estabilidade do OAuth2 beta do Roblox a médio prazo → camada `RobloxIdentityProvider` isolada, permitindo trocar estratégia sem tocar no domínio.

## 3.4 Probe controlado do Roblox (roteiro, fase P2)

`scripts/probe-roblox.mjs` (a criar na fase P2), executado de **IP residencial/CI confiável**, sem segredos:

1. `GET /v1/users/{userId}/can-view-inventory` para 3 usuários (o dono, um tester, um inexistente).
2. `GET /v1/users/{userId}/items/Place/15327728308/is-owned` para os mesmos, espaçados ≥1 s, **máx. 6 requisições totais**.
3. Registrar status code + corpo bruto + headers de rate limit para cada caso (possui / não possui / inventário privado / usuário inexistente).
4. Resultado alimenta a tabela de estados do `GameAccessChecker` (documento 10, §10.4). Se todos retornarem 403 mesmo em IP residencial com `can-view` = true → endpoint mudou; alternativa: `cloud/v2/users/{id}/inventory-items` com API key do dono [CONFIRMADO que existe].
5. Nenhum dado sensível envolvido; nenhuma tentativa de burlar proteção; respeitar `Retry-After`.
