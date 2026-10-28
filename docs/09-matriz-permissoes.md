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
