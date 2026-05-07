---
name: agents-md-maintainer
description: "Maintain AGENTS.md as the stable, repo-level rule file for AI agents. Use when (1) adding or updating project constraints, build commands, test commands, or collaboration rules, (2) auditing AGENTS.md for stale or misplaced content, (3) ensuring only durable rules live in AGENTS.md while ephemeral context stays elsewhere. Prevents polluting AGENTS.md with temporary session context."
---

# AGENTS.md 维护 Skill

识别仓库级长期规则，维护 AGENTS.md 的结构、规范、约束、测试命令、协作原则。避免将临时上下文写入仓库级规则。

## 何时使用

- 项目初始化时创建 AGENTS.md
- 发现新的项目约束、构建规则、测试命令需要记录
- 审计 AGENTS.md 是否有过时或错位的内容
- 团队规范发生变化需要更新 AGENTS.md
- 有人误将临时上下文写入了 AGENTS.md，需要清理

**不适用**：项目文档体系（README、设计文档等）、个人记忆、一次性笔记。

## 输入

- 现有 AGENTS.md 内容（若存在）
- 项目根目录结构（用于推断技术栈和约束）
- 用户要求新增或修改的规则内容
- 项目配置文件（package.json, Makefile, pyproject.toml 等，可选）

## 输出

- 更新后的 AGENTS.md 完整内容
- 变更说明：新增了什么、修改了什么、删除了什么、为什么
- 被拒绝写入的内容及原因（临时性、不适用等）

## AGENTS.md 标准结构

```markdown
# 项目规则

## 项目概述
（一两句话说明项目是什么、做什么）

## 技术栈
- 语言 / 框架 / 主要依赖

## 构建与运行
- 安装命令
- 构建/编译命令
- 启动命令

## 测试
- 运行测试的命令
- 测试约定（文件命名、目录结构）

## 代码规范
- 命名约定
- 目录结构约定
- import / module 约定

## 协作原则
- 分支策略
- 提交消息格式
- PR 审查要求

## 约束与注意事项
- 已知限制
- 环境要求
- 安全相关注意事项
```

## 执行步骤

### 1. 审计现有 AGENTS.md

若 AGENTS.md 已存在：

- 逐项检查每条规则是否仍然有效
- 标记过时内容（构建命令变了、依赖升级了等）
- 标记错位内容（临时上下文、一次性结论、个人偏好）
- 检查结构是否完整（是否缺少标准结构中的必要章节）

### 2. 收集规则来源

从以下来源识别应写入 AGENTS.md 的规则：

| 来源 | 提取什么 |
|------|----------|
| 项目配置文件 | 构建命令、测试命令、lint 规则 |
| 代码结构 | 目录约定、模块组织 |
| 用户输入 | 团队约定的规范、约束 |
| 踩坑记录 | 已知限制、注意事项 |
| CI/CD 配置 | 构建流程、部署约束 |

### 3. 过滤：什么不该写入

```
以下内容不应写入 AGENTS.md:
- 一次性结论（"这个 bug 是因为 X 导致的"）
- 未验证的推测
- 个人偏好（"我喜欢用 vim"）
- 临时工作状态（"正在重构模块 A"）
- 可从代码推导的信息（不需要重复写函数签名）
- 与项目规则无关的上下文
```

### 4. 更新 AGENTS.md

- 按标准结构组织内容
- 新增内容放在对应章节
- 修改已有内容时保留原始意图
- 删除过时内容时在变更说明中记录原因
- 确保每条规则都是可执行的、有明确含义的

### 5. 输出变更报告

```markdown
## AGENTS.md 变更报告

### 新增
- [构建与运行] 添加 `pnpm build` 作为构建命令
- [约束] 添加"生产环境必须使用 Node 18+"

### 修改
- [测试] 从 `npm test` 更新为 `pnpm test --coverage`

### 删除
- [约束] 移除"暂不支持 TypeScript strict mode"（已在 v2.0 启用）

### 拒绝写入
- "当前正在迁移数据库" → 临时工作状态，不属于长期规则
- "建议使用 iTerm2" → 个人偏好，不属于项目规则
```

## 边界与非目标

- **不做**知识沉淀判断 — 这是 knowledge-consolidation 的职责
- **不做**memory 管理 — 这是 memory-governance 的职责
- **不做**项目文档生成 — 这是 project-doc-generator 的职责
- **不做**hooks 设计 — 这是 hooks-designer 的职责
- 只维护 AGENTS.md，不维护其他配置文件

## 验收标准

- AGENTS.md 中每条规则都是长期有效的、可执行的
- 没有临时上下文或一次性结论混入
- 结构清晰，按标准章节组织
- 构建和测试命令可直接复制执行
- 变更报告清晰说明了每项改动的原因
