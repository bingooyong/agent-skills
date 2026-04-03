# 项目文档治理 Skill — 交付说明

本文档按你的要求顺序汇总整个 Skill 方案，便于验收与落地。

---

## 1. Skill 概述

本 Skill 面向**企业级项目的文档体系治理**：在存在 Memory ROOT（或类似长期记忆）的前提下，帮助自动规划、生成、补全和维护完整项目文档，并保持与项目上下文一致。核心能力包括：

- 以 **Memory ROOT 为基线**，避免重复提问和文档与记忆冲突；
- **文档盘点与缺口分析**，按文档类型清单与依赖关系确定应生成/更新的文档；
- **按文档类型与读者**选择合适深度，统一术语、版本、交叉引用与风格；
- **增量更新优于全量重写**，**事实优先、不确定处明确标记**，**对外与对内文档区分**；
- 支持**持续迭代**：需求/架构/接口/发布/事故后的文档跟进与质量校验。

设计上吸收了 **agentic-project-management** 的思想：结构化流程、长周期上下文保持、角色/步骤分层、从发现到执行闭环、memory/context continuity，并可在 Claude Code 环境中通过命令落地。

---

## 2. Skill 名称与 description

- **名称**：`project-docs-governance`（小写、短横线，不含 skill 单词）
- **description**（用于触发）：  
  *"Enterprise project documentation governance: plan, generate, complete and maintain a full documentation system. Use when (1) auditing or generating project docs (README, design, deployment, API, runbook, ADR, etc.), (2) aligning docs with Memory ROOT and existing context, (3) identifying doc gaps and inconsistencies, (4) incremental doc updates and cross-reference sync, (5) enforcing doc quality and reader-specific depth. Prioritizes Memory ROOT as single source of truth; avoids duplicate or conflicting content."*

- **目标用户**：使用 AI 辅助开发/运维的团队、项目负责人、技术写作与文档维护者。
- **适用场景**：项目初始化文档、文档审计、缺口分析、发布前文档检查、架构/接口/发布/事故后的文档更新、术语与交叉引用统一。
- **不适用场景**：纯代码注释、单文件临时说明、与项目文档体系无关的笔记。
- **能力边界**：依赖用户提供或项目已有的 Memory ROOT（或等价长期记忆）；不替代人工评审与合规审批；模板与指南为最佳实践建议，可按项目裁剪。

---

## 3. 目录结构

```
project-docs-governance/
├── SKILL.md
├── DELIVERY.md                 # 本交付说明
├── agents/
│   └── openai.yaml             # ChatGPT/OpenAI 技能元数据
├── references/
│   ├── document-map.md         # 文档体系清单、依赖与同步策略
│   ├── document-practices-summary.md  # 文档类型与章节最佳实践总表、治理机制、落地建议
│   ├── README-guide.md
│   ├── design-doc-guide.md
│   ├── deployment-guide.md
│   ├── api-doc-guide.md
│   ├── operations-guide.md
│   ├── testing-guide.md
│   ├── security-guide.md
│   ├── adr-guide.md
│   ├── ai-project-guide.md
│   ├── quality-checklist.md
│   └── commands.md             # Claude Code 命令设计
├── templates/
│   ├── README-template.md
│   ├── project-overview-template.md
│   ├── docs-index-template.md
│   ├── detailed-design-template.md
│   ├── deployment-manual-template.md
│   ├── api-manual-template.md
│   ├── operations-manual-template.md
│   ├── runbook-template.md
│   ├── adr-template.md
│   └── release-notes-template.md
└── assets/                     # 按需放置，当前为空
```

若需在 Claude Code 中通过 Slash 命令使用，可将 `references/commands.md` 中的各命令实现为 `.claude/commands/doc-*.md` 或 `.cursor/commands/doc-*.md`，在命令正文中引用本 Skill 与 Memory ROOT 读取步骤。

---

## 4. 完整 SKILL.md

已写入 `project-docs-governance/SKILL.md`。要点：

- frontmatter 仅含 `name` 与 `description`；
- 明确：何时用、如何先读 Memory ROOT、如何盘点与缺口识别、如何确定生成范围、如何选深度、如何做一致性、如何增量更新、如何避免重复/冲突/过时；
- 复杂细节放在 references/*.md，SKILL.md 中通过表格与列表引用。

---

## 5. references 文件逐个说明

| 文件 | 内容概要 |
|------|----------|
| **document-map.md** | 文档索引用途；文档类型清单（通用 + AI 专项）；依赖与同步策略；状态与版本；对外 vs 对内。 |
| **document-practices-summary.md** | 文档类型与章节最佳实践总表；文档治理机制摘要；质量校验使用方式；落地建议。 |
| **README-guide.md** | README 目的、读者、必备/可选章节、禁忌、质量标准。 |
| **design-doc-guide.md** | 系统架构、详细设计、技术选型三类的目的、读者、必备/可选、禁忌。 |
| **deployment-guide.md** | 部署手册目的、读者、必备章节（前置、拓扑、步骤、配置、验证、回滚）、禁忌与质量。 |
| **api-doc-guide.md** | 接口文档必备内容、形式建议（OpenAPI + Markdown）、禁忌与质量。 |
| **operations-guide.md** | 操作/运维手册、监控与告警、Runbook 的目的、必备内容、禁忌。 |
| **testing-guide.md** | 测试计划、验收/UAT、测试用例设计说明。 |
| **security-guide.md** | 安全设计说明、权限模型说明。 |
| **adr-guide.md** | ADR 单则结构、组织方式、禁忌。 |
| **ai-project-guide.md** | AI 项目专项：模型与推理、Prompt、Tool/MCP、Evaluation、Guardrails、Token/Cost、记忆/上下文。 |
| **quality-checklist.md** | 元数据、Memory 一致性、读者与深度、可执行与可验证、交叉引用、冲突与过时、安全；治理规则摘要。 |
| **commands.md** | 各 Claude Code 命令的输入、动作、输出、与 Memory ROOT 的关系、适用场景。 |

---

## 6. templates 文件建议

- **README-template.md**：项目名、描述、功能、快速开始、文档索引、版本与许可、贡献。
- **project-overview-template.md**：目标、范围、成功准则、约束与假设、相关文档、变更记录。
- **docs-index-template.md**：按类型与按角色的文档表、上游与依赖说明。
- **detailed-design-template.md**：范围、模块划分、关键接口、数据流、关键逻辑、变更记录。
- **deployment-manual-template.md**：前置、拓扑、步骤、验证、回滚、变更记录。
- **api-manual-template.md**：基础信息、错误码、接口列表、变更与兼容性。
- **operations-manual-template.md**：组件、日常操作、配置变更、监控与故障入口。
- **runbook-template.md**：现象、原因、诊断、处理、升级、变更记录。
- **adr-template.md**：标题、状态、背景、决策、选项、后果、相关。
- **release-notes-template.md**：版本、概述、新增/变更/修复/安全、升级与兼容性。

使用建议：生成文档时以对应 template 为骨架，从 Memory ROOT 与现有代码/配置中填充内容，并遵守对应 *-guide 的必备章节与禁忌。

---

## 7. Claude Code 命令设计

| 命令 | 输入 | 执行动作 | 输出 | 与 Memory ROOT |
|------|------|----------|------|----------------|
| **/doc-audit** | 可选路径 | 盘点、对照 document-map、检查元数据与一致性、交叉引用 | 审计报告（已有/缺失/过时/冲突/建议） | 以 Memory 为基准校验 |
| **/doc-generate** | 文档类型或名称、可选草稿 | 读 Memory；增量补全；按 guide 生成；写路径 | 文件路径与变更说明 | 内容与 Memory 一致，不足处标记待补充 |
| **/doc-sync** | 可选文档列表或「全部」、同步维度 | 统一术语、修正引用、对齐版本与状态、与 Memory 对齐 | 变更文件列表与说明 | 术语与状态以 Memory 为源 |
| **/doc-update** | 变更原因、可选受影响列表 | 确定受影响文档；增量更新；冲突提示 | 更新文件列表与摘要 | 变更若来自 Memory 则以 Memory 为准 |
| **/doc-gap-analysis** | 可选项目类型、路径 | 读 Memory；盘点；按 document-map 列应有文档；标缺失/占位/过旧；优先级与顺序 | 缺口分析报告 | 依 Memory 判断阶段与类型 |
| **/doc-release-check** | 可选版本号/标签 | 检查 Release Notes、部署版本、API 变更、链接；可选 quality-checklist | 通过/不通过项与建议 | 版本与阶段与 Memory 一致 |
| **/doc-runbook-create** | 故障现象/告警名、可选现有文档路径 | 按模板生成 Runbook；与索引衔接 | 新 Runbook 路径与索引更新建议 | 组件名与架构/运维一致 |

详细定义见 `references/commands.md`。实现时每个命令应先读取 Memory ROOT（若存在），再执行动作。

---

## 8. 文档类型与章节最佳实践总表

总表已集中在 `references/document-practices-summary.md`，包括：

- 各文档类型的目的、读者、必备/可选章节、禁忌、质量判断；
- 覆盖：README、项目概述、文档索引、术语表、架构、技术选型、详细设计、环境、配置、部署、运维、监控、API、数据库设计、测试计划、验收、安全、权限、Runbook、ADR、Release Notes、FAQ、Onboarding，以及 AI 专项（模型与推理、Prompt、评测等）。

---

## 9. 文档治理与质量校验机制

- **文档盘点规则**：按 document-map 扫描约定路径，列出已有文档及最后修改时间。
- **缺失识别**：对照 document-map 与项目类型，标出缺失与仅占位/过旧；按依赖决定生成顺序。
- **新旧/版本识别**：依据元数据版本、日期、状态；与 Release Notes 比对。
- **冲突检测**：与 Memory ROOT、术语表、ADR、同主题文档比对；冲突时提示并建议处理方式。
- **交叉引用规则**：相对路径或稳定 ID；生成时写「相关文档」；同步时校验有效。
- **术语统一**：术语表为权威；新术语先入表再使用。
- **状态标记**：draft → reviewed → approved；deprecated 时注明替代。
- **更新触发**：需求→验收/测试；架构→架构/设计/部署/运维；接口→API；发布→Release Notes/部署；事故/复盘→Runbook/监控。

质量校验使用 `references/quality-checklist.md`：单份文档逐项打勾；全量审计对关键文档执行；发布前结合 /doc-release-check 与清单中发布相关项。

---

## 10. 落地建议

1. **首次引入**：将本 Skill 或 references/templates 放入项目或技能库；指定 Memory ROOT 路径；运行 /doc-gap-analysis 得到缺口与优先级。
2. **补齐基础**：先生成文档索引、术语表、项目概述（与 Memory 对齐）；再按依赖顺序生成架构、环境、配置、部署、API、运维、Runbook 等。
3. **日常维护**：需求/架构/接口/发布/事故后执行 /doc-update 或按更新触发规则人工更新；定期 /doc-sync；发布前 /doc-release-check。
4. **与 APM 结合**：Memory ROOT 使用 APM 的 `.apm/Memory/Memory_Root.md`；Phase 完成后在 Memory 写摘要，并触发相关文档更新。
5. **演进**：按项目反馈调整 document-map、各 guide 的必备章节、模板与命令。

---

## 最终打包建议

- **作为 Cursor/Codex Skill 使用**：将 `project-docs-governance/` 整体复制到 `.cursor/skills/` 或 `$CODEX_HOME/skills/`，确保 SKILL.md 的 frontmatter 与 description 可被识别。
- **作为 ChatGPT 技能**：使用 `agents/openai.yaml` 中的 display_name、short_description、default_prompt 配置到 ChatGPT 技能平台（若平台支持类似字段）。
- **在 Claude Code 中落地**：根据 `references/commands.md` 在 `.claude/commands/` 或 `.cursor/commands/` 下创建 `doc-audit.md`、`doc-generate.md` 等，每份命令正文中注明「先读取 Memory ROOT（路径…），再执行…」，并引用本 Skill 的 SKILL.md 或 references。
- **仅用参考资料与模板**：不安装 Skill 时，也可仅复制 `references/` 与 `templates/` 到项目的 `docs/` 或专用目录，按文档地图与各 guide 人工或脚本驱动文档生成与更新。

以上为项目文档治理 Skill 的完整交付内容；所有正文已落盘于 `project-docs-governance/` 目录，可直接使用或按需裁剪。

---

## 如何使 Skill 可用（本项目已配置）

1. **Skill 安装位置**
   - 本项目：`.cursor/skills/project-docs-governance/`（与根目录 `project-docs-governance/` 内容一致，供 Cursor 识别）。
   - 全局（可选）：已复制到 `~/.cursor/skills/project-docs-governance/`，在其他项目中也可被 Cursor 加载。

2. **Slash 命令（本项目内直接可用）**
   - 在 Claude Code / Cursor 中输入以下命令即可触发对应流程：
     - `/doc-audit` — 文档审计
     - `/doc-gap-analysis` — 文档缺口分析
     - `/doc-generate` — 生成/补全指定文档
     - `/doc-sync` — 文档同步（术语、引用、版本）
     - `/doc-update` — 按变更原因增量更新文档
     - `/doc-release-check` — 发布前文档检查
     - `/doc-runbook-create` — 创建单则 Runbook
   - 命令定义位于 `.claude/commands/doc-*.md`，执行前会自动读取 Memory ROOT（`.apm/Memory/Memory_Root.md`）与本 Skill。

3. **在其他项目使用**
   - 若需在其他仓库使用同一套命令：将 `.claude/commands/doc-*.md` 复制到该项目的 `.claude/commands/`，并将本 Skill 置于该项目的 `.cursor/skills/project-docs-governance/`，或依赖全局 `~/.cursor/skills/project-docs-governance/`（命令中读取路径需能访问到 Skill 的 references/templates）。
