---
name: knowledge-card-generator
description: "Convert Markdown knowledge base files into structured JSON + YAML LLM Wiki card sets. Use when (1) transforming module/feature/process/rule docs into machine-readable cards, (2) building LLM-consumable knowledge bases from Markdown, (3) generating dependency-linked card sets for Codex or Vibecoding AI, (4) user mentions 'knowledge card', 'wiki card', 'card set', or 'MD to JSON/YAML'. Outputs dual-format card collections with cross-references and index definitions."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# 知识卡片生成 Skill

将 Markdown 知识库文件转换为结构化 JSON + YAML LLM Wiki 卡片集，供 AI Agent 直接消费。支持模块、功能、Phase、规则、实体、UI、流程等多类型卡片的自动识别与依赖关系构建。支持单文件和多文件批量处理，生成向量/元数据/图谱索引定义供下游系统消费。

## 快速上手

给定 Markdown 知识库文件和版本号，输出结构化卡片集和索引定义：

```
输入: Document_Review_Module_Knowledge.md + v1.0
输出: cards.json（AI 消费）+ cards.yaml（人工维护）+ index.json（索引定义）
```

每张卡片包含 7 个卡片类型之一、6 个 fields 字段、完整依赖关系和元数据。详见 [reference.md](reference.md) 中的完整 Schema 和示例。

## 何时使用

- 需要将 Markdown 知识文档转为 AI 可直接消费的结构化卡片
- 为 Codex / Vibecoding AI 构建上下文完整的知识库
- 知识库文档需要同时支持机器消费（JSON）和人工维护（YAML）
- 需要建立知识条目间的依赖关系与交叉引用
- 对现有 Markdown 知识库进行结构化升级
- 需要为下游向量数据库 / 知识图谱生成索引定义

**不适用**：纯文本摘要、单文件格式转换（无结构化需求）、非知识库类 Markdown（博客/文章）、已有结构化数据的逆向转换。

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| Markdown 文件内容 | 知识库文档，支持单个或多个文件（模块/功能/Phase/规则/实体/UI/流程） | 是 |
| 版本号 | 卡片版本标识，格式 `vX.Y` | 是 |
| 卡片 ID 前缀 | 用于生成唯一 ID 的模块前缀，如 `doc_review` | 否（默认从 H1 标题推断） |
| 输出格式偏好 | `json` / `yaml` / `both` | 否（默认 both） |
| ID 生成策略 | `structured`（`{prefix}_{feature}_{phase}`） / `uuid` | 否（默认 structured） |

## 输出

- **JSON 卡片集** — 结构化卡片数组，供 AI 会话直接消费，不丢失上下文
- **YAML 卡片集** — 可人工编辑和迭代的卡片集合
- **索引定义** — 向量索引、元数据索引、图谱索引的结构化定义（JSON）
- 每条卡片 `fields` 完整包含：dependencies、rules、entities、apis、ui_components、test_coverage
- 卡片间依赖关系显式保存，支持单文件内和跨文件关联
- 保留 Markdown 原文作为 `source` 字段

## 执行步骤

### 1. 解析 Markdown 结构

逐层解析 Markdown 文档，识别结构元素并映射到卡片字段：

| Markdown 元素 | 解析目标 | 卡片字段映射 |
|---------------|----------|-------------|
| H1 标题 | 模块/顶级实体 | `card_type: module`，生成 `id` 和 `title` |
| H2 标题 | 功能/子模块 | `card_type: feature`，父级为 H1 对应卡片 |
| H3 标题 | 子功能/细节 | 归属上层 feature 卡片，或生成独立卡片 |
| 表格 | 规则/约束/覆盖度 | `rules`、`test_coverage`、`dependencies` |
| 有序/无序列表 | 实体/组件/依赖 | `entities`、`ui_components`、`dependencies` |
| 代码块 | 流程示例/方法/测试 | 保留原文，归入对应卡片的 `description` 或独立 process 卡片 |
| Mermaid / 流程图 | 流程定义 | `card_type: process`，关联涉及的模块和功能 |
| Phase 关键词 | 阶段信息 | `card_type: phase`，自动关联涉及的模块与功能 |

多文件输入时，对每个文件独立执行本步骤，保留文件来源标记。

### 2. 识别卡片类型与生成 ID

对每个解析出的知识单元，按优先级判断 `card_type`：

1. 明确标注 Phase 或阶段 → `phase`
2. H1 级别且包含多个子功能 → `module`
3. 包含流程图或步骤序列 → `process`
4. 包含约束/门禁/NFR/安全规则 → `rule`
5. 描述数据模型/Service/Adapter → `entity`
6. 描述页面/组件/布局 → `ui`
7. 描述具体功能或工作流 → `feature`

ID 生成规则：
- `structured` 策略：优先使用 `{prefix}_{feature}_{phase}` 格式，若无法推断则用 `{prefix}_{序号}`
- `uuid` 策略：生成标准 UUID v4
- 多文件场景下，`structured` 策略自动在 prefix 前加文件序号避免冲突：`{file_seq}_{prefix}_{feature}_{phase}`
- 确保全局唯一

### 3. 提取卡片字段

对每张卡片，按以下规则填充 `fields`：

- **dependencies**：从表格和列表中提取模块/功能/Phase/Rule/Entity 依赖，记录目标卡片 ID
- **rules**：提取 P0/P1/P2 门禁规则、NFR 约束、安全约束，保留原始描述
- **entities**：提取数据模型名称、Service 名称、Adapter 名称
- **apis**：提取接口路径和方法（从代码块或表格中识别 URL 模式 `/api/...`）
- **ui_components**：提取页面组件名称、布局结构
- **test_coverage**：提取测试类型和覆盖范围（单元测试 / UI 自动化 / 集成测试）

对于无法从原文提取的字段，设为空数组 `[]` 而非省略，保证结构完整。

### 4. 建立依赖关系

**单文件内依赖**：
- 扫描所有卡片的 `dependencies`、`entities` 字段
- 将文本引用匹配到对应卡片的 `id`
- 匹配优先级：精确匹配卡片 ID → 标题关键词匹配 → 模块名前缀匹配
- 未匹配到的依赖保留原文并标记 `unresolved: true`

**跨文件关联**（多文件场景）：
- 合并所有文件的卡片集后，重新扫描 `dependencies` 和 `entities`
- 对每个 `unresolved` 依赖，在跨文件卡片集中二次匹配
- 仍无法匹配的保留 `unresolved: true`
- 对跨文件匹配成功的依赖，标记 `cross_file: true` 和 `source_file` 字段

### 5. 元数据增强

对每张卡片补充：

- `source`：原始 Markdown 文件名
- `version`：输入的版本号
- `last_updated`：当前日期（YYYY-MM-DD）
- `module`：所属模块名（从 H1 或 ID 前缀推断）
- `phase`：所属阶段（从上下文推断，若存在）

### 6. 生成双格式输出

按 [reference.md](reference.md) 中的 Schema 生成 JSON 和 YAML 卡片集。两者内容完全等价：
- JSON：供 AI 会话直接消费
- YAML：供人工编辑和迭代

多文件场景输出单个合并的卡片集。

### 7. 生成索引定义

基于卡片集生成三类索引定义，供下游系统（向量数据库、知识图谱等）消费。skill 只生成定义文件，不执行实际的向量化和图谱构建。

**向量索引定义**：指定哪些字段需要 embedding，以及 embedding 模型建议。

**元数据索引定义**：指定哪些字段用于过滤和检索，以及索引类型。

**图谱索引定义**：基于 `dependencies` 和 `entities` 生成节点和边的定义。

索引 Schema 详见 [reference.md](reference.md)。

### 8. 一致性校验

- 每张卡片的 `fields` 六个字段全部存在（可为空数组）
- 所有 `dependencies` 中的 ID 在卡片集中有对应卡片，或标记 `unresolved`
- `card_type` 值限定为：module / feature / process / rule / ui / phase / entity
- `version` 和 `last_updated` 格式正确
- `source` 字段与输入文件名一致
- JSON 可被标准 JSON 解析器解析，YAML 可被标准 YAML 解析器解析
- 多文件场景下无重复 ID
- 索引定义中引用的卡片 ID 在卡片集中存在

## 边界与非目标

- **不做**向量数据库写入 / embedding 实际计算 — 只生成索引定义文件，实际向量化和存储由下游系统执行
- **不做**知识图谱运行时构建 — 只生成图谱节点/边的定义，运行时图谱构建由下游系统执行
- **不做**Markdown 逆向生成 — 只做 MD → 卡片，不做卡片 → MD
- **不做**知识库版本管理和 diff — 只负责单次转换，版本管理是外部工具的职责
- **不做**自动写入知识库系统 — 只输出 JSON/YAML 文件内容，由用户决定存储方式
- **不做**AI 模型训练数据生成 — 输出供推理时上下文消费，非训练语料
- 依赖 Markdown 源文件的结构化程度，非结构化文本可能导致字段提取不完整（标记为空数组）

## 验收标准

- 输出的 JSON 和 YAML 可被标准解析器直接解析，无语法错误
- 每张卡片 `fields` 包含完整的六个字段（dependencies / rules / entities / apis / ui_components / test_coverage）
- 每张卡片包含 `module` 和 `phase` 元数据字段
- 所有卡片 ID 全局唯一
- `dependencies` 中已解析的 ID 在卡片集中存在
- `card_type` 值均为合法枚举值
- `source` 和 `version` 与输入一致
- JSON 与 YAML 输出内容完全等价（格式差异除外）
- 无法从原文提取的字段为空数组 `[]`，非 null 或缺失
- 多文件场景下跨文件依赖已正确关联或标记 `unresolved`
- 索引定义中所有引用的卡片 ID 在卡片集中存在
