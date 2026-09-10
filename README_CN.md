# Dify Workflow DSL Skill

一套帮助 AI Agent 编写、修改、审查、调试**可直接导入 Dify** 的 Workflow / Chatflow
DSL(YAML)的技能包。用自然语言描述需求,Agent 自动生成包含节点、边、布局与配置的完整
工作流文件。

> 维护者:**RookieTvTao** · 仓库:https://github.com/RookieTvTao/dify-workflow-dsl-skill

---

## 来源与署名(请先读)

本仓库由多个来源组合而成,授权姿态各不相同。维护者的原创内容(节点路由表、Schema 陷阱、
模板、README、工程化文件)采用 **MIT License**(`LICENSE`);基础技能 fork 自未授权的上游。

| 内容 | 来源 | 授权 |
| --- | --- | --- |
| 原创增强(节点路由表、Schema 陷阱、`templates.md`、README、`requirements.txt`、`examples/`、`.github/`、CI) | RookieTvTao | **MIT**,Copyright (c) 2026 RookieTvTao |
| 基础技能(SKILL.md 结构、`references/`\*、`scripts/validate_dsl.py`、`install.sh`、`agents/`) | fork 自 [`yzmw123/dify-workflow-dsl-skill`](https://github.com/yzmw123/dify-workflow-dsl-skill) | **无 LICENSE**(All Rights Reserved);保留并署名,再分发需取得上游授权 |
| 增强思路(节点路由表 / Schema 陷阱 / 模板) | 改编自 [`jspi-fu/Aeson-skills`](https://github.com/jspi-fu/Aeson-skills) | **MIT**,Copyright (c) 2025 jspi-fu |

完整的来源与各授权的精确范围见 `NOTICE`。MIT `LICENSE` 仅覆盖原创贡献,不延伸至 All
Rights Reserved 的基础技能部分。

---

## 本版本相对上游的改动

在 yzmw123 原版基础上(保留全部 `references/` 与校验脚本),新增:

- **节点路由表**:SKILL.md 内置 16 行快速选型表,按用途映射节点 `data.type` 与关键字段,
  指向 `references/node-schemas.md` 对应锚点。
- **常见 Schema 陷阱**:6 条最易导致导入失败的字段形状错误——变量列表形状随节点而异、
  `memory` 仅属于 chatflow 的 LLM、`end.outputs` 与 `code.outputs` 形状不同、迭代需两处设尺寸
  并遵守子节点接线规则、`output_type` 须匹配真实元素类型、`dataset_ids` 跨租户失效且
  `code_language` 必填。
- **`references/templates.md`**:5 个可直接导入的骨架(chatbot / RAG / 分类器智能体流 /
  translation / agent 节点),含完整节点、边与布局坐标;外加面向写操作的**副作用安全模式**
  (状态分类分流、`unknown` 绝不自动重试、写前人工确认、写后回读核对)。
- **多版本支持**:面向 0.5.x / 0.6.x / 0.7.x,附实测的 Dify 版本 ↔ DSL 版本映射
  (DSL `0.7.0` 随 Dify ≥ 1.16.0 发布;≤ 1.15 仍为 `0.6.0`)与升级重验协议;
  见 `references/dsl-versions.md`。
- **`references/import-troubleshooting.md`**:Dify 导入/运行报错 → 原因 → 修复的闭环手册,
  含租户绑定 `dataset_ids`、工作区盘点优先的生成前置规则。
- **Marketplace 公开 API 检索**:免登录拉取插件索引(manifest)与声明(batch),
  直接取到工具名、参数 schema、授权要求与当前依赖标识,组装 tool/agent 节点;
  见 `references/plugin-marketplace-tools.md`。
- **Agent 节点生成**:支持官方 `langgenius/agent` 的 `function_calling` / `ReAct`
  策略与第三方策略(按声明生成),workflow 与 chatflow 均可。
- **工作流片段(Dify 1.15+)**:生成独立的 `kind: snippet` DSL——输入字段、
  虚拟 start 引用、禁用节点规则;并明确插入的片段在 app DSL 中以普通节点展开
  (`references/snippets.md`)。
- **工程化**:`requirements.txt`、由 `validate` GitHub Actions 工作流校验的 `examples/` 语料、
  以及贡献模板。
- 关键标题与术语增加**中文/英文双语**标注。

---

## 能做什么

- 生成可导入 Dify 的 `workflow` 与 `advanced-chat` DSL YAML。
- 根据业务需求判断 `workflow` vs `advanced-chat`,并选择节点组合。
- 新文件默认面向用户所选 DSL 版本(0.5.x / 0.6.x / 0.7.x;默认 0.7.0)。
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

> `install.sh` 只把 `SKILL.md`、`references/`、`scripts/`、`agents/` 复制到目标 skills
> 目录;已安装过可加 `--force` 覆盖。

### 离线 / zip 安装

适用于无法访问 GitHub 的同事(在外部 agent 中生成 DSL,再导入内网 Dify):

1. **制包**(在有克隆的机器上):`git archive --format=zip -o
   dify-workflow-dsl-skill.zip HEAD` —— 只含 git 跟踪文件,不含 `.git/`。
   也可手动压缩文件夹,排除 `.git/`。
2. **安装**:解压后把 `SKILL.md`、`references/`、`scripts/`、`agents/` 拷入你的
   agent skills 目录(如 `~/.claude/skills/dify-workflow-dsl/`)。有 bash 环境可
   `bash install.sh --platform claude` 等效完成;Windows 无 Git Bash 时直接手动拷贝
   这 4 项即可。
3. **校验脚本**:需要 `pip install pyyaml`(见 `requirements.txt`)。
4. **导入内网 Dify 前**:先确认服务端 Dify 版本(Web 界面 → 关于/系统信息,或问
   管理员),并告知 agent,以便选对 DSL 版本(见 `references/dsl-versions.md`)。

## 校验

先安装依赖,再校验:

```bash
pip install -r requirements.txt          # 或:pip install pyyaml
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
├── LICENSE               # MIT — 仅覆盖原创贡献(见 NOTICE)
├── NOTICE                # 来源与授权姿态
├── requirements.txt      # 校验依赖(pyyaml)
├── agents/
│   └── openai.yaml
├── examples/             # 可导入 YAML,由 CI 校验
│   ├── translation.yml
│   ├── if-else.yml
│   ├── http-code.yml
│   └── chatflow.yml
├── install.sh
├── references/
│   ├── complete-examples.md
│   ├── database-tools.md
│   ├── dsl-structure.md
│   ├── dsl-versions.md     # ← 版本选择(0.5/0.6/0.7)
│   ├── node-schemas.md
│   ├── official-target.md
│   ├── plugin-marketplace-tools.md
│   ├── real-world-yml-study.md
│   ├── templates.md
│   └── usecase-node-selection.md
├── scripts/
│   └── validate_dsl.py
├── .github/
│   ├── workflows/validate.yml
│   ├── CONTRIBUTING.md
│   ├── ISSUE_TEMPLATE/
│   └── pull_request_template.md
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
