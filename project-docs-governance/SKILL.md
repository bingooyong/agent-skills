---
name: project-docs-governance
description: "Enterprise project documentation governance: plan, generate, complete and maintain a full documentation system. Use when (1) auditing or generating project docs (README, design, deployment, API, runbook, ADR, etc.), (2) aligning docs with Memory ROOT and existing context, (3) identifying doc gaps and inconsistencies, (4) incremental doc updates and cross-reference sync, (5) enforcing doc quality and reader-specific depth. Prioritizes Memory ROOT as single source of truth; avoids duplicate or conflicting content."
---

# 项目文档治理 Skill

本 Skill 帮助为项目规划、生成、补全和维护完整文档体系。优先以 **Memory ROOT** 为上下文基线，增量更新优于全量重写，对外/对内文档区分，事实优先、不确定处明确标记。

## 何时使用

- 用户要求盘点/生成/补全/同步项目文档
- 用户要求文档审计、缺口分析、发布前文档检查
- 项目有 Memory ROOT（如 `.apm/Memory/Memory_Root.md`）或类似长期记忆，需与文档一致
- 需要统一术语、版本、交叉引用或文档风格
- 需求/架构/接口/发布/事故后需更新多份文档

**不适用**：纯代码注释、单文件说明、与项目文档体系无关的临时笔记。

## 执行前：必读 Memory ROOT

1. **定位 Memory ROOT**  
   常见路径：`.apm/Memory/Memory_Root.md`、`docs/Memory_Root.md`、或用户指定。若不存在，向用户确认是否以其他“项目长期记忆”文件替代。

2. **读取并吸收**  
   提取：项目目标、阶段状态、架构决策、技术栈、约束、术语、环境、流程、规范。用于后续所有生成与更新，避免与已有记忆冲突。

3. **冲突与不足**  
   - 若用户输入与 Memory ROOT 冲突：先提示冲突并给出处理建议（以 Memory 为准 / 以用户为准 / 建议更新 Memory），再执行。
   - 若 Memory 信息不足：在生成内容中明确标记「待补充」或「待与项目方确认」，不捏造细节。

## 文档盘点与缺口识别

1. **盘点现有文档**  
   扫描项目（如 `docs/`、仓库根、`.apm/`）中 Markdown 与约定文档路径，列出已有文档及其大致用途、最后修改时间。

2. **对照文档体系清单**  
   使用 [references/document-map.md](references/document-map.md) 中的文档类型清单，按项目类型（常规软件 / 含 AI 组件）勾选应有文档，识别**缺失**与**仅存占位/过旧**的文档。

3. **输出缺口报告**  
   列出：缺失文档、建议优先级、与 Memory ROOT/其他文档的依赖关系。可据此决定先生成哪些、后补哪些。

## 确定生成/更新范围

- **按用户指令**：若用户指定「只生成 README」或「只补全部署手册」，则仅处理指定项。
- **按缺口与依赖**：先生成被多处引用的基础文档（如项目概述、文档索引、术语表），再生成依赖它们的文档（如架构、详细设计、接口、部署）。
- **按读者与深度**：每份文档目标读者不同（开发/测试/运维/架构/产品/客户/审计）。深度与风格见 [references/document-map.md](references/document-map.md) 及各 `*-guide.md`。

## 生成与更新规则

1. **增量优先**  
   若文档已存在且部分有效：在原有结构上补全章节、修正过时处、统一术语与编号，不整篇重写除非用户明确要求。

2. **事实优先，不确定即标记**  
   以代码、配置、Memory ROOT、接口定义等可验证内容为准。无法确认的内容写「待确认」或「见 [某文档]」，不编造。

3. **对外 vs 对内**  
   - 对外：客户/合作方/合规审计用，注意脱敏、版本号、发布说明、合规表述。
   - 对内：开发/运维/测试用，可含内部路径、环境、账号约定、故障预案。

4. **元数据与关系**  
   每份文档包含：版本、状态（draft/reviewed/approved/deprecated）、作者/维护者、更新时间、变更摘要；文末说明「相关文档」与「上游/下游」关系。规范见 [references/quality-checklist.md](references/quality-checklist.md)。

5. **一致性**  
   - 术语与 Memory ROOT、术语表一致。  
   - 交叉引用使用相对路径或文档 ID，便于后续自动校验。  
   - 版本号、发布日期与发布说明/变更记录一致。

## 按文档类型选择参考

| 文档类型 | 参考 | 模板 |
|----------|------|------|
| README、项目概述、文档索引 | document-map.md, README-guide.md | templates/ |
| 架构、详细设计、技术选型 | design-doc-guide.md | detailed-design-template.md |
| 部署、环境、配置 | deployment-guide.md | deployment-manual-template.md |
| API/接口 | api-doc-guide.md | api-manual-template.md |
| 操作/运维、监控、Runbook | operations-guide.md | operations-manual-template.md, runbook-template.md |
| 测试、验收 | testing-guide.md | — |
| 安全、权限 | security-guide.md | — |
| ADR、决策记录 | adr-guide.md | adr-template.md |
| AI 模型/推理/Prompt/评测 | ai-project-guide.md | — |
| 质量与治理 | quality-checklist.md | — |

每类文档的推荐章节结构、必备/可选节、禁忌与质量标准见对应 `*-guide.md`。

## 文档治理机制概要

- **盘点与缺口**：按 document-map 定期盘点；缺口识别见上文。  
- **冲突检测**：与 Memory ROOT、术语表、其他文档比对；冲突时提示并建议处理方式。  
- **状态与更新触发**：draft → reviewed → approved；在需求/架构/接口/发布/事故/回顾后触发相关文档更新。  
- **交叉引用与术语**：统一使用术语表；引用可解析、可校验。  

细节见 [references/quality-checklist.md](references/quality-checklist.md) 与 [references/document-map.md](references/document-map.md) 中的「依赖与同步策略」。

## Claude Code 命令

在 Claude Code 中可通过命令触发固定流程，见 [references/commands.md](references/commands.md)。典型命令示例：`/doc-audit`、`/doc-generate`、`/doc-sync`、`/doc-gap-analysis`、`/doc-release-check`。每个命令的输入、动作、输出及与 Memory ROOT 的关系在 references/commands.md 中定义。

## 输出语言与格式

- 文档内容与 Skill 输出**中文**为主（除非项目明确要求英文）。  
- 统一使用 **Markdown**，章节层级清晰，可被自动化工具解析。

## 总结

本 Skill 将项目文档视为可长期演进的体系：以 Memory ROOT 为基线，结合文档地图与各类型指南，完成盘点→缺口识别→生成/补全→一致性维护→质量校验，并支持通过 Claude Code 命令做审计、生成、同步与发布前检查。
