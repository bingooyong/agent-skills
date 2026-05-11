# 知识卡片拆分规则与示例

本文档定义拆分规则、manifest Schema 和完整示例，供 `knowledge-card-splitter` skill 参考。

## 拆分规则详解

### 按模块拆分

以 `card_type: module` 的卡片为核心，聚合其依赖的 feature、rule、entity、ui、phase 卡片。

**归属判定**（满足任一即归入该模块文件）：

1. 卡片本身是 module 卡片
2. 卡片的 `module` 字段值 = 该 module 的 id
3. 卡片被该 module 的 `fields.dependencies` 直接引用
4. 卡片被该 module 聚合的 feature 卡片的 `fields.dependencies` 引用（递归一层）

**依赖处理**：

- 依赖目标在当前文件中 → 保留完整卡片对象
- 依赖目标不在当前文件中 → 保留引用摘要：

```json
{
  "id": "other_module_feature",
  "title": "其他模块功能",
  "external_ref": true,
  "source_file": "module_other_module.json"
}
```

### 按功能拆分

以 `card_type: feature` 的卡片为核心，仅聚合其直接依赖（不递归）。

**归属判定**：

1. 卡片本身是 feature 卡片
2. 卡片被该 feature 的 `fields.dependencies` 直接引用

**依赖处理**：与按模块拆分相同，外部依赖保留引用摘要。

### 按类型拆分

按 `card_type` 分组，每组一个文件。不处理依赖关系，保留原始引用。

---

## Manifest Schema

```json
{
  "source_file": "cards.json",
  "total_cards": 25,
  "split_strategy": "all",
  "generated_at": "2026-05-11",
  "files": [
    {
      "filename": "module_doc_review_platform.json",
      "card_count": 8,
      "card_types": {
        "module": 1,
        "feature": 3,
        "rule": 2,
        "entity": 1,
        "ui": 1
      },
      "dependencies": ["module_rule_engine.json", "rules.json"]
    }
  ]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `source_file` | string | 原始 cards.json 路径 |
| `total_cards` | number | 原始卡片总数 |
| `split_strategy` | string | 使用的拆分策略（module / feature / type / all） |
| `generated_at` | string | 生成日期，格式 YYYY-MM-DD |
| `files` | array | 拆分文件清单 |
| `files[].filename` | string | 文件名 |
| `files[].card_count` | number | 包含的卡片数量 |
| `files[].card_types` | object | 各类型卡片数量 |
| `files[].dependencies` | string[] | 依赖的其他拆分文件名 |

---

## 拆分示例

### 输入 cards.json

```json
[
  {
    "card_type": "module",
    "id": "doc_review_platform",
    "title": "文档评审平台",
    "description": "统一评审任务模型 + AI 评审引擎 + 闭环治理",
    "fields": {
      "dependencies": ["requirement_review_workbench", "design_review_workbench"],
      "rules": ["P0: 必须通过 6Cs 检查"],
      "entities": ["ReviewTaskEntity", "ReviewFinding"],
      "apis": ["/api/review/document"],
      "ui_components": ["Sidebar", "Canvas"],
      "test_coverage": ["unit_test_coverage"]
    },
    "module": "doc_review",
    "phase": "phase-1",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "feature",
    "id": "requirement_review_workbench",
    "title": "需求评审工作台",
    "description": "需求文档上传、AI 评审、结果确认的完整工作流",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": ["P0: 需求评审不可跳过"],
      "entities": ["ReviewTaskEntity"],
      "apis": ["/api/review/requirement", "/api/review/requirement/submit"],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer"],
      "test_coverage": ["unit_test_coverage: 80%", "ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "feature",
    "id": "design_review_workbench",
    "title": "设计评审工作台",
    "description": "设计文档评审工作流，支持设计规范检查",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": ["P1: 设计评审建议不阻塞"],
      "entities": ["ReviewFinding"],
      "apis": ["/api/review/design"],
      "ui_components": ["DesignCanvas", "FindingPanel"],
      "test_coverage": ["unit_test_coverage: 75%"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "rule",
    "id": "p0_gate_policy",
    "title": "P0 门禁策略",
    "description": "所有 P0 门禁必须通过，不可跳过或降级",
    "fields": {
      "dependencies": [],
      "rules": ["P0: 不可跳过", "P0: 不可降级"],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 100%"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "entity",
    "id": "review_task_entity",
    "title": "ReviewTaskEntity",
    "description": "评审任务实体，包含任务状态、关联文档、评审结果",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 90%"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "ui",
    "id": "requirement_review_workbench_ui",
    "title": "需求评审工作台 UI",
    "description": "包含上传面板、评审结果抽屉、确认对话框",
    "fields": {
      "dependencies": ["requirement_review_workbench"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer", "ConfirmDialog"],
      "test_coverage": ["ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "phase",
    "id": "phase_116",
    "title": "Phase 116",
    "description": "需求评审 MVP 阶段，完成核心评审流程",
    "fields": {
      "dependencies": ["doc_review_platform", "requirement_review_workbench"],
      "rules": ["P0: MVP 必须包含 6Cs 检查"],
      "entities": ["ReviewTaskEntity"],
      "apis": [],
      "ui_components": ["RequirementUploadPanel"],
      "test_coverage": ["unit_test_coverage", "ui_automation_coverage"]
    },
    "module": "doc_review",
    "source": "Document_Review_Module_Knowledge.md",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：module_doc_review_platform.json

```json
[
  {
    "card_type": "module",
    "id": "doc_review_platform",
    "title": "文档评审平台",
    "description": "统一评审任务模型 + AI 评审引擎 + 闭环治理",
    "fields": {
      "dependencies": ["requirement_review_workbench", "design_review_workbench"],
      "rules": ["P0: 必须通过 6Cs 检查"],
      "entities": ["ReviewTaskEntity", "ReviewFinding"],
      "apis": ["/api/review/document"],
      "ui_components": ["Sidebar", "Canvas"],
      "test_coverage": ["unit_test_coverage"]
    },
    "module": "doc_review",
    "phase": "phase-1",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "feature",
    "id": "requirement_review_workbench",
    "title": "需求评审工作台",
    "description": "需求文档上传、AI 评审、结果确认的完整工作流",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": ["P0: 需求评审不可跳过"],
      "entities": ["ReviewTaskEntity"],
      "apis": ["/api/review/requirement", "/api/review/requirement/submit"],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer"],
      "test_coverage": ["unit_test_coverage: 80%", "ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "feature",
    "id": "design_review_workbench",
    "title": "设计评审工作台",
    "description": "设计文档评审工作流，支持设计规范检查",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": ["P1: 设计评审建议不阻塞"],
      "entities": ["ReviewFinding"],
      "apis": ["/api/review/design"],
      "ui_components": ["DesignCanvas", "FindingPanel"],
      "test_coverage": ["unit_test_coverage: 75%"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "rule",
    "id": "p0_gate_policy",
    "title": "P0 门禁策略",
    "description": "所有 P0 门禁必须通过，不可跳过或降级",
    "fields": {
      "dependencies": [],
      "rules": ["P0: 不可跳过", "P0: 不可降级"],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 100%"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "entity",
    "id": "review_task_entity",
    "title": "ReviewTaskEntity",
    "description": "评审任务实体，包含任务状态、关联文档、评审结果",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 90%"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "ui",
    "id": "requirement_review_workbench_ui",
    "title": "需求评审工作台 UI",
    "description": "包含上传面板、评审结果抽屉、确认对话框",
    "fields": {
      "dependencies": ["requirement_review_workbench"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer", "ConfirmDialog"],
      "test_coverage": ["ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  },
  {
    "card_type": "phase",
    "id": "phase_116",
    "title": "Phase 116",
    "description": "需求评审 MVP 阶段，完成核心评审流程",
    "fields": {
      "dependencies": ["doc_review_platform", "requirement_review_workbench"],
      "rules": ["P0: MVP 必须包含 6Cs 检查"],
      "entities": ["ReviewTaskEntity"],
      "apis": [],
      "ui_components": ["RequirementUploadPanel"],
      "test_coverage": ["unit_test_coverage", "ui_automation_coverage"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：feature_requirement_review_workbench.json

```json
[
  {
    "card_type": "feature",
    "id": "requirement_review_workbench",
    "title": "需求评审工作台",
    "description": "需求文档上传、AI 评审、结果确认的完整工作流",
    "fields": {
      "dependencies": [
        {
          "id": "doc_review_platform",
          "title": "文档评审平台",
          "external_ref": true,
          "source_file": "module_doc_review_platform.json"
        }
      ],
      "rules": ["P0: 需求评审不可跳过"],
      "entities": ["ReviewTaskEntity"],
      "apis": ["/api/review/requirement", "/api/review/requirement/submit"],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer"],
      "test_coverage": ["unit_test_coverage: 80%", "ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：rules.json

```json
[
  {
    "card_type": "rule",
    "id": "p0_gate_policy",
    "title": "P0 门禁策略",
    "description": "所有 P0 门禁必须通过，不可跳过或降级",
    "fields": {
      "dependencies": [],
      "rules": ["P0: 不可跳过", "P0: 不可降级"],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 100%"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：entity.json

```json
[
  {
    "card_type": "entity",
    "id": "review_task_entity",
    "title": "ReviewTaskEntity",
    "description": "评审任务实体，包含任务状态、关联文档、评审结果",
    "fields": {
      "dependencies": ["doc_review_platform"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": [],
      "test_coverage": ["unit_test_coverage: 90%"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：ui.json

```json
[
  {
    "card_type": "ui",
    "id": "requirement_review_workbench_ui",
    "title": "需求评审工作台 UI",
    "description": "包含上传面板、评审结果抽屉、确认对话框",
    "fields": {
      "dependencies": ["requirement_review_workbench"],
      "rules": [],
      "entities": [],
      "apis": [],
      "ui_components": ["RequirementUploadPanel", "ReviewResultDrawer", "ConfirmDialog"],
      "test_coverage": ["ui_automation_coverage: e2e_req_review"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：phase.json

```json
[
  {
    "card_type": "phase",
    "id": "phase_116",
    "title": "Phase 116",
    "description": "需求评审 MVP 阶段，完成核心评审流程",
    "fields": {
      "dependencies": ["doc_review_platform", "requirement_review_workbench"],
      "rules": ["P0: MVP 必须包含 6Cs 检查"],
      "entities": ["ReviewTaskEntity"],
      "apis": [],
      "ui_components": ["RequirementUploadPanel"],
      "test_coverage": ["unit_test_coverage", "ui_automation_coverage"]
    },
    "module": "doc_review",
    "source": "cards.json",
    "version": "v1.0",
    "last_updated": "2026-05-10"
  }
]
```

### 输出：manifest.json

```json
{
  "source_file": "cards.json",
  "total_cards": 7,
  "split_strategy": "all",
  "generated_at": "2026-05-11",
  "files": [
    {
      "filename": "module_doc_review_platform.json",
      "card_count": 7,
      "card_types": {
        "module": 1,
        "feature": 2,
        "rule": 1,
        "entity": 1,
        "ui": 1,
        "phase": 1
      },
      "dependencies": []
    },
    {
      "filename": "feature_requirement_review_workbench.json",
      "card_count": 1,
      "card_types": {
        "feature": 1
      },
      "dependencies": ["module_doc_review_platform.json"]
    },
    {
      "filename": "feature_design_review_workbench.json",
      "card_count": 1,
      "card_types": {
        "feature": 1
      },
      "dependencies": ["module_doc_review_platform.json"]
    },
    {
      "filename": "rules.json",
      "card_count": 1,
      "card_types": {
        "rule": 1
      },
      "dependencies": []
    },
    {
      "filename": "entity.json",
      "card_count": 1,
      "card_types": {
        "entity": 1
      },
      "dependencies": ["module_doc_review_platform.json"]
    },
    {
      "filename": "ui.json",
      "card_count": 1,
      "card_types": {
        "ui": 1
      },
      "dependencies": ["module_doc_review_platform.json", "feature_requirement_review_workbench.json"]
    },
    {
      "filename": "phase.json",
      "card_count": 1,
      "card_types": {
        "phase": 1
      },
      "dependencies": ["module_doc_review_platform.json", "feature_requirement_review_workbench.json"]
    }
  ]
}
```
