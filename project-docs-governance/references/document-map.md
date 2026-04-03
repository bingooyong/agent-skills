# 文档地图与体系清单

## 1. 文档索引（文档地图）用途

- 列出项目全部正式文档、存放路径、目标读者、状态、与上下游关系。
- 作为「应有文档」的检查清单，用于缺口分析与发布前校验。
- 建议放在 `docs/README.md` 或 `docs/INDEX.md`，并在项目主 README 中链接。

## 2. 文档类型清单（按角色与生命周期）

### 2.1 通用必选（多数项目）

| 文档 | 典型路径 | 主要读者 | 依赖上游 | 备注 |
|------|----------|----------|----------|------|
| README | 仓库根 README.md | 所有人 | — | 入口，简要说明、快速开始、文档索引链接 |
| 项目概述/项目章程 | docs/project-overview.md | 产品、PM、架构 | — | 目标、范围、成功准则、约束 |
| 文档索引/文档地图 | docs/INDEX.md 或 docs/README.md | 所有人 | — | 本清单的实例化 |
| 术语表 | docs/glossary.md | 全员 | — | 统一术语，被多文档引用 |
| 系统架构设计 | docs/architecture.md | 架构、开发、运维 | 项目概述、技术选型 | 组件、边界、数据流、部署拓扑 |
| 技术选型说明 | docs/tech-stack.md | 架构、开发 | 项目概述 | 选型理由、版本、替代方案 |
| 详细设计文档 | docs/design.md 或按模块拆分 | 开发 | 架构、API 约定 | 模块、接口、数据流、关键逻辑 |
| 环境说明 | docs/environments.md | 开发、运维 | — | 开发/测试/预发/生产差异 |
| 配置说明 | docs/configuration.md | 开发、运维 | 环境说明 | 配置项、默认值、安全注意 |
| 部署手册 | docs/deployment.md 或 docs/Deployment.md | 运维、发布 | 环境、配置、架构 | 可执行步骤、回滚 |
| 操作/运维手册 | docs/operations.md | 运维 | 部署、监控 | 日常操作、扩缩容、备份 |
| 接口/API 手册 | docs/api/ 或 OpenAPI | 开发、测试、对接方 | 详细设计 | 请求/响应、错误码、示例 |
| 数据库设计/数据字典 | docs/database.md 或 docs/schema/ | 开发、DBA | 详细设计 | 表结构、索引、迁移约定 |
| 测试计划 | docs/testing-plan.md | 测试、PM | 需求/PRD | 范围、策略、资源 |
| 验收标准/UAT | docs/acceptance.md | 产品、测试、客户 | 需求 | 可验证条件 |
| 运维手册 | 见「操作/运维」 | 运维 | — | 可与 operations 合并 |
| 监控与告警 | docs/monitoring.md | 运维 | 部署、架构 | 指标、告警规则、值班 |
| 故障处理/Runbook | docs/runbooks/ | 运维、值班 | 监控、部署 | 按故障类型分篇 |
| 安全设计说明 | docs/security.md | 架构、安全、审计 | 架构 | 威胁模型、控制措施 |
| 权限模型 | docs/access-control.md | 开发、安全 | 安全设计 | 角色、权限、鉴权方式 |
| 发布说明/Release Notes | docs/releases/ 或 CHANGELOG | 全员、客户 | — | 版本、变更、升级注意 |
| 变更记录规范 | docs/changelog-policy.md | 开发、PM | — | 如何写 CHANGELOG/Release Notes |
| 风险与问题清单 | docs/risks-issues.md | PM、架构 | — | 可选，跟踪风险与已知问题 |
| ADR（架构决策记录） | docs/adr/ | 架构、开发 | — | 每则决策一篇 |
| FAQ | docs/faq.md | 全员、支持 | — | 常见问题与答案 |
| Onboarding | docs/onboarding.md | 新成员 | 文档索引、环境、配置 | 环境搭建、权限、首次运行 |
| 项目文档索引 | 同「文档地图」 | — | — | — |

### 2.2 可选但推荐

- 数据模型/数据字典（若与数据库设计分开）
- 测试用例设计说明（测试计划下游）
- 配置说明（若与环境说明合并则不必单列）
- 对外交付物清单（客户/合规）

### 2.3 AI 项目专项（适用时补充）

| 文档 | 典型路径 | 主要读者 | 备注 |
|------|----------|----------|------|
| 模型与推理架构 | docs/ai/architecture.md | 架构、算法、运维 | 模型选型、推理服务、配额 |
| Prompt/System Prompt 设计 | docs/ai/prompts.md | 算法、产品、安全 | 提示词版本、评审、敏感信息 |
| Tool/MCP/外部依赖 | docs/ai/tools.md | 开发、运维 | 工具列表、权限、限流 |
| Evaluation/Benchmark | docs/ai/evaluation.md | 算法、质量 | 指标、数据集、回归 |
| Guardrails/安全策略 | docs/ai/guardrails.md | 安全、产品 | 内容过滤、滥用防护 |
| Token/Cost 管理 | docs/ai/costs.md | 架构、运维、成本 | 用量、预算、优化 |
| 记忆/上下文管理 | docs/ai/context-memory.md | 架构、开发 | 上下文窗口、Memory ROOT 等 |

## 3. 文档依赖与同步策略

- **上游优先**：生成或更新某文档前，先确保其上游文档（见上表）已存在且可引用；若上游缺失，在本文档中标记「待补充：见 [上游文档]」。
- **交叉引用**：使用相对路径或稳定文档 ID，例如 `[架构设计](architecture.md)`、`[术语表](glossary.md)#术语A`。
- **同步触发**：架构变更 → 更新架构、详细设计、部署/运维中受影响部分；接口变更 → 更新 API 文档、调用方说明；发布 → 更新 Release Notes、部署手册中的版本号；事故/复盘 → 更新 Runbook、监控、故障处理。
- **术语统一**：术语表为权威；生成或更新任何文档时优先使用术语表中的表述，新增术语先入术语表再在正文使用。

## 4. 文档状态与版本

- **状态**：draft（草稿）→ reviewed（已评审）→ approved（已批准）→ deprecated（已废弃）。
- **版本**：建议语义化或日期版本（如 v1.0、2024-03-01）；与 Release Notes 对应时保持一致。
- 在文档 frontmatter 或文首元数据中注明：版本、状态、作者/维护者、最后更新、变更摘要。

## 5. 对外 vs 对内

- **对外**：客户、合作方、合规/审计；注意脱敏、版本号、发布说明、合规用语；不暴露内部环境、账号、预案细节。
- **对内**：开发、测试、运维；可含内部路径、环境、账号约定、故障预案、成本与限流策略。
- 若同一主题需同时对外与对内，可拆成两篇（如 `deployment.md` 对外摘要 + `deployment-internal.md` 完整步骤），或在同一文档中用「对外/对内」章节区分。
