---
name: knowledge-consolidation
description: "Review completed work and consolidate learnings into long-term knowledge artifacts. Use when (1) a task or session is finishing and you want to capture durable insights, (2) deciding what belongs in AGENTS.md vs memory vs project docs, (3) surfacing reusable patterns from one-off work, (4) generating concrete update patches for knowledge files. Outputs prioritized suggestions with exact file locations and diffs."
---

# 知识沉淀 Skill

在任务完成后回顾本轮工作，判断哪些内容应沉淀到 AGENTS.md、skills、memory、知识体系文档，输出具体的更新建议和补丁。

## 何时使用

- 任务完成、会话即将结束时
- 刚解决了一个复杂 bug 或完成了一个重要功能
- 发现了新的项目约束、团队规范、架构决策
- 用户主动要求"总结一下这次工作"或"把这些记下来"

**不适用**：纯对话问答、未完成的任务、临时性探索。

## 输入

- 本轮对话的完整上下文
- 项目中已有的 AGENTS.md（若存在）
- 项目中已有的 memory 文件（若存在）
- 用户明确要求沉淀的目标文件（可选）

## 输出

一份结构化的沉淀报告，包含：

1. **沉淀候选项列表** — 每项标注来源、类型、推荐落点
2. **推荐落点判断** — AGENTS.md / memory / skill / 项目文档 / 不沉淀
3. **具体改动内容** — 每个落点的具体新增/修改文本
4. **优先级排序** — 高（长期复用价值）/ 中（可能复用）/ 低（仅供参考）
5. **最终补丁** — 可直接应用的 diff 或完整文本块

## 执行步骤

### 1. 回顾本轮工作

- 梳理本轮对话中完成的所有任务
- 识别关键决策、踩过的坑、发现的约束、验证过的假设
- 标记哪些是事实（已验证）、哪些是推测（未验证）

### 2. 判断沉淀价值

对每个候选知识点，依次判断：

| 判断维度 | 问题 |
|----------|------|
| 复用性 | 未来还会遇到吗？ |
| 稳定性 | 这个结论会很快过时吗？ |
| 通用性 | 只对当前项目有效，还是跨项目有效？ |
| 验证度 | 已验证还是推测？ |
| 唯一性 | 已有知识体系中是否已覆盖？ |

### 3. 确定落点

```
IF 跨项目通用 AND 稳定 → 写入用户的 memory 或全局 AGENTS.md
IF 项目级规则 AND 稳定 → 写入项目 AGENTS.md
IF 与特定能力相关 AND 可复用 → 提炼为 skill 的一部分
IF 与项目文档体系相关 → 写入对应项目文档
IF 一次性结论 OR 未验证 OR 短期有效 → 不沉淀，仅在对话中说明
```

### 4. 生成具体改动

- 对每个推荐沉淀项，给出目标文件路径
- 给出要插入/修改的具体文本
- 标注插入位置（文件末尾、特定章节之后等）
- 若有冲突与已有内容，明确说明如何合并

### 5. 输出沉淀报告

格式示例：

```markdown
## 沉淀报告

### 候选项 1: [标题]
- 来源: 本轮调试 X 问题时的发现
- 类型: 项目约束
- 落点: AGENTS.md → 构建约束章节
- 优先级: 高
- 改动:
  文件: AGENTS.md
  位置: "构建约束" 章节末尾
  内容: |
    - 生产环境构建必须使用 Node 18+，低于此版本会出现 Buffer.alloc 不可用错误

### 候选项 2: ...
```

## 边界与非目标

- **不做**自动写入 — 只提供建议和补丁，由用户确认后执行
- **不做**文档体系治理 — 这是 project-doc-generator 的职责
- **不做**memory 治理（去重、过期清理）— 这是 memory-governance 的职责
- **不做**AGENTS.md 结构维护 — 这是 agents-md-maintainer 的职责
- 只关注"本轮工作产出了什么值得沉淀的内容"，不负责历史知识审计

## 验收标准

- 沉淀报告中每一项都有明确的落点和具体改动文本
- 没有将临时性、未验证内容标记为"高优先级沉淀"
- 至少识别出 1 个值得沉淀的知识点（如果本轮工作有实质内容）
- 生成的补丁文本可以直接复制到目标文件中生效
