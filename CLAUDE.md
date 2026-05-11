# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个多 skill 单体仓库，收集可复用的 AI Agent 技能。每个 skill 是独立的、可复用的能力单元，兼容 Claude Code / Cursor / Codex 等 agent 环境。所有 skill 均为 Markdown 文档，无代码、无依赖、无构建步骤。

## 仓库结构

- `skills/` — 15 个标准 skill，每个目录下必需 `SKILL.md`
- `project-docs-governance/` — 独立的企业级文档治理 skill（含 references/ 和 templates/）
- `templates/skill-template.md` — 新建 skill 的模板
- `AGENTS.md` — 仓库级 skill 治理规范（必须遵守）
- `references/`、`scripts/` — 跨 skill 共享资源（目前为空）

## Skill 目录规范

```
skills/<skill-name>/
├── SKILL.md              # 必需，唯一入口，必须自包含
├── references/           # 可选，skill 专属参考资料
└── templates/            # 可选，skill 专属模板
```

SKILL.md 通过相对路径引用自己的 references/ 和 templates/，不依赖仓库级共享资源。

## Skill 命名

- 小写短横线格式：`knowledge-consolidation`
- 描述能力而非实现：`memory-governance` 而非 `memory-file-manager`
- 不含 `skill`、`agent`、`ai` 等冗余前缀

## SKILL.md 必需结构

1. **YAML frontmatter**：`name` + `description`（description 用英文，含 "Use when" 触发场景）
2. **正文必需章节**：何时使用（含不适用场景）、输入（表格）、输出、执行步骤（编号）、边界与非目标、验收标准
3. **质量红线**：无空话、无模糊步骤、必须声明边界、必须有验收标准、不含临时上下文

## 新增 Skill 前置检查

必须 5 项全"是"才可新增：复用性（3+ 项目）、独立性（不依赖特定项目）、可执行性（明确输入输出步骤）、边界清晰、与已有 skill 不重叠。参见 `AGENTS.md` 中的完整边界表。

## 关键约束

- SKILL.md 正文用中文，frontmatter description 用英文
- 不把短期上下文（临时状态、未验证推测、一次性选择）写入 SKILL.md
- 仓库级 `templates/`、`references/`、`scripts/` 只放确认通用的内容，skill 专属资源放在 skill 自己目录下
- `project-docs-governance` 是特殊的独立 skill，不放在 `skills/` 下

## Skill 协作关系

`knowledge-consolidation` 是路由中枢，判断知识沉淀到哪里：`agents-md-maintainer`、`memory-governance` 或 `project-doc-generator`。代码质量生态 skill 之间互相关联（review checklist、error pattern、api review、refactoring、dependency、test strategy）。`hooks-designer` 与所有 skill 协作。`commit-message-craftsman` 和 `onboarding-guide-generator` 相对独立。
