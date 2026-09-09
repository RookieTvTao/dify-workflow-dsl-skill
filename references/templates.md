# Templates (模板)

Starter graph skeletons you can adapt, then complete with the patterns in
`SKILL.md` and the node schemas in `node-schemas.md`. All target the user's
chosen version (default `0.7.0`). Replace model providers, dataset IDs, and tool
identifiers with values exported from your target Dify workspace before
importing. Model names like `qwen3.5-flash` are placeholders — substitute a
model available in your workspace.

## How to use (使用方法)

1. Pick the closest template by matching condition.
2. Copy the YAML block, rename the app, and adjust `model`, prompts, IDs.
3. Validate with `python3 scripts/validate_dsl.py <file.yml>`.

## Node ID and layout conventions

- Node `id`: any unique quoted string works (the validator only requires a
  string, never reused). Real exports commonly use 13-digit timestamps
  (e.g. `"1711536487001"`); templates here follow that style, while
  `complete-examples.md` uses shorter readable IDs for clarity.
- Start position `{x: 80, y: 282}`; each subsequent column `x + 300`; parallel
  branches offset `y + 200`.
- Edge `id`: `{source}-source-{target}-target`. Linear edges use
  `sourceHandle: source`, `targetHandle: target`.
- Every edge carries `data.sourceType`/`data.targetType` matching the endpoint
  node `data.type`, plus `isInIteration`/`isInLoop`.

---

## 1. Chatbot (简单对话机器人)

**Match when:** a single assistant answers `sys.query` with no retrieval or tools.

Shape: `Start -> LLM -> Answer` (advanced-chat).

```yaml
version: "0.7.0"
kind: app
app:
  name: "Simple Chatbot"
  mode: advanced-chat
  description: "A minimal chatbot: Start -> LLM -> Answer."
  icon: "🤖"
  icon_type: emoji
  icon_background: "#FFEAD5"
  use_icon_as_answer_icon: false
dependencies: []
workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload: { enabled: false }
    opening_statement: "Hello! How can I help you today?"
    retriever_resource: { enabled: false }
    sensitive_word_avoidance: { enabled: false }
    speech_to_text: { enabled: false }
    suggested_questions: []
    suggested_questions_after_answer: { enabled: false }
    text_to_speech: { enabled: false }
  graph:
    nodes:
      - id: "1711536487001"
        type: custom
        position: { x: 80, y: 282 }
        data:
          type: start
          title: "Start"
          variables: []
      - id: "1711536522001"
        type: custom
        position: { x: 380, y: 282 }
        data:
          type: llm
          title: "LLM"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0.7 }
          prompt_template:
            - { role: system, text: "You are a helpful assistant." }
            - { role: user, text: "{{#sys.query#}}" }
          context: { enabled: false, variable_selector: [] }
          memory:
            query_prompt_template: "{{#sys.query#}}"
            window: { enabled: false, size: 10 }
          vision: { enabled: false }
      - id: "1711536558001"
        type: custom
        position: { x: 680, y: 282 }
        data:
          type: answer
          title: "Answer"
          answer: "{{#1711536522001.text#}}"
          variables: []
    edges:
      - id: "1711536487001-source-1711536522001-target"
        source: "1711536487001"
        sourceHandle: source
        target: "1711536522001"
        targetHandle: target
        type: custom
        zIndex: 0
        data: { sourceType: start, targetType: llm, isInIteration: false, isInLoop: false }
      - id: "1711536522001-source-1711536558001-target"
        source: "1711536522001"
        sourceHandle: source
        target: "1711536558001"
        targetHandle: target
        type: custom
        zIndex: 0
        data: { sourceType: llm, targetType: answer, isInIteration: false, isInLoop: false }
    viewport: { x: 0, y: 0, zoom: 0.7 }
```

---

## 2. RAG (知识库问答)

**Match when:** answers must be grounded in a knowledge base; user asks a question
and the assistant cites retrieved context.

Shape: `Start -> Knowledge Retrieval -> LLM (context enabled) -> Answer`.

```yaml
version: "0.7.0"
kind: app
app:
  name: "RAG Chatbot"
  mode: advanced-chat
  description: "Retrieve from a knowledge base and answer with grounded context."
  icon: "📚"
  icon_type: emoji
  icon_background: "#E4FBCC"
  use_icon_as_answer_icon: false
dependencies: []
workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload: { enabled: false }
    opening_statement: "Ask me anything about the knowledge base."
    retriever_resource: { enabled: true }
    sensitive_word_avoidance: { enabled: false }
    speech_to_text: { enabled: false }
    suggested_questions: []
    suggested_questions_after_answer: { enabled: false }
    text_to_speech: { enabled: false }
  graph:
    nodes:
      - id: "1711536487001"
        type: custom
        position: { x: 80, y: 282 }
        data: { type: start, title: "Start", variables: [] }
      - id: "1711536600001"
        type: custom
        position: { x: 380, y: 282 }
        data:
          type: knowledge-retrieval
          title: "Knowledge Retrieval"
          dataset_ids: ["REPLACE_WITH_DATASET_ID"]
          query_variable_selector: ["sys", query]
          retrieval_mode: multiple
          multiple_retrieval_config:
            top_k: 4
            reranking_enable: false
            score_threshold_enabled: false
            score_threshold: 0
          metadata_filtering_mode: disabled
      - id: "1711536522001"
        type: custom
        position: { x: 680, y: 282 }
        data:
          type: llm
          title: "LLM"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0.3 }
          prompt_template:
            - { role: system, text: "Answer the user's question using only the provided context. If the context is insufficient, say you don't know." }
            - { role: user, text: "{{#sys.query#}}" }
          context:
            enabled: true
            variable_selector: ["1711536600001", result]
          memory:
            query_prompt_template: "{{#sys.query#}}"
            window: { enabled: false, size: 10 }
          vision: { enabled: false }
      - id: "1711536558001"
        type: custom
        position: { x: 980, y: 282 }
        data:
          type: answer
          title: "Answer"
          answer: "{{#1711536522001.text#}}"
          variables: []
    edges:
      - { id: "1711536487001-source-1711536600001-target", source: "1711536487001", sourceHandle: source, target: "1711536600001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: start, targetType: knowledge-retrieval, isInIteration: false, isInLoop: false } }
      - { id: "1711536600001-source-1711536522001-target", source: "1711536600001", sourceHandle: source, target: "1711536522001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: knowledge-retrieval, targetType: llm, isInIteration: false, isInLoop: false } }
      - { id: "1711536522001-source-1711536558001-target", source: "1711536522001", sourceHandle: source, target: "1711536558001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: llm, targetType: answer, isInIteration: false, isInLoop: false } }
    viewport: { x: 0, y: 0, zoom: 0.7 }
```

---

## 3. Agent (工具调用 + 分支)

**Match when:** the workflow must classify intent or extract parameters, then route
to different handling (tool call, retrieval, or direct answer).

Shape: `Start -> Question Classifier -> (branches) -> LLM -> Answer`. The skeleton
below shows two classifier branches re-joined by a Variable Aggregator. Extend each
branch with the nodes it needs (tool, knowledge-retrieval, code).

```yaml
version: "0.7.0"
kind: app
app:
  name: "Router Agent"
  mode: advanced-chat
  description: "Classify intent, route each branch, then answer."
  icon: "🧭"
  icon_type: emoji
  icon_background: "#D5F5F6"
  use_icon_as_answer_icon: false
dependencies: []
workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload: { enabled: false }
    opening_statement: "How can I help?"
    retriever_resource: { enabled: false }
    sensitive_word_avoidance: { enabled: false }
    speech_to_text: { enabled: false }
    suggested_questions: []
    suggested_questions_after_answer: { enabled: false }
    text_to_speech: { enabled: false }
  graph:
    nodes:
      - id: "1711536487001"
        type: custom
        position: { x: 80, y: 282 }
        data: { type: start, title: "Start", variables: [] }
      - id: "1711536700001"
        type: custom
        position: { x: 380, y: 282 }
        data:
          type: question-classifier
          title: "Question Classifier"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0 }
          query_variable_selector: ["sys", query]
          classes:
            - { id: "1", name: "需要工具" }
            - { id: "2", name: "直接回答" }
          instruction: "将用户问题分类到最合适的一类。"
          vision: { enabled: false }
      - id: "1711536800001"
        type: custom
        position: { x: 680, y: 182 }
        data:
          type: llm
          title: "LLM (with tool)"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0.3 }
          prompt_template:
            - { role: system, text: "Use available tools to help the user." }
            - { role: user, text: "{{#sys.query#}}" }
          context: { enabled: false, variable_selector: [] }
          memory:
            query_prompt_template: "{{#sys.query#}}"
            window: { enabled: false, size: 10 }
          vision: { enabled: false }
      - id: "1711536800002"
        type: custom
        position: { x: 680, y: 382 }
        data:
          type: llm
          title: "LLM (direct)"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0.5 }
          prompt_template:
            - { role: system, text: "Answer the user directly." }
            - { role: user, text: "{{#sys.query#}}" }
          context: { enabled: false, variable_selector: [] }
          memory:
            query_prompt_template: "{{#sys.query#}}"
            window: { enabled: false, size: 10 }
          vision: { enabled: false }
      - id: "1711536900001"
        type: custom
        position: { x: 980, y: 282 }
        data:
          type: variable-aggregator
          title: "Merge Branches"
          output_type: string
          variables:
            - ["1711536800001", text]
            - ["1711536800002", text]
      - id: "1711536558001"
        type: custom
        position: { x: 1280, y: 282 }
        data:
          type: answer
          title: "Answer"
          answer: "{{#1711536900001.output#}}"
          variables: []
    edges:
      - { id: "1711536487001-source-1711536700001-target", source: "1711536487001", sourceHandle: source, target: "1711536700001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: start, targetType: question-classifier, isInIteration: false, isInLoop: false } }
      - { id: "1711536700001-1-1711536800001-target", source: "1711536700001", sourceHandle: "1", target: "1711536800001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: question-classifier, targetType: llm, isInIteration: false, isInLoop: false } }
      - { id: "1711536700001-2-1711536800002-target", source: "1711536700001", sourceHandle: "2", target: "1711536800002", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: question-classifier, targetType: llm, isInIteration: false, isInLoop: false } }
      - { id: "1711536800001-source-1711536900001-target", source: "1711536800001", sourceHandle: source, target: "1711536900001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: llm, targetType: variable-aggregator, isInIteration: false, isInLoop: false } }
      - { id: "1711536800002-source-1711536900001-target", source: "1711536800002", sourceHandle: source, target: "1711536900001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: llm, targetType: variable-aggregator, isInIteration: false, isInLoop: false } }
      - { id: "1711536900001-source-1711536558001-target", source: "1711536900001", sourceHandle: source, target: "1711536558001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: variable-aggregator, targetType: answer, isInIteration: false, isInLoop: false } }
    viewport: { x: 0, y: 0, zoom: 0.7 }
```

Note: classifier branch edges use the class `id` as `sourceHandle` (`"1"`, `"2"`).
Add a `dependencies` entry for any plugin-backed tool you wire into a branch.

---

## 4. Translation (文本转换/翻译)

**Match when:** input text is transformed by an LLM with a fixed system prompt and
returned as a result. Shown as a one-shot `workflow` ending at `end` — switch to
`advanced-chat` + `answer` if it should be conversational.

Shape: `Start (text input) -> LLM -> End`.

```yaml
version: "0.7.0"
kind: app
app:
  name: "EN->ZH Translator"
  mode: workflow
  description: "Translate English input to Chinese."
  icon: "🌐"
  icon_type: emoji
  icon_background: "#E0F2FE"
  use_icon_as_answer_icon: false
dependencies: []
workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload: { enabled: false }
    opening_statement: ""
    retriever_resource: { enabled: false }
    sensitive_word_avoidance: { enabled: false }
    speech_to_text: { enabled: false }
    suggested_questions: []
    suggested_questions_after_answer: { enabled: false }
    text_to_speech: { enabled: false }
  graph:
    nodes:
      - id: "1711536487001"
        type: custom
        position: { x: 80, y: 282 }
        data:
          type: start
          title: "Start"
          variables:
            - label: "英文文本"
              variable: input_text
              type: paragraph
              required: true
              max_length: 50000
      - id: "1711536522001"
        type: custom
        position: { x: 380, y: 282 }
        data:
          type: llm
          title: "Translate"
          model:
            provider: langgenius/tongyi/tongyi
            name: qwen3.5-flash
            mode: chat
            completion_params: { temperature: 0.3 }
          prompt_template:
            - { role: system, text: "You are a professional translator. Translate the user's English text into natural Chinese. Output only the translation." }
            - { role: user, text: "{{#1711536487001.input_text#}}" }
          context: { enabled: false, variable_selector: [] }
          vision: { enabled: false }
      - id: "1711536558001"
        type: custom
        position: { x: 680, y: 282 }
        data:
          type: end
          title: "End"
          outputs:
            - variable: result
              value_selector: ["1711536522001", text]
    edges:
      - { id: "1711536487001-source-1711536522001-target", source: "1711536487001", sourceHandle: source, target: "1711536522001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: start, targetType: llm, isInIteration: false, isInLoop: false } }
      - { id: "1711536522001-source-1711536558001-target", source: "1711536522001", sourceHandle: source, target: "1711536558001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: llm, targetType: end, isInIteration: false, isInLoop: false } }
    viewport: { x: 0, y: 0, zoom: 0.7 }
```

This `workflow` LLM has **no `memory`** block and no `sys.query` — both are
chatflow-only. See the Schema Pitfalls in `SKILL.md`.

## 5. Side-effect safety pattern (写操作安全模式)

**Match when:** the workflow writes to the outside world — ERP/CRM records,
database inserts, payments, outgoing messages. This is the pattern proven by
conversation-driven workflow builders that compile-then-fix against a real
Dify instance (e.g. an AgentGen-style e2e loop over order-to-cache flows).

A plain `Start -> HTTP write -> End` is not acceptable for side effects. Use
this shape instead:

```
Start -> validate input -> JSON-valid? --no--> End (reject, with reason)
                          |--yes-> lookup/prepare -> exists? --yes--> human confirm
                                                            |--no--> create path ─┘
human confirm -> write (draft) -> status branch:
    success      -> read back -> matches? -> End (ok)
    auth_error   -> End (auth failure, user action)
    failed       -> End (failure, safe to retry manually)
    unknown      -> End (UNKNOWN — never auto-retry; result is indeterminate)
```

Key rules, each earned from real write-flow failures:

1. **Classify outcomes, don't binary-branch.** After a write call, branch on
   `success / failed / auth_error / unknown` (in an `if-else` over a Code node
   that parses the response). `unknown` (timeout, truncated response, unclear
   body) must **never auto-retry** — a retry may double-write. Route it to a
   human with the request ID.
2. **Human confirmation before the write.** Insert a confirmation step
   (operator review of the assembled payload) before any irreversible call.
   In current Dify this is a pause/trigger pattern or an approval step before
   the workflow proceeds; at minimum, write as `Draft` status when the target
   system supports it.
3. **Read back and verify.** After writing, read the record back and compare
   key fields before declaring success; a 200 response is not proof of the
   intended state.
4. **Validate before you build.** Parse/validate user input (JSON schema,
   required fields) *before* assembling the write payload, and reject early
   with a reason the user can act on.
5. **Return the foreign key.** The `end.outputs` should include the created
   record's ID (e.g. `docname` from an ERPNext-style API) so operators can
   trace the result even when later steps fail.

This pattern composes with template 3 (Agent) when preparation needs tools,
and with `database-tools.md` when the write target is SQL (keep writes
parameterized; prefer draft/staging tables).

## 6. Agent node (agent 节点最小示例)

**Match when:** the workflow needs an autonomous LLM + tools loop via a strategy
plugin (`langgenius/agent`: `function_calling` or `ReAct`), instead of a
hand-wired classifier flow (template 3). Works in `workflow` and
`advanced-chat`; shown here as a one-shot `workflow`.

Shape: `Start (query) -> Agent -> End`. The dependency identifier below was
current at authoring time (v0.0.47) — re-fetch `latest_package_identifier` via
the marketplace API (`plugin-marketplace-tools.md`) before importing, and
install/authorize the plugin in the target workspace first.

```yaml
version: "0.7.0"
kind: app
app:
  name: "Agent Node Demo"
  mode: workflow
  description: "Start -> Agent (function_calling) -> End."
  icon: "🤖"
  icon_type: emoji
  icon_background: "#FFEAD5"
  use_icon_as_answer_icon: false
dependencies:
  - current_identifier: null
    type: marketplace
    value:
      marketplace_plugin_unique_identifier: langgenius/agent:0.0.47@b14b5a5259094510a272e5b7facd09aa63352511c13fabcde36a4cac57d1da94
      version: null
workflow:
  environment_variables: []
  conversation_variables: []
  features:
    file_upload: { enabled: false }
    opening_statement: ""
    retriever_resource: { enabled: false }
    sensitive_word_avoidance: { enabled: false }
    speech_to_text: { enabled: false }
    suggested_questions: []
    suggested_questions_after_answer: { enabled: false }
    text_to_speech: { enabled: false }
  graph:
    nodes:
      - id: "1711536487001"
        type: custom
        position: { x: 80, y: 282 }
        data:
          type: start
          title: "Start"
          variables:
            - label: "问题"
              variable: query
              type: paragraph
              required: true
              max_length: 480
      - id: "1711536522001"
        type: custom
        position: { x: 380, y: 282 }
        data:
          type: agent
          title: "Agent"
          desc: ""
          agent_strategy_provider_name: langgenius/agent/agent
          agent_strategy_name: function_calling
          agent_strategy_label: FunctionCalling
          agent_parameters:
            model:
              type: constant
              value:
                provider: langgenius/tongyi/tongyi
                model: qwen3.5-flash
                mode: chat
                model_type: llm
                type: model-selector
            query:
              type: constant
              value: "{{#1711536487001.query#}}"
            instruction:
              type: constant
              value: "Answer the user's question. Use tools when they help."
            tools:
              type: constant
              value: []
            maximum_iterations:
              type: constant
              value: 5
          output_schema: null
          selected: false
      - id: "1711536558001"
        type: custom
        position: { x: 680, y: 282 }
        data:
          type: end
          title: "End"
          outputs:
            - variable: result
              value_selector: ["1711536522001", text]
    edges:
      - { id: "1711536487001-source-1711536522001-target", source: "1711536487001", sourceHandle: source, target: "1711536522001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: start, targetType: agent, isInIteration: false, isInLoop: false } }
      - { id: "1711536522001-source-1711536558001-target", source: "1711536522001", sourceHandle: source, target: "1711536558001", targetHandle: target, type: custom, zIndex: 0, data: { sourceType: agent, targetType: end, isInIteration: false, isInLoop: false } }
    viewport: { x: 0, y: 0, zoom: 0.7 }
```

Variations:

- **Chatflow**: set `mode: advanced-chat`, replace the `end` node with
  `answer`, and bind `query` to `{{#sys.query#}}`.
- **ReAct strategy**: switch `agent_strategy_name`/`agent_strategy_label` to
  `ReAct`; parameter set is identical minus `files` (see `node-schemas.md#agent`).
- **Tools**: add installed tools to `agent_parameters.tools`; entries follow
  the tool node identity fields (`plugin`/`provider`/`tool_name`/`parameters`).
  `tools: []` gives an LLM-only loop with no tool calls.
