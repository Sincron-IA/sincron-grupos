---
name: sincron-grupos
description: Configure e opere o Sincron Grupos (gestão de grupos WhatsApp) inteiramente pela IA. Use quando o usuário quiser assinar o serviço, conectar o WhatsApp/celular, importar ou listar grupos, enviar ou agendar mensagens, configurar transcrição/resumo de áudio, criar gatilhos (automações), criar agentes de IA e ativá-los nos grupos, ou checar o status da conta. Aciona em pedidos como "configurar meu Sincron Grupos", "conectar meu WhatsApp", "assinar o Grupos", "mandar mensagem pros grupos", "agendar mensagem", "ativar resumo de áudio", "criar um gatilho", "criar uma IA pro grupo".
---

# Sincron Grupos — Onboarding e operação pela IA

Você opera o Sincron Grupos via servidor MCP `sincron-grupos` (já conectado por este plugin).
A autenticação é automática por OAuth — no primeiro uso o Claude Code abre o navegador para o
usuário fazer login. **Não peça API key manualmente.**

## Ferramentas MCP disponíveis

### Conta e conexão
- `get_account` — plano, status da assinatura, uso e limites (`recursos_ia` indica se IA está liberada).
- `create_checkout_link` — gera link de checkout Stripe (trial 7 dias). args: `plan` (starter|pro), `period` (monthly|annual).
- `list_instances` / `connect_phone` / `get_instance_status` — conexão do WhatsApp. `list_instances`/`get_instance_status` retornam status **ao vivo** + campo `conectado` (boolean).

### Grupos e mensagens
- `list_groups` / `list_categories` — grupos e categorias. Grupos têm `id` (UUID) e `jid`.
- `send_message` — envio imediato. args: `group_ids[]` (UUIDs), `message`.
- `schedule_message` — agendamento. args: `group_ids[]`, `message`, `scheduled_at` (ISO 8601 futuro).

### Transcrição / resumo de áudio
- `set_transcription_mode` — define o modo por **grupo** OU **categoria**.
  args: `group_id` (UUID) **ou** `category_id` (UUID); `mode` (`disabled`|`auto`|`manual`|`auto_summary`); `tipo_transcricao?` (`simples`|`com_resumo`).
  `auto`/`auto_summary` exigem `recursos_ia: true`. `auto_summary` = "Auto + Resumo IA".

### Gatilhos (automações por evento)
- `list_triggers` — lista gatilhos e status.
- `create_trigger` — cria automação.
  args: `name`, `event_type` (`keyword`|`new_member`|`member_leave`|`scheduled`|`reaction`),
  `action_type` (`send_message`|`auto_reply`|`notify_admin`|`delete_member`|`webhook`|`ai_agent`),
  `event_config` (ex.: keyword → `{ "keyword_text": "oi,ola" }`, vírgula = OR),
  `action_config` (send_message → `{ "message": "...", "destination_type": "own_group" }`; auto_reply → `{ "reply_message": "..." }`),
  `target_group_ids[]` (UUIDs), `target_category_ids[]?`, `is_active?` (default true).

### Agentes de IA (respondem dentro do grupo)
- `list_agents` — lista agentes + escopo de grupos vinculados.
- `create_ai_agent` — cria o agente. args: `nome`, `instrucao` (system prompt, mín. 20 chars), `modelo?` (default `gpt-4o-mini`), `temperatura?` (0..1), `resposta_privada?`. Exige `recursos_ia`.
- `set_agent_scope` — **ativa** o agente em grupos/categorias (o "colocar pra funcionar no grupo").
  args: `agent_id`, `scope_type` (`all`|`groups`|`categories`), `group_ids[]` (UUIDs, quando `groups`) ou `category_ids[]`.
  Fluxo: `create_ai_agent` → `set_agent_scope`. O agente nasce `ativo: true`.

## Fluxo recomendado

1. **Diagnóstico**: chame `get_account`.
   - 401/sem autorização → login OAuth não feito; o Claude Code abre o navegador. Oriente concluir o login e tente de novo.
   - `subscription_status` `none`/`blocked` → **Assinar**.
   - Ativo/trial → **Conectar WhatsApp** (se não houver instância conectada).

2. **Assinar** (skill `assinar`): `create_checkout_link` e **apresente o `checkout_url`**. Trial 7 dias. Confirme com `get_account`.

3. **Conectar WhatsApp** (skill `conectar-whatsapp`): `connect_phone`, mostre o QR, faça polling com `get_instance_status` até `conectado`.

4. **Operar grupos**: `list_groups`/`list_categories` para descobrir; `send_message`/`schedule_message` para enviar.

5. **Automação/IA**: `set_transcription_mode` para resumo de áudio; `create_trigger` para automações; `create_ai_agent` + `set_agent_scope` para um agente responder no grupo. Use `list_triggers`/`list_agents` para conferir.

## Notificações no celular
O Sincron Grupos envia avisos (ex.: confirmação, código) como **mensagem de WhatsApp para o número
conectado**, usando a própria instância. Conectar o celular já habilita receber notificações nele.

## Regras
- **Confirme antes de agir** em qualquer ação que escreve: enviar mensagem, criar gatilho, criar/ativar agente, mudar transcrição. Mostre conteúdo + grupos-alvo e peça OK.
- **Escopo de categoria afeta TODOS os grupos da categoria** — confirme explicitamente antes de usar `category_id` em `set_transcription_mode` ou `scope_type=categories`. Na dúvida, prefira por grupo (`group_id`/`group_ids`).
- Gatilhos `auto_reply`/`send_message` e agentes ativos podem **disparar mensagens reais** nos grupos. Avise o usuário ao ativar.
- Datas de agendamento sempre no futuro, ISO 8601 (ex.: `2026-06-10T14:00:00-03:00`).
- Se um limite de plano ou `recursos_ia: false` bloquear, ofereça `create_checkout_link` para upgrade.
- Não há tool de exclusão de grupo por design — operações são aditivas/seguras.
