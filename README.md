# Plugin Sincron Grupos (Claude Code / Codex)

Configure e opere o **Sincron Grupos** inteiramente pela IA — assinar, conectar o WhatsApp e
gerenciar grupos/mensagens — via servidor **MCP** hospedado em `grupos.sincronia.digital/api/mcp`.

Autenticação por **OAuth 2.0**: no primeiro uso o Claude Code abre o navegador para login.
**O usuário não configura MCP nem cola API key.**

## Estrutura

```
plugins/sincron-grupos/
  .claude-plugin/plugin.json     # manifesto do plugin
  .mcp.json                      # registra o MCP remoto (auto-conectado no install)
  skills/
    sincron-grupos/SKILL.md      # orquestrador (onboarding completo)
    conectar-whatsapp/SKILL.md   # conexão por QR
    assinar/SKILL.md             # checkout self-serve
```

## Instalar (Claude Code)

```
/plugin marketplace add Sincron-IA/sincron-grupos
/plugin install sincron-grupos
```

No primeiro uso de uma tool, o Claude Code abre o navegador para o login OAuth. Pronto.

## Usar (Codex)

Codex não tem plugins, mas conecta ao mesmo MCP com um comando:

```
codex mcp add sincron-grupos --url https://grupos.sincronia.digital/api/mcp
```

## Publicação — duas opções

1. **Repo próprio** `Sincron-IA/sincron-grupos` (este diretório): `/plugin marketplace add Sincron-IA/sincron-grupos`.
2. **Marketplace `sincron-tools`** (compartilhado, no repo `gtm`): adicione a entrada abaixo em
   `.claude-plugin/marketplace.json` e referencie o source (path local se mover a pasta, ou um
   `source` git apontando para o repo do plugin):

```json
{
  "name": "sincron-grupos",
  "description": "Configure e opere o Sincron Grupos pela IA via MCP (assinatura, WhatsApp, grupos, mensagens).",
  "version": "1.0.0",
  "author": { "name": "Sincron IA" },
  "source": "./plugins/sincron-grupos",
  "category": "productivity"
}
```

## Backend necessário (já implementado no app sincron-grupos)

- Servidor OAuth 2.0 com DCR + PKCE: `/.well-known/oauth-authorization-server`,
  `/.well-known/oauth-protected-resource`, `/oauth/register`, `/oauth/authorize`, `/oauth/token`.
- Servidor MCP JSON-RPC protegido: `/api/mcp`.
- Migration `20260608000001_oauth_mcp.sql` aplicada (`npm run db:migrate`).
- Env var opcional `NEXT_PUBLIC_APP_URL=https://grupos.sincronia.digital` (senão resolve pela request).

## Tools MCP expostas
`get_account`, `create_checkout_link`, `list_instances`, `connect_phone`, `get_instance_status`,
`list_groups`, `list_categories`, `send_message`, `schedule_message`.
