# 仓库规范

本文件约束 `agent-skills` 仓库中新增和维护 skill 的规范。

## 仓库定位

公开的 agent skills 合集。每个 skill 是独立的、可复用的能力单元，兼容 Claude Code / Cursor / Codex 等 agent 环境。

## Skill 命名

- 使用小写短横线格式：`knowledge-consolidation`、`hooks-designer`
- 名称应描述能力而非实现方式：用 `memory-governance` 而非 `memory-file-manager`
- 不在名称中包含 `skill`、`agent`、`ai` 等冗余前缀

## 目录结构

每个 skill 必须遵循：

```
skills/<skill-name>/
├── SKILL.md              # 必需，skill 的唯一定义文件
├── references/           # 可选，详细参考资料
└── templates/            # 可选，模板文件
```

- SKILL.md 是 skill 的唯一入口，必须自包含
- references/ 放详细参考资料，SKILL.md 中通过相对路径引用
- templates/ 放模板文件，供 skill 执行时使用

## SKILL.md 质量要求

### Frontmatter（必需）

```yaml
---
name: <skill-name>
description: "一句话描述，包含触发场景关键词，用于 agent 自动匹配"
---
```

description 要求：
- 用英文编写（agent 匹配主要依赖英文）
- 包含 "Use when" 列出典型触发场景
- 精确描述能力范围，避免过度承诺

### 正文结构（必需）

```markdown
# 标题

一段话说明这个 skill 做什么。

## 何时使用
（触发条件 + 不适用场景）

## 输入
（表格列出输入类型、说明、是否必需）

## 输出
（列出 skill 执行后产出的内容）

## 执行步骤
（编号步骤，每步有明确输入和输出）

## 边界与非目标
（明确不做什么，与其他 skill 的边界）

## 验收标准
（如何判断 skill 执行成功）
```

### 文档质量红线

以下内容不得出现在 SKILL.md 中：

- **空话** — "这个 skill 非常有用" 之类的无信息量描述
- **模糊步骤** — "根据情况处理" 必须细化为具体的判断逻辑
- **缺失边界** — 必须说明不做什么
- **缺失验收标准** — 必须有可判断的标准
- **临时上下文** — 不把当前任务的状态、进度写入 SKILL.md

## 不要把短期上下文写成长期规则

```
✅ 正确：写入 skill
- 稳定的项目约束（构建命令、技术栈要求）
- 反复验证过的最佳实践
- 团队一致同意的协作规范

❌ 错误：写入 skill
- "正在修复 X bug"（临时状态）
- "可能是 Y 的问题"（未验证推测）
- "这次任务中用了 Z"（一次性选择）
- "接下来要做 A"（计划，非规则）
```

## templates / references / scripts 维护

- **templates/** — 只放跨 skill 可复用的模板。新 skill 的模板放在 skill 自己的目录下，除非确认通用。
- **references/** — 只放跨 skill 共享的参考资料。skill 专属参考资料放在 skill 自己的 references/ 下。
- **scripts/** — 只放仓库级辅助脚本（如安装脚本、校验脚本）。

## 新增 Skill 前的判断

新增 skill 前回答以下问题：

1. **复用性** — 这个能力在 3 个以上不同项目中会用到吗？
2. **独立性** — 这个能力可以独立于特定项目运行吗？
3. **可执行性** — 能否定义明确的输入、输出和执行步骤？
4. **边界清晰** — 能否明确说明这个 skill 不做什么？
5. **不重叠** — 与已有 skill 没有功能重叠？

5 个问题都回答"是"才值得新增。

## 与已有 Skill 的关系

| 本仓库名称 | 职责边界 |
|------------|----------|
| knowledge-consolidation | 只判断"什么值得沉淀"和"沉淀到哪里"，不做实际写入 |
| project-doc-generator | 只生成文档，不做文档治理（缺口分析、质量审计） |
| agents-md-maintainer | 只维护 AGENTS.md，不管理 memory 或项目文档 |
| memory-governance | 只管理 memory 文件，不做知识沉淀判断或 AGENTS.md 管理 |
| hooks-designer | 只设计 hooks 配置和策略，不做运行时实现 |
| project-docs-governance | 企业级文档治理，包含完整 references 和 templates |
| code-review-checklist | 只生成 review 检查清单，不做实际代码 review 或自动化检查 |
| error-pattern-library | 只积累和检索错误模式，不做实时监控或自动修复 |
| api-design-reviewer | 只审查 API 设计，不做 API 文档生成或代码 review |
| refactoring-planner | 只生成重构计划，不做实际代码修改或代码 review |
| dependency-analyzer | 只分析依赖关系，不做自动升级或安全 exploit 分析 |
| test-strategy-designer | 只设计测试策略，不做具体测试用例编写或测试执行 |
| commit-message-craftsman | 只生成 commit message，不做自动提交或 changelog 生成 |
| onboarding-guide-generator | 只生成上手指南，不做项目文档体系或架构文档 |
