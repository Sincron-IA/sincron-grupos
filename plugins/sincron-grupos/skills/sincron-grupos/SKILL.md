---
name: sincron-grupos
description: Configure e opere o Sincron Grupos (gestão de grupos WhatsApp) pela IA. Use as tools MCP do servidor `sincron-grupos` (instaladas com este plugin); o CLI `sincron` e a REST ficam para automação sem MCP. Use quando o usuário quiser configurar ou começar a usar a conta, assinar, conectar o WhatsApp/celular, importar/listar/organizar grupos, criar categorias, enviar ou agendar mensagens (inclusive para muitos grupos), escolher o que fazer com os áudios (transcrever/destacar), criar/gerir gatilhos, criar/ativar agentes de IA, ver estatísticas, ou checar a conta. Aciona em "configurar o Sincron Grupos", "me ajuda a começar", "conectar WhatsApp", "assinar o Grupos", "importar grupos", "mandar/agendar mensagem", "disparo para vários grupos", "criar categoria/gatilho", "transcrever áudio", "resumo de áudio", "ver estatísticas".
---

# Sincron Grupos — Operação pela IA

As três interfaces (MCP, CLI, REST) falam com o MESMO backend — mesmos recursos e ações.

## Ordem de preferência

1. **MCP (PADRÃO).** Este plugin já registra o servidor MCP `sincron-grupos`. Use as tools nativas.
   - Login (OAuth) no navegador. No Claude Code: `/mcp` → `sincron-grupos` → Authenticate. Não peça API key.
   - Sem as tools: rode `/reload-plugins` (ou reinicie). No Claude.ai, o usuário adiciona o conector em
     **Conectores → Adicionar conector personalizado** com a URL `https://grupos.sincronia.digital/api/mcp`.
2. **CLI `sincron`.** Para scripts, CI ou quando não houver MCP mas houver terminal.
3. **REST `/api/v1`.** Último recurso, HTTP direto.

---

## Onboarding guiado — COMECE POR AQUI

Quando o usuário quiser configurar, começar, ou não souber o próximo passo:

1. Chame `setup({ action: "guide" })`. Ele diz em que passo a conta está, o próximo passo, como fazer e
   **a pergunta exata** a fazer em cada passo.
2. Conduza **um passo por vez**, em linguagem simples, e **espere a resposta** antes de seguir.
3. Depois de cada passo, chame `setup guide` de novo.

As duas perguntas que nunca podem faltar:

**Áudios** (passo 6). Pergunte com as opções numeradas e deixe a pessoa escolher:
> O que o Sincron deve fazer com os áudios que chegam nos grupos?
> 1. **Desligado** — nada acontece.
> 2. **Só transcrever** — todo áudio vira texto no grupo.
> 3. **Transcrever e destacar** — vira texto e, quando tiver tarefa, prazo ou decisão, isso vem destacado em cima (bom para grupos de trabalho).
> 4. **Só quando pedirem** — transcreve só o áudio que alguém reagir com 📝.

Depois: `transcription({ action: "set", group_id OU category_id, mode })` com
`disabled | auto | auto_summary | manual` (na ordem acima). Por categoria afeta todos os grupos dela:
confirme. Modos ativos exigem plano com IA (Pro).

**Ritmo dos envios para vários grupos** (passo 7). Explique e pergunte:
> Quando você mandar a mesma mensagem para vários grupos, o Sincron envia **um grupo por vez**, com
> intervalo variável, para o seu número não ser bloqueado pelo WhatsApp (ex.: 40 grupos levam uns 40 min).
> Prefere deixar o Sincron decidir o ritmo (recomendado) ou quer um intervalo fixo seu?

- Sincron decide → nada a fazer (`disparo_intervalo_seg` fica `null`).
- Intervalo fixo → `organization({ action: "update", disparo_intervalo_seg: N })` (5–3600 s). Se N for
  menor que o recomendado, diga que é arriscado e confirme.

## Disparo seguro (mensagens para vários grupos)

Todo envio sai pela fila: o 1º grupo na hora, os demais um de cada vez, com intervalo médio que varia
±35% (para não parecer robô). Disparos simultâneos do mesmo número **não se somam**: dividem o mesmo ritmo.

| Grupos | Intervalo médio (Sincron decide) | Pausa longa | Exemplo |
|---|---|---|---|
| 2–5 | ~20 s | — | 5 grupos ≈ 1 min |
| 6–20 | ~30 s | — | 20 ≈ 10 min |
| 21–60 | ~60 s | — | 40 ≈ 39 min |
| mais de 60 | ~75 s | ~8 min a cada 20 | 100 ≈ 2 h 36 |

Antes de enviar para vários grupos:
1. `messages({ action: "plan", group_ids })` → mostre o `resumo` ("40 grupos · cerca de 39 min · um a cada ~60 s").
2. Confirme texto + grupos + tempo com o usuário.
3. `messages({ action: "send" | "schedule", group_ids, message, ... })`. A resposta traz `message_id` e o plano.
4. Acompanhe com `messages({ action: "get", message_id })`; para parar: `messages({ action: "cancel", message_id })`.

Ritmo manual: `interval_mode: "manual", interval_seconds: N`. Abaixo do recomendado o backend **recusa**
sem `accept_risk: true` — só envie isso depois de dizer ao usuário que o número pode ser bloqueado e ele
confirmar que segue mesmo assim. Nunca use `accept_risk` por conta própria.

## Conectar o WhatsApp

Skill `conectar-whatsapp`. Resumo: `instances({ action: "connect", name, phone? })` devolve `link_qr`
(página do painel com o QR, atualiza sozinha) e, com `phone`, `codigo_pareamento`. **Mande o link ou o
código** — a imagem do QR nem sempre aparece para o usuário. Depois `instances status` a cada ~5 s.

---

## 1) MCP — padrão

11 tools no formato **recurso + `action`**: `setup, account, organization, instances, groups, categories, messages, triggers, agents, transcription, stats`.
Ex.: `categories({ action: "create", name: "Leads", color: "#10B981" })`.

## 2) CLI `sincron` — automação sem MCP

```bash
curl -s https://grupos.sincronia.digital/sincron.mjs -o sincron.mjs   # arquivo único, Node 18+
node sincron.mjs login            # OAuth no navegador; guarda o token em ~/.sincron (0600)
node sincron.mjs endpoints        # lista recursos e ações
node sincron.mjs groups list
node sincron.mjs messages send --group_ids a,b --message "Olá a todos!" --dry-run
```
Headless/CI: `node sincron.mjs login --key sg_live_...` ou `export SINCRON_GRUPOS_API_KEY=sg_live_...`
(gere a key no painel em Settings → API keys). Arrays por vírgula (`--group_ids a,b,c`), objetos em
JSON (`--event_config '{"keyword_text":"oi"}'`), `--dry-run` mostra a request sem enviar.

## 3) REST `/api/v1` — último recurso

Base: `https://grupos.sincronia.digital/api/v1`. Auth: `Authorization: Bearer <sg_live_ ou token OAuth>`.
Descoberta: `GET /api/v1` · Docs: `/docs` · OpenAPI: `/api/v1/openapi.json`.

---

## Recursos & ações (valem para MCP, CLI e REST)

| Recurso | Ações |
|---|---|
| **setup** | `guide` (passo atual, próximo passo, perguntas a fazer) |
| **account** | `get` · `checkout` (plan=starter\|pro, period=monthly\|annual → link de pagamento Cakto) |
| **organization** | `get` · `update` (nome?, timezone IANA — afeta a hora dos agendamentos —, `disparo_intervalo_seg` = ritmo padrão, `null` = Sincron decide) |
| **instances** | `list` · `status`(instance_id) · `connect`(name, phone? → link_qr / codigo_pareamento) · `reconnect`(instance_id, phone?) · `disconnect`(instance_id) · `delete`(instance_id) · `check_phone`(phone) |
| **groups** | `list`(category_id?/search?) · `available`(search?) · `import`(group_jids[] OU name_filter) · `move_category`(group_ids[]+category_id\|null) · `settings`(group_id + guardar_todas_mensagens/processar_simultaneo/**visibilidade**=cliente\|interno) · `archive`(group_ids[]) |
| **categories** | `list` · `create`(name,color #RRGGBB) · `update`(category_id+name?/color?) · `delete`(category_id) |
| **messages** | `plan`(group_ids[] ou group_count, ritmo? — não envia) · `send`(group_ids[]+message, ritmo?) · `schedule`(+scheduled_at ISO futuro) · `get`(message_id → progresso) · `list`(page?/status?) · `cancel`(message_id; agendada ou em andamento) · `retry`(message_id; só os grupos que falharam) |
| **triggers** | `list` · `create`(event/action + alvo `target_group_ids[]` **e/ou** `target_category_ids[]`) · `update`(is_active pausa/ativa; mesmos alvos) · `delete` · `duplicate` · `run`(executa real) · `executions` · `reorder`(ordered_ids[]) · `test_webhook`(url) |
| **agents** | `list` · `create` · `update` · `delete` · `scope`(ativa em grupos **OU** categorias) · `test` · `bulk_toggle` · `logs` · `check_keys` |
| **transcription** | `get`(group_id) · `set`(group_id OU category_id, mode=disabled\|auto\|auto_summary\|manual) |
| **stats** | `get` |

Ritmo (send/schedule): `interval_mode=auto|manual`, `interval_seconds` (manual), `accept_risk` (manual abaixo do recomendado).
Enums: trigger event_type `keyword\|new_member\|member_leave\|scheduled\|reaction`; action_type `send_message\|auto_reply\|notify_admin\|delete_member\|webhook\|ai_agent`; agente scope_type `all\|groups\|categories`.

## Modelo mental — EXPLIQUE no onboarding

A montagem é sempre **instância → grupos → configuração**:

1. **Instância** = um número de WhatsApp conectado. Tudo pendura nela.
2. **Grupo** = importado da instância. Cada grupo pode ficar **isolado** (configurado sozinho) **ou** dentro de uma **categoria**.
3. **Categoria** = um balde que junta vários grupos para aplicar **a MESMA configuração a todos de uma vez** (e novos grupos movidos pra categoria herdam). **Não é só rótulo — é configuração consolidada.**

Por isso **toda configuração tem dois caminhos: por grupo (afeta 1) ou por categoria (afeta todos os grupos dela)**:

| Configuração | Por grupo | Por categoria |
|---|---|---|
| Áudios | `transcription set` com `group_id` | `transcription set` com `category_id` |
| Agente de IA (onde responde) | `agents scope` com `scope_type: groups` + `group_ids` | `agents scope` com `scope_type: categories` + `category_ids` |
| Gatilho (alvo) | `triggers create` com `target_group_ids` | `triggers create` com `target_category_ids` |
| Enviar/agendar mensagem | `messages send` com `group_ids` | **nativo não** → `groups list` com `category_id` e envie pros IDs |
| Ajustes do grupo (histórico, paralelo, visibilidade interno/cliente) | `groups settings` com `group_id` | **só por grupo** |

## Receita: radar de propostas/oportunidades (caso de uso comum)

Objetivo: **escutar** palavras como "proposta/oportunidade/orçamento" em grupos monitorados e **alertar a equipe** — sem nunca responder no grupo escutado.

1. **Organize a escuta:** `categories create` ("Monitorados") → `groups move_category` com os grupos a escutar.
2. **Escolha o destino do alerta com o usuário:**
   - **Celular conectado (default seguro):** `action_type: notify_admin` — zero risco de vazamento.
   - **Grupo interno da equipe:** primeiro marque `groups settings` com `visibilidade: interno` no grupo da equipe, depois `action_type: send_message` + `action_config: {"destination_type":"other_group","destination_group_id":"<id_grupo_equipe>", ...}`.
3. **Crie PAUSADO, mirando a categoria:**
```json
triggers({
  "action": "create", "name": "Radar oportunidades", "event_type": "keyword",
  "event_config": { "keyword_text": "proposta,oportunidade,orçamento,orcamento" },
  "action_type": "notify_admin",
  "action_config": { "notify_message": "⚠️ Possível oportunidade em {{nome_grupo}} ({{data_hora}}): {{mensagem}}" },
  "target_category_ids": ["<id_categoria_monitorados>"], "is_active": false
})
```
   Placeholders disponíveis na mensagem da ação: `{{nome_grupo}}`, `{{mensagem}}`, `{{nome_membro}}`, `{{numero_membro}}`, `{{data_hora}}`.
4. **Teste e ative:** `triggers run` (confirme onde o alerta chegou) → revisão do destino com o usuário → `triggers update` com `is_active: true`.
5. **NUNCA** use `auto_reply`/`own_group` nesta receita — o alerta cairia no próprio grupo escutado (vazamento). O backend recusa (`TRIGGER_UNSAFE_DESTINATION`) se o destino não for interno.

> Gatilho que envia para **vários grupos** também sai pela fila do disparo seguro (um grupo por vez).

## Receita: agente de IA respondendo no grupo

Objetivo: o agente **responde no próprio grupo** (ou no privado do membro, com `resposta_privada`) a partir de instruções que o usuário escreve. Exige plano com recursos de IA.

1. `agents check_keys` → confirma que há chave de IA disponível para o modelo escolhido.
2. `agents create` (nasce **pausado**) com uma instrução clara: o que responder, o que nunca dizer, tom.
3. `agents test` com perguntas reais do grupo → revise as respostas com o usuário.
4. `agents scope` no grupo ou categoria certos — **confirme que são grupos onde o agente pode falar em público**.
5. `agents update` com `ativo: true` só depois do OK explícito. Acompanhe com `agents logs`.

## 🔒 Segurança — travas OBRIGATÓRIAS (gatilhos & agentes)

> **Regra de ouro:** a automação que ESCUTA num grupo **não responde nesse mesmo grupo com conteúdo interno**. Escuta ≠ destino.

1. **Todo grupo é CLIENTE por padrão.** Antes de criar OU ativar qualquer gatilho/agente que POSTA num grupo, **declare o destino em voz alta** e pergunte: "esse grupo é **interno** (sua equipe) ou de **cliente**?". Conteúdo de uso interno (detecção de proposta/lead/oportunidade, alerta pra equipe, "avisar que…") **NUNCA** pode cair em grupo de cliente.

2. **Proibido `own_group` para notificação/detecção interna.** `action_config.destination_type: "own_group"` (responder no próprio grupo que disparou) e `auto_reply` só servem para resposta **pública genuína** que todos no grupo podem ver (boas-vindas, FAQ, "pong"). Para alerta **interno** use:
   - `notify_admin` → cai no **celular conectado** (mais seguro), ou
   - `send_message` com `destination_type: "other_group"` apontando para um grupo **marcado como interno**, ou `destination_type: "whatsapp"` para um número.

   **Marcar grupo como interno** (pré-requisito do destino interno): `groups settings` com `visibilidade: interno`. Todo grupo nasce `cliente` (protegido); só marque `interno` após o usuário confirmar que é grupo DA EQUIPE.

3. **Palavras sensíveis = freio duplo.** Se o conteúdo da ação contém *proposta, lead, oportunidade, orçamento, valor, preço, custo, fechar, contrato, fatura, comissão, margem* **E** o destino é `own_group` ou um grupo de cliente → **PARE**, mostre alerta forte e exija confirmação literal do usuário ("confirmo postar isso no grupo do cliente") antes de criar/ativar. O **backend também bloqueia** (`TRIGGER_UNSAFE_DESTINATION`): conteúdo sensível + destino `own_group`/grupo não-interno → recusado. Override só com `action_config.confirm_client_facing: true` — **nunca** envie esse campo sem a confirmação literal do usuário.

4. **Nasce PAUSADO.** Todo gatilho/agente criado pela IA → `is_active:false` / `ativo:false`. Só ative depois de: (a) revisar o destino com o usuário, (b) testar com `triggers run` / `agents test`, (c) OK explícito.

5. **Auditoria pré-uso (rodar antes de liberar pra equipe):** `triggers list` + `agents list` → sinalize toda automação **ativa** cujo destino seja `own_group` ou inclua grupo de cliente, principalmente com conteúdo sensível. Ofereça pausar.

## Regras
- **Confirme antes de qualquer escrita** (enviar/agendar mensagem, criar/editar/excluir gatilho/agente, mudar áudios, importar/arquivar grupos): mostre conteúdo + alvos e peça OK. No CLI, use `--dry-run` para revisar.
- **Escopo de categoria afeta TODOS os grupos** dela — confirme antes de usar `category_id` em `transcription set`/`agents scope`.
- Ações que disparam no WhatsApp de verdade: `messages send/retry`, `triggers run`, gatilhos/agentes ativos.
- Teste de envio: só num grupo que o usuário indicar.
- Datas de agendamento sempre futuras, ISO 8601, no fuso da organização.
- **Sem ferramenta de billing avançado** nem ações sensíveis (senha, API keys, equipe, excluir conta) — humano-only, pelo painel.
