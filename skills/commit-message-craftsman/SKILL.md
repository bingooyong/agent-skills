---
name: commit-message-craftsman
description: "Generate well-structured commit messages from staged changes. Use when (1) crafting commit messages that follow Conventional Commits or team-specific formats, (2) summarizing complex multi-file changes, (3) ensuring commit history is searchable, (4) splitting large changes into logical commits. Outputs commit messages with type, scope, and body based on actual diffs."
---

# Commit Message 工匠 Skill

基于暂存区变更内容生成规范的 commit message，支持 Conventional Commits 等多种格式，帮助保持提交历史清晰可检索。

## 何时使用

- 准备提交代码，需要生成规范的 commit message
- 团队要求统一提交格式
- 复杂变更需要拆分为多个逻辑提交
- 审视暂存区变更，确认提交内容的合理性

**不适用**：与 Git 无关的文本生成、changelog 生成、release notes。

## Commit Message 格式规范

| 格式名称 | 结构 | 适用场景 |
|----------|------|----------|
| Conventional Commits | `type: description` | 通用，支持自动 changelog |
| Conventional + Scope | `type(scope): description` | 多模块项目 |
| Merge Commit | `Merge branch/PR #N` | 合并分支 |
| Revert Commit | `Revert "original message"` | 回退提交 |

Conventional Commits 类型：

| type | 说明 |
|------|------|
| feat | 新功能 |
| fix | 修复 bug |
| docs | 仅文档变更 |
| style | 格式化（不影响逻辑） |
| refactor | 重构（不新增功能、不修复 bug） |
| perf | 性能优化 |
| test | 测试相关 |
| build | 构建系统或外部依赖 |
| ci | CI 配置 |
| chore | 其他不修改 src 或 test 的变更 |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| 暂存区 diff 内容 | `git diff --cached` 的输出 | 是 |
| 项目 commit 历史风格 | 最近几条 commit message | 否（默认 Conventional Commits） |
| 目标格式 | 指定 commit message 格式 | 否 |
| 是否需要拆分 | 大变更是否拆为多个 commit | 否（默认判断） |

## 输出

- 规范化的 commit message（含 type / scope / body / footer）
- 拆分建议（如适用）
- 格式说明

## 执行步骤

### 1. 分析暂存区变更

- 统计变更文件列表和变更类型（新增/修改/删除）
- 识别变更涉及的模块（从文件路径前缀推断）
- 提取关键变更内容（新增了什么、修改了什么、删除了什么）
- 判断变更的复杂度（单模块简单 / 多模块复杂）

### 2. 推断变更类型和范围

```
推断变更类型 (type):
  IF 新增了文件 AND 无删除 AND 非测试非配置 → type: feat
  IF 修改了 bug 相关代码 OR 文件名含 fix → type: fix
  IF 只修改了文档文件 (.md, .txt, .rst) → type: docs
  IF 只修改了格式/空格/import 排序 → type: style
  IF 修改了构建配置/CI/依赖配置 → type: build 或 chore
  IF 只修改了测试文件 → type: test
  IF 重构了代码且未改变外部行为 → type: refactor
  IF 修改了性能相关逻辑 → type: perf
  IF 修改了 CI 配置文件 → type: ci

推断范围 (scope):
  从变更文件的公共路径前缀提取
  例: src/auth/login.ts + src/auth/logout.ts → scope: auth
  例: 仅修改根目录配置 → 无 scope
```

### 3. 生成 commit message

```markdown
type(scope): 简短描述（不超过 50 字符）

详细说明变更的内容和原因。每行不超过 72 字符。
解释为什么做这个变更，而非做了什么。

如果有关联的 issue:
Refs #123
```

生成原则：
- 标题行使用祈使语气（"add feature" 而非 "added feature"）
- 标题行不超过 50 字符
- 正文每行不超过 72 字符
- 正文说明"为什么"而非"做了什么"（做什么已经在 diff 中）
- 不在 message 中重复 diff 内容

### 4. 拆分建议（可选）

```
判断是否需要拆分:
  IF 变更涉及 3+ 不相关模块 → 建议按模块拆分为多个 commit
  IF 变更同时包含 feat 和 fix → 建议按类型拆分
  IF 变更同时包含代码和文档更新 → 建议拆分
  IF 单个文件变更超过 500 行 → 建议拆分

拆分方案:
  Commit 1: type(scope-a): 变更描述 A
  Commit 2: type(scope-b): 变更描述 B
  Commit 3: docs: 更新相关文档
```

## 边界与非目标

- **不做**自动执行 git commit — 只生成 message，提交由用户执行
- **不做**changelog 或 release notes 生成
- **不做**代码 review — 这是 code-review-checklist 的职责
- **不做**分支管理策略
- 只生成 commit message，不改变暂存区内容
- 不强制要求用户使用生成的 message

## 验收标准

- 生成的 commit message 符合 Conventional Commits 格式
- type 推断与实际变更内容匹配
- scope 从文件路径正确推断（多模块项目）
- 标题行不超过 50 字符，正文不超过 72 字符
- 拆分建议只在变更确实跨模块时才提出
