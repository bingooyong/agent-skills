---
name: code-review-checklist
description: "Generate structured code review checklists tailored by language, framework, and change type. Use when (1) preparing for a code review and needing a systematic checklist, (2) reviewing PRs for specific concern areas like security, performance, or maintainability, (3) establishing team-level review standards. Outputs categorized checklist items with severity and rationale."
---

# Code Review 检查清单 Skill

根据 PR 变更内容，生成按关注维度分类的结构化 review 检查清单，帮助 review 者系统性检查而非凭感觉。

## 何时使用

- 准备 review 一个 PR，需要系统化的检查清单
- 团队需要统一的 review 标准，避免遗漏关键检查项
- 新人需要 review 参考指南
- 变更涉及安全/性能等高风险领域，需要专项检查

**不适用**：自动化 lint/CI 检查配置、与 code review 无关的代码分析。

## 检查清单维度分类

| 维度 | 关注点 | 默认严重性 |
|------|--------|------------|
| 正确性 | 逻辑正确、边界条件、空值处理、并发安全 | 高 |
| 安全性 | 输入校验、注入防护、鉴权、敏感数据处理 | 高 |
| 性能 | 查询效率、内存使用、缓存策略、N+1 问题 | 中 |
| 可维护性 | 命名清晰、函数长度、职责单一、复杂度 | 中 |
| 可测试性 | 依赖可注入、副作用隔离、可 mock | 中 |
| 向后兼容 | 接口签名、返回结构、枚举值、序列化格式 | 高 |
| 文档与注释 | 公共接口文档、复杂逻辑注释、变更说明 | 低 |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| 变更的 diff 内容 | PR 的完整 diff 或关键变更 | 是 |
| 项目技术栈 | 语言、框架、主要依赖 | 是 |
| 变更类型 | feature / bugfix / refactor / chore | 否 |
| 团队 review 规范 | 团队特有的检查要求 | 否 |

## 输出

- 按维度分类的检查清单，每项含检查要点和判断依据
- 按严重性排序的条目
- 语言/框架专项检查项
- 跳过项说明（本次变更不涉及的维度及原因）

## 执行步骤

### 1. 分析变更类型与范围

- 统计变更文件数、新增/删除/修改行数
- 识别变更涉及的模块和层次（UI / API / 数据层 / 基础设施）
- 判断变更类型：新功能 / 修复 / 重构 / 配置 / 文档
- 标记高风险区域：涉及认证、支付、数据迁移、外部集成

### 2. 匹配检查维度

```
IF 变更涉及数据流或状态变更 → 激活"正确性"维度
IF 变更涉及输入处理或鉴权 → 激活"安全性"维度
IF 变更涉及循环/查询/缓存 → 激活"性能"维度
IF 变更涉及接口签名 → 激活"向后兼容"维度
IF 变更超过 200 行 OR 涉及 3+ 文件 → 激活"可维护性"维度
IF 变更涉及外部依赖 OR 新增模块 → 激活"可测试性"维度
IF 变更涉及公共接口 OR 配置项 → 激活"文档与注释"维度
```

### 3. 语言框架专项补充

根据技术栈补充专项检查项：

| 技术栈 | 专项检查 |
|--------|----------|
| TypeScript | 类型安全、any 使用、枚举 vs 联合类型 |
| React | 重渲染、key 使用、useEffect 依赖、状态提升 |
| Go | goroutine 泄漏、error 处理、interface 滥用 |
| Python | 类型注解、异常层级、可变默认参数 |
| SQL | 索引使用、N+1 查询、事务范围、SQL 注入 |

### 4. 生成检查清单

```markdown
## Review 检查清单

### 正确性 (严重性: 高)
- [ ] **空值处理**: 检查新增路径上是否存在未处理的 null/undefined
  - 判断依据: 所有外部输入和 API 返回值是否做了空值检查
  - 常见问题: 解构时未处理 undefined、可选链使用不一致
- [ ] **边界条件**: 检查新增逻辑的边界输入
  - 判断依据: 空数组、零值、超大值、特殊字符是否正常处理

### 安全性 (严重性: 高)
- [ ] **输入校验**: 检查新增的输入处理路径
  - 判断依据: 是否使用校验库、是否白名单优于黑名单
  - 常见问题: 仅前端校验、正则拒绝服务
```

### 5. 输出结构化结果

- 完整检查清单（按严重性排序）
- 跳过项说明
- 建议关注的高风险区域
- 参考已有的 error-pattern-library 中相关模式

## 边界与非目标

- **不做**实际代码 review — 只生成检查清单，review 由人执行
- **不做**自动化检查执行 — 这是 lint/CI 的职责
- **不做**API 设计审查 — 这是 api-design-reviewer 的职责
- **不做**测试策略设计 — 这是 test-strategy-designer 的职责
- **不做**重构建议 — 这是 refactoring-planner 的职责
- **不做**错误模式管理 — 这是 error-pattern-library 的职责

## 验收标准

- 每个激活的维度都有具体的检查项，而非笼统提醒
- 检查项包含判断依据，review 者知道怎么判断通过/不通过
- 高严重性维度排在前面
- 跳过的维度有明确的跳过原因
- 语言框架专项检查项与项目实际技术栈匹配
