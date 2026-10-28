# 15 — Telas e wireframes (descritivos)

Estética-alvo: escura, "tática/sobrevivência" condizente com Aftermath — fundo `#0B0F14`, superfícies `#131A22`, destaque âmbar `#F59E0B` e verde `#34D399`; tipografia display forte (ex.: Archivo/Space Grotesk) + corpo Inter; cards com borda 1px e sombra sutil; micro-animações discretas. Assets de armas do jogo (pasta `Downloads/aftermath`, `.webp`) podem decorar seções de regras.

## 15.1 Público

**T1 Home** — hero com nome do torneio, status da inscrição (contagem de vagas), CTA "Entrar com Discord"; cartões: prêmios/formato, datas, regras resumidas; feed dos últimos 5 eventos públicos.

**T2 Regras** — renderização de `rules_md`; seções fixas: elegibilidade, W.O., suplentes, punições.

**T3 Chaveamento público** — colunas por rodada (desktop) / abas (mobile); cards de confronto com avatares, estado colorido, vencedora; overlay de partida "AO VIVO — 12:37" quando IN_PROGRESS; atualização SSE; badge BYE/W.O.

**T4 Equipes** — grade de equipes inscritas (avatar do dono, tag, prontidão ✓/⚠, integrantes com estado Roblox); busca.

**T5 Feed** — timeline dos `public_events` (inícios/fins com duração, sorteios, anúncios), infinito paginado.

## 15.2 Jogador (área autenticada)

**T6 Dashboard** — passo a passo do jogador: 1) Vincular Roblox (estado), 2) Criar/entrar em equipe, 3) Convites, 4) Aguardando chaveamento. Barra de progresso + alertas contextuais.

**T7 Minha equipe** — nome/tag, lista de integrantes com checklist (Roblox vinculado, acesso ao jogo), prontidão; ações do owner: convidar (modal busca por Discord), remover, transferir; ações do membro: sair.

**T8 Convites** — recebidos (Aceitar/Recusar) e enviados (estado, cancelar); aviso quando DM falhou ("aceite por aqui").

**T9 Roblox** — card de perfil (avatar, nome, ID), estado do vínculo; estado de acesso com explicação conforme resultado (HAS_ACCESS/NO_ACCESS com instruções/INCONCLUSIVE com "tentar novamente" + `nextAttemptAt`); histórico das últimas checagens.

## 15.3 Admin (painel web)

**T10 Torneios** — lista + criar (wizard: básico → regras → datas → visibilidade → canal Discord).

**T11 Configuração** — formulário da config corrente; ao salvar com conflito: modal "Revisar impactos" (diff + lista de equipes afetadas + botão Confirmar com `confirmToken`); histórico de versões (who/when/diff).

**T12 Equipes (admin)** — tabela com filtros (estado, prontidão, Roblox); ações: aprovar, desqualificar (razão), ajustar roster com audit.

**T13 Partidas** — fila de "aguardando início" e "em andamento" com cronômetro; iniciar/finalizar/cancelar/W.O. com os mesmos modais do bot; correção com o relatório de impactos (13.5); link para histórico.

**T14 Sorteio** — estado atual (equipes prontas x vagas), prévia da distribuição (mock), botões Confirmar/Cancelar, histórico de sorteios.

**T15 Administradores** — lista de grants, conceder (busca por Discord ID + picker de permissões), revogar; aviso "permissões de dono não são editáveis aqui".

**T16 Auditoria** — filtros por ator/ação/alvo/data; linha expansível com before/after JSON; exportação CSV (P5+).

**T17 Anúncios** — editor + prévia + publicar (outbox).

## 15.4 Bot (mapeamento 1:1 com a API)

- `/painel` → menu ephemeral: Minha equipe · Convites · Roblox · (Admin, se for admin) — botões abrem views (T7/T8/T9 equivalentes).
- `/equipe criar|ver|convidar|sair` · `/roblox vincular|status` · `/partida ver` (público) · `/admin start|finalizar|sorteio|equipe|configurar` (configurar abre link p/ T11).
- Todos os comandos admin usam autocomplete + selects paginados; confirmações por botão; respostas ephemeral por padrão; logs públicos via outbox (não via comandos).

## 15.5 Prioridades do painel admin e mapeamento tela → endpoint

Prioridade de construção/refino (o que o dono mais usa):

1. **T13 Central de partidas** — fila "aguardando início"/"em andamento" com cronômetro; iniciar/finalizar/cancelar/W.O. sem sair da tela.
2. **T3 Chaveamento visual** — cartões, estados coloridos, SSE ao vivo, duração.
3. **T14 Sorteio guiado** — prévia → confirmar → histórico.
4. **T11 Configuração** — form + modal de impactos com `confirmToken`.
5. **T12/T4 Equipes** — pesquisa instantânea, filtros (estado, prontidão, Roblox).
6. **T15/T16/T17** — permissões, auditoria, anúncios.

Mapeamento tela → endpoints (contratos completos em 08):

| Tela | Endpoints | Permissão |
|---|---|---|
| T1 Home | `GET /tournaments` · `GET /tournaments/:id` · `GET /tournaments/:id/events` | PUBLIC |
| T2 Regras | `GET /tournaments/:id` | PUBLIC |
| T3 Chaveamento | `GET /tournaments/:id/bracket` · SSE `GET /api/v1/stream/tournaments/:id` | PUBLIC |
| T4 Equipes | `GET /tournaments/:id/teams` | PUBLIC |
| T5 Feed | `GET /tournaments/:id/events` | PUBLIC |
| T6 Dashboard | `GET /me` | SESSION |
| T7 Minha equipe | `POST /teams` · `GET /teams/:id` · `PATCH /teams/:id` · `POST /teams/:id/invites` · `POST /teams/:id/leave` | SESSION |
| T8 Convites | `GET /me/invites` · `POST /invites/:id/accept` · `POST /invites/:id/decline` · `DELETE /invites/:id` | SESSION |
| T9 Roblox | `GET /roblox/link/start` (302) · `GET /roblox/status` · `POST /roblox/access-check` · `DELETE /roblox/link` | SESSION |
| T10 Torneios | `GET /tournaments` · `POST /admin/tournaments` | ADMIN `tournaments.config` |
| T11 Config | `PATCH /admin/tournaments/:id/config` (prévia → confirm) · `POST /admin/tournaments/:id/status` | ADMIN `tournaments.config` |
| T12 Equipes admin | `GET /admin/tournaments/:id/teams` · `PATCH /teams/:id` · desqualificar | ADMIN `teams.manage` |
| T13 Partidas | `GET /admin/tournaments/:id/matches?status=` · `POST /admin/matches/:id/start\|finish\|correct\|cancel\|walkover` | ADMIN `matches.manage` |
| T14 Sorteio | `POST /admin/tournaments/:id/draw/preview\|confirm\|redraw` · `GET /admin/tournaments/:id/draws` | ADMIN `draws.run` |
| T15 Admins | `GET /admin/admins` · `POST /admin/admins` · `DELETE /admin/admins/:discordId` | OWNER |
| T16 Auditoria | `GET /admin/audit` | ADMIN `audit.view` |
| T17 Anúncios | `POST /admin/tournaments/:id/announcements` | ADMIN `announcements.publish` |

Esta tabela é a referência da UI track do AI Studio (21, §21.2): **nenhuma tela inventa endpoint fora dela**.
