---
name: conectar-whatsapp
description: Conecta um número de WhatsApp ao Sincron Grupos pela IA, por link do QR code ou código de pareamento. Use quando o usuário pedir para conectar o celular/WhatsApp, parear o número, ler o QR code, digitar um código, ou quando uma ação falhar por não haver instância conectada.
---

# Conectar WhatsApp

Conecta o celular do usuário ao Sincron Grupos usando a tool MCP `instances` do servidor `sincron-grupos`.

## Passos

1. **Veja o que já existe**: `instances({ action: "list" })`.
   - Já há uma conectada → informe e pergunte se quer conectar outra.
   - Há uma desconectada → prefira `reconnect` com o `instance_id` dela em vez de criar outra.

2. **Pergunte como a pessoa está**:
   > Você está no computador (escaneia um QR com o celular) ou só no celular (digita um código)?
   > Se for o celular, qual o número com DDD?

3. **Crie ou reconecte**:
   - Nova: `instances({ action: "connect", name: "WhatsApp Principal", phone? })`
   - Existente: `instances({ action: "reconnect", instance_id, phone? })`

   A resposta traz:
   - `link_qr`: página do painel com o QR, que se atualiza sozinha. **Mande este link** para quem está no
     computador. A imagem do QR que vem junto nem sempre aparece para o usuário (no Claude.ai fica escondida
     dentro do bloco da tool).
   - `codigo_pareamento` (só quando você mandou `phone`): a pessoa abre **WhatsApp → Aparelhos conectados →
     Conectar aparelho → Conectar com número de telefone** e digita o código.

   Avise antes: conectar o número aqui pode desconectar outro sistema que use o mesmo número como aparelho
   conectado.

4. **Aguarde a conexão**: `instances({ action: "status", instance_id })` a cada ~5 s.
   - Pare quando estiver `conectado` (mostre o número e o nome do perfil).
   - Passou de ~2 min sem conectar → `reconnect` (o QR e o código expiram) e mande o novo link/código.

## Observações
- Erro "Limite de instâncias do plano atingido" → ofereça upgrade com a skill `assinar`.
- Próximo passo natural: `setup({ action: "guide" })` — importar grupos, escolher o que fazer com os áudios
  e o ritmo dos envios.
