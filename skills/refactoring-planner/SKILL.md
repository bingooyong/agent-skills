---
name: refactoring-planner
description: "Analyze code smells, generate refactoring plans with impact assessment and risk evaluation. Use when (1) identifying code smells and technical debt hotspots, (2) planning safe refactoring sequences with dependency awareness, (3) estimating refactoring scope and risk, (4) creating step-by-step refactoring checklists. Outputs prioritized refactoring plans with rollback strategies."
---

# 重构计划 Skill

分析代码坏味道和技术债，生成按优先级排序的重构计划，评估影响范围和风险，确保重构安全可回退。

## 何时使用

- 发现代码重复、过长函数、过深嵌套等坏味道，需要系统化处理
- 需要规划技术债偿还顺序
- 准备大规模重构前，需要评估影响和风险
- 生成重构 checklist 供团队执行

**不适用**：实时性能优化、从零开始的架构设计、纯 cosmetic 的代码格式化。

## 代码坏味道分类

| 坏味道 | 严重性指标 | 典型表现 | 推荐重构手法 |
|--------|------------|----------|--------------|
| 重复代码 | 同一代码块出现 2+ 次 | 复制粘贴的逻辑、相似的 if-else 分支 | Extract Method / Extract Class / 模板方法 |
| 过长函数 | 函数超过 50 行 | 做太多事的函数、难以理解的逻辑块 | Extract Method / Replace Temp with Query |
| 过大类 | 类超过 300 行或方法超过 15 个 | 承担多个职责的"上帝类" | Extract Class / Move Method |
| 过深嵌套 | 嵌套层级超过 3 | if-for-if-for 套娃、箭头型代码 | Guard Clauses / Extract Method / 早返回 |
| 发散变化 | 一个类因 N 种不同原因修改 | 修改时总需要改同一个文件 | Extract Class / 按职责拆分 |
| 霰弹修改 | 一个变更需要修改 N 个文件 | 新功能需要改 5+ 个文件 | Move Method / 合并相关逻辑 |
| 特性嫉妒 | 频繁访问其他类的数据 | 大量 getter 调用链 | Move Method / Extract Method |
| 过度耦合 | 模块间依赖超过 3 层 | 改一个模块影响一串模块 | 引入接口 / Dependency Injection |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| 目标代码路径或范围 | 要分析的文件/目录/模块 | 是 |
| 已知问题清单 | 团队已知的痛点 | 否 |
| 重构目标 | 可维护性 / 性能 / 可测试性 | 否（默认可维护性） |
| 约束条件 | 不可修改的接口、冻结的模块等 | 否 |

## 输出

- 坏味道分析报告（按严重性排序）
- 重构计划（P0/P1/P2 分级）
- 影响范围评估
- 每步的验证条件
- 回退策略

## 执行步骤

### 1. 识别代码坏味道

```
FOR each 目标文件/模块:
  IF 函数超过 50 行 → 标记"过长函数", 严重性: 中
  IF 类超过 300 行 OR 方法超过 15 个 → 标记"过大类", 严重性: 中
  IF 嵌套层级超过 3 → 标记"过深嵌套", 严重性: 高
  IF 重复代码块超过 6 行 AND 出现 2+ 次 → 标记"重复代码", 严重性: 高
  IF 文件修改原因超过 3 种（从 git history 推断）→ 标记"发散变化", 严重性: 中
  IF 函数频繁访问其他类的属性 → 标记"特性嫉妒", 严重性: 中
  IF 模块间依赖超过 3 层 → 标记"过度耦合", 严重性: 高
```

### 2. 评估影响范围

对每个识别到的坏味道：

- 统计受影响的文件数和函数数
- 识别依赖此代码的上游调用方
- 识别此代码依赖的下游模块
- 评估修复所需的最小变更范围

```
影响评估:
  受影响文件数: N
  受影响函数数: M
  上游调用方: [列表]
  最小变更范围: [文件列表]
```

### 3. 确定重构优先级

```
优先级排序:
  P0 (高收益低风险):
    - 严重性高 AND 影响范围小（< 3 文件）
    - 重复代码提取
    - 过深嵌套改为 guard clause

  P1 (高收益中风险):
    - 严重性高 AND 影响范围中等（3-10 文件）
    - 过大类拆分
    - 发散变化重构

  P2 (中等收益):
    - 严重性中 AND 影响范围可控
    - 过长函数提取
    - 特性嫉妒修复

  建议延后:
    - 影响范围超过 10 文件
    - 涉及跨团队接口
    - 当前无测试覆盖的区域
```

### 4. 制定重构计划

```markdown
## 重构计划

### P0 — 高收益低风险
1. **提取重复的参数校验逻辑**
   - 目标文件: src/utils/validate.ts
   - 影响范围: 2 个文件
   - 重构手法: Extract Method
   - 验证条件: 现有测试全部通过
   - 预估时间: 30 分钟

### P1 — 高收益中风险
2. **拆分 UserService 的权限逻辑**
   - 目标文件: src/services/UserService.ts
   - 影响范围: 5 个文件
   - 重构手法: Extract Class
   - 验证条件: 现有测试通过 + 新增权限测试用例
   - 预估时间: 2 小时
```

### 5. 设计回退策略

```
回退策略:
  1. 每步重构前确保 Git 工作区干净
  2. 每步完成后运行完整测试套件
  3. 测试失败时: git revert 到上一步的提交
  4. 大范围重构: 先创建 feature branch，完成后合并
  5. 重构过程中不混合功能变更
```

## 边界与非目标

- **不做**实际代码修改 — 只生成重构计划，修改由用户执行
- **不做**代码 review — 这是 code-review-checklist 的职责
- **不做**API 设计审查 — 这是 api-design-reviewer 的职责
- **不做**测试策略设计 — 这是 test-strategy-designer 的职责
- **不做**依赖分析 — 这是 dependency-analyzer 的职责
- **不做**错误模式管理 — 这是 error-pattern-library 的职责
- 只规划重构，不涉及新功能设计

## 验收标准

- 每个识别的坏味道都有严重性评估和影响范围
- 重构计划按 P0/P1/P2 分级，P0 可以立即执行
- 每步重构有明确的验证条件（通常是测试通过）
- 回退策略具体可执行，不是笼统的"回退"
- 没有将高风险大范围重构标记为 P0
