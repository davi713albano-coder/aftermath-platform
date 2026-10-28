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
