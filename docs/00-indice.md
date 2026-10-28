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
