# Aftermath Tournament Platform — Planejamento

> Status: **FASE DE PLANEJAMENTO** — nenhum código de produto foi escrito ainda.
> Pesquisa web realizada em **28/10/2026**. Fontes oficiais citadas em `docs/03-pesquisa-web.md`.

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

## Identificadores do projeto (configuráveis, nunca código fixo)

| Item | Valor inicial | Onde configurado |
|---|---|---|
| Place ID | `15327728308` | `ROBLOX_PLACE_ID` (env) |
| Universe ID | `5286749994` | `ROBLOX_UNIVERSE_ID` (env) |
| Proprietário inicial | `1102745175098404935` | `OWNER_DISCORD_ID` (env) |

Esses identificadores **não substituem autenticação**; são apenas sementes de configuração, com procedimento seguro de inicialização descrito em `docs/10-fluxos-auth.md`.

## Decisões que exigem aprovação

A lista consolidada está em [`docs/19-riscos-decisoes.md`](docs/19-riscos-decisoes.md) — inclui escolha de hosting, estratégia de sessão, fila, e o risco crítico do OAuth2 do Roblox estar em **beta com limite de 10 usuários** até aprovação em produção.
