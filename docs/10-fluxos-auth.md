# 10 — Fluxos de autenticação (Discord) e Roblox

## 10.1 Login Discord (OAuth2) [CONFIRMADO na doc oficial]

Escopo mínimo: `identify` (ID, username, avatar). `email` **não** é necessário. Sem `guilds.join` na v1.

Passos:
1. `GET /api/auth/signin/discord` (Auth.js v5, provider Discord).
2. Auth.js gera `state` aleatório, armazena no cookie httpOnly de verificação e monta `https://discord.com/oauth2/authorize?...&state=...`.
3. Callback `/api/auth/callback/discord`: valida `state` (CSRF) → troca `code` por token (server-to-server) → `GET /users/@me`.
4. Upsert `users` por `discord_user_id` (nunca confiar em ID vindo do navegador).
5. Sessão JWT (cookie `HttpOnly`, `Secure`, `SameSite=Lax`, maxAge 7 dias). Claims: `userId`, `discordId` **apenas**. Permissões **não** vivem no token: recarregadas do banco a cada checagem.
6. Logout: limpa cookie; sessões não são persistentes no servidor (JWT). Re-login forçado em 7 dias.

Validações de callback: `state` confere; `code` usado uma vez; erro do Discord → página de erro sem vazar detalhes.

## 10.2 Bootstrap seguro do proprietário

1. Deploy define `OWNER_DISCORD_ID=1102745175098404935` (env; validado: 17–20 dígitos).
2. No primeiro login do usuário cujo `discordId == OWNER_DISCORD_ID`: o sistema garante papel OWNER (idempotente), grava `audit_logs` (`OWNER_BOOTSTRAP`, com timestamp e fonte=env) e responde "Proprietário ativado".
3. Enquanto o dono não loga, `/painel` mostra aviso apenas a quem logou (não há elevação sem login).
4. Procedimento de verificação (ops): comando `pnpm ops:verify-owner` conecta ao banco, compara `users.discord_user_id` com env, e relata o estado — não altera dados.
5. Se o env mudar (transferência de dono), documentar: exige intervenção de ops + audit `OWNER_TRANSFERRED`; nenhum admin pode fazê-lo pela UI.

## 10.3 Vínculo Roblox (OAuth2 Open Cloud) [CONFIRMADO: beta]

Registro prévio (fase P1): app em `create.roblox.com/dashboard/credentials` com redirect `https://<domínio>/api/v1/roblox/callback` (HTTPS obrigatório fora de localhost) e escopos `openid profile`. **App não aprovado = limite de 10 usuários** → solicitar aprovação de produção imediatamente.

Fluxo:
1. Usuário autenticado clica "Vincular Roblox" → `POST /roblox/link/start` → server gera `state` assinado (HMAC com segredo do servidor, contém `userId`, expira em 10 min, uso único) e redireciona para `https://apis.roblox.com/oauth/v1/authorize?client_id=…&redirect_uri=…&scope=openid%20profile&response_type=code&state=…&nonce=…`.
2. Callback: valida `state` + `nonce` → `POST /oauth/v1/token` (client_secret_basic) → `GET /oauth/v1/userinfo`.
3. Extrai: `sub` (Roblox ID), `preferred_username`, `name`, `picture`, `created_at` [CONFIRMADO].
4. Upsert `roblox_accounts` com invariantes:
   - robloxId já vinculado a outro usuário → **409** `CONFLICT_ROBLOX_LINKED` (mostra hint genérico, sem expor quem vinculou);
   - usuário já vinculado → exige desvincular primeiro;
   - username/display são snapshot; ID é a chave.
5. Dispara checagem de acesso (fila) e mostra perfil: nome, ID, avatar, estado do vínculo e **estado de acesso separado**.
6. Tokens Roblox **não são armazenados** (decisão D-05): o vínculo é identidade, não sessão Roblox; re-verificações usam endpoints públicos.
7. Desvincular: `DELETE /roblox/link` — audit `ROBLOX_UNLINKED`; convites/equipes existentes permanecem (elegibilidade reavaliada nas próximas checagens).

Nunca pedir senha ou `.ROBLOSECURITY` do jogador [requisito]. O fluxo nunca envia o code para o navegador (server-to-server).

## 10.4 Verificação de acesso ao jogo — máquina de estados

Endpoint primário (informado pelo dono do projeto):
`GET https://inventory.roblox.com/v1/users/{robloxId}/items/Place/{ROBLOX_PLACE_ID}/is-owned` [TESTAR — probe em 03, §3.4]

Apoio: `GET https://inventory.roblox.com/v1/users/{robloxId}/can-view-inventory` [CONFIRMADO que existe].

| Entrada | Resultado do sistema |
|---|---|
| 200 + indicador de posse | `HAS_ACCESS` |
| 200 + indicador de não-posse (ou 403 "don't own") | `NO_ACCESS` — jogador vê "entre no jogo uma vez e re-verifique" |
| 403 com `can-view-inventory=false` | `INCONCLUSIVE` (`HTTP_403_PRIVACY`) — mensagem orienta tornar inventário visível temporariamente; **não é fraude** |
| 429 / 5xx / timeout | `INCONCLUSIVE` (`HTTP_429`/`UPSTREAM`) — retry automático com backoff; jogador pode re-tentar |
| 404 usuário | erro de dado → `INCONCLUSIVE` + alerta admin |
| Bloqueio anti-bot de IP | `INCONCLUSIVE` + alerta ops (padrão de 403 uniforme) |

Regras:
- Checagem **assíncrona** (worker) com orçamento global (30 req/min) e por usuário (5 re-tentativas/min); jitter; circuit breaker se taxa de falha > 50% em 5 min.
- `INCONCLUSIVE` expõe `nextAttemptAt`; re-tentativa manual nunca marca fraude.
- Elegibilidade de convite/sorteio exige `HAS_ACCESS` (configurável: `requireGameAccess`).
- Alternativa documentada se o endpoint mudar: `apis.roblox.com/cloud/v2/users/{id}/inventory-items` com API key do dono + escopo de inventário [CONFIRMADO que existe, mesma info de posse de places/passes]; e, a médio prazo, verificação nativa no jogo (MessagingService/DataStore) — sugerida ao desenvolvedor do jogo, fora do escopo v1.

## 10.5 Rate limits externos resumidos

| Destino | Limite aplicado pelo sistema |
|---|---|
| Discord REST | ≤ 50 req/s global; buckets por rota respeitados pelo discord.js; outbox com bucket dedicado |
| Roblox inventário/thumbnails | orçamento 30 req/min (fila serial); backoff respeitando `Retry-After` |
| Roblox OAuth | apenas no fluxo do usuário (sem automação) |
