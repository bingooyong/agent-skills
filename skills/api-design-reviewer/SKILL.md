---
name: api-design-reviewer
description: "Review API designs for consistency, backward compatibility, security, and extensibility. Use when (1) designing new API endpoints or interfaces, (2) reviewing existing API changes for breaking changes, (3) establishing API design standards, (4) auditing API surface for security and evolution readiness. Outputs structured review findings with severity and recommendations."
---

# API 设计审查 Skill

审查 API 设计的一致性、向后兼容性、安全性和可扩展性，确保 API 演进不破坏现有使用者。

## 何时使用

- 设计新的 API 接口，需要系统化审查
- 评审 API 变更是否引入 breaking change
- 制定团队 API 设计规范
- 审计现有 API 的一致性和安全性

**不适用**：API 文档生成、代码层面的 review、UI/UX 设计评审、API 性能压测。

## API 审查维度

| 维度 | 审查要点 | 常见问题 |
|------|----------|----------|
| 命名一致性 | 资源命名、动词选择、大小写风格 | 混用 camelCase/snake_case、名词单复数不一致 |
| 参数设计 | 必填/可选、默认值、参数类型、分页方式 | 过多必填参数、缺乏分页、body 和 query 混用 |
| 错误处理 | 错误码体系、错误信息结构、HTTP 状态码 | 状态码语义错误、错误信息泄露内部实现 |
| 版本策略 | 版本位置（URL/Header）、版本演进规则 | 无版本控制、版本跳级、多版本共存策略缺失 |
| 鉴权授权 | 认证方式、权限粒度、scope 设计 | 过度授权、缺少 scope 校验、token 未校验 |
| 分页过滤 | 分页参数、过滤语法、排序方式 | 无上限分页、过滤注入、排序不稳定 |
| 向后兼容 | 字段增删、类型变更、语义变更 | 删除字段不通知、类型隐式转换、枚举值重命名 |
| 幂等性 | 重复请求的处理、幂等键设计 | 创建接口不幂等、状态变更无幂等保护 |
| 限流 | 频率限制、配额管理、超限响应 | 无限流、限流响应不符合标准 |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| API 定义 | OpenAPI / Swagger / GraphQL Schema / 代码接口 | 是 |
| 变更 diff | 增量审查时的前后对比 | 否（全量审查时不需要） |
| 已有 API 规范 | 团队的 API 设计规范文档 | 否 |
| 目标 API 风格 | REST / GraphQL / gRPC | 否（从定义推断） |

## 输出

- 审查发现列表，按严重性分类（breaking / 重要 / 建议）
- 一致性评分（命名、错误处理、分页等维度的达标率）
- Breaking change 分析（受影响的端点和消费者）
- 安全风险项
- 改进建议（按优先级排序）

## 执行步骤

### 1. 解析 API 定义

- 从输入格式（OpenAPI / Schema / 代码）中提取所有端点
- 提取每个端点的：路径、方法、参数、请求体、响应体、错误码
- 识别认证方式和通用配置
- 构建端点清单和依赖关系

### 2. 一致性检查

逐项检查所有端点是否符合统一规范：

```
命名一致性:
  IF 同一 API 中混用 camelCase 和 snake_case → 标记不一致
  IF 资源路径使用动词（/getUser）→ 标记不规范，建议用名词（/users/{id}）
  IF 同一资源的列表/详情使用不同路径前缀 → 标记不一致

参数一致性:
  IF 分页参数不统一（page/offset 混用）→ 标记不一致
  IF 日期参数格式不统一 → 标记不一致
  IF ID 参数类型不统一（string vs integer）→ 标记不一致

错误处理一致性:
  IF 错误响应结构不统一 → 标记不一致
  IF 同类错误使用不同 HTTP 状态码 → 标记不一致
```

### 3. 向后兼容性分析

```
检查向后兼容性:
  IF 删除了字段/参数 → 标记 breaking, 严重性: 高
  IF 修改了字段类型 → 标记 breaking, 严重性: 高
  IF 新增了必需参数 → 标记 breaking, 严重性: 中
  IF 修改了错误码语义 → 标记 breaking, 严重性: 中
  IF 缩小了参数取值范围 → 标记 breaking, 严重性: 中
  IF 新增了可选参数 → 标记兼容, 建议设默认值
  IF 新增了端点/字段 → 标记兼容, 无影响
  IF 废弃了端点 → 标记兼容, 建议提供迁移路径和截止日期
```

### 4. 安全性审查

```
安全检查:
  IF 端点无认证要求 AND 非 public API → 标记风险
  IF 接受用户输入拼接到查询/命令 → 标记注入风险
  IF 返回敏感字段（密码/密钥/token）→ 标记泄露风险
  IF 缺少限流配置 → 标记滥用风险
  IF 使用 HTTP 传输敏感数据 → 标记传输风险
  IF 权限校验仅在特定端点存在 → 标记遗漏风险
```

### 5. 输出审查报告

```markdown
## API 审查报告

### 摘要
- 审查端点数: N
- Breaking changes: X (高严重性)
- 一致性问题: Y (中严重性)
- 安全风险: Z (高严重性)
- 改进建议: W (低严重性)

### Breaking Changes (严重性: 高)
- **BC-001**: `DELETE /api/v1/users/{id}` 返回值从 `{success: bool}` 变为 `{data: null}`
  - 影响: 所有消费此端点的客户端
  - 建议: 保持原返回结构或提供过渡期

### 一致性问题 (严重性: 中)
- **CON-001**: 分页参数不统一，部分使用 `page/size`，部分使用 `offset/limit`

### 安全风险 (严重性: 高)
- **SEC-001**: `POST /api/users` 无认证要求

### 改进建议 (严重性: 低)
- **SUG-001**: 建议为所有列表接口添加 `Link` header 实现分页
```

## 边界与非目标

- **不做**API 文档生成 — 这是 project-doc-generator 的职责
- **不做**代码层面的 review — 这是 code-review-checklist 的职责
- **不做**测试策略设计 — 这是 test-strategy-designer 的职责
- **不做**重构建议 — 这是 refactoring-planner 的职责
- 只审查 API 设计，不负责实现细节和性能优化

## 验收标准

- 每个 breaking change 都标注了影响的端点和消费者
- 一致性检查覆盖了所有审查维度
- 安全审查没有遗漏无认证端点
- 审查报告按严重性分类，高优先级问题排在前面
- 每个发现都有明确的改进建议
