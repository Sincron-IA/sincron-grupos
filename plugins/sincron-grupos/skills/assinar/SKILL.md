---
name: assinar
description: Assina o Sincron Grupos (checkout Stripe self-serve) pela IA. Use quando o usuário quiser assinar, contratar, fazer upgrade de plano, iniciar o trial, ou quando uma ação for bloqueada por assinatura inativa/limite de plano.
---

# Assinar o Sincron Grupos

Gera um link de checkout Stripe (com 7 dias de trial) usando o servidor MCP `sincron-grupos`.

## Passos

1. **Cheque o estado atual**: `get_account` (veja `plano` e `subscription_status`).

2. **Escolha o plano** com o usuário:
   - `starter` — 1 instância, 50 grupos, 3 gatilhos. (R$147/mês)
   - `pro` — 3 instâncias, grupos ilimitados, recursos de IA. (R$347/mês)
   - Período: `monthly` (padrão) ou `annual`.

3. **Gere o link**: `create_checkout_link({ plan, period })`.
   - Apresente o `checkout_url` para o usuário **abrir no navegador** e concluir o pagamento.
   - Informe que o trial é de 7 dias e que pode cancelar pelo portal do cliente.

4. **Confirme**: após o usuário avisar que assinou, chame `get_account` novamente e confirme
   `subscription_status` ativo/trial. Em seguida, ofereça **conectar o WhatsApp** (skill `conectar-whatsapp`).

## Observações
- `create_checkout_link` exige login OAuth (não funciona com API key avulsa). Como o plugin usa OAuth,
  isso é transparente.
- Enterprise é negociado manualmente — não há checkout self-serve; oriente o usuário a falar com a Sincron.
