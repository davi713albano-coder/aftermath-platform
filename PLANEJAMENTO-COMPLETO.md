# AFTERMATH TOURNAMENT PLATFORM - PLANEJAMENTO COMPLETO (DOCUMENTO UNICO)

> **O que e este arquivo:** consolidacao integral do planejamento da plataforma de torneios
> do jogo Roblox **Aftermath** (site + painel admin + bot Discord + backend compartilhado).
> Gerado automaticamente a partir dos arquivos individuais do repositorio **aftermath-platform**
> em **28/10/2026**, ja com a **revisao tecnica v2** incorporada - nenhum conteudo foi alterado ou resumido.
>
> **Como ler:** o documento esta dividido em 23 partes (DOCUMENTO 0 a 22), onde o
> DOCUMENTO 0 e o README do repositorio. Cada secao comeca com um separador indicando o arquivo de origem.
>
> **Para um assistente de IA:** trate cada secao como fonte autoritativa daquela area; as fontes
> de pesquisa citam URL e data de consulta (28/10/2026); afirmacoes marcadas [CONFIRMADO] /
> [TESTAR] / [INCERTO] indicam o grau de verificacao. Contratos de API e catalogo de eventos:
> DOCUMENTO 9 (docs/08-api-contratos.md), secoes 8.0 (convencoes canonicas) e 8.10 (eventos).
> UI track do Google AI Studio: DOCUMENTO 22 (docs/21-ai-studio-integration.md).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 0/22 | ARQUIVO ORIGEM: README.md -->
<!-- ================================================================ -->

# Aftermath Tournament Platform — Planejamento

> Status: **FASE DE PLANEJAMENTO** — nenhum código de produto foi escrito ainda.
> Pesquisa web realizada em **28/10/2026**. Fontes oficiais citadas em `docs/03-pesquisa-web.md`.
> **Revisão técnica v2 aplicada em 28/10/2026** — docs 03, 05–08, 10–15, 17, 19–21 revisados; contratos e eventos canônicos centralizados em `docs/08-api-contratos.md`.

Plataforma profissional de torneios para o jogo Roblox **Aftermath** (Place `15327728308`, Universe `5286749994`): site público, área de jogadores, painel administrativo, bot Discord e backend compartilhado com **fonte única de verdade**.

## Como navegar

| Documento | Conteúdo |
|---|---|
| [`docs/00-indice.md`](docs/00-indice.md) | Índice completo e mapa dos 15 entregáveis exigidos |
| [`docs/01-resumo-executivo.md`](docs/01-resumo-executivo.md) | Resumo executivo e requisitos confirmados |
| [`docs/02-requisitos.md`](docs/02-requisitos.md) | Requisitos funcionais e não funcionais detalhados |
| [`docs/03-pesquisa-web.md`](docs/03-pesquisa-web.md) | Relatório de pesquisa: fontes, datas, fatos × hipóteses |
| [`docs/04-reuso-repositorio-atual.md`](docs/04-reuso-repositorio-atual.md) | Análise do repositório existente ("ap de campeonato") e o que será reutilizado |
| [`docs/05-arquitetura.md`](docs/05-arquitetura.md) | Arquitetura proposta, justificativas e trade-offs |
| [`docs/06-diagramas.md`](docs/06-diagramas.md) | Diagramas de componentes, dados e sequências (Mermaid) |
| [`docs/07-modelo-dados.md`](docs/07-modelo-dados.md) | Modelo de dados, relações, invariantes e esboço de schema |
| [`docs/08-api-contratos.md`](docs/08-api-contratos.md) | Endpoints, contratos, códigos de erro e eventos |
| [`docs/09-matriz-permissoes.md`](docs/09-matriz-permissoes.md) | Papéis e matriz de permissões |
| [`docs/10-fluxos-auth.md`](docs/10-fluxos-auth.md) | Login Discord (OAuth2) e vinculação/verificação Roblox |
| [`docs/11-fluxos-equipes-convites.md`](docs/11-fluxos-equipes-convites.md) | Equipes, convites, elegibilidade |
| [`docs/12-fluxos-sorteio-chaveamento.md`](docs/12-fluxos-sorteio-chaveamento.md) | Sorteio, chaveamento, BYEs, reexecução segura |
| [`docs/13-fluxos-partidas.md`](docs/13-fluxos-partidas.md) | Início/finalização/correção de partidas, W.O., concorrência |
| [`docs/14-sincronizacao-site-bot.md`](docs/14-sincronizacao-site-bot.md) | Estratégia site–bot: API única, outbox, idempotência |
| [`docs/15-telas-ux.md`](docs/15-telas-ux.md) | Wireframes descritivos de todas as telas |
| [`docs/16-plano-testes.md`](docs/16-plano-testes.md) | Testes unitários, integração, concorrência, E2E |
| [`docs/17-seguranca.md`](docs/17-seguranca.md) | Segurança: CSRF, rate limit, segredos, logs |
| [`docs/18-deploy-operacao.md`](docs/18-deploy-operacao.md) | Deploy, variáveis de ambiente, backups, monitoramento |
| [`docs/19-riscos-decisoes.md`](docs/19-riscos-decisoes.md) | Riscos, incertezas, alternativas e decisões pendentes |
| [`docs/20-roadmap.md`](docs/20-roadmap.md) | Roadmap em fases pequenas, ordenadas e verificáveis |
| [`docs/21-ai-studio-integration.md`](docs/21-ai-studio-integration.md) | Integração com Google AI Studio (UI track): contratos, regras de port, fluxo de PR |

## Identificadores do projeto (configuráveis, nunca código fixo)

| Item | Valor inicial | Onde configurado |
|---|---|---|
| Place ID | `15327728308` | `ROBLOX_PLACE_ID` (env) |
| Universe ID | `5286749994` | `ROBLOX_UNIVERSE_ID` (env) |
| Proprietário inicial | `1102745175098404935` | `OWNER_DISCORD_ID` (env) |

Esses identificadores **não substituem autenticação**; são apenas sementes de configuração, com procedimento seguro de inicialização descrito em `docs/10-fluxos-auth.md`.

## Decisões

A lista consolidada está em [`docs/19-riscos-decisoes.md`](docs/19-riscos-decisoes.md). Após a **revisão técnica v2 (28/10/2026)**, estão **RESOLVIDAS**: D-02 a D-09 e D-11 (API no Next, sessão JWT, BullMQ+Redis, sem armazenar tokens Roblox, ordem da verificação de acesso, AI Studio só UI, eliminação simples, cadeia de hash na P5, política de suplentes). **Pendentes** (não bloqueiam o início): **D-01** hosting, **D-10** domínio, **D-12** sua aprovação do roadmap. O risco crítico do OAuth2 Roblox (modo privado ≤10 usuários até aprovação de produção, com processo oficial de review) permanece mapeado como **R-1**.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 1/22 | ARQUIVO ORIGEM: docs/00-indice.md -->
<!-- ================================================================ -->

# 00 — Índice e mapa de entregáveis

Requisito original: 15 entregáveis antes de implementar. Mapa abaixo mostra onde cada um está.

> **Revisão técnica v2 — 28/10/2026**: docs 03, 05, 06, 07, 08, 10, 11, 12, 13, 14, 15, 17, 19 e 20 atualizados e `docs/21-ai-studio-integration.md` criado; `PLANEJAMENTO-COMPLETO.md` regenerado. As convenções canônicas da API (§8.0) e o catálogo de eventos (§8.10) em `08` são a referência única — nenhuma variante de nome de evento ou endpoint é válida em outro documento.

| # | Entregável exigido | Documento |
|---|---|---|
| 1 | Resumo executivo e requisitos confirmados | `01-resumo-executivo.md` + `02-requisitos.md` |
| 2 | Relatório da pesquisa web com links e limitações | `03-pesquisa-web.md` |
| 3 | Arquitetura proposta e justificativa | `05-arquitetura.md` |
| 4 | Diagrama dos componentes e fluxos de dados | `06-diagramas.md` |
| 5 | Modelo de dados e relações | `07-modelo-dados.md` |
| 6 | Lista de endpoints e contratos da API | `08-api-contratos.md` |
| 7 | Matriz de permissões | `09-matriz-permissoes.md` |
| 8 | Fluxos de login Discord e vinculação Roblox | `10-fluxos-auth.md` |
| 9 | Fluxos de equipes, convites, sorteios e partidas | `11`, `12`, `13` |
| 10 | Estratégia de sincronização site–bot | `14-sincronizacao-site-bot.md` |
| 11 | Wireframes / descrição das telas | `15-telas-ux.md` |
| 12 | Plano de testes e segurança | `16-plano-testes.md` + `17-seguranca.md` |
| 13 | Plano de deploy, env vars e backups | `18-deploy-operacao.md` |
| 14 | Riscos, decisões pendentes e alternativas | `19-riscos-decisoes.md` |
| 15 | Roadmap em fases pequenas e verificáveis | `20-roadmap.md` |

## Anexos

- `04-reuso-repositorio-atual.md` — inspeção do repositório existente (`Downloads/ap de campeonato`, monorepo "elite-clubs" para EA FC Pro Clubs). Nada será destruído; o planejamento identifica o que reutilizar, adaptar ou descartar.
- `21-ai-studio-integration.md` — integração com o Google AI Studio (UI track): pacote de contexto, regras de port e fluxo de PR.

## Convenções

- Todas as fontes de pesquisa citam URL + data de consulta (28/10/2026).
- Cada afirmação técnica é classificada: **[CONFIRMADO]** (documentação oficial lida nesta sessão), **[TESTAR]** (contrato plausível, exige teste controlado), **[INCERTO]** (não verificável agora; plano de contingência definido).
- Nenhuma decisão destrutiva sobre dados existentes foi tomada; o repositório atual permanece intacto.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 2/22 | ARQUIVO ORIGEM: docs/01-resumo-executivo.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 3/22 | ARQUIVO ORIGEM: docs/02-requisitos.md -->
<!-- ================================================================ -->

# 02 — Requisitos

Legenda: **RF** = requisito funcional; **RNF** = requisito não funcional. "Fase" indica a fase do roadmap (`20-roadmap.md`) em que entra.

## RF — Identidade e autenticação

- **RF-01 (P1)** Login exclusivo por Discord OAuth2 (authorization code + `state` anti-CSRF), sessão segura em cookie `HttpOnly`, logout e expiração.
- **RF-02 (P1)** Identidade do jogador = ID numérico do Discord (snowflake validado). Nomes/avatares são cache de exibição, nunca chave.
- **RF-03 (P1)** Proprietário inicial definido por `OWNER_DISCORD_ID`; procedimento de bootstrap seguro (nada de ID colado no código).
- **RF-04 (P2)** Vínculo de conta Roblox via OAuth2 do Open Cloud (escopos `openid profile`); perfil exibe nome, ID, avatar e estado de verificação.
- **RF-05 (P2)** Verificação de acesso ao jogo (endpoint de inventário informado) é **estado separado** do vínculo de conta; resultados inconclusivos não são fraude.
- **RF-06 (P1)** É proibido confiar em IDs enviados livremente pelo navegador para autorização; todo ID vira lookup no banco.

## RF — Configuração do torneio (web, dono/admin)

- **RF-07 (P2)** Configurar: nome, descrição, regras, máx. equipes, jogadores por equipe, datas de inscrição/encerramento, formato, política de suplentes, regras de W.O./desistência/equipes incompletas, visibilidade pública, canal do Discord de registros públicos.
- **RF-08 (P2)** Valores default (ex.: 12 equipes × 4 jogadores) editáveis; nada fixo em código.
- **RF-09 (P2)** Alteração de configuração com inscrições/partidas existentes calcula **impacto**, exige confirmação explícita e grava versão da config + auditoria. Mudanças incompatíveis são bloqueadas com 409 + lista de impactos.

## RF — Equipes e convites

- **RF-10 (P2)** Jogador autenticado cria equipe (se dentro das regras), vê a própria equipe, integrantes, estado de verificação de cada um.
- **RF-11 (P2)** Convites a membros elegíveis por seleção no Discord (user select) ou e-mail/id no site; convites têm validade, estados explícitos e proteção contra dupla aceitação (transação + constraint).
- **RF-12 (P2)** Falha no envio de DM **não perde o convite**: convite continua pendente e aceitável pelo site/`/painel`.
- **RF-13 (P2)** Impedida a pertença simultânea a equipes incompatíveis (uma equipe ativa por usuário por torneio — partial unique index).
- **RF-14 (P2)** Equipe só é "pronta" quando cumpre a composição configurada (titulares + suplentes conforme política).

## RF — Sorteio e chaveamento

- **RF-15 (P3)** Sorteio iniciado pelo site ou Discord com **prévia** antes de confirmar; valida elegibilidade (equipes prontas, sem duplicidade).
- **RF-16 (P3)** Sorteio não substitui chaveamento já iniciado; reexecução exige fluxo explícito e o resultado gravado é imutável (autor, data, seed, config snapshot).
- **RF-17 (P3)** Chaveamento visual responsivo: rodadas, confrontos, vencedores, estados; avanço automático dos vencedores; tratamento de quantidade ímpar e BYEs; histórico de sorteios.
- **RF-18 (P3)** Recarregar a página nunca altera resultado (dados derivam do banco, não do cliente).

## RF — Partidas

- **RF-19 (P4)** Registro por admin (site ou Discord `/admin start`): seleciona confronto em lista pesquisável (sem digitar nomes), confirma; timestamp confiável do servidor.
- **RF-20 (P4)** Finalização: admin seleciona equipe vencedora (entre as duas); duração calculada dos timestamps persistidos; sem campos de placar.
- **RF-21 (P4)** Proteções: não iniciar partida já iniciada/concluída, não finalizar inexistente, não registrar resultado incompatível com o chaveamento; transações + condicional `WHERE status='PENDING'` contra corrida.
- **RF-22 (P4)** Correção de resultado com análise de impacto nas partidas dependentes; invalidação controlada; histórico preservado.
- **RF-23 (P4)** W.O. e desistência como operações administrativas auditadas.

## RF — Permissões

- **RF-24 (P1)** Três categorias: PROPRIETÁRIO (env), ADMINISTRADORES (concedidos pelo dono, permissões granulares), JOGADORES.
- **RF-25 (P1)** Dono concede/revoga/configura admins; admin não se promove, não concede acima do próprio nível, não altera identidade do dono; validação sempre no backend.

## RF — Bot Discord

- **RF-26 (P5)** Comandos: `/painel`, `/equipe`, `/roblox`, `/partida`, `/admin start`, `/admin finalizar`, `/admin sorteio`, `/admin equipe`, `/admin configurar`.
- **RF-27 (P5)** Menus pesquisáveis e seletores (autocomplete + string select ≤25 opções + paginação) para escolher equipes/confrontos.
- **RF-28 (P5)** Sem comandos de resultado para jogadores; o bot lê da mesma API (sem banco próprio).

## RF — Logs

- **RF-29 (P5)** Log público no Discord (início/fim de partida, vencedora + duração, sorteios, mudanças relevantes, anúncios) via outbox com reentrega.
- **RF-30 (P1)** Auditoria privada append-only: autor, ação, data, entidade, before/after; não editável por admins comuns; fonte oficial é o banco, não mensagens do Discord.

## RF — Público

- **RF-31 (P3)** Site público: regras, equipes inscritas, chaveamento, feed de eventos, atualização em tempo real (SSE com fallback de polling).

## RNF

- **RNF-01** Segurança: validação de entrada (Zod) em 100% dos endpoints; CSRF; rate limiting; segredos só no servidor; logs sem tokens.
- **RNF-02** Confiabilidade: transações, idempotência, retries com backoff, outbox, reconciliação.
- **RNF-03** Qualidade: testes unitários/integração/E2E; testes de concorrência dedicados (convites, partidas, sorteios).
- **RNF-04** Operação: migrações controladas (Prisma Migrate, nunca no boot), backups diários com teste de restauração, healthchecks, logs estruturados (pino).
- **RNF-05** Desempenho alvo: chaveamento com 128 equipes renderiza < 2 s; API p95 < 300 ms (exceto chamadas externas Roblox/Discord).
- **RNF-06** Acessibilidade básica do site (contraste, foco, labels) e mobile-first.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 4/22 | ARQUIVO ORIGEM: docs/03-pesquisa-web.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 5/22 | ARQUIVO ORIGEM: docs/04-reuso-repositorio-atual.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 6/22 | ARQUIVO ORIGEM: docs/05-arquitetura.md -->
<!-- ================================================================ -->

# 05 — Arquitetura proposta

## 5.1 Visão geral

```
                        ┌─────────────────────────────┐
   Navegador ──────────▶│  apps/web (Next.js 16)      │
   Jogadores/Admins     │  Site + Painel + API /api/v1│──────┐
                        └───────────────┬─────────────┘      │
                                        │ REST (service      │
                                        │ token + ator)      │
                        ┌───────────────▼─────────────┐      │
   Discord ──gateway───▶│  apps/bot (discord.js v14)  │      │
                        │  sem banco; só consome API  │      │
                        └───────────────┬─────────────┘      │
                                        │                    │
                        ┌───────────────▼─────────────┐      ▼
                        │  apps/worker (BullMQ)       │   PostgreSQL 16
                        │  outbox Discord, checagens  │   (Prisma 6)
                        │  Roblox, expirações, cron   │
                        └───────┬───────────────┬─────┘      ▲
                                │               │            │
                            Redis 7      APIs externas──────┘
                            (fila+lock)  (Discord REST, Roblox)
```

## 5.2 Componentes e responsabilidades

| Componente | Tecnologia | Responsabilidade | Proibido |
|---|---|---|---|
| `apps/web` | Next.js 16 (App Router), React 19 | Site público, área do jogador, painel admin, **API REST `/api/v1`** (route handlers), SSE, autenticação Auth.js | Regra de negócio inline; acesso direto do cliente ao banco |
| `apps/bot` | discord.js 14 | Interações (comandos, selects, botões), envio de DMs, painéis V2 | Ler/escrever banco; guardar resultados; decidir permissão |
| `apps/worker` | Node + BullMQ | Outbox→Discord, checagens Roblox (orçamento de rate), expiração de convites, reconciliação, cron de manutenção | Regra de negócio (apenas executa tarefas e grava resultados) |
| `packages/core` | TS puro, sem IO | Máquinas de estado, motor de chaveamento, validação de config, política de permissões | Imports de Prisma/Discord/Next |
| `packages/schemas` | Zod | Contratos de entrada/saída compartilhados web/bot | — |
| `packages/db` | Prisma 6 + Postgres | Modelos, migrações, repositories com transações | Lógica de domínio |
| `packages/integrations` | fetch | Clientes Roblox (OAuth2, inventário, thumbnails) e Discord REST, com cache/rate-limit/backoff (padrão do repo antigo) | Conhecer o domínio |
| `packages/shared` | pino, ioredis | env tipada, logger, lock, idempotência, erros de domínio | — |
| `packages/ui` | discord.js builders | Painéis Components V2, custom-ids tipados | Chamadas de rede de negócio |

## 5.3 Decisões arquiteturais e justificativas

**D1 — API dentro do Next (BFF único), não API separada.**
O requisito "site e bot usam a mesma API e a mesma fonte de dados" é atendido por `/api/v1` (route handlers Node). Uma API separada (Fastify/Nest) duplicaria auth, deploy e tipos na v1. O core fica em `packages/core`, então extrair uma API dedicada depois é mecânico. *Trade-off*: route handlers compartilham o processo do site (escala conjunta) — aceitável para o porte do produto; documentado como ponto de extração futura.

**D2 — Bot sem banco.**
Diferente do repo antigo (bot → core/DB direto), aqui o bot chama `/api/v1` com `SERVICE_TOKEN` + ator (`X-Acting-Discord-Id`). Garante fonte única, permite rate limit/auditoria centralizados e mantém o bot stateless (facilita deploy). *Trade-off*: latência extra + o bot depende da API no ar — mitigado com mensagens de erro claras e healthcheck.

**D3 — PostgreSQL + Prisma como fonte de verdade; Redis apenas para fila/lock/rate-limit.**
Invariantes críticos (uma partida uma vez, convite único, uma equipe por jogador) vivem em **constraints do Postgres + transações**, nunca só no Redis (o lock é otimização de contenção — padrão já provado no repo antigo).

**D4 — Outbox transacional para Discord.**
Toda mudança que deve gerar log público grava um `outbox_event` **na mesma transação** do domínio; o worker publica no Discord com retries (BullMQ, backoff exponencial + jitter). Falha no Discord nunca perde o fato — o feed do site lê o banco, não o Discord. Atende exatamente o requisito "se a publicação falhar, a partida continua salva e a notificação pode ser reenviada".

**D5 — Fila BullMQ + Redis 7 (AOF).**
Já é o padrão do repositório atual (ioredis no lock), o worker existe de qualquer forma (Roblox + Discord). Alternativa avaliada: `pg-boss` (fila no Postgres, um serviço a menos) — mantida como alternativa B em 19, decisão D-04 para aprovação.

**D6 — Realtime com SSE + fallback de polling.**
SSE (route handler Node, `runtime='nodejs'`, heartbeat 25 s, `Last-Event-ID`) para chaveamento/feed; o cliente troca para polling de 10 s automaticamente se o stream falhar. WebSockets rejeitados na v1: só leitura servidor→cliente é necessária.

**D7 — Monorepo pnpm sem Turborepo na v1.** Scripts `pnpm -r` bastam neste porte (padrão do repo atual). Turbo é adição trivial depois.

**D8 — Deploy alvo: Docker Compose em VPS.** O bot exige conexão de gateway persistente; manter web+bot+worker+pg+redis no mesmo host simplifica operação da v1. Alternativas (Cloud Run para web + VPS para bot; tudo na Oracle/AWS) em 19/D-01.

## 5.4 Fluxo de dados canônico (exemplo: finalizar partida)

1. Admin clica "Finalizar" no site (ou `/admin finalizar`) → `POST /api/v1/matches/:id/finish {winnerTeamId}` (site: cookie; bot: service token + ator).
2. Route handler valida Zod + permissão (banco) → chama `core.match.finish()` dentro de transação Prisma:
   - `UPDATE matches SET status='COMPLETED', winner_team_id=..., ended_at=now(), duration_seconds=... WHERE id=? AND status='IN_PROGRESS'` → `rowCount===1` obrigatório;
   - avança vencedora no confronto seguinte; grava `match_events` (histórico) e `audit_logs`;
   - insere `outbox_events` (`match.finished`) — mesma transação.
3. Commit. Resposta 200 com o estado novo.
4. Worker consome outbox → publica no canal do Discord → marca `DELIVERED` + `discord_message_id`. Falha → retry com backoff; `DEAD` após N tentativas → alarme.
5. SSE empurra snapshot do chaveamento; clientes atualizam.

Nenhum componente mantém cópia: site e bot leem o mesmo banco via a mesma API.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 7/22 | ARQUIVO ORIGEM: docs/06-diagramas.md -->
<!-- ================================================================ -->

# 06 — Diagramas

## 6.1 Diagrama de componentes

```mermaid
flowchart LR
    subgraph Clientes
        B[Navegador - jogador/admin]
        D[Discord]
    end

    subgraph Plataforma
        W[apps/web - Next.js 16<br/>site + painel + API /api/v1 + SSE]
        BOT[apps/bot - discord.js<br/>stateless]
        WRK[apps/worker - BullMQ<br/>outbox / roblox / cron]
    end

    subgraph Infra
        PG[(PostgreSQL 16<br/>fonte única de verdade)]
        RD[(Redis 7<br/>filas + locks + rate limit)]
    end

    subgraph Externos
        DAPI[Discord REST/Gateway]
        ROB[Roblox APIs<br/>OAuth2 / inventário / thumbnails]
    end

    B -->|HTTPS + cookie sessão| W
    D <-->|gateway intents| BOT
    BOT -->|REST + service token + ator| W
    W -->|Prisma| PG
    W -->|BullMQ producer| RD
    WRK <-->|BullMQ consumer| RD
    WRK -->|Prisma| PG
    WRK -->|REST outbox| DAPI
    WRK -->|checagens c/ orçamento| ROB
    W -->|OAuth callbacks| ROB
    W -->|login OAuth2| DAPI
```

## 6.2 Modelo entidade-relacionamento (simplificado)

```mermaid
erDiagram
    USER ||--o| ROBLOX_ACCOUNT : "1:1 vinculado"
    USER ||--o{ TEAM_MEMBER : "participa"
    USER ||--o{ INVITE_RECEIVED : "recebe"
    USER ||--o{ ADMIN_GRANT : "tem"
    TOURNAMENT ||--o{ TEAM : "tem"
    TOURNAMENT ||--o{ BRACKET : "1 versão ativa"
    TOURNAMENT ||--o{ CONFIG_VERSION : "versionado"
    TEAM ||--o{ TEAM_MEMBER : "composta"
    TEAM ||--o{ INVITE : "recebe"
    BRACKET ||--o{ MATCH : "contém"
    MATCH }o--|| TEAM : "teamA/teamB/winner"
    MATCH ||--o{ MATCH_EVENT : "histórico"
    TOURNAMENT ||--o{ DRAW : "sorteios"
    TOURNAMENT ||--o{ PUBLIC_EVENT : "feed público"
    TOURNAMENT ||--o{ OUTBOX_EVENT : "entregas Discord"
    USER ||--o{ AUDIT_LOG : "ator"
```

## 6.3 Sequência — login Discord + bootstrap do proprietário

```mermaid
sequenceDiagram
    actor U as Usuário
    participant W as apps/web
    participant D as Discord OAuth2
    participant PG as PostgreSQL

    U->>W: GET /api/auth/signin/discord
    W->>D: authorize (scope=identify, state anti-CSRF)
    D-->>U: tela de consentimento
    U->>D: autoriza
    D->>W: callback ?code&state
    W->>D: POST token (code)
    W->>D: GET users/@me (id, username, avatar)
    W->>PG: upsert user (por discordId) + sessão
    alt discordId == OWNER_DISCORD_ID (env)
        W->>PG: garante papel OWNER (idempotente) + audit OWNER_BOOTSTRAP
    end
    W-->>U: cookie HttpOnly (JWT) + redirect
```

## 6.4 Sequência — vínculo Roblox e verificação de acesso

```mermaid
sequenceDiagram
    actor U as Jogador
    participant W as apps/web
    participant R as Roblox
    participant Q as Fila (BullMQ)
    participant WK as Worker

    U->>W: GET /api/v1/roblox/link/start (302)
    W-->>U: redirect apis.roblox.com/oauth/v1/authorize (openid profile, state assinado 10 min)
    U->>R: consentimento
    R->>W: callback ?code&state
    W->>R: POST /oauth/v1/token → userinfo
    W->>W: upsert roblox_accounts (robloxId único; conflito → 409)
    W->>Q: enfileira game-access-check (dedup por robloxId)
    WK->>R: GET inventory .../is-owned (+ can-view-inventory se 403)
    alt 200 possui
        WK->>W: state=HAS_ACCESS
    else 403 "don't own"
        WK->>W: state=NO_ACCESS
    else 403 privacidade / 429 / timeout
        WK->>W: state=INCONCLUSIVE + nextAttemptAt (backoff)
    end
    U->>W: GET /api/v1/roblox/status (estado + botão "tentar novamente")
```

## 6.5 Sequência — início e finalização de partida (com concorrência)

```mermaid
sequenceDiagram
    actor A as Admin
    participant C as Cliente (site/bot)
    participant API as /api/v1
    participant PG as PostgreSQL (transação)
    participant O as Outbox→Worker→Discord

    A->>C: /admin start (autocomplete de confrontos pendentes)
    C->>API: POST /admin/matches/:id/start
    API->>PG: BEGIN; UPDATE matches SET status='IN_PROGRESS', started_at=now() WHERE id=? AND status='PENDING'
    alt rowCount = 1
        PG->>PG: match_events(iniciar) + audit + outbox(match.started); COMMIT
        API-->>C: 200 {match}
        O->>O: publica log público (retry com backoff)
    else rowCount = 0
        PG-->>API: ROLLBACK
        API-->>C: 409 CONFLICT_STATUS (alguém iniciou/estado mudou)
    end

    A->>C: /admin finalizar (select entre teamA/teamB)
    C->>API: POST /admin/matches/:id/finish {winnerTeamId}
    API->>PG: valida winner ∈ {A,B}; UPDATE ... WHERE status='IN_PROGRESS'
    PG->>PG: avança chaveamento + match_events + audit + outbox(match.finished); COMMIT
    API-->>C: 200 {match, nextMatch}
    O->>O: log "Equipe X venceu em 14m32s"
```

## 6.6 Fluxo do outbox (requisito de sincronização)

```mermaid
flowchart TB
    TX[Transação de domínio] -->|INSERT outbox_events na mesma tx| PG
    PG -->|job enfileirado pós-commit| RD[(Redis/BullMQ)]
    RD --> WK[Worker]
    WK -->|POST /channels/:id/messages| DC[Discord]
    DC -->|200| WK -->|status=DELIVERED + message_id| PG
    DC -->|429/5xx| WK -->|attempts++, next_attempt_at backoff| PG
    WK -->|N falhas| PG[status=DEAD → alarme]
    CRON[Cron de reconciliação] -->|reenvia PENDING antigos| RD
    SITE[Feed do site] -->|lê public_events| PG
```

---

<!-- ================================================================ -->
<!-- DOCUMENTO 8/22 | ARQUIVO ORIGEM: docs/07-modelo-dados.md -->
<!-- ================================================================ -->

# 07 — Modelo de dados

Convenções herdadas do repositório atual [CONFIRMADO como boas práticas]: PK UUID gerada pelo banco; `timestamptz`; snowflakes do Discord como `String`; enums nomeados; constraints UNIQUE como garantia definitiva de concorrência; histórico via tabelas de eventos + `AuditLog` append-only. **Sem campos de placar.**

## 7.1 Entidades e relações

### users — identidade de login
| Campo | Tipo | Notas |
|---|---|---|
| id | uuid PK | |
| discord_user_id | String unique | identidade estável |
| username / global_name / avatar_hash | String? | cache de exibição, sincronizado no login |
| status | enum ACTIVE/BANNED | ban bloqueia novas ações, não histórico |
| last_login_at | timestamptz? | |

### roblox_accounts — vínculo 1:1 com Roblox (OAuth2)
| Campo | Tipo | Notas |
|---|---|---|
| user_id | uuid FK unique | 1 usuário = 1 conta Roblox |
| roblox_id | String unique | `sub` do userinfo — 1 conta Roblox = 1 usuário |
| roblox_username / display_name | String | snapshot (mudam; nunca chave) |
| avatar_url | String? | `picture` do userinfo ou thumbnails API |
| linked_at / unlinked_at | timestamptz | desvinculo mantém histórico |
| **Sem tokens Roblox armazenados** | — | decisão D-05 (19): só identidade; re-verificação usa endpoints públicos |

### game_access_checks — estado de acesso ao jogo (separado do vínculo)
| Campo | Tipo | Notas |
|---|---|---|
| roblox_id | String | |
| place_id | int | de `ROBLOX_PLACE_ID` |
| state | enum HAS_ACCESS / NO_ACCESS / INCONCLUSIVE / UNKNOWN | INCONCLUSIVE ≠ fraude (timeout/403 privacidade/429) |
| reason_code | String? | `HTTP_403_PRIVACY`, `HTTP_429`, `TIMEOUT`, `HTTP_403_NOT_OWNED`… |
| checked_at / next_attempt_at | timestamptz | backoff exponencial p/ inconclusivos |
| attempts | int | |

### tournaments — torneio + config corrente
Campos fortes: `name`, `description`, `rules_md`, `status` (DRAFT/REG_OPEN/REG_CLOSED/RUNNING/FINISHED/CANCELLED), `max_teams`, `players_per_team`, `substitutes_per_team` (0 = desativado), `format` (SINGLE_ELIMINATION; modelo pronto p/ DOUBLE), `registration_opens_at/closes_at`, `starts_at`, `visibility` (jsonb: o que é público), `discord_log_channel_id`, `wo_grace_minutes`, `incomplete_team_policy` (ALLOW/BLOCK_AT_DRAW), `created_by_user_id`. Default: 12 equipes × 4 jogadores — **editáveis, nada fixo**.

### tournament_config_versions — trilha de configuração
`config jsonb` (snapshot completo), `changed_by`, `changed_at`, `impacts jsonb` (relatório confirmado), `request_ip_hash`. Uma linha por mudança aceita; a config corrente é a última.

### teams
| Campo | Notas |
|---|---|
| tournament_id FK, name, tag? | `@@unique([tournament_id, lower(name)])` |
| owner_user_id | criador/capitão |
| status | DRAFT / READY / LOCKED (pós-sorteio) / DISQUALIFIED |
| locked_at | congelada no sorteio |

### team_members
`team_id`, `tournament_id` (FK **denormalizada** na linha do membro — preenchida na transação de entrada e mantida em sync com `teams.tournament_id`; nunca editável isoladamente), `user_id`, `role` (OWNER/MEMBER/SUBSTITUTE), `joined_at`, `left_at?`.

**Invariantes (regra definitiva — sem alternativas)**:
- `@@unique([team_id, user_id])` — um vínculo por equipe;
- **Um usuário só tem UMA participação ativa por torneio — incluindo suplentes.** Implementação definitiva: coluna `tournament_id` denormalizada + **partial unique index**:
  ```sql
  CREATE UNIQUE INDEX uq_team_members_active_per_tournament
    ON team_members (tournament_id, user_id)
    WHERE left_at IS NULL;
  ```
  Sem exceção por `role`: suplente também não pode estar ativo em duas equipes. O Prisma não modela partial unique index → criar via migration SQL manual + teste de integração dedicado (16, §16.4).
- **Concorrência de aceitação de convite**: a transação de accept executa como primeira instrução `SELECT pg_advisory_xact_lock(hashtextextended('team:' || :tournamentId || ':' || :userId, 0))`, depois revalida elegibilidade/vagas, insere e commita. Dois accepts paralelos do mesmo usuário (mesmo contra equipes diferentes) serializam no lock; o segundo encontra a participação já ativa e falha com `CONFLICT_TEAM_MEMBER`. A partial unique index permanece como **última linha de defesa**: o lock é otimização de contenção, a constraint é a invariante.

### invites
| Campo | Notas |
|---|---|
| team_id, tournament_id, invitee_user_id, invited_by_user_id | |
| status | PENDING / ACCEPTED / DECLINED / CANCELLED / EXPIRED |
| dm_state | SENT / FAILED / UNKNOWN (falha de DM não perde convite) |
| expires_at | padrão 48 h configurável |
| responded_at | |

**Invariantes**: `@@unique([invitee_user_id, tournament_id, status])` não funciona p/ multi-status ⇒ partial unique index: `(invitee_user_id, tournament_id) WHERE status='PENDING'` — no máx. 1 convite pendente por usuário por torneio; aceitação em transação com `UPDATE invites SET status='ACCEPTED' WHERE id=? AND status='PENDING'`.

### brackets
`tournament_id @unique` (v1 ativa), `format`, `size` (potência de 2), `rounds`, `seed_hash` (determinístico p/ auditoria do sorteio), `structure jsonb` (slots + links), `draw_id`, `status` (ACTIVE/FINALIZED), `version`. Chaveamento **nunca** é regerado implicitamente.

### matches
| Campo | Notas |
|---|---|
| bracket_id, round, slot | `@@unique([bracket_id, round, slot])` — idempotência da geração |
| team_a_id?, team_b_id? | `NULL` = TBD (aguardando confrontos anteriores/BYE resolvido) |
| status | PENDING / IN_PROGRESS / COMPLETED / CANCELLED (sem DISPUTED: sem contestação de jogador) |
| match_kind | NORMAL / BYE / WALKOVER |
| started_at, ended_at | **server clock** UTC |
| duration_seconds | calculado no UPDATE (`EXTRACT(EPOCH FROM ended_at-started_at)`) |
| winner_team_id? | ∈ {team_a, team_b} validado |
| recorded_by_user_id | admin responsável |
| next_match_id?, next_slot? ('A'|'B') | para avanço automático |

### match_events — histórico por partida
`match_id`, `type` (STARTED/FINISHED/CORRECTED/CANCELLED/WALKOVER/TEAM_SWAPPED), `actor_user_id?`, `before jsonb`, `after jsonb`, `reason?`, `created_at`. **Append-only.** Correções nunca UPDATE em silêncio.

### draws — sorteios imutáveis
`tournament_id`, `performed_by_user_id`, `source` (WEB/DISCORD), `seed` (random cripto do servidor), `config_snapshot jsonb`, `eligible_teams jsonb` (ids no momento), `draw_result jsonb` (ordem final), `created_at`. Reexecutar a página não re-sortear: resultado vem daqui.

### config_change_requests (opcional P2+)
`proposed jsonb`, `impacts jsonb`, `status` (PENDING/APPLIED/DISCARDED), `requested_by`. Suporta o modal "revise os impactos e confirme".

### public_events — feed público (fonte de verdade do log)
`tournament_id`, `type` — valores do **catálogo canônico de eventos** (08, §8.10): `match.started`, `match.finished`, `match.corrected`, `match.cancelled`, `match.walkover`, `draw.confirmed`, `draw.superseded`, `bracket.updated`, `tournament.config_changed`, `tournament.status_changed`, `tournament.announcement_published`, `team.created`, `team.disqualified`, `team.member_joined`, `team.member_left` —, `payload jsonb` (dados já **públicos**), `created_at`, `visibility`. O site lê daqui; o Discord é espelho. Eventos privados (convites, vínculo Roblox) vão apenas para `audit_logs`/outbox dirigido — nunca ao feed público.

### outbox_events — entrega Discord
`dedup_key @unique` (ex.: `outbox:match.finished:<matchId>`), `type`, `target` (`channel_id` | `user_id`), `payload jsonb`, `status` (PENDING/DELIVERED/FAILED/DEAD), `attempts`, `next_attempt_at`, `discord_message_id?`, `created_at`. **Insert na mesma transação do domínio.**

### admin_grants — permissões granulares
`discord_user_id` (chave por identidade Discord — grant existe antes do primeiro login), `permissions String[]` (enum fechado), `granted_by_discord_user_id`, `granted_at`, `revoked_at?`, `note?`. **Sem linha OWNER aqui**: dono é derivado de `OWNER_DISCORD_ID` + tabela `owner_bootstrap` (ver 10).

### audit_logs — append-only
`actor_user_id?`, `actor_discord_id?`, `actor_type` (USER/ADMIN/SYSTEM/BOT), `action`, `target_type`, `target_id`, `before jsonb?`, `after jsonb?`, `reason?`, `ip_hash?`, `correlation_id?`, `created_at`. Índices por target/ator/data. **Sem endpoint de edição/exclusão**; opcional P4: cadeia de hash (`prev_hash`) p/ tamper-evidence (decisão D-09).

### announcements
`tournament_id`, `title`, `body`, `published_by`, `published_at`, `discord_message_id?` (pós-outbox).

## 7.2 Diagrama

Ver `06-diagramas.md` §6.2.

## 7.3 Invariantes globais (resumo para os testes)

1. Partida só transitada uma vez: `UPDATE ... WHERE status=<esperado>` + rowCount check.
2. Um convite pendente por usuário/torneio; aceitação atômica com transição de status.
3. Uma equipe ativa por usuário por torneio; uma equipe por torneio por nome.
4. Vencedora ∈ {teamA, teamB}; partida finalizada só com started_at válido; duração ≥ 0.
5. Avanço de chaveamento apenas se partida COMPLETED e slot destino ainda vazio ou idêntico (idempotente).
6. Config incompatível com estado atual → bloqueada (409 + impactos) até confirmação explícita.
7. `outbox_events.dedup_key` único ⇒ reexecução nunca duplica mensagem.
8. AuditLog só INSERT (sem UPDATE/DELETE em API; revoke via permission no banco, não em app).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 9/22 | ARQUIVO ORIGEM: docs/08-api-contratos.md -->
<!-- ================================================================ -->

# 08 — API: endpoints e contratos (referência canônica)

Base: **`/api/v1`** (route handlers do `apps/web`). Este documento é a **única fonte de contratos** para site, bot, worker e a UI track do AI Studio (21). Schemas Zod em `packages/schemas`, compartilhados por web e bot. **Nenhum outro documento pode definir endpoint, nome de evento ou formato de payload divergente.**

## 8.0 Convenções canônicas

| Tema | Regra |
|---|---|
| JSON | campos em `camelCase` |
| IDs | sempre **string** (UUID interno, snowflake do Discord, ID Roblox) — nunca `number` (perda de precisão em snowflakes) |
| Datas | ISO-8601 UTC com sufixo `Z` (`2026-10-28T18:00:00Z`); durações em segundos inteiros (`durationSeconds`) |
| Enums | `UPPER_SNAKE_CASE` (`IN_PROGRESS`, `SINGLE_ELIMINATION`, `HAS_ACCESS`) |
| Paginação | query `?page=1&limit=25` (máx. 100) → resposta `{ "items": [...], "page": 1, "limit": 25, "total": 42, "hasMore": true }` |
| Erros | envelope único em **todos** os erros: `{ "error": { "code": "…", "message": "…", "details": { … } } }` (`details` opcional) |
| Eventos | catálogo canônico §8.10 — nomes `dotted lowercase` em `public_events`, `outbox_events` e SSE; nenhuma variante (`match_started`, `MATCH_STARTED`) é válida |
| Segredos | nunca em query string; `SERVICE_TOKEN` e cookies só em headers |

**Classes de autenticação** (coluna "Auth" das tabelas abaixo):

- `PUBLIC` — sem auth; somente dados públicos conforme `visibility` do torneio;
- `SESSION` — cookie Auth.js (navegador);
- `ADMIN` — SESSION + permissão granular **recarregada do banco a cada chamada** (09) — esconder botão na UI não é autorização;
- `INTERNAL` — `Authorization: Bearer SERVICE_TOKEN` (bot/worker); quando houver ator humano: header `X-Acting-Discord-Id` (resolvido e autorizado no banco).

**Idempotência**: mutações originadas no bot enviam header `Idempotency-Key` (derivada do `interactionId`); repetição devolve a resposta original sem novo efeito.

**Códigos de erro**: `VALIDATION_ERROR` (400) · `UNAUTHENTICATED` (401) · `FORBIDDEN` (403) · `NOT_FOUND` (404) · `CONFLICT_*` (409: `_MATCH_STATUS`, `_INVITE_STATE`, `_TEAM_MEMBER`, `_CONFIG_INCOMPATIBLE`, `_BRACKET_EXISTS`, `_BRACKET_EDIT`, `_ROBLOX_LINKED`, `_CONFIRM_TOKEN`) · `RATE_LIMITED` (429 + `Retry-After`) · `UPSTREAM_*` (502/504 p/ Discord/Roblox) · `INTERNAL` (500).

## 8.1 Auth (Auth.js v5) — fora de `/api/v1`

| Método | Rota | Descrição |
|---|---|---|
| GET | `/api/auth/signin/discord` | inicia login Discord (scope `identify`) |
| GET | `/api/auth/callback/discord` | callback OAuth2 (valida `state` anti-CSRF) |
| POST | `/api/auth/signout` | logout (limpa cookie) — **é o único logout; não existe `/me/logout`** |

## 8.2 Sessão / perfil

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/me` | SESSION | user + roblox (nome/ID/avatar/estado de acesso) + equipe ativa + permissões efetivas |
| GET | `/me/invites` | SESSION | convites recebidos/enviados |
| GET | `/me/permissions` | SESSION | o que o próprio usuário pode |

## 8.3 Roblox

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/roblox/link/start` | SESSION | gera `state` assinado (HMAC, 10 min, uso único) e responde **302** para o authorize do Roblox (`openid profile`) — navegação sempre via GET |
| GET | `/roblox/callback` | — | valida state/nonce → token → userinfo → upsert `roblox_accounts`; 409 `_ROBLOX_LINKED` se o robloxId já pertence a outro usuário |
| DELETE | `/roblox/link` | SESSION | desvincula (audit) |
| GET | `/roblox/status` | SESSION | estado `HAS_ACCESS/NO_ACCESS/INCONCLUSIVE/UNKNOWN` + `nextAttemptAt` |
| POST | `/roblox/access-check` | SESSION | força nova checagem (rate 5/min; fila do worker) |

## 8.4 Torneio e configuração

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments` | PUBLIC | lista (respeita `visibility`) |
| GET | `/tournaments/:id` | PUBLIC | detalhes + config pública |
| POST | `/admin/tournaments` | ADMIN `tournaments.config` | cria |
| PATCH | `/admin/tournaments/:id/config` | ADMIN `tournaments.config` | sem `confirm`: 200 com `impacts` + `confirmToken`; com `confirm=true&token=…`: aplica (audita) |
| POST | `/admin/tournaments/:id/status` | ADMIN `tournaments.config` | transições de status com guardas |

**Impacts** (exemplos): `max_teams` abaixo das equipes confirmadas → `INCOMPATIBLE` (bloqueia); `players_per_team`/`substitutes_per_team` mudos → lista equipes fora do novo tamanho (exige confirm; equipes incompatíveis perdem READY); formato após chaveamento → bloqueado; datas → warnings.

## 8.5 Equipes e convites

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/teams` | PUBLIC* | equipes + composição + prontidão (*conforme `visibility`) |
| GET | `/admin/tournaments/:id/teams` | ADMIN `teams.manage` | todas as equipes (todos os estados) + filtros — alimenta T12 |
| POST | `/teams` | SESSION | cria equipe (janela de inscrição aberta) |
| GET | `/teams/:id` | PUBLIC/membro | detalhes |
| PATCH | `/teams/:id` | owner ou ADMIN `teams.manage` | renomear etc. |
| POST | `/teams/:id/leave` | membro | sair (owner transfere ou bloqueia) |
| POST | `/teams/:id/invites` | owner | cria convite (`dm_state` pode sair FAILED sem perder o convite) |
| POST | `/invites/:id/accept` | convidado | transação com advisory lock + partial unique index (07/11) |
| POST | `/invites/:id/decline` | convidado | transição com guarda |
| DELETE | `/invites/:id` | owner ou ADMIN `teams.manage` | cancela pendente |

## 8.6 Sorteio e chaveamento

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/tournaments/:id/bracket` | PUBLIC | chaveamento (matches, slots, estados, vencedores) |
| POST | `/admin/tournaments/:id/draw/preview` | ADMIN `draws.run` | prévia determinística (seed provisório) — **NÃO persiste** |
| POST | `/admin/tournaments/:id/draw/confirm` | ADMIN `draws.run` | seed novo no servidor → grava `draws`+`brackets`+`matches`+locks em transação; 409 `_BRACKET_EXISTS` se já há bracket com partida iniciada |
| POST | `/admin/tournaments/:id/draw/redraw` | ADMIN `bracket.manage` + dono | só com todas as partidas PENDING/CANCELLED; bracket antigo → SUPERSEDED (evento `draw.superseded`) |
| GET | `/admin/tournaments/:id/draws` | ADMIN `draws.run` | histórico de sorteios |
| POST | `/admin/brackets/:id/edits/preview` | ADMIN `bracket.manage` + dono | prévia de **edição manual** (mutações declarativas) + `confirmToken`; NÃO persiste (12, §12.6) |
| POST | `/admin/brackets/:id/edits/confirm` | ADMIN `bracket.manage` + dono | aplica; `version++`; audit; outbox `bracket.updated`; 409 `_BRACKET_EDIT` se há partida IN_PROGRESS/COMPLETED |

## 8.7 Partidas

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/admin/tournaments/:id/matches?status=` | ADMIN `matches.view` | lista p/ seleção (alimenta autocomplete/select do bot via §8.9) |
| POST | `/admin/matches/:id/start` | ADMIN `matches.manage` | PENDING→IN_PROGRESS; timestamp `now()` do Postgres; 409 `_MATCH_STATUS` |
| POST | `/admin/matches/:id/finish` | ADMIN `matches.manage` | `{winnerTeamId}`; valida ∈ {A,B}; duração calculada; avança bracket; **última rodada → torneio FINISHED** (12, §12.3) |
| POST | `/admin/matches/:id/correct` | ADMIN `matches.manage` + dono | dry-run com `impacts` + `confirmToken`; confirm aplica invalidação controlada (13, §13.5); 409 se dependentes IN_PROGRESS/COMPLETED |
| POST | `/admin/matches/:id/cancel` | ADMIN `matches.manage` | razão obrigatória |
| POST | `/admin/matches/:id/walkover` | ADMIN `matches.walkover` | W.O. com razão; `wo_grace_minutes` respeitado |
| GET | `/admin/matches/:id/history` | ADMIN `audit.view` | `match_events` (histórico da partida) |

## 8.8 Administração, logs e realtime

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/admin/audit?target=&actor=&action=&page=` | ADMIN `audit.view` | paginado, append-only |
| GET · POST · DELETE | `/admin/admins`, `/admin/admins/:discordId` | OWNER | concessão/revogação granular; admin não pode conceder (403) |
| GET | `/tournaments/:id/events?page=` | PUBLIC | feed público paginado (`public_events`) |
| GET | `/stream/tournaments/:id` | SESSION ou PUBLIC conforme visibility | **SSE** — caminho completo: `/api/v1/stream/tournaments/:id`; eventos do catálogo §8.10; heartbeat 25 s; reconexão com `Last-Event-ID` |
| POST | `/admin/tournaments/:id/announcements` | ADMIN `announcements.publish` | anúncio + outbox |

## 8.9 Internos (bot/worker)

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| GET | `/internal/health` | INTERNAL | liveness web/api |
| POST | `/internal/outbox/ack` | INTERNAL (worker) | marca DELIVERED/DEAD na reconciliação manual |
| GET | `/internal/bot/select/matches?status=&page=&q=` | INTERNAL + ator | páginas de ≤25 confrontos p/ autocomplete/string select do bot |
| GET | `/internal/bot/select/teams?status=&page=&q=` | INTERNAL + ator | páginas de ≤25 equipes |

## 8.10 Catálogo canônico de eventos

**Única fonte de nomes** para `public_events`, `outbox_events` e tópicos SSE. Nomes `dotted lowercase`; payloads `camelCase`. Payloads públicos não contêm IDs de admins nem dados privados. Eventos privados (convites, Roblox) vão para `audit_logs`/outbox dirigido — nunca ao feed público.

| Evento | Origem | Payload (essência) |
|---|---|---|
| `tournament.created` | criação | `{tournamentId, name}` |
| `tournament.config_changed` | config aplicada (pós-confirmação) | `{version, changedFields[]}` |
| `tournament.status_changed` | transição de status | `{from, to}` (inclui FINISHED com campeão) |
| `tournament.announcement_published` | anúncio | `{announcementId, title, excerpt}` |
| `team.created` | criação de equipe | `{teamId, name}` |
| `team.member_joined` | convite aceito | `{teamId, userId}` |
| `team.member_left` | saída/remoção | `{teamId, userId}` |
| `team.disqualified` | desqualificação | `{teamId, reason}` |
| `invite.accepted` / `invite.declined` / `invite.expired` / `invite.cancelled` | convites (privados) | `{inviteId, teamId, userId}` |
| `match.started` | admin | `{matchId, tournamentId, round, teams{a,b}}` |
| `match.finished` | admin | `{matchId, winnerTeamId, durationSeconds}` |
| `match.walkover` | admin | `{matchId, winnerTeamId, reason}` |
| `match.corrected` | admin (pós-confirmação) | `{matchId, oldWinnerTeamId, newWinnerTeamId, reason}` |
| `match.cancelled` | admin | `{matchId, reason}` |
| `draw.confirmed` | sorteio confirmado | `{drawId, size, teamsCount}` |
| `draw.superseded` | redraw | `{oldBracketId, newBracketId}` |
| `bracket.updated` | avanço automático ou edição manual | `{version, reason}` |
| `roblox.linked` / `roblox.unlinked` | vínculo (privado → audit) | `{userId}` |
| `roblox.access_state_changed` | máquina de acesso (privado → audit) | `{robloxId, state}` |

Dedup key do outbox: `outbox:<evento>:<id da entidade>` (ex.: `outbox:match.finished:<matchId>`). Heartbeat do SSE é comentário (`: ping`), não evento.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 10/22 | ARQUIVO ORIGEM: docs/09-matriz-permissoes.md -->
<!-- ================================================================ -->

# 09 — Matriz de permissões

## 9.1 Papéis

| Papel | Origem | Escopo |
|---|---|---|
| **PROPRIETÁRIO** | `OWNER_DISCORD_ID` (env), verificado no bootstrap (10, §10.2) | tudo, irrevogável pela UI; identidade não alterável por admins |
| **ADMINISTRADOR** | concessão do dono (`admin_grants`, granular) | apenas permissões explicitamente concedidas |
| **JOGADOR** | qualquer autenticado | funções de participante; resultados oficiais são só de admin |

Regra central: **a interface esconder botões é conveniência; a autorização é validada no backend em toda requisição** — inclusive nas chamadas do bot (ator resolvido no banco).

## 9.2 Permissões granulares

| Permissão | Descrição |
|---|---|
| `tournaments.config` | criar/configurar torneio, mudar status |
| `registrations.manage` | abrir/fechar inscrição, aprovar/afastar equipes |
| `teams.manage` | editar/desqualificar equipes alheias, forçar ajustes |
| `matches.manage` | iniciar/finalizar/cancelar/corrigir partidas |
| `matches.walkover` | aplicar W.O. |
| `bracket.manage` | editar chaveamento, reexecutar sorteio |
| `draws.run` | prévia + confirmar sorteio |
| `announcements.publish` | anúncios públicos |
| `audit.view` | consultar auditoria |
| `admins.manage` | **exclusivo do dono** (nunca concedido) |

## 9.3 Matriz (✅ = pode · ❌ = não pode)

| Ação | Proprietário | Admin | Jogador |
|---|---|---|---|
| Configurar torneio | ✅ | ✅ se `tournaments.config` | ❌ |
| Gerenciar inscrições/equipes alheias | ✅ | ✅ se `registrations.manage`/`teams.manage` | ❌ |
| Iniciar/finalizar partida | ✅ | ✅ se `matches.manage` | ❌ |
| Corrigir resultado com invalidação | ✅ | ✅ se `matches.manage` **e** dono aprova impacto | ❌ |
| Aplicar W.O./punições | ✅ | ✅ se `matches.walkover` | ❌ |
| Executar sorteio | ✅ | ✅ se `draws.run` | ❌ |
| Editar/reexecutar chaveamento | ✅ | ✅ se `bracket.manage` | ❌ |
| Publicar anúncios | ✅ | ✅ se `announcements.publish` | ❌ |
| Consultar auditoria | ✅ | ✅ se `audit.view` | ❌ |
| Conceder/revogar admins | ✅ | ❌ | ❌ |
| Alterar `OWNER_DISCORD_ID` | via env/ops apenas | ❌ | ❌ |
| Criar equipe, convidar, aceitar convites, vincular Roblox | ✅ | ✅ (como jogador) | ✅ |
| Ver resultado oficial | ✅ | ✅ | ✅ (somente leitura) |

## 9.4 Guardas de elevação

1. `admins.manage` nunca entra no array de permissões de um admin (unique no enum + validação no service).
2. Admin concede apenas subconjunto das próprias permissões — como o dono é o único com `admins.manage`, admin não concede nada.
3. Revogação é imediata: permissões sempre lidas do banco na hora da checagem (cache ≤ 5 s com invalidação na revogação).
4. Bootstrap do dono idempotente e auditado (`OWNER_BOOTSTRAP`), ver 10, §10.2.
5. Toda checagem emite `audit_logs` para ações de escrita com ator resolvido (incluindo via bot: `actor_type=BOT` + `actor_discord_id`).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 11/22 | ARQUIVO ORIGEM: docs/10-fluxos-auth.md -->
<!-- ================================================================ -->

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
1. Usuário autenticado clica "Vincular Roblox" → **`GET /api/v1/roblox/link/start`** (navegação sempre via GET; resposta **302**): server gera `state` assinado (HMAC com segredo do servidor, contém `userId`, expira em 10 min, uso único) e responde `302 Location: https://apis.roblox.com/oauth/v1/authorize?client_id=…&redirect_uri=…&scope=openid%20profile&response_type=code&state=…&nonce=…`.
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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 12/22 | ARQUIVO ORIGEM: docs/11-fluxos-equipes-convites.md -->
<!-- ================================================================ -->

# 11 — Fluxos: equipes, convites e elegibilidade

## 11.1 Criar equipe (jogador, site ou `/equipe criar`)

Pré-condições: janela de inscrição aberta (`REG_OPEN`), usuário sem equipe ativa no torneio, configuração permite auto-criação (ou admin cria).

1. `POST /teams {tournamentId, name}` — validação: nome único por torneio (case/acentos normalizados), tamanho, filtro de palavras.
2. Transação: `teams` (status `DRAFT`, owner = usuário) + `team_members` (OWNER) + audit.
3. Resposta: equipe + checklist de "prontidão" (jogadores faltantes, Roblox vinculado?, acesso ao jogo?).

## 11.2 Convite

**Por Discord** (preferido): `/equipe convidar` → user select nativo (sem digitar) → `POST /teams/:id/invites {inviteeDiscordId}`. O backend resolve discordId→user; se o convidado nunca logou, o convite fica pendente por identidade Discord e é ativado no primeiro login.

Validações no backend (site ou bot — mesmo código):
- convidado elegível: sem equipe ativa no torneio, não banido, Roblox vinculado (se `requireLinkedRoblox`), acesso HAS_ACCESS (se `requireGameAccess`);
- sem convite pendente duplicado (partial unique);
- convites respeitam vaga: `titulares = players_per_team`, `suplentes = substitutes_per_team` conforme política.

Envio da DM: o bot tenta DM **depois** do convite persistido. Falha (privacidade 50007 etc.) → `dm_state=FAILED`; convite continua `PENDING` e aceitável pelo site ou `/painel`. DM nunca é passo obrigatório.

Aceitação (`POST /invites/:id/accept`) — transação única, ordem fixa:
0. **lock de contenção**: `SELECT pg_advisory_xact_lock(hashtextextended('team:' || :tournamentId || ':' || :inviteeId, 0))` — serializa accepts paralelos do mesmo usuário (mesmo que em equipes diferentes);
1. `UPDATE invites SET status='ACCEPTED' WHERE id=? AND status='PENDING' AND expires_at>now()` → rowCount 1 obrigatório (senão 409 `CONFLICT_INVITE_STATE`);
2. revalidar elegibilidade + vagas **dentro da tx** (titulares × suplentes conforme política);
3. inserir `team_members` com `tournament_id` denormalizada — a **partial unique index** `(tournament_id, user_id) WHERE left_at IS NULL` (07, regra definitiva) bloqueia dupla participação ativa, **incluindo suplentes**;
4. audit + outbox `team.member_joined` (feed público apenas se config disser; convite em si é privado).

Recusa/cancelamento/expiração: transições de status com guarda `WHERE status='PENDING'`; expiração por job cron do worker (batch, idempotente).

## 11.3 Estados da equipe

| Estado | Significado | Transições |
|---|---|---|
| DRAFT | em composição | → READY (auto quando regras cumpridas), → CANCELLED (owner) |
| READY | pronta p/ sorteio | → DRAFT (mudança de roster), → LOCKED (sorteio) |
| LOCKED | congelada no chaveamento | edições bloqueadas; saída só via admin |
| DISQUALIFIED | punida por admin | irreversible + audit |

**Prontidão** = fórmula da config: `titulares ativos == players_per_team` E (se política exige) `substitutos == substitutes_per_team` E todos com Roblox vinculado (e HAS_ACCESS, se configurado). A UI mostra checklist item a item; o bot espelha em `/equipe`.

## 11.4 Elegibilidade e conflitos

- Um usuário não pode estar em duas equipes ativas do mesmo torneio (invariante do banco; tentativa → `CONFLICT_TEAM_MEMBER`).
- Troca de equipe = sair (deixa `left_at`) + ser aceito em outra; durante LOCKED, bloqueado para membros (admin pode liberar com audit).
- Suplentes (decisão D-11): default **2** por equipe (`substitutes_per_team`, configurável); **não contam** para a prontidão mínima; também estão sujeitos à regra "1 participação ativa por torneio" (07 — inclusive suplentes); promoção a titular pelo owner enquanto DRAFT/READY (nunca em LOCKED).
- Admin (`teams.manage`) pode remover membro / desqualificar equipe — sempre com razão + audit.

## 11.5 Fluxo no Discord (`/equipe`)

1. `/equipe ver` → painel ephemeral: nome, estado, checklist de prontidão, integrantes (com estado Roblox), botões: convidar (user select), sair.
2. `/equipe criar` → modal (nome) → mesma criação via API.
3. Convite recebido: o usuário pode checar `/painel` → aba convites (lista com Aceitar/Recusar) — mesmo endpoint do site.
4. Todos os comandos do jogador são **leitura/escrita de própria conta**; guard no router rejeita qualquer parâmetro de ID de terceiro (exceto user select do convite, que é a função de convidar).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 13/22 | ARQUIVO ORIGEM: docs/12-fluxos-sorteio-chaveamento.md -->
<!-- ================================================================ -->

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
- grava `public_events`/`outbox` `bracket.updated` (throttle: 1 por transação);
- **se `M` é a partida da última rodada**: torneio transita para `FINISHED` com campeão = vencedora de `M`, na mesma transação (`tournament.status_changed` + audit + outbox) — o sistema só conclui o torneio quando o admin encerra a partida final; nada é encerrado automaticamente antes disso.

## 12.4 Chaveamento visual (site)

- Render server-side do `bracket` + hidratação; colunas por rodada; cada card mostra equipes (avatar), estado (pendente/em andamento/concluída/BYE/W.O.), vencedora destacada, duração quando concluída.
- Responsivo: desktop colunas lado a lado; mobile em abas por rodada.
- Atualização: SSE (`bracket.updated` → refetch do snapshot) com fallback polling 10 s.
- Padrões visuais: fonte forte, cartões com borda, estados por cor (pendente cinza, andamento âmbar, concluída verde, W.O. roxo) — detalhes em 15.

## 12.5 Histórico de sorteios

`GET /admin/tournaments/:id/draws` — lista com autor, data, seed (visível só p/ `audit.view`), nº de equipes, config. Cada sorteio tem link para o bracket que gerou. Auditoria `draw.confirmed` contém antes/depois (bracket arquivado, se houve).

## 12.6 Edição manual do chaveamento (admin)

Corrige erro de seed/montagem **antes de qualquer partida começar** — não substitui a correção de resultado (13, §13.5). Mesmo padrão de prévia → confirmação → auditoria:

1. `POST /admin/brackets/:id/edits/preview` — mutações declarativas (ex.: `[{"op":"swap","matchA":id,"matchB":id}]`); backend valida contra o gabarito (`next_match_id`/`next_slot` consistentes) e retorna prévia + `confirmToken` (HMAC, 5 min, uso único); **não persiste**.
2. `POST /admin/brackets/:id/edits/confirm` — aplica em transação: `matches` ajustadas, `brackets.version++`, `match_events(TEAM_SWAPPED)` nas partidas afetadas, audit (before/after do `structure`), outbox `bracket.updated`.

Guards: qualquer partida `IN_PROGRESS`/`COMPLETED` → 409 `CONFLICT_BRACKET_EDIT` (use 13.5 nesses casos); mutação fora do gabarito → 400 `VALIDATION_ERROR`; token inválido/expirado → 409 `CONFLICT_CONFIRM_TOKEN`.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 14/22 | ARQUIVO ORIGEM: docs/13-fluxos-partidas.md -->
<!-- ================================================================ -->

# 13 — Fluxos: partidas

Domínio: **eliminação por sobrevivência** — sem placar. Campos: equipes, rodada/slot, estado, início, fim, duração (calculada), vencedora, responsável, histórico.

## 13.1 Iniciar

**Site**: painel admin → lista de confrontos pendentes → botão Iniciar. **Discord**: `/admin start` → autocomplete alimentado por `GET /internal/bot/select/matches` (busca por "R2 — EquipeA vs EquipeB", ≤25 resultados por tecla) ou string select paginado; confirmação por botão (2 cliques, zero digitação de nomes).

`POST /admin/matches/:id/start` — transação:
```sql
UPDATE matches
   SET status='IN_PROGRESS', started_at=now(), recorded_by_user_id=:actor
 WHERE id=:id AND status='PENDING'
   AND team_a_id IS NOT NULL AND team_b_id IS NOT NULL;
```
`rowCount == 1` → grava `match_events(STARTED)` + `audit_logs` + `outbox` (`match.started`); COMMIT; 200.
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
4. `match_events(FINISHED, before/after)`, `audit_logs`, `outbox` (`match.finished`); COMMIT.

Iniciar sem finalizar é permitido (estado persiste); finalizar sem iniciar → 409. Cancelamento (`/cancel`, razão obrigatória): partida volta ao fluxo admin — não avança chaveamento.

**Partida final**: quando a partida finalizada é a da última rodada do bracket, a mesma transação encerra o torneio (`FINISHED`, campeão = vencedora), com audit + outbox `tournament.status_changed` (12, §12.3). **Sem placar em nenhuma etapa** e **sem exigência de evidências** — o registro é ato administrativo auditado (requisito do dono).

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 15/22 | ARQUIVO ORIGEM: docs/14-sincronizacao-site-bot.md -->
<!-- ================================================================ -->

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

- Chaveamento/feed: SSE em `/api/v1/stream/tournaments/:id` (eventos do catálogo 08 §8.10: `match.started`, `match.finished`, `draw.confirmed`, `bracket.updated`, `tournament.announcement_published`), heartbeat 25 s, reconexão com `Last-Event-ID`.
- Fallback: se `EventSource` falhar 2× (proxy/CDN bloqueando), cliente alterna polling SWR 10 s automaticamente.
- O bot consulta a API por demanda (interação) — não mantém cache de resultados; painéis V2 publicados no Discord são atualizados pelo worker (edit da mensagem) quando o evento correspondente é entregue.

## 14.6 Semântica de falha externa (requisito explícito)

| Falha | Efeito no domínio | Recuperação |
|---|---|---|
| Discord REST fora / 429 | partida/resultados **já salvos**; mensagem fica PENDING | worker reenvia com backoff; reconciliação |
| Roblox inventário fora | estado `INCONCLUSIVE` | re-tentativa agendada + manual |
| API fora (bot sem resposta) | bot responde "serviço indisponível, tente em instantes" | nenhuma ação perdida: usuário re-tenta |
| Redis fora | API continua (leitura/escrita sem fila); jobs enfileirados falham e ficam `PENDING` no outbox | fila reprocessa ao voltar; alarme |

---

<!-- ================================================================ -->
<!-- DOCUMENTO 16/22 | ARQUIVO ORIGEM: docs/15-telas-ux.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 17/22 | ARQUIVO ORIGEM: docs/16-plano-testes.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 18/22 | ARQUIVO ORIGEM: docs/17-seguranca.md -->
<!-- ================================================================ -->

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

---

<!-- ================================================================ -->
<!-- DOCUMENTO 19/22 | ARQUIVO ORIGEM: docs/18-deploy-operacao.md -->
<!-- ================================================================ -->

# 18 — Deploy, variáveis de ambiente, backups e operação

## 18.1 Topologia alvo (v1)

Docker Compose em VPS (4 vCPU/8 GB mínimo): `web` (Next.js standalone), `bot`, `worker`, `postgres:16`, `redis:7` (AOF). Alternativas (Cloud Run p/ web + VPS p/ bot) em 19/D-01.

```
compose/
  docker-compose.yml      # prod
  docker-compose.dev.yml  # dev (só pg+redis)
services:
  web:    build apps/web,  healthcheck /internal/health
  bot:    build apps/bot,  restart unless-stopped
  worker: build apps/worker
  postgres: volume pgdata + backup sidecar (pg_dump cron)
  redis:    --appendonly yes
```

Deploy: CI (GitHub Actions) roda typecheck/lint/testes → build images → push → `docker compose pull && up -d` → `prisma migrate deploy` (passo explícito, nunca no boot) → smoke tests. Rollback: tag anterior das images + migrate não-destrutivo.

## 18.2 Variáveis de ambiente (`.env.example` completo)

```bash
# Ambiente
NODE_ENV=production
LOG_LEVEL=info
APP_PUBLIC_URL=https://seu-dominio

# Discord (bot + OAuth2)
DISCORD_TOKEN=              # bot token
DISCORD_CLIENT_ID=
DISCORD_CLIENT_SECRET=
DISCORD_GUILD_ID=           # guild única v1
DISCORD_INVITE_URL=         # link de convite p/ o site
OWNER_DISCORD_ID=1102745175098404935

# Roblox (OAuth2 Open Cloud + jogo)
ROBLOX_OAUTH_CLIENT_ID=
ROBLOX_OAUTH_CLIENT_SECRET=
ROBLOX_PLACE_ID=15327728308
ROBLOX_UNIVERSE_ID=5286749994
ROBLOX_CHECK_BUDGET_PER_MINUTE=30

# Banco / fila
DATABASE_URL=postgresql://user:pass@postgres:5432/aftermath?schema=public
REDIS_URL=redis://redis:6379

# Segredos internos
SERVICE_TOKEN=              # bot/worker → API
SESSION_SECRET=             # Auth.js
STATE_SIGNING_SECRET=       # HMAC dos states OAuth Roblox
```

Validação: env schema (Zod) no boot de cada app — faltou chave, não sobe. `OWNER_DISCORD_ID`, `ROBLOX_PLACE_ID`, `ROBLOX_UNIVERSE_ID` são **configuração inicial validada**, nunca substitutos de autenticação.

## 18.3 Backups e recuperação

- Postgres: `pg_dump` diário (retenção 30 dias) + WAL archiving se volume gerenciado; testes de restauração **mensais** (script `ops:restore-drill`).
- Redis: AOF + snapshot 6 h (fila é reprocessável; dados críticos estão no Postgres).
- Offsite: upload do dump para storage externo (S3/B2) com criptografia.
- RTO alvo: 1 h; RPO: 24 h (WAL reduz p/ minutos).

## 18.4 Monitoramento e operação

- Logs: pino JSON com correlationId; agregados via `docker logs` → Loki/Grafana (opcional) ou logfile rotativo.
- Health: `/internal/health` (web), worker heartbeat em Redis (TTL 60 s), bot: `ready` + presença.
- Alarmes mínimos (UptimeRobot/healthchecks.io + webhook no Discord de ops): web down, worker heartbeat expirado, bot desconectado >5 min, outbox DEAD >0, taxa 5xx >2%, circuit breaker Roblox aberto.
- Rotinas: reconciliação outbox (cron 5 min), expiração de convites (cron 15 min), watchdog partidas (cron 30 min), digest de auditoria (diário, opcional).
- Runbooks: token vazado (17.6), Roblox bloqueando IP (trocar egress/proxy de saída + abrir circuit), Discord rate 429 em massa (throttle do outbox).

## 18.5 Migrações e seeds

- `prisma migrate dev` local; `migrate deploy` em prod via CI; migrações destrutivas em duas etapas (expand/contract) com revisão.
- Seed de produção: **nenhum dado fake**; apenas criação do torneio raiz pelo painel. Seed de dev separado.

---

<!-- ================================================================ -->
<!-- DOCUMENTO 20/22 | ARQUIVO ORIGEM: docs/19-riscos-decisoes.md -->
<!-- ================================================================ -->

# 19 — Riscos, decisões pendentes e alternativas

## 19.1 Riscos

| # | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| R-1 | OAuth2 Roblox: app novo fica em **modo privado (máx. 10 usuários)** até aprovação de produção; publicar exige **Submit for Review** — vídeo de demonstração + justificativa, escopos mínimos, conta registradora com **ID verificado**, app não editável durante a revisão e irreversível para privado [CONFIRMADO 28/10/2026] | Média | Alto — trava vinculação em massa | Registrar app na P1 e submeter revisão imediatamente; preparar vídeo/justificativa com antecedência; contingência: torneio fechado ≤10 contas até aprovação; monitorar docs |
| R-2 | `is-owned` depende de privacidade do jogador; respostas 403 ambíguas [TESTAR] | Alta | Médio — falsos "sem acesso" | Máquina de estados com INCONCLUSIVE + `can-view-inventory` + mensagens orientando o jogador; probe na P2 |
| R-3 | Bloqueio anti-bot Roblox a IPs de datacenter [INCERTO — 403 observado nesta sessão] | Média | Alto se produção em datacenter | Probe residencial; se confirmado, saída por IP residencial/proxy confiável ou alternativa com API key do dono (cloud v2 inventory-items) |
| R-4 | Intents privilegiadas do Discord exigem revisão se app crescer (>10k usuários) [CONFIRMADO] | Baixa | Baixo/Médio | Solicitar só `GUILD_MEMBERS`; sem `MESSAGE_CONTENT` |
| R-5 | Rate limits Discord/Roblox em pico (anúncios + logs simultâneos) | Média | Baixo | Outbox com bucket + throttle; orçamento global Roblox |
| R-6 | SSE bloqueado por proxy/CDN do ambiente de deploy | Média | Baixo | Fallback polling automático (10 s) já previsto |
| R-7 | Relógios divergentes (duração de partida) | Baixa | Baixo | Timestamps via `now()` do Postgres em transação |
| R-8 | Abuso de re-tentativa de verificação | Média | Baixo | Rate limit 5/min + orçamento global + circuit breaker |
| R-9 | Correções de resultado em cadeia afetando partidas futuras | Média | Médio | Dry-run de impactos + bloqueio + invalidação explícita + histórico |
| R-10 | OAuth2 beta do Roblox mudar escopos/contratos | Média | Médio | Camada `RobloxIdentityProvider` isolada; fixtures + probe contratual |

## 19.2 Alternativas avaliadas

| Tema | Escolhida | Alternativa | Por que a escolhida |
|---|---|---|---|
| API | Route handlers Next (`/api/v1`) | API Fastify/Nest separada | Menos superfície p/ v1; extração futura barata (core isolado) |
| Fila | BullMQ + Redis 7 | pg-boss (Postgres) | Redis já necessário p/ lock/rate-limit (padrão do repo atual); pg-boss é plano B de 1 dependência a menos |
| Sessão | Auth.js v5 JWT | Database sessions (adapter) | JWT simplifica scale-out; permissões ficam no banco de qualquer forma |
| Realtime | SSE + polling fallback | WebSocket | Só precisa server→client; SSE é HTTP puro, simples atrás de proxies |
| Host | VPS + Compose | Cloud Run (web) + VPS (bot); Render/Railway all-in-one | Bot requer processo persistente; Compose dá controle e custo previsível |
| Verificação Roblox | `is-owned` público | cloud v2 inventory-items (API key do dono) | Endpoint indicado pelo dono; alternativa robusta pronta se R-2/R-3 confirmarem |
| Vínculo Roblox | OAuth2 Open Cloud | none (só username digitado) | Username digitado não é prova; OAuth dá ID estável + avatar oficial |

## 19.3 DECISÕES — status após a revisão técnica v2 (28/10/2026)

| # | Decisão | Escolha | Status |
|---|---|---|---|
| D-01 | Hosting da v1 | VPS + Docker Compose (um provedor, ex.: Hetzner/Contabo/Scaleway) | ⏳ **PENDENTE** — escolher provedor antes da P7 (não bloqueia P0–P6) |
| D-02 | API dentro do Next vs API separada | Dentro do Next (`/api/v1`) | ✅ **RESOLVIDA** |
| D-03 | Sessão JWT (Auth.js) vs sessão em banco | JWT + permissões sempre do banco | ✅ **RESOLVIDA** |
| D-04 | Fila BullMQ+Redis vs pg-boss | BullMQ + Redis 7 (AOF) | ✅ **RESOLVIDA** |
| D-05 | Armazenar tokens OAuth Roblox? | Não — só identidade; re-verificação por endpoints públicos | ✅ **RESOLVIDA** |
| D-06 | Ordem da verificação de acesso | `is-owned` → `can-view-inventory` → fallback cloud v2 (API key do dono) | ✅ **RESOLVIDA** |
| D-07 | Uso do Google AI Studio Build | Apenas protótipo de telas (UI track — doc 21); produção no repo | ✅ **RESOLVIDA** |
| D-08 | Formato v1 | Eliminação simples (modelo pronto p/ double) | ✅ **RESOLVIDA** |
| D-09 | Cadeia de hash no AuditLog | Adotar na P5 (tamper-evidence opcional) | ✅ **RESOLVIDA** |
| D-10 | Domínio/e-mail p/ apps OAuth e redirects | Definir antes da P1 | ⏳ **PENDENTE** — bloqueia redirects reais dos apps OAuth |
| D-11 | Política de suplentes | 2 suplentes por equipe (configurável); **não contam** p/ prontidão mínima; sujeitos à regra "1 participação ativa por torneio" | ✅ **RESOLVIDA** |
| D-12 | Aprovar roadmap (20) e iniciar P0 | Sim | ⏳ **PENDENTE** — aguarda aprovação do dono |

Decisões resolvidas não são reabertas sem necessidade; reversão exige justificativa e nova linha de auditoria. As decisões pendentes (D-01, D-10, D-12) **não bloqueiam o início da implementação** — detalhes em 20.

## 19.4 Incertezas que não bloqueiam o começo

- Contrato exato do `is-owned` (probe P2 — a máquina de estados tolera qualquer resposta).
- Prazo da aprovação do app OAuth Roblox (R-1) — o desenvolvimento continua com mock provider.
- Comportamento exato de DMs bloqueadas (teste de staging P5).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 21/22 | ARQUIVO ORIGEM: docs/20-roadmap.md -->
<!-- ================================================================ -->

# 20 — Roadmap (fases pequenas, ordenadas, verificáveis)

Regra: cada fase tem **critérios de aceite testáveis** e nada de fase posterior começa sem os anteriores verdes. Estimativas em dias de trabalho concentrado (não calendário).

## P0 — Bootstrap do repositório (~1–2 d)
- Monorepo pnpm (apps: web/bot/worker; packages: core/db/schemas/shared/ui/integrations) — copiando padrões do repo atual (04).
- TS strict, eslint, prettier, vitest, CI (typecheck+lint+test), docker-compose dev (pg+redis), `.env.example`, README.
- **Aceite**: `pnpm test` verde no CI; containers sobem.

## P1 — Identidade: Discord OAuth2 + dono + permissões (~3–4 d)
- Auth.js provider Discord; upsert users; sessão JWT; logout.
- Bootstrap do OWNER por env (10.2) + `admin_grants` + policy service (matriz 9.3) + audit append-only.
- `/me`, `/me/permissions`, `/admin/admins` (conceder/revogar, só dono).
- **Aceite**: login E2E; matriz de permissões coberta por testes; bootstrap idempotente auditado. *Paralelo: registrar app OAuth Roblox + pedir produção (R-1) e definir domínio (D-10).*

## P2 — Torneio + config + Roblox (~4–5 d)
- Modelo tournaments/config_versions + wizard T11 com análise de impacto e confirmação (8.4).
- `packages/integrations/roblox`: OAuth2 link (openid profile), userinfo, thumbnails; **probe do `is-owned`** (03 §3.4); máquina de estados de acesso com INCONCLUSIVE/backoff; fila de checagens com orçamento.
- **Aceite**: config salva com trilha; mudança incompatível bloqueada com impactos; vínculo Roblox funciona com mock + real (se app aprovado); estados de acesso conforme tabela 10.4.

## P3 — Equipes, convites + sorteio/chaveamento (~5–6 d)
- Teams/members/invites com invariantes (07); fluxos site (T7/T8) e `/equipe`.
- Motor de chaveamento puro + testes de propriedade; sorteio com prévia/confirmar/redraw guards (12); render do bracket (T3) com SSE + fallback polling.
- **Aceite**: testes de concorrência 16.4 (convites/vaga/sorteio) verdes; bracket renderiza 12 e 128 equipes; reload não altera resultado.

## P4 — Partidas (~3–4 d)
- start/finish/cancel/walkover/correct com guards de status + avanço automático + match_events; modais T13; outbox `match.*` (worker entrega no canal configurado).
- **Aceite**: 16.4 (partida) verde; correção com impacto; falha do Discord simulada não perde o resultado e reconcilia.

## P5 — Bot completo + logs públicos + anúncios (~4–5 d)
- Comandos RF-26 com autocomplete/selects/paginação; painéis V2; defer+edit (3 s); idempotency keys; mapeamento de erros.
- Feed público no site (T5) lendo `public_events`; anúncios (T17); digest opcional.
- **Aceite**: todos os comandos do checklist em guild de staging; outbox DEAD alarmado; nenhuma ação de resultado disponível a jogadores.

## P6 — Endurecimento + E2E completos (~3–4 d)
- Rate limits finais, CSRF/Origin checks, redação de logs, auditoria com filtros (T16), backups automáticos + drill de restauração, alarmes, runbooks.
- **Aceite**: suíte completa verde; restauração de backup testada; alarmes disparando em staging.

## P7 — Deploy de produção + go-live (~2–3 d)
- VPS + compose prod; DNS/TLS; migrate deploy via CI; seeds zerados; monitoramento externo; plano de rollback.
- **Aceite**: smoke tests em produção; dono ativa OWNER com login; primeiro torneio configurado pela UI.

## Pós-v1 (backlog ordenado)
Double elimination · round-robin de grupos · estatísticas de equipes · página de histórico de torneios · webhook nativo do jogo p/ verificação (proposta ao dev do Aftermath) · melhorias visuais com assets do jogo · AI Studio Build para experimentos de UI (D-07).

## Dependências externas (paralelas ao código)
1. Registro + aprovação do app OAuth2 Roblox (P1) — caminho crítico do R-1.
2. Criação do app/bot Discord + intents (`GUILD_MEMBERS`).
3. Definição de domínio + canal de logs no Discord (D-10).

## UI track paralela (Google AI Studio)

As telas (15) podem ser prototipadas **em paralelo** com P3–P5 no Google AI Studio Build, seguindo `docs/21-ai-studio-integration.md` (pacote de contexto, regras de port, fluxo de PR). A UI track não altera as fases de backend: cada tela integrada entra no PR da sua fase, e a troca de mock pela API real acontece apenas na camada `api-client.ts`, já tipada pelos schemas Zod (08, §8.0). Nada do backend (API/auth/DB/worker/bot) é gerado pelo AI Studio (D-07).

---

<!-- ================================================================ -->
<!-- DOCUMENTO 22/22 | ARQUIVO ORIGEM: docs/21-ai-studio-integration.md -->
<!-- ================================================================ -->

# 21 — Integração com Google AI Studio (UI track)

Objetivo: usar o **Google AI Studio Build** (Gemini) para prototipar e refinar a **interface** (site + painel admin) **sem criar uma arquitetura paralela**. O backend (API, auth, banco, worker, bot) **nunca** é gerado pelo AI Studio — decisão **D-07** (RESOLVIDA em 19).

## 21.1 O que o AI Studio pode e não pode

| Pode | Não pode |
|---|---|
| Prototipar telas consumindo os contratos reais (08) | Gerar endpoints/auth/banco ("arquitetura paralela") |
| Gerar componentes React/Next em `apps/web/src/**` | Alterar `packages/core`, `packages/db`, `apps/bot`, `apps/worker` |
| Propor design system (cores/tipografia/motion) | Introduzir biblioteca de estado/fetch fora do padrão |
| Consumir mock data layer com as mesmas formas dos contratos | Fazer deploy para produção (deploy é CI — doc 18) |
| Sincronizar via GitHub (sync nativo do Build mode) | Push direto em `master` (sempre branch + PR) |

## 21.2 Pacote de contexto fornecido ao AI Studio

Anexar ao projeto do AI Studio, nesta ordem:

1. `docs/08-api-contratos.md` — contratos + convenções (§8.0) + catálogo de eventos (§8.10);
2. `docs/15-telas-ux.md` — wireframes + mapeamento tela→endpoint (§15.5);
3. `docs/21-ai-studio-integration.md` — este documento (regras de port);
4. `docs/02-requisitos.md` — RF/RNF que legitimam comportamentos;
5. `packages/schemas/*` — schemas Zod compartilhados (fonte única de tipos);
6. Design tokens (quando existirem): `apps/web/src/styles/tokens.css` + paleta do doc 15 (`#0B0F14`, `#131A22`, âmbar `#F59E0B`, verde `#34D399`);
7. Mock data layer: `apps/web/src/mocks/*` — fixtures com as **mesmas formas** dos contratos (camelCase, IDs string, ISO-8601, eventos do catálogo).

## 21.3 Regras de port (o que o AI Studio gera entra como)

- Branch `design/ai-studio/<tela>`; integração **sempre via PR** revisado por humano — nunca push em `master`.
- Só toca em: `apps/web/src/app/**` (páginas), `apps/web/src/components/**`, `apps/web/src/styles/**`, `apps/web/src/mocks/**`.
- Fetch: camada única `apps/web/src/lib/api-client.ts`, tipada pelos schemas Zod. O AI Studio escreve chamadas contra esta camada; **não cria clients próprios**.
- Auth no protótipo: mock data layer com perfis fake (jogador, admin, dono) — **nunca tocar no Auth.js** nem em cookies de sessão reais.
- SSE no protótipo: simulado com timers emitindo os eventos do catálogo §8.10; o contrato do cliente é o mesmo (`onEvent(type, payload)`).
- Proibido instalar dependências novas sem aprovação (base aprovada: React/Next/Tailwind/zod; motion via CSS).
- Toda página nova nasce com estados **loading / empty / error** + skeleton; acessibilidade mínima (labels, foco visível, contraste — RNF-06).
- **Nenhum endpoint fora do mapeamento §15.5 / contratos §8.x**; se uma tela precisa de dado inexistente, abre issue para estender o contrato primeiro.

## 21.4 Fluxo de integração

1. Gerar tela no AI Studio com o pacote de contexto (21.2) anexado.
2. Exportar via sync nativo do Build mode para o GitHub → branch `design/ai-studio/tXX-<nome>`.
3. PR no repositório; revisão humana confere: conformidade com contratos (08), regras de port (21.3), acessibilidade (15/RNF-06).
4. Substituir mock pela API real — só a implementação da `api-client.ts` muda; as chamadas/tipos permanecem.
5. Testes E2E (16, §16.3) cobrem a tela integrada antes de marcar a fase como concluída.

## 21.5 Prioridades visuais do painel (eco de 15.5)

1. **Chaveamento visual (T3)** — cartões por rodada, estados coloridos, SSE ao vivo.
2. **Central de partidas (T13)** — fila com cronômetro, ações com modais de confirmação.
3. **Sorteio guiado (T14)** — prévia → confirmação → resultado.
4. **Configuração com impactos (T11)** — diff antes de aplicar (`confirmToken`).
5. **Equipes com pesquisa instantânea e filtros (T4/T12)**.
6. **Permissões (T15) e auditoria (T16)** — tabelas densas com filtros e before/after expansível.

## 21.6 Antipadrões a rejeitar no review de PR

- Chamadas `fetch` espalhadas nos componentes (fora da `api-client.ts`);
- Tipos locais duplicando schemas Zod;
- Estados de autenticação inventados no cliente (permissões vêm sempre da API);
- Eventos/nomes fora do catálogo §8.10;
- Estilos inline mágicos fora dos tokens do design system;
- Lógica de domínio (chaveamento, elegibilidade, permissões) reimplementada na UI — a UI só exibe o que a API responde.

---

