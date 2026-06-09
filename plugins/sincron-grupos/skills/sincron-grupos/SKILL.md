---
name: sincron-grupos
description: Configure e opere o Sincron Grupos (gestão de grupos WhatsApp) pela IA. PREFIRA o CLI `sincron`; sem terminal, use o MCP; em último caso, a REST. Use quando o usuário quiser assinar, conectar o WhatsApp/celular, importar/listar/organizar grupos, criar categorias, enviar ou agendar mensagens, configurar transcrição/resumo de áudio, criar/gerir gatilhos, criar/ativar agentes de IA, ver estatísticas, ou checar a conta. Aciona em "configurar Sincron Grupos", "conectar WhatsApp", "assinar o Grupos", "importar grupos", "mandar/agendar mensagem", "criar categoria/gatilho", "ativar resumo de áudio", "ver estatísticas".
---

# Sincron Grupos — Operação pela IA

As três interfaces (CLI, MCP, REST) falam com o MESMO backend — mesmos recursos e ações.

## ⚠️ Ordem de preferência (SEMPRE nesta ordem)

1. **CLI `sincron` (PADRÃO).** Se você tem acesso a terminal/shell, **use o CLI**. Instala em 1 linha (arquivo único, zero-dependência), não exige plugin nem reiniciar o cliente.
2. **MCP (fallback — sem terminal).** Se NÃO houver shell, use as tools MCP nativas. Pode exigir instalar o plugin `sincron-grupos` e **reiniciar** o cliente; a auth é OAuth (abre o navegador).
3. **REST `/api/v1` (último recurso).** HTTP direto, quando nem CLI nem MCP servem.

---

## 1) CLI `sincron` — preferido

**Instalar (1 linha, sem precisar do repositório):**
```bash
curl -s https://grupos.sincronia.digital/sincron.mjs -o sincron.mjs
node sincron.mjs help
# (alternativa: dentro do repo → `cd cli && npm link` → comando global `sincron`)
```

**Autenticar (preferir OAuth no navegador — sem key manual):**
```bash
node sincron.mjs login                        # OAuth no navegador (recomendado); guarda token em ~/.sincron (0600)
# alternativa CI/headless:
node sincron.mjs login --key sg_live_...        # API key (gere em Settings → API keys)
# ou: export SINCRON_GRUPOS_API_KEY=sg_live_...
```
O `login` sem `--key` faz o fluxo OAuth (DCR+PKCE+loopback) e refresca o token sozinho. Onboarding de máquina nova = instalar + `login`, 1 clique.

**Usar** (`node sincron.mjs ...` ou `sincron ...` se global):
```bash
sincron endpoints                              # lista recursos e ações
sincron account get
sincron stats get
sincron groups list
sincron groups import --name_filter "Vendas"
sincron categories create --name "Leads" --color "#10B981"
sincron messages send --group_ids a,b --message "Olá a todos!"
sincron messages schedule --group_ids a --message "lembrete" --scheduled_at 2026-06-10T14:00:00-03:00
sincron transcription set --group_id <id> --mode auto_summary
sincron <recurso> <ação> --dry-run             # mostra a request sem enviar
```
Arrays por vírgula (`--group_ids a,b,c`), objetos em JSON (`--event_config '{"keyword_text":"oi"}'`), `--k=v` escapa valores com `--`.

## 2) MCP — fallback (sem terminal)

9 tools no formato **recurso + `action`**: `account, instances, groups, categories, messages, triggers, agents, transcription, stats`. Ex.: `categories({action:"create", name, color})`.
Se as tools não aparecerem, oriente o usuário a instalar o plugin `sincron-grupos` e **reiniciar** o Claude Code (a lista de tools só atualiza no restart). Auth OAuth abre o navegador (sem API key manual).

## 3) REST `/api/v1` — último recurso

Base: `https://grupos.sincronia.digital/api/v1`. Auth: `Authorization: Bearer <sg_live_ ou token OAuth>`.
Descoberta: `GET /api/v1` · Docs: `/docs` · OpenAPI: `/api/v1/openapi.json`.

---

## Recursos & ações (valem para CLI, MCP e REST)

| Recurso | Ações |
|---|---|
| **account** | `get` · `checkout` (plan=starter\|pro, period=monthly\|annual) |
| **instances** | `list` · `status`(instance_id) · `connect`(name→QR) · `reconnect`(instance_id) · `disconnect`(instance_id) |
| **groups** | `list`(category_id?/search?) · `available`(search?) · `import`(group_jids[] OU name_filter) · `move_category`(group_ids[]+category_id\|null) · `settings`(group_id+guardar_todas_mensagens/processar_simultaneo) · `archive`(group_ids[]) |
| **categories** | `list` · `create`(name,color #RRGGBB) · `update`(category_id+name?/color?) · `delete`(category_id) |
| **messages** | `list`(page?/status?) · `send`(group_ids[]+message) · `schedule`(+scheduled_at ISO futuro) · `cancel`(message_id) · `retry`(message_id) |
| **triggers** | `list` · `create` · `update`(is_active pausa/ativa) · `delete` · `duplicate` · `run`(executa real) · `executions` · `reorder`(ordered_ids[]) |
| **agents** | `list` · `create` · `update` · `delete` · `scope`(ativa no grupo) · `test` · `bulk_toggle` · `logs` |
| **transcription** | `get`(group_id) · `set`(group_id OU category_id, mode=disabled\|auto\|manual\|auto_summary) |
| **stats** | `get` |

Enums: trigger event_type `keyword\|new_member\|member_leave\|scheduled\|reaction`; action_type `send_message\|auto_reply\|notify_admin\|delete_member\|webhook\|ai_agent`; transcrição mode `disabled\|auto\|manual\|auto_summary`; agente scope_type `all\|groups\|categories`.

## Fluxo recomendado
1. **Diagnóstico:** `account get`. Sem assinatura → checkout. Ativo → seguir.
2. **Conectar WhatsApp:** `instances connect` → QR → `instances status` até conectado.
3. **Grupos:** sem grupos? `groups available` → `groups import`. Depois organize com `categories` + `groups move_category`.
4. **Operar:** `messages send/schedule`; automações `triggers`; resumo de áudio `transcription set`; números `stats get`.

## Regras
- **Confirme antes de qualquer escrita** (enviar/agendar mensagem, criar/editar/excluir gatilho/agente, mudar transcrição, importar/arquivar grupos): mostre conteúdo + alvos e peça OK. No CLI, use `--dry-run` para revisar.
- **Escopo de categoria afeta TODOS os grupos** dela — confirme antes de usar `category_id` em `transcription set`/`agents scope`.
- Ações que disparam no WhatsApp de verdade: `messages send/retry`, `triggers run`, gatilhos/agentes ativos.
- Datas de agendamento sempre futuras, ISO 8601.
- **Sem ferramenta de billing avançado** nem ações sensíveis (senha, API keys, equipe, excluir grupo) — humano-only.
