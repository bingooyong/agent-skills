---
name: dependency-analyzer
description: "Analyze dependency graphs to identify circular dependencies, outdated packages, security vulnerabilities, and coupling hotspots. Use when (1) auditing project dependency health, (2) investigating build failures caused by dependency conflicts, (3) planning dependency upgrades with impact assessment, (4) visualizing module coupling. Outputs structured dependency reports with actionable recommendations."
---

# 依赖分析 Skill

分析项目依赖关系图谱，识别循环依赖、过时依赖、安全风险和耦合热点，提供可操作的治理建议。

## 何时使用

- 定期审计项目依赖健康状况
- 构建失败时排查依赖冲突
- 规划依赖升级，需要评估影响和风险
- 分析模块间耦合度，指导架构优化

**不适用**：运行时性能分析、代码逻辑审查、非依赖相关的构建问题。

## 依赖问题分类

| 问题类型 | 严重性 | 检测方式 |
|----------|--------|----------|
| 循环依赖 | 高 | 有向图环路检测 |
| 安全漏洞 | 高 | 版本范围与已知 CVE 比对 |
| 严重过时（2+ major 落后） | 高 | 当前版本与最新版本比对 |
| 过时（1 major 落后） | 中 | 当前版本与最新版本比对 |
| 版本冲突 | 中 | 同一包的不同版本范围是否冲突 |
| 废弃依赖 | 低 | 依赖声明了但代码中未引用 |
| 幽灵依赖 | 低 | 代码中引用了但未声明 |
| 耦合热点 | 中 | 被依赖次数排序，top N 标记 |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| 项目路径 | 包含依赖配置文件的根目录 | 是 |
| 包管理器 | npm / yarn / pnpm / pip / poetry / go / maven 等 | 否（从文件推断） |
| 分析范围 | 全部 / 仅直接 / 仅开发 | 否（默认全部） |
| 关注维度 | 安全 / 过时 / 循环 / 耦合 | 否（默认全部） |

## 输出

- 依赖图谱摘要（总数、直接/传递比例）
- 按严重性分类的问题列表
- 升级建议（含影响评估和风险等级）
- 治理优先级排序
- 建议立即处理的安全漏洞

## 执行步骤

### 1. 解析依赖清单

```
识别包管理器:
  IF 存在 pnpm-lock.yaml → pnpm
  IF 存在 yarn.lock → yarn
  IF 存在 package-lock.json → npm
  IF 存在 pyproject.toml → poetry / pip
  IF 存在 go.sum → go modules
  IF 存在 pom.xml → maven

提取依赖信息:
  1. 解析依赖配置文件
  2. 提取直接依赖和版本范围
  3. 解析锁文件获取传递依赖和精确版本
  4. 记录开发依赖 vs 生产依赖
```

### 2. 构建依赖图谱

```
构建依赖图谱:
  1. 创建 {包名 → 依赖包名集合} 的有向图
  2. 标注每条边的版本范围
  3. 区分直接依赖和传递依赖
  4. 识别同一包的多版本共存情况
```

### 3. 检测问题模式

```
检测问题:
  IF 有向图中存在环 (A → B → C → A) → 标记"循环依赖", 严重性: 高
  IF 最新版本 > 当前版本 2+ major → 标记"严重过时", 严重性: 高
  IF 最新版本 > 当前版本 1 major → 标记"过时", 严重性: 中
  IF 已知 CVE 影响当前版本范围 → 标记"安全漏洞", 严重性: 高
  IF 同一包存在 2+ 版本 → 标记"版本冲突", 严重性: 中
  IF 依赖声明了但代码中未 import → 标记"废弃依赖", 严重性: 低
  IF 代码中 import 了但未声明 → 标记"幽灵依赖", 严重性: 低
  IF 包被 5+ 其他包依赖 → 标记"耦合热点", 严重性: 中
```

### 4. 评估升级影响

对每个建议升级的依赖：

```
影响评估:
  1. 识别哪些模块直接依赖此包
  2. 识别哪些传递依赖会受影响
  3. 检查升级是否引入 breaking change（查 changelog）
  4. 评估升级的连锁影响范围

风险分级:
  IF patch 升级 AND 无 breaking change → 风险: 低
  IF minor 升级 AND 遵循 semver → 风险: 低
  IF major 升级 → 风险: 中
  IF major 升级 AND 被依赖数 > 3 → 风险: 高
```

### 5. 输出分析报告

```markdown
## 依赖分析报告

### 摘要
- 总依赖数: N (直接: X, 传递: Y)
- 问题数: 高严重性 A / 中严重性 B / 低严重性 C

### 安全漏洞 (严重性: 高)
| 依赖 | 当前版本 | 漏洞编号 | 修复版本 | 影响范围 |
|------|----------|----------|----------|----------|

### 循环依赖 (严重性: 高)
```
A → B → C → A
```

### 过时依赖
| 依赖 | 当前版本 | 最新版本 | 落后版本 | 升级风险 |
|------|----------|----------|----------|----------|

### 升级建议（按优先级排序）
1. **[依赖名]** X.Y.Z → A.B.C — 修复 CVE-XXXX, 风险: 低
2. **[依赖名]** X.Y.Z → A.B.C — 消除循环依赖, 风险: 中
```

## 边界与非目标

- **不做**自动执行依赖升级 — 只提供分析和建议
- **不做**安全漏洞的深度 exploit 分析 — 只识别和标记
- **不做**代码重构 — 这是 refactoring-planner 的职责
- **不做**错误模式管理 — 这是 error-pattern-library 的职责
- **不做**测试策略设计 — 这是 test-strategy-designer 的职责
- 只分析依赖关系，不涉及业务逻辑层面的依赖

## 验收标准

- 所有安全漏洞都被识别并标注了修复版本
- 循环依赖被完整列出，包含环路路径
- 升级建议包含影响评估和风险分级
- 没有将低风险升级标记为高优先级
- 报告格式可以直接用于团队评审
