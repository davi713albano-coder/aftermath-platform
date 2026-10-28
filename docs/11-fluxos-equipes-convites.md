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

Aceitação (`POST /invites/:id/accept`) — transação única:
1. `UPDATE invites SET status='ACCEPTED' WHERE id=? AND status='PENDING' AND expires_at>now()` → rowCount 1 obrigatório;
2. inserir `team_members` (checa vagas + elegibilidade de novo, dentro da tx);
3. respeitar invariante "1 equipe ativa por usuário por torneio" (partial unique + re-checagem na tx);
4. audit + outbox (público apenas se config disser; convite em si é privado).

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
- Suplentes: participam da contagem de prontidão conforme política; podem ser promovidos a titular pelo owner enquanto DRAFT/READY (não em LOCKED).
- Admin (`teams.manage`) pode remover membro / desqualificar equipe — sempre com razão + audit.

## 11.5 Fluxo no Discord (`/equipe`)

1. `/equipe ver` → painel ephemeral: nome, estado, checklist de prontidão, integrantes (com estado Roblox), botões: convidar (user select), sair.
2. `/equipe criar` → modal (nome) → mesma criação via API.
3. Convite recebido: o usuário pode checar `/painel` → aba convites (lista com Aceitar/Recusar) — mesmo endpoint do site.
4. Todos os comandos do jogador são **leitura/escrita de própria conta**; guard no router rejeita qualquer parâmetro de ID de terceiro (exceto user select do convite, que é a função de convidar).
