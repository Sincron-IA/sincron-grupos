---
name: conectar-whatsapp
description: Conecta um número de WhatsApp ao Sincron Grupos via QR code, pela IA. Use quando o usuário pedir para conectar o celular/WhatsApp, parear o número, ler o QR code, ou quando uma ação falhar por não haver instância conectada.
---

# Conectar WhatsApp (QR code)

Conecta o celular do usuário ao Sincron Grupos usando a tool MCP `instances` do servidor `sincron-grupos`.

## Passos

1. **Verifique instâncias existentes**: `instances({ action: "list" })`.
   - Se já houver uma conectada, informe e pergunte se quer conectar outra.
   - Se houver uma desconectada, prefira `instances({ action: "reconnect", instance_id })` em vez de criar outra.

2. **Crie a conexão**: `instances({ action: "connect", name: "WhatsApp Principal" })`.
   - A resposta traz `qr_code` (imagem base64), `instance_id` e `instrucoes`.
   - Apresente o QR ao usuário. Se o ambiente não renderizar a imagem base64, oriente:
     **WhatsApp → Aparelhos conectados → Conectar um aparelho → escanear o QR**.
   - Avise antes: conectar o número aqui pode desconectar outro sistema que use o mesmo número
     como aparelho conectado.

3. **Aguarde a conexão**: `instances({ action: "status", instance_id })` a cada ~5s.
   - Pare quando o status indicar conectado (mostre o número e o nome do perfil).
   - Se após ~2 min continuar desconectado, gere um novo QR com `instances({ action: "reconnect", instance_id })` (o QR expira).

## Sem câmera para ler o QR (pareamento por código)

1. `instances({ action: "check_phone", phone })` — diz se o número aceita código.
2. `instances({ action: "send_otp", phone })` — o WhatsApp do usuário recebe um código.
3. `instances({ action: "verify_otp", phone, verification_id, code })`.

## Observações
- Depois de conectado, o número passa a **receber avisos do sistema como mensagens de WhatsApp**.
- Erro "Limite de instâncias do plano atingido" → ofereça upgrade com a skill `assinar`.
- Próximo passo natural: importar grupos (`groups({ action: "available" })` → `groups({ action: "import" })`).
