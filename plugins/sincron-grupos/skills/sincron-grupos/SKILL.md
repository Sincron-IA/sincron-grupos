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
| **groups** | `list`(category_id?/search?) · `available`(search?) · `import`(group_jids[] OU name_filter) · `move_category`(group_ids[]+category_id\|null) · `settings`(group_id + guardar_todas_mensagens/processar_simultaneo/**visibilidade**=cliente\|interno) · `archive`(group_ids[]) |
| **categories** | `list` · `create`(name,color #RRGGBB) · `update`(category_id+name?/color?) · `delete`(category_id) |
| **messages** | `list`(page?/status?) · `send`(group_ids[]+message) · `schedule`(+scheduled_at ISO futuro) · `cancel`(message_id) · `retry`(message_id) |
| **triggers** | `list` · `create`(event/action + alvo `target_group_ids[]` **e/ou** `target_category_ids[]`) · `update`(is_active pausa/ativa; mesmos alvos) · `delete` · `duplicate` · `run`(executa real) · `executions` · `reorder`(ordered_ids[]) |
| **agents** | `list` · `create` · `update` · `delete` · `scope`(ativa em grupos **OU** categorias) · `test` · `bulk_toggle` · `logs` |
| **transcription** | `get`(group_id) · `set`(group_id OU category_id, mode=disabled\|auto\|manual\|auto_summary) |
| **stats** | `get` |

Enums: trigger event_type `keyword\|new_member\|member_leave\|scheduled\|reaction`; action_type `send_message\|auto_reply\|notify_admin\|delete_member\|webhook\|ai_agent`; transcrição mode `disabled\|auto\|manual\|auto_summary`; agente scope_type `all\|groups\|categories`.

## Modelo mental — EXPLIQUE no onboarding

A montagem é sempre **instância → grupos → configuração**:

1. **Instância** = um número de WhatsApp conectado (QR). Tudo pendura nela.
2. **Grupo** = importado da instância. Cada grupo pode ficar **isolado** (configurado sozinho) **ou** dentro de uma **categoria**.
3. **Categoria** = um balde que junta vários grupos para aplicar **a MESMA configuração a todos de uma vez** (e novos grupos movidos pra categoria herdam). **Não é só rótulo — é configuração consolidada.**

Por isso **toda configuração tem dois caminhos: por grupo (afeta 1) ou por categoria (afeta todos os grupos dela)**. No onboarding, sempre ofereça as duas opções:
> "Quer cadastrar **grupos isolados** (configura cada um) ou criar **categorias** pra agrupar e configurar tudo de uma vez?"

### Por grupo vs por categoria — o que cada um aceita

| Configuração | Por grupo | Por categoria |
|---|---|---|
| Transcrição/resumo de áudio | `transcription set --group_id` | `transcription set --category_id` |
| Agente de IA (onde responde) | `agents scope --scope_type groups --group_ids` | `agents scope --scope_type categories --category_ids` |
| Gatilho (alvo) | `triggers create --target_group_ids` | `triggers create --target_category_ids` |
| Enviar/agendar mensagem | `messages send --group_ids` | **nativo não** → `groups list --category_id <id>` e envie pros IDs |
| Ajustes do grupo (histórico, paralelo, visibilidade interno/cliente) | `groups settings --group_id` | **só por grupo** |

> **Mensagem não aceita categoria direto.** Para mandar pra uma categoria inteira: `groups list --category_id <id>`, pegue os `group_ids` e passe em `messages send/schedule`.

## Fluxo recomendado (onboarding)
1. **Diagnóstico:** `account get`. Sem assinatura → checkout. Ativo → seguir.
2. **Conectar WhatsApp:** `instances connect` → QR → `instances status` até conectado.
3. **Importar grupos:** sem grupos? `groups available` → `groups import`.
4. **Decidir a organização (PERGUNTE ao usuário):**
   - **Grupos isolados** → configure cada um por `--group_id`.
   - **Categorias** → `categories create` → `groups move_category` pra encaixar → configure a categoria inteira de uma vez (`--category_id` / `scope categories` / `target_category_ids`).
5. **Operar:** `messages send/schedule`; automações `triggers`; resumo de áudio `transcription set`; agentes `agents`; números `stats get`.

## Receita: radar de propostas/oportunidades (caso de uso comum)

Objetivo: **escutar** palavras como "proposta/oportunidade/orçamento" em grupos monitorados e **alertar a equipe** — sem nunca responder no grupo escutado.

1. **Organize a escuta:** `categories create --name "Monitorados" --color "#10B981"` → `groups move_category` com os grupos a escutar.
2. **Escolha o destino do alerta com o usuário:**
   - **Celular conectado (default seguro):** `action_type: notify_admin` — zero risco de vazamento.
   - **Grupo interno da equipe:** primeiro marque `groups settings --group_id <id_grupo_equipe> --visibilidade interno`, depois `action_type: send_message` + `action_config: {"destination_type":"other_group","destination_group_id":"<id_grupo_equipe>", ...}`.
3. **Crie PAUSADO, mirando a categoria:**
```bash
sincron triggers create --name "Radar oportunidades" --event_type keyword \
  --event_config '{"keyword_text":"proposta,oportunidade,orçamento,orcamento"}' \
  --action_type notify_admin \
  --action_config '{"notify_message":"⚠️ Possível oportunidade em {{nome_grupo}} ({{data_hora}}): {{mensagem}}"}' \
  --target_category_ids <id_categoria_monitorados> --is_active false
```
   Placeholders disponíveis na mensagem da ação: `{{nome_grupo}}`, `{{mensagem}}`, `{{nome_membro}}`, `{{numero_membro}}`, `{{data_hora}}`.
4. **Teste e ative:** `triggers run` (confirme onde o alerta chegou) → revisão do destino com o usuário → `triggers update --trigger_id <id> --is_active true`.
5. **NUNCA** use `auto_reply`/`own_group` nesta receita — o alerta cairia no próprio grupo escutado (vazamento). O backend recusa (`TRIGGER_UNSAFE_DESTINATION`) se o destino não for interno.

## 🔒 Segurança — travas OBRIGATÓRIAS (gatilhos & agentes)

> **Regra de ouro:** a automação que ESCUTA num grupo **não responde nesse mesmo grupo com conteúdo interno**. Escuta ≠ destino. (Foi assim que uma detecção interna de "proposta" vazou no grupo do cliente.)

1. **Todo grupo é CLIENTE por padrão.** Antes de criar OU ativar qualquer gatilho/agente que POSTA num grupo, **declare o destino em voz alta** e pergunte: "esse grupo é **interno** (sua equipe) ou de **cliente**?". Conteúdo de uso interno (detecção de proposta/lead/oportunidade, alerta pra equipe, "avisar que…") **NUNCA** pode cair em grupo de cliente.

2. **Proibido `own_group` para notificação/detecção interna.** `action_config.destination_type: "own_group"` (responder no próprio grupo que disparou) e `auto_reply` só servem para resposta **pública genuína** que todos no grupo podem ver (boas-vindas, FAQ, "pong"). Para alerta **interno** use:
   - `notify_admin` → cai no **celular conectado** (mais seguro), ou
   - `send_message` com `destination_type: "other_group"` apontando para um grupo **marcado como interno**, ou `destination_type: "whatsapp"` para um número.

   **Marcar grupo como interno** (pré-requisito do destino interno): `groups settings --group_id <id> --visibilidade interno`. Todo grupo nasce `cliente` (protegido); só marque `interno` após o usuário confirmar que é grupo DA EQUIPE.

3. **Palavras sensíveis = freio duplo.** Se o conteúdo da ação contém *proposta, lead, oportunidade, orçamento, valor, preço, custo, fechar, contrato, fatura, comissão, margem* **E** o destino é `own_group` ou um grupo de cliente → **PARE**, mostre alerta forte e exija confirmação literal do usuário ("confirmo postar isso no grupo do cliente") antes de criar/ativar. O **backend também bloqueia** (`assertTriggerSafe`, erro `TRIGGER_UNSAFE_DESTINATION`): conteúdo sensível + destino `own_group`/grupo não-interno → recusado. Override só com `action_config.confirm_client_facing: true` — **nunca** envie esse campo sem a confirmação literal do usuário.

4. **Nasce PAUSADO.** Todo gatilho/agente criado pela IA → `is_active:false` / `ativo:false`. Só ative depois de: (a) revisar o destino com o usuário, (b) testar com `triggers run` / `agents test`, (c) OK explícito.

5. **Auditoria pré-uso (rodar antes de liberar pra equipe):** `triggers list` + `agents list` → sinalize toda automação **ativa** cujo destino seja `own_group` ou inclua grupo de cliente, principalmente com conteúdo sensível. Ofereça pausar.

## Regras
- **Confirme antes de qualquer escrita** (enviar/agendar mensagem, criar/editar/excluir gatilho/agente, mudar transcrição, importar/arquivar grupos): mostre conteúdo + alvos e peça OK. No CLI, use `--dry-run` para revisar.
- **Escopo de categoria afeta TODOS os grupos** dela — confirme antes de usar `category_id` em `transcription set`/`agents scope`.
- Ações que disparam no WhatsApp de verdade: `messages send/retry`, `triggers run`, gatilhos/agentes ativos.
- Datas de agendamento sempre futuras, ISO 8601.
- **Sem ferramenta de billing avançado** nem ações sensíveis (senha, API keys, equipe, excluir grupo) — humano-only.
