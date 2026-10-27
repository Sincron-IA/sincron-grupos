# Sincron Grupos para Claude

Opere o [Sincron Grupos](https://grupos.sincronia.digital) conversando com o Claude: assinar, conectar o
WhatsApp, importar e organizar grupos, enviar e agendar mensagens, configurar resumo de áudio, gatilhos e
agentes de IA.

O plugin registra o servidor MCP `https://grupos.sincronia.digital/api/mcp` e instala três skills que
ensinam o Claude a usá-lo com segurança. O login é feito no navegador (OAuth): você não precisa colar
nenhuma chave.

## Instalar

### Claude Code

```
/plugin install sincron-grupos --marketplace Sincron-IA/sincron-grupos
```

Em versões mais antigas do Claude Code, use dois comandos:

```
/plugin marketplace add Sincron-IA/sincron-grupos
```
```
/plugin install sincron-grupos
```

Feche e abra o Claude Code. Na primeira vez que pedir algo, o navegador abre para o login.

### Claude.ai (web ou desktop), sem plugin

**Configurações → Conectores → Adicionar conector personalizado** e informe a URL:

```
https://grupos.sincronia.digital/api/mcp
```

### Codex

```
codex mcp add sincron-grupos --url https://grupos.sincronia.digital/api/mcp
```

## Como usar

Peça em português, como pediria a uma pessoa:

- "Quais grupos eu tenho?"
- "Manda 'Reunião às 15h hoje' no grupo Equipe Comercial."
- "Agenda um bom dia para os grupos da categoria Clientes amanhã às 8h."
- "Ativa o resumo de áudio nos grupos da categoria Turmas."

Antes de enviar ou alterar qualquer coisa, o Claude mostra o que vai fazer e para quais grupos, e pede
sua confirmação.

## O que vem no plugin

| Skill | Para quê |
|---|---|
| `sincron-grupos` | Operação completa: grupos, categorias, mensagens, gatilhos, agentes, transcrição |
| `conectar-whatsapp` | Conectar um número por QR code (ou código de pareamento) |
| `assinar` | Gerar o link de pagamento (Pix ou cartão) e começar o trial de 7 dias |

O servidor MCP expõe 10 tools no formato recurso + `action`: `account`, `organization`, `instances`,
`groups`, `categories`, `messages`, `triggers`, `agents`, `transcription`, `stats`.

## Suporte

https://grupos.sincronia.digital
