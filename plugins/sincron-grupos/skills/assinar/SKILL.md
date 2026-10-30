---
name: assinar
description: Assina o Sincron Grupos (checkout Cakto self-serve, Pix ou cartão) pela IA. Use quando o usuário quiser assinar, contratar, fazer upgrade de plano, ou quando uma ação for bloqueada por assinatura inativa/limite de plano.
---

# Assinar o Sincron Grupos

Gera um link de checkout da **Cakto** (Pix, cartão e carteiras; sem fidelidade) usando a tool MCP `account` do servidor `sincron-grupos`.

## Passos

1. **Cheque o estado atual**: `account({ action: "get" })` (veja `plano` e `subscription_status`).

2. **Escolha o plano** com o usuário:
   - `starter` — 1 número de WhatsApp, 50 grupos, 3 gatilhos. (R$ 147/mês ou R$ 1.470/ano)
   - `pro` — 3 números, grupos e gatilhos ilimitados, 5 pessoas da equipe, agentes de IA, transcrição de áudio. (R$ 347/mês ou R$ 3.470/ano)
   - Período: `monthly` (padrão) ou `annual` (equivale a 2 meses grátis).

3. **Gere o link**: `account({ action: "checkout", plan, period })`.
   - Apresente o `checkout_url` para o usuário **abrir no navegador** e pagar via Pix, cartão ou carteira.
   - Informe que a cobrança começa no pagamento (não há período de teste) e que a assinatura é gerenciada na área do cliente da Cakto.

4. **Confirme**: depois que o usuário avisar que assinou, chame `account({ action: "get" })` de novo e confirme
   `subscription_status` ativo. Em seguida, ofereça **conectar o WhatsApp** (skill `conectar-whatsapp`).

## Observações
- `account checkout` exige login OAuth (não funciona com API key avulsa). Como o plugin usa OAuth, isso é transparente.
- Precisa de mais números ou de uma operação sob medida? Não há checkout self-serve — oriente o usuário a falar com a Sincron em https://grupos.sincronia.digital.
