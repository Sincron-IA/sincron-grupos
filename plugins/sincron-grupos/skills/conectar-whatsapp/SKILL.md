---
name: conectar-whatsapp
description: Conecta um número de WhatsApp ao Sincron Grupos via QR code, pela IA. Use quando o usuário pedir para conectar o celular/WhatsApp, parear o número, ler o QR code, ou quando uma ação falhar por não haver instância conectada.
---

# Conectar WhatsApp (QR code)

Conecta o celular do usuário ao Sincron Grupos usando o servidor MCP `sincron-grupos`.

## Passos

1. **Verifique instâncias existentes**: `list_instances`.
   - Se já houver uma com status `conectado`, informe e pergunte se quer conectar outra.

2. **Crie a conexão**: `connect_phone` com `name` (ex.: "WhatsApp Principal").
   - A resposta traz `qr_code` (imagem base64), `instance_id` e `instrucoes`.
   - Apresente o QR ao usuário. Se o ambiente não renderizar a imagem base64, oriente:
     **WhatsApp → Aparelhos conectados → Conectar um aparelho → escanear o QR**.

3. **Aguarde a conexão**: faça polling com `get_instance_status({ instance_id })` a cada ~5s.
   - Pare quando `status` for `conectado` (mostre o `numero_telefone`/`profile_name`).
   - Se após ~2 min continuar `desconectado`, gere um novo QR com `connect_phone` (o QR expira).

## Observações
- O ciclo de conexão (QR, status) é feito pelo Sincron-Conexao por trás do MCP — o usuário não
  configura nada além de escanear.
- Depois de conectado, o número passa a **receber notificações do sistema como mensagens de WhatsApp**.
- Em caso de erro "Limite de instâncias do plano atingido", ofereça upgrade via `create_checkout_link`.
