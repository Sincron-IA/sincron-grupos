---
name: sincron-grupos
description: Configure e opere o Sincron Grupos (gestão de grupos WhatsApp) inteiramente pela IA. Use quando o usuário quiser assinar o serviço, conectar o WhatsApp/celular, importar ou listar grupos, enviar ou agendar mensagens, ou checar o status da conta. Aciona em pedidos como "configurar meu Sincron Grupos", "conectar meu WhatsApp", "assinar o Grupos", "mandar mensagem pros grupos", "agendar mensagem".
---

# Sincron Grupos — Onboarding e operação pela IA

Você opera o Sincron Grupos via servidor MCP `sincron-grupos` (já conectado por este plugin).
A autenticação é automática por OAuth — no primeiro uso o Claude Code abre o navegador para o
usuário fazer login. **Não peça API key manualmente.**

## Ferramentas MCP disponíveis
- `get_account` — plano, status da assinatura, uso e limites.
- `create_checkout_link` — gera link de checkout Stripe (trial 7 dias). args: `plan` (starter|pro), `period` (monthly|annual).
- `list_instances` / `connect_phone` / `get_instance_status` — conexão do WhatsApp.
- `list_groups` / `list_categories` — grupos e categorias.
- `send_message` — envio imediato. args: `group_ids[]`, `message`.
- `schedule_message` — agendamento. args: `group_ids[]`, `message`, `scheduled_at` (ISO 8601 futuro).

## Fluxo recomendado

1. **Diagnóstico**: chame `get_account`.
   - Se a chamada retornar 401/sem autorização, o login OAuth ainda não foi feito — o Claude Code
     abrirá o navegador automaticamente; oriente o usuário a concluir o login e tente de novo.
   - Se `subscription_status` for `none`/`blocked` → vá para **Assinar**.
   - Se ativo/trial → siga para **Conectar WhatsApp** (se não houver instância conectada).

2. **Assinar** (ver skill `assinar`): chame `create_checkout_link` e **apresente o `checkout_url`**
   para o usuário abrir no navegador. Avise que há 7 dias de trial. Depois de assinar, confirme com
   `get_account`.

3. **Conectar WhatsApp** (ver skill `conectar-whatsapp`): use `connect_phone`, mostre o QR e faça
   polling com `get_instance_status` até `conectado`.

4. **Operar**: use `list_groups`/`list_categories` para descobrir os grupos, e
   `send_message`/`schedule_message` para enviar. Sempre confirme o conteúdo e os grupos-alvo com o
   usuário antes de enviar.

## Notificações no celular
O Sincron Grupos envia avisos (ex.: confirmação, código de verificação) como **mensagem de WhatsApp
para o número conectado**, usando a própria instância. Ou seja: conectar o celular já habilita receber
notificações nele. Explique isso ao usuário quando perguntarem sobre notificações.

## Regras
- Nunca envie mensagens sem confirmação explícita do usuário (conteúdo + grupos).
- Datas de agendamento sempre no futuro, em ISO 8601 (ex.: `2026-06-10T14:00:00-03:00`).
- Se um limite de plano bloquear a ação, ofereça `create_checkout_link` para upgrade.
