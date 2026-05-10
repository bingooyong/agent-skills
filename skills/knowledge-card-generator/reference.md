# 知识卡片 Schema 与示例

本文档定义知识卡片集和索引定义的完整 Schema 和示例输出，供 `knowledge-card-generator` skill 参考。

## 卡片 Schema

每张卡片包含以下字段：

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `card_type` | enum | 是 | `module` / `feature` / `process` / `rule` / `ui` / `phase` / `entity` |
| `id` | string | 是 | 全局唯一标识，格式见下方 ID 生成规则 |
| `title` | string | 是 | 卡片标题，从 Markdown 标题提取 |
| `description` | string | 是 | 摘要说明，一段话概括卡片内容 |
| `fields` | object | 是 | 结构化字段，包含以下 6 个子字段 |
| `fields.dependencies` | string[] | 是 | 依赖的其他卡片 ID，无依赖时为 `[]` |
| `fields.rules` | string[] | 是 | P0/P1/P2 门禁、NFR、安全约束，无规则时为 `[]` |
| `fields.entities` | string[] | 是 | 数据模型/Service/Adapter 名称，无实体时为 `[]` |
| `fields.apis` | string[] | 是 | 前后端接口路径，无接口时为 `[]` |
| `fields.ui_components` | string[] | 是 | 页面组件/布局名称，无 UI 时为 `[]` |
| `fields.test_coverage` | string[] | 是 | 测试类型和覆盖范围，无测试时为 `[]` |
| `module` | string | 是 | 所属模块名，从 H1 或 ID 前缀推断 |
| `phase` | string | 否 | 所属阶段（如 `phase-1`），无法推断时省略 |
| `source` | string | 是 | 原始 Markdown 文件名 |
| `version` | string | 是 | 卡片版本号，格式 `vX.Y` |
| `last_updated` | string | 是 | 日期，格式 `YYYY-MM-DD` |

跨文件依赖匹配成功的卡片，`dependencies` 中对应条目为对象形式：

```json
{
  "id": "other_module_feature",
  "cross_file": true,
  "source_file": "Other_Module_Knowledge.md"
}
```

### ID 生成规则

**structured 策略**（默认）：
- 单文件：`{prefix}_{feature}_{phase}`，如 `doc_review_ai_engine`
- 无法推断时：`{prefix}_{序号}`，如 `doc_review_01`
- 多文件：`{file_seq}_{prefix}_{feature}_{phase}`，如 `1_doc_review_ai_engine`

**uuid 策略**：
- 生成标准 UUID v4，如 `a1b2c3d4-e5f6-7890-abcd-ef1234567890`

### 卡片类型说明

| card_type | 触发条件 | 典型内容 |
|-----------|----------|----------|
| `module` | H1 级别，包含多个子功能 | 模块概述、核心能力、整体架构 |
| `feature` | 描述具体功能或工作流 | 功能说明、输入输出、业务逻辑 |
| `process` | 包含流程图或步骤序列 | 流程步骤、状态转换、时序 |
| `rule` | 包含约束/门禁/NFR/安全规则 | P0/P1/P2 门禁、NFR、安全策略 |
| `entity` | 描述数据模型/Service/Adapter | 数据结构、Service 接口、Adapter 配置 |
| `ui` | 描述页面/组件/布局 | 组件树、布局结构、交互行为 |
| `phase` | 明确标注 Phase 或阶段 | 阶段目标、涉及模块、里程碑 |

---

## 索引定义 Schema

索引定义与卡片集一同输出，供下游系统消费。skill 只生成定义，不执行实际的向量化和图谱构建。

### 索引定义顶层结构

```json
{
  "card_set_version": "v1.0",
  "generated_at": "2026-05-10",
  "vector_index": { ... },
  "metadata_index": { ... },
  "graph_index": { ... }
}
```

### 向量索引定义

指定哪些卡片字段需要生成 embedding，以及建议的 embedding 配置。

```json
{
  "vector_index": {
    "fields": [
      {
        "field": "description",
        "embedding_model": "text-embedding-3-small",
        "dimension": 1536
      },
      {
        "field": "rules",
        "embedding_model": "text-embedding-3-small",
        "dimension": 1536
      },
      {
        "field": "ui_components",
        "embedding_model": "text-embedding-3-small",
        "dimension": 1536
      }
    ],
    "similarity_metric": "cosine",
    "index_type": "hnsw"
  }
}
```

| 字段 | 说明 |
|------|------|
| `fields[].field` | 需要生成 embedding 的卡片字段名 |
| `fields[].embedding_model` | 建议的 embedding 模型 |
| `fields[].dimension` | 向量维度 |
| `similarity_metric` | 相似度计算方式（cosine / dot_product / l2） |
| `index_type` | 索引类型（hnsw / flat / ivf） |

### 元数据索引定义

指定哪些卡片字段用于过滤和检索，以及索引类型。

```json
{
  "metadata_index": {
    "fields": [
      { "field": "module", "index_type": "keyword" },
      { "field": "phase", "index_type": "keyword" },
      { "field": "card_type", "index_type": "keyword" },
      { "field": "version", "index_type": "keyword" },
      { "field": "source", "index_type": "keyword" },
      { "field": "rules", "index_type": "keyword", "sub_index": "rule_level" }
    ]
  }
}
```

| 字段 | 说明 |
|------|------|
| `fields[].field` | 卡片字段名 |
| `fields[].index_type` | 索引类型（keyword / text / numeric） |
| `fields[].sub_index` | 子索引名称，用于嵌套字段（如 rules 中的 P0/P1/P2 级别） |

### 图谱索引定义

基于卡片的 `dependencies` 和 `entities` 生成节点和边的定义。

```json
{
  "graph_index": {
    "nodes": [
      { "id": "doc_review_platform", "label": "module", "properties": ["title", "module"] },
      { "id": "doc_review_ai_engine", "label": "feature", "properties": ["title", "module"] },
      { "id": "doc_review_workflow", "label": "process", "properties": ["title", "module"] }
    ],
    "edges": [
      { "source": "doc_review_ai_engine", "target": "doc_review_platform", "relation": "depends_on" },
      { "source": "doc_review_workflow", "target": "doc_review_ai_engine", "relation": "depends_on" },
      { "source": "doc_review_workflow", "target": "doc_review_platform", "relation": "depends_on" }
    ],
    "edge_types": ["depends_on", "contains", "implements", "references"]
  }
}
```

| 字段 | 说明 |
|------|------|
| `nodes[].id` | 卡片 ID |
| `nodes[].label` | 等同于 `card_type` |
| `nodes[].properties` | 节点上需要存储的卡片属性 |
| `edges[].source` | 起始节点 ID |
| `edges[].target` | 目标节点 ID |
| `edges[].relation` | 关系类型 |
| `edge_types` | 本卡片集中出现的所有关系类型枚举 |

关系类型推断规则：
- `dependencies` 中的引用 → `depends_on`
- 同一 module 下的 feature → `contains`（父→子）
- `entities` 中引用的实体卡片 → `references`
- phase 卡片关联的模块 → `implements`

---

## JSON 示例

### 单文件示例

输入：`Document_Review_Module_Knowledge.md` + `v1.0`

```json
[
  {
    "card_type": "module",
    "id": "doc_review_platform",
    "title": "文档评审平台",
    "description": "统一评审任务模型 + AI 评审引擎 + 闭环治理 + 审计报告 + 全生命周期追溯",
    "fields": {
      "dependencies": ["review_rules_yaml", "requirement_review_workbench"],
      "rules": ["P0: 必须通过 6Cs 检查", "NFR: 完整性审计"],
      "entities": ["ReviewBatch", "ReviewFinding"],
      "apis": ["/api/review/document", "/api/review/document/generate-reviewed-docx"],
      "ui_components": ["Sidebar", "Canvas", "Drawer"],
      "test_coverage": ["unit_test_coverage", "ui_automation_coverage"]
    },
    "module": "doc_review",
    "phase": "phase-1",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "feature",
    "id": "doc_review_ai_engine",
    "title": "AI 评审引擎",
    "description": "基于规则引擎的自动评审，支持 6Cs 检查和自定义规则扩展",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": ["P0: 6Cs 检查不可跳过", "P1: 自定义规则需审核后生效"],
      "entities": ["ReviewEngine", "RuleExecutor"],
      "apis": ["/api/review/engine/execute", "/api/review/engine/rules"],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 85%"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "process",
    "id": "doc_review_workflow",
    "title": "文档评审流程",
    "description": "从文档上传到评审报告生成的完整流程",
    "fields": {
      "dependencies": ["doc_review_ai_engine", "doc_review_platform"],
      "rules": ["P0: 评审结果需人工确认后才能生效"],
      "entities": [],
      "apis": [],
      "ui_components": ["ReviewStepIndicator", "ResultConfirmDialog"],
      "test_coverage": ["ui_automation_coverage: e2e_review_flow"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 多文件跨文件关联示例

输入：`Document_Review_Module_Knowledge.md` + `Rule_Engine_Module_Knowledge.md` + `v1.0`

```json
[
  {
    "card_type": "module",
    "id": "1_doc_review_platform",
    "title": "文档评审平台",
    "description": "统一评审任务模型 + AI 评审引擎 + 闭环治理 + 审计报告 + 全生命周期追溯",
    "fields": {
      "dependencies": [
        {
          "id": "2_rule_engine_core",
          "cross_file": true,
          "source_file": "Rule_Engine_Module_Knowledge.md"
        }
      ],
      "rules": ["P0: 必须通过 6Cs 检查"],
      "entities": ["ReviewBatch", "ReviewFinding"],
      "apis": ["/api/review/document"],
      "ui_components": ["Sidebar", "Canvas"],
      "test_coverage": ["unit_test_coverage"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "module",
    "id": "2_rule_engine_core",
    "title": "规则引擎",
    "description": "通用规则执行引擎，支持 YAML 规则定义和动态加载",
    "fields": {
      "dependencies": [],
      "rules": ["P0: 规则执行失败必须回滚", "P1: 规则变更需审核"],
      "entities": ["RuleDefinition", "RuleExecutor"],
      "apis": ["/api/rules/execute", "/api/rules/validate"],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 90%"]
    },
    "module": "rule_engine",
    "source": "Rule_Engine_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

---

## YAML 示例

与 JSON 单文件示例内容完全等价，格式适合人工编辑：

```yaml
- card_type: module
  id: doc_review_platform
  title: 文档评审平台
  description: 统一评审任务模型 + AI 评审引擎 + 闭环治理 + 审计报告 + 全生命周期追溯
  fields:
    dependencies:
      - review_rules_yaml
      - requirement_review_workbench
    rules:
      - "P0: 必须通过 6Cs 检查"
      - "NFR: 完整性审计"
    entities:
      - ReviewBatch
      - ReviewFinding
    apis:
      - /api/review/document
      - /api/review/document/generate-reviewed-docx
    ui_components:
      - Sidebar
      - Canvas
      - Drawer
    test_coverage:
      - unit_test_coverage
      - ui_automation_coverage
  module: doc_review
  phase: phase-1
  source: Document_Review_Module_Knowledge.md
  version: v1.0
  last_updated: "2026-05-10"

- card_type: feature
  id: doc_review_ai_engine
  title: AI 评审引擎
  description: 基于规则引擎的自动评审，支持 6Cs 检查和自定义规则扩展
  fields:
    dependencies:
      - doc_review_platform
    rules:
      - "P0: 6Cs 检查不可跳过"
      - "P1: 自定义规则需审核后生效"
    entities:
      - ReviewEngine
      - RuleExecutor
    apis:
      - /api/review/engine/execute
      - /api/review/engine/rules
    ui_components: []
    test_coverage:
      - "unit_test_coverage: 85%"
  module: doc_review
  source: Document_Review_Module_Knowledge.md
  version: v1.0
  last_updated: "2026-05-10"

- card_type: process
  id: doc_review_workflow
  title: 文档评审流程
  description: 从文档上传到评审报告生成的完整流程
  fields:
    dependencies:
      - doc_review_ai_engine
      - doc_review_platform
    rules:
      - "P0: 评审结果需人工确认后才能生效"
    entities: []
    apis: []
    ui_components:
      - ReviewStepIndicator
      - ResultConfirmDialog
    test_coverage:
      - "ui_automation_coverage: e2e_review_flow"
  module: doc_review
  source: Document_Review_Module_Knowledge.md
  version: v1.0
  last_updated: "2026-05-10"
```

---

## 索引定义完整示例

对应上述单文件卡片集的索引定义：

```json
{
  "card_set_version": "v1.0",
  "generated_at": "2026-05-10",
  "vector_index": {
    "fields": [
      { "field": "description", "embedding_model": "text-embedding-3-small", "dimension": 1536 },
      { "field": "rules", "embedding_model": "text-embedding-3-small", "dimension": 1536 },
      { "field": "ui_components", "embedding_model": "text-embedding-3-small", "dimension": 1536 }
    ],
    "similarity_metric": "cosine",
    "index_type": "hnsw"
  },
  "metadata_index": {
    "fields": [
      { "field": "module", "index_type": "keyword" },
      { "field": "phase", "index_type": "keyword" },
      { "field": "card_type", "index_type": "keyword" },
      { "field": "version", "index_type": "keyword" },
      { "field": "source", "index_type": "keyword" },
      { "field": "rules", "index_type": "keyword", "sub_index": "rule_level" }
    ]
  },
  "graph_index": {
    "nodes": [
      { "id": "doc_review_platform", "label": "module", "properties": ["title", "module"] },
      { "id": "doc_review_ai_engine", "label": "feature", "properties": ["title", "module"] },
      { "id": "doc_review_workflow", "label": "process", "properties": ["title", "module"] }
    ],
    "edges": [
      { "source": "doc_review_platform", "target": "doc_review_ai_engine", "relation": "contains" },
      { "source": "doc_review_ai_engine", "target": "doc_review_platform", "relation": "depends_on" },
      { "source": "doc_review_workflow", "target": "doc_review_ai_engine", "relation": "depends_on" },
      { "source": "doc_review_workflow", "target": "doc_review_platform", "relation": "depends_on" }
    ],
    "edge_types": ["depends_on", "contains"]
  }
}
```
