# 17 — Segurança

## 17.1 Autenticação e sessão

- OAuth2 Discord/Roblox com `state` anti-CSRF validado server-side; `code` de uso único; trocas server-to-server [CONFIRMADO nas docs].
- Cookie de sessão: nome padrão Auth.js (`authjs.session-token`), `HttpOnly`, `Secure`, `SameSite=Lax`, maxAge 7 dias; JWT assinado (Auth.js v5) com claims mínimos (`userId`, `discordId`), rotacionado a cada login; permissões sempre do banco. Logout por `POST /api/auth/signout` (Auth.js) — limpa o cookie; não há lista de sessões server-side (JWT stateless). **Não existe `/me/logout`** (removido da API canônica — 08, §8.1).
- IDs vindos do cliente **nunca** autorizam: todo endpoint resolve ator no banco e checa permissão (inclusive via bot).

## 17.2 Autorização

- Checagem central em `packages/core` (policy service) — chamada por route handlers antes de qualquer efeito; testes cobrem a matriz 9.3.
- Proteções anti-elevação (9.4): `admins.manage` só dono; admin concede nada; owner só via env+bootstrap.

## 17.3 Validação e abuso

- Zod em 100% dos inputs (query, path, body, headers de ator); snowflakes validados (17–20 dígitos), UUIDs, datas futuras/passadas coerentes.
- **Rate limiting** (Redis, sliding window) — tabela canônica (referência para 08/18):

| Escopo | Limite |
|---|---|
| Login/logout (rotas `/api/auth/*`) | 10/min por IP |
| SSE (novas conexões) | 10/min por usuário, máx. 3 simultâneas |
| Mutações de jogador (equipes/convites) | 30/min por usuário |
| Mutações admin | 120/min por usuário |
| `/roblox/access-check` | 5/min por usuário (+ orçamento global Roblox de 30 req/min no worker) |
| Rotas `INTERNAL` (bot/worker) | 600/min por service token |
| Leitura pública (default) | 60/min por IP |

Respostas 429 com `Retry-After`; abuso repetido → bloqueio temporário + audit.
- **CSRF**: `SameSite=Lax` + validação de `Origin`/`Host` em **toda** mutação com cookie (middleware `proxy.ts` do Next 16); mutações sempre em POST/PATCH/DELETE (GET nunca causa efeito); ações de alto impacto (config, correção de resultado, edição de bracket, redraw, desqualificação) exigem **token de confirmação** assinado — HMAC, 5 min, uso único, emitido na prévia (`confirmToken`), rejeitado com 409 `CONFLICT_CONFIRM_TOKEN` se inválido/expirado.
- **XSS**: sem `dangerouslySetInnerHTML`; regras markdown renderizadas por sanitizer; conteúdo do Discord (nomes) sempre texto escapado.
- **SSRF**: outbound fixo para hosts conhecidos (Discord/Roblox); nenhum fetch por URL informada pelo usuário.
- Uploads: nenhum na v1 (avatars vêm das APIs oficiais).

## 17.4 Segredos e dados

- Segredos só em env (17 no `18-deploy-operacao.md`); nenhum token em banco (tokens Roblox nem são armazenados); logs pino com `redact` de `authorization`, `cookie`, `token` [padrão do repo atual].
- Banco: usuário de app com privilégios mínimos; migrations com usuário próprio; **AuditLog sem UPDATE/DELETE** (revoke no banco: `REVOKE UPDATE, DELETE ON audit_logs FROM app_user`).
- PII mínima: Discord ID/username/avatar + Roblox ID/username/avatar; sem e-mails; sem documentos; LGPD: dados públicos do Discord conforme exibição consentida no fluxo OAuth.

## 17.5 Integridade de domínio

- Transações + `WHERE status=` para todo transit; constraints UNIQUE como última linha; idempotency keys em ações do bot; outbox dedup.
- Migrações só por CI/pipeline revisado; nunca no boot [padrão do repo atual].

## 17.6 Backups e resposta

- Backups diários + WAL (18); teste de restauração mensal.
- Rotação do `SERVICE_TOKEN` e secrets via .env + reinício; playbook de token vazado: revogar no Discord/Roblox dashboards, rotacionar, auditar.
- Dependências: `pnpm audit` no CI; Dependabot/renovate; lockfile commitado.

## 17.7 Monitoramento de segurança

- Alarme: taxa de 401/403 anormal, rate-limit trips, `outbox DEAD`, falha do circuit-breaker Roblox, 5xx.
- Audit feed diário (digest) opcional para o dono em canal privado.
