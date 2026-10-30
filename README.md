# Sincron Grupos para Claude

Opere o [Sincron Grupos](https://grupos.sincronia.digital) conversando com o Claude: assinar, conectar o
WhatsApp, importar e organizar grupos, enviar e agendar mensagens com disparo seguro, escolher o que fazer
com os áudios, gatilhos e agentes de IA.

O plugin registra o servidor MCP `https://grupos.sincronia.digital/api/mcp` e instala três skills que
ensinam o Claude a usá-lo com segurança. O login é feito no navegador (OAuth): você não precisa colar
nenhuma chave.

## Instalar

### Claude Code

```
/plugin install sincron-grupos --marketplace Sincron-IA/sincron-grupos
```

Esse comando de uma linha precisa do Claude Code 2.1.275 ou mais novo. Em versões anteriores, use dois:

```
/plugin marketplace add Sincron-IA/sincron-grupos
```
```
/plugin install sincron-grupos@sincron-grupos
```

Depois rode `/mcp`, escolha `sincron-grupos` e clique em **Authenticate**: o navegador abre para o login.
Se as tools não aparecerem, rode `/reload-plugins`.

### Claude.ai (web, desktop e celular), sem plugin

1. Na web, abra **Conectores** (em Configurações/Personalizar) e clique em **Adicionar conector personalizado**.
2. Nome: `Sincron Grupos`. URL:

   ```
   https://grupos.sincronia.digital/api/mcp
   ```
3. Faça o login quando pedir. Na conversa, ligue o conector em **+ → Conectores**.

Depois de adicionado na web, o conector funciona também no app do celular e no Claude Desktop. Em planos
Team e Enterprise, quem adiciona é o dono da organização.

### Codex

```
codex mcp add sincron-grupos --url https://grupos.sincronia.digital/api/mcp
```

O login abre no navegador. Se não abrir: `codex mcp login sincron-grupos`.

## Como começar

Peça: **"Configura minha conta do Sincron Grupos"**. O Claude segue um roteiro guiado e pergunta, em
linguagem simples, o que precisa: conectar o WhatsApp (link do QR ou código para digitar no celular),
quais grupos acompanhar, o que fazer com os áudios e o ritmo dos envios para vários grupos.

Depois, peça em português, como pediria a uma pessoa:

- "Quais grupos eu tenho?"
- "Manda 'Reunião às 15h hoje' no grupo Equipe Comercial."
- "Agenda um bom dia para os grupos da categoria Clientes amanhã às 8h."
- "Nos grupos da categoria Turmas, transcreve os áudios e destaca o que tiver de tarefa."

Antes de enviar ou alterar qualquer coisa, o Claude mostra o que vai fazer e para quais grupos, e pede
sua confirmação.

## Disparo seguro

Mandar a mesma mensagem para muitos grupos de uma vez pode bloquear o número. O Sincron envia um grupo
por vez, com intervalo variável (mais grupos, mais espaço entre eles), e o Claude mostra antes quanto tempo
vai levar. Dá para deixar o Sincron decidir (recomendado) ou escolher o intervalo.

## O que vem no plugin

| Skill | Para quê |
|---|---|
| `sincron-grupos` | Configuração guiada e operação: grupos, categorias, mensagens, áudios, gatilhos, agentes |
| `conectar-whatsapp` | Conectar um número por link do QR code ou código de pareamento |
| `assinar` | Gerar o link de pagamento (Pix ou cartão) e assinar |

O servidor MCP expõe 11 tools no formato recurso + `action`: `setup`, `account`, `organization`,
`instances`, `groups`, `categories`, `messages`, `triggers`, `agents`, `transcription`, `stats`.

## Suporte

https://grupos.sincronia.digital
