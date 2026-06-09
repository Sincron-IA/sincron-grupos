---
name: sincron-grupos
description: Configure e opere o Sincron Grupos (gestão de grupos WhatsApp) inteiramente pela IA via MCP. Use quando o usuário quiser assinar, conectar o WhatsApp/celular, importar/listar/organizar grupos, criar categorias, enviar ou agendar mensagens, configurar transcrição/resumo de áudio, criar/gerir gatilhos (automações), criar/ativar agentes de IA nos grupos, ver estatísticas, ou checar a conta. Aciona em pedidos como "configurar meu Sincron Grupos", "conectar meu WhatsApp", "assinar o Grupos", "importar meus grupos", "mandar mensagem pros grupos", "agendar mensagem", "criar categoria", "ativar resumo de áudio", "criar um gatilho", "ver estatísticas".
---

# Sincron Grupos — Operação pela IA (MCP)

Você opera o Sincron Grupos via servidor MCP `sincron-grupos` (já conectado por este plugin).
Autenticação automática por OAuth — no primeiro uso o Claude Code abre o navegador para login.
**Não peça API key manualmente.**

## Modelo das ferramentas: RECURSO + ACTION

São **9 tools**, uma por recurso. Cada uma recebe um campo **`action`** que escolhe a operação,
mais os parâmetros daquela action (estilo REST). Ex.: `categories({ action: "create", name, color })`.

| Tool | Actions |
|---|---|
| **`account`** | `get` (plano/status/uso/limites) · `checkout` (plan=starter\|pro, period=monthly\|annual) |
| **`instances`** | `list` · `status`(instance_id) · `connect`(name → QR) · `reconnect`(instance_id → QR) · `disconnect`(instance_id) |
| **`groups`** | `list`(category_id?/search?) · `available`(search? — do WhatsApp) · `import`(group_jids[] OU name_filter) · `move_category`(group_ids[]+category_id\|null) · `settings`(group_id+guardar_todas_mensagens/processar_simultaneo) · `archive`(group_ids[]) |
| **`categories`** | `list` · `create`(name,color #RRGGBB) · `update`(category_id+name?/color?) · `delete`(category_id) |
| **`messages`** | `list`(page?/status?) · `send`(group_ids[]+message) · `schedule`(+scheduled_at ISO futuro) · `cancel`(message_id) · `retry`(message_id) |
| **`triggers`** | `list` · `create` · `update`(is_active pausa/ativa) · `delete` · `duplicate` · `run`(executa real) · `executions` · `reorder`(ordered_ids[]) |
| **`agents`** | `list` · `create` · `update` · `delete` · `scope`(ativa no grupo) · `test` · `bulk_toggle` · `logs` |
| **`transcription`** | `get`(group_id) · `set`(group_id OU category_id, mode=disabled\|auto\|manual\|auto_summary) |
| **`stats`** | `get` |

Detalhes de cada action e parâmetros estão na `description` de cada tool. As funções validam os
campos obrigatórios e retornam erro claro se faltar algo — leia o erro e corrija.

### Referência rápida de enums
- **trigger event_type**: `keyword`, `new_member`, `member_leave`, `scheduled`, `reaction`.
- **trigger action_type**: `send_message`, `auto_reply`, `notify_admin`, `delete_member`, `webhook`, `ai_agent`.
- **trigger event_config** (keyword): `{ keyword_text: "oi,ola" }` (vírgula = OU). **action_config**: send_message `{ message, destination_type:"own_group" }`; auto_reply `{ reply_message }`.
- **transcrição mode**: `disabled`, `auto`, `manual`, `auto_summary` (= "Auto + Resumo IA"). Grupo usa o UUID do grupo; categoria usa o UUID da categoria.
- **agente scope**: `scope_type` = `all` | `groups` (group_ids[] UUIDs) | `categories`.

## Fluxo recomendado

1. **Diagnóstico**: `account({action:"get"})`. 401 → conclua o login OAuth no navegador.
   `subscription_status` none/blocked → **Assinar**. Ativo/trial → seguir.
2. **Assinar**: `account({action:"checkout", plan, period})` → apresente `checkout_url`. Trial 7 dias.
3. **Conectar WhatsApp**: `instances({action:"connect", name})` → mostre o QR → `instances({action:"status", instance_id})` até `conectado`.
4. **Grupos**: sem grupos? `groups({action:"available"})` → `groups({action:"import", name_filter})`. Depois `groups({action:"list"})`, organize com `categories` + `groups({action:"move_category"})`.
5. **Operar**: `messages({action:"send"|"schedule"})`; automações com `triggers`; resumo de áudio com `transcription({action:"set"})`; números com `stats({action:"get"})`.

## Notificações no celular
Avisos (confirmação, código) chegam como **mensagem de WhatsApp para o número conectado**, via a
própria instância. Conectar o celular já habilita receber notificações nele.

## Regras
- **Confirme antes de qualquer escrita**: enviar/agendar mensagem, criar/editar/excluir gatilho ou
  agente, mudar transcrição, importar/arquivar grupos. Mostre o conteúdo + alvos e peça OK.
- **Escopo de categoria afeta TODOS os grupos da categoria** — confirme antes de usar `category_id`
  em `transcription set` ou `agents scope (categories)`. Na dúvida, use por grupo.
- **Ações que disparam no WhatsApp de verdade**: `messages send/retry`, `triggers run`, e gatilhos/
  agentes ativos. Avise o usuário.
- Datas de agendamento sempre no futuro, ISO 8601 (ex.: `2026-06-10T14:00:00-03:00`).
- Limite de plano ou `recursos_ia:false` bloqueia? Ofereça `account({action:"checkout"})`.
- **Não há ferramenta de billing avançado** (trocar plano/portal) nem para ações **sensíveis**
  (senha, API keys, equipe, excluir grupo) — por design são humano-only.
- O **agente in-group** (`agents`) é uma feature **em desenvolvimento**; pode não estar 100%.
