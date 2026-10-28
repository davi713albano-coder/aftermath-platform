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
