# Dify Workflow DSL Skill

一套帮助 AI Agent 编写、修改、审查、调试**可直接导入 Dify** 的 Workflow / Chatflow
DSL(YAML)的技能包。用自然语言描述需求,Agent 自动生成包含节点、边、布局与配置的完整
工作流文件。

> 维护者:**RookieTvTao** · 仓库:https://github.com/RookieTvTao/dify-workflow-dsl-skill(私有)

---

## 来源与署名(请先读)

**本仓库是派生作品,非从零原创**,基于以下两个上游项目组合而成:

| 内容 | 来源 | 授权 |
| --- | --- | --- |
| 基础技能(SKILL.md 结构、`references/`、`scripts/validate_dsl.py`、`install.sh`、`agents/`) | fork 自 [`yzmw123/dify-workflow-dsl-skill`](https://github.com/yzmw123/dify-workflow-dsl-skill) | **无 LICENSE**(默认 All Rights Reserved);本仓库出于个人使用目的保留其内容并保留原作者署名 |
| 增强内容(节点路由表 / 常见 Schema 陷阱 / 模板 的思路) | 改编自 [`jspi-fu/Aeson-skills`](https://github.com/jspi-fu/Aeson-skills) | **MIT License**,Copyright (c) 2025 jspi-fu |

由于上游基础技能未授权,本仓库**不附加任何 LICENSE**,也不主张对 yzmw123 原作部分的版权;
Aeson-skills 派生部分遵循其 MIT 条款。如需将本项目用于公开分发或商用,请先取得上游授权。

---

## 本版本相对上游的改动

在 yzmw123 原版基础上(保留全部 `references/` 与校验脚本),新增:

- **节点路由表**:SKILL.md 内置 15 行快速选型表,按用途映射节点 `data.type` 与关键字段,
  指向 `references/node-schemas.md` 对应锚点。
- **常见 Schema 陷阱**:5 条最易导致导入失败的字段形状错误——变量列表形状随节点而异、
  `memory` 仅属于 chatflow 的 LLM、`end.outputs` 与 `code.outputs` 形状不同、迭代需两处设尺寸
  并遵守子节点接线规则、`output_type` 须匹配真实元素类型。
- **`references/templates.md`**:4 个可直接导入的骨架模板(chatbot / RAG / agent / translation),
  含完整节点、边与布局坐标。
- 关键标题与术语增加**中文/英文双语**标注。

**刻意未引入**(遵循 YAGNI):Admin API 自动部署、基于 `config.yml` 的多版本检测系统。

---

## 能做什么

- 生成可导入 Dify 的 `workflow` 与 `advanced-chat` DSL YAML。
- 根据业务需求判断 `workflow` vs `advanced-chat`,并选择节点组合。
- 新文件默认面向 Dify 官方 app DSL `version: "0.6.0"`。
- 编写常见节点:Start、End、Answer、LLM、Code、IF/ELSE、HTTP Request、Template Transform、
  Variable Aggregator、Assigner、Document Extractor、Question Classifier、Parameter Extractor、
  Knowledge Retrieval、Agent、Iteration、Loop、Tool、Datasource、各类 Trigger 等。
- 自动规划节点 ID、边连接与分支 handle。
- 补齐 marketplace / package / GitHub 插件依赖。
- 编写数据库读写流程,支持 `spance/db_client_node` 与 `hjlarry/database` 模式。
- 审查已有 DSL 的导入风险与逻辑问题。
- 用 `scripts/validate_dsl.py` 做本地结构校验。

---

## 怎么用

把本目录放到 Agent 的 skills 目录(如 `~/.claude/skills/`),或在提问时显式调用:

```text
Use $dify-workflow-dsl to create an advanced-chat Dify workflow.
用户上传 PDF,提取文本,用通义千问总结,写入 PostgreSQL,最后回复用户。
```

审查已有 DSL:

```text
Use $dify-workflow-dsl to review this Dify YAML and fix import-breaking issues.
重点检查 tool 节点、数据库 SQL、节点边连接。
```

新增插件工具时,最稳的做法是先在 Dify 里配置一次该工具节点、导出最小 DSL,再让 Agent
复用其中的 `provider_id` / `tool_name` / `paramSchemas` / `tool_parameters` / 依赖字段。

---

## 安装

```bash
git clone https://github.com/RookieTvTao/dify-workflow-dsl-skill.git
cd dify-workflow-dsl-skill
bash install.sh --platform claude      # 或 codex / openclaw / hermes / all
```

> 该仓库为私有,`clone` 需要有访问权限的 GitHub 凭据。`install.sh` 只把 `SKILL.md`、
> `references/`、`scripts/`、`agents/` 复制到目标 skills 目录;已安装过可加 `--force` 覆盖。

## 校验

```bash
python scripts/validate_dsl.py path/to/workflow.yml
python scripts/validate_dsl.py examples/*.yml   # 批量
```

校验脚本检查:YAML 解析、DSL version 类型、图连接、节点类型一致性、LLM/tool 基础字段、
变量引用,以及 `INSERT` 字段列表尾逗号等常见 SQL 问题。

---

## 项目结构

```text
.
├── SKILL.md              # 主文档(含节点路由表、Schema 陷阱)
├── agents/
│   └── openai.yaml
├── install.sh
├── references/
│   ├── complete-examples.md
│   ├── database-tools.md
│   ├── dsl-structure.md
│   ├── node-schemas.md
│   ├── official-0.6-target.md
│   ├── plugin-marketplace-tools.md
│   ├── real-world-yml-study.md
│   ├── templates.md        # ← 本版本新增
│   └── usecase-node-selection.md
├── scripts/
│   └── validate_dsl.py
├── README.md
└── README_CN.md
```

---

## 上游与致谢

本项目站在以下工作的肩膀上:

- 基础技能:[`yzmw123/dify-workflow-dsl-skill`](https://github.com/yzmw123/dify-workflow-dsl-skill)
- 增强参考:[`jspi-fu/Aeson-skills`](https://github.com/jspi-fu/Aeson-skills)(MIT)
- Dify:[`langgenius/dify`](https://github.com/langgenius/dify)
- 公开 DSL 语料参考:`BannyLon/DifyAIA`、`svcvit/Awesome-Dify-Workflow`、
  `wwwzhouhui/dify-for-dsl`、`TheOneWithChair/Dify-DSL-generator`、
  `g-krishna0/dify-export-test`、`Petrus-Han/dify-usecase-playground`
- 规范参考:[Agent Skills specification](https://agentskills.io/specification)、
  [Anthropic skills examples](https://github.com/anthropics/skills)

English version: [README.md](./README.md)
