---
name: knowledge-card-splitter
description: "Split a large JSON LLM Wiki card set into multiple small JSON files for AI task-driven context loading. Use when (1) splitting cards.json by module/feature/type, (2) reducing AI context window load, (3) user mentions 'split cards', 'card splitter', 'chunk cards', or 'task-driven loading'. Outputs per-module, per-feature, per-type JSON files with preserved dependencies and a manifest."
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# 知识卡片拆分 Skill

将大型 JSON 卡片集（cards.json）按模块、功能、类型拆分为多个小 JSON 文件，供 AI 按任务按需加载上下文。保留依赖关系，支持增量更新。

## 快速上手

```
输入: cards.json（由 knowledge-card-generator 生成）
输出: module_*.json / feature_*.json / rules.json / entity.json / ui.json / phase.json + manifest.json
```

每个拆分文件保持原有字段完整性，可直接被 Codex / Vibecoding AI 加载。详见 [reference.md](reference.md) 中的拆分规则和示例。

## 何时使用

- 大型 cards.json 超出 AI 上下文窗口，需要按任务按需加载
- 需要按模块/功能粒度给 AI 提供精确上下文
- 已有 cards.json 需要拆分为可独立消费的小文件
- 知识库新增模块或功能，需要增量生成拆分文件

**不适用**：非 JSON 卡片集的文件拆分、cards.json 的生成（这是 knowledge-card-generator 的职责）、卡片内容的编辑或修改。

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| JSON 卡片集文件 | 由 knowledge-card-generator 生成的 cards.json | 是 |
| 拆分策略 | `module` / `feature` / `type` / `all`（默认 all） | 否 |
| 输出目录 | 拆分文件的存放目录 | 否（默认与输入同目录下的 `cards/` 子目录） |

## 输出

- **按模块拆分文件**：`module_<id>.json` — 每个 module 及其依赖的 feature/rule/entity/ui/phase
- **按功能拆分文件**：`feature_<id>.json` — 每个 feature 及其直接依赖
- **按类型拆分文件**：`rules.json`、`entity.json`、`ui.json`、`phase.json`
- **拆分清单**：`manifest.json` — 列出每个拆分文件包含的卡片类型、数量、依赖关系
- 每个拆分文件保持原有字段完整性，保留 `source` 字段指向原 JSON

## 执行步骤

### 1. 读取并验证输入

- 读取 cards.json，验证 JSON 格式合法
- 验证每张卡片包含必需字段：card_type、id、title、description、fields、source、version、last_updated
- 统计卡片总数和各类型数量
- 若输入不合法，输出错误信息并终止

### 2. 构建卡片索引

建立三组索引供后续查询：

- **ID 索引**：`id` → 卡片对象，O(1) 查找
- **类型索引**：`card_type` → 卡片列表
- **模块索引**：`module` 字段 → 卡片列表（同一模块下的所有卡片）

### 3. 按模块拆分

对每个 `card_type: module` 的卡片：

1. 以该 module 卡片为核心
2. 收集其 `fields.dependencies` 中引用的所有卡片（递归一层）
3. 收集 `module` 字段值与该 module id 匹配的 feature、rule、entity、ui、phase 卡片
4. 将 module 卡片和收集到的所有卡片写入 `module_<module_id>.json`
5. 依赖指向的卡片不在当前文件中时，保留 `id` 和 `title` 作为引用，标记 `external_ref: true`

### 4. 按功能拆分

对每个 `card_type: feature` 的卡片：

1. 以该 feature 卡片为核心
2. 收集其 `fields.dependencies` 中直接引用的卡片（不递归）
3. 将 feature 卡片和收集到的卡片写入 `feature_<feature_id>.json`
4. 依赖指向的卡片不在当前文件中时，保留 `id` 和 `title` 作为引用，标记 `external_ref: true`

### 5. 按类型拆分

按 `card_type` 分组输出辅助文件：

| 文件名 | 包含的 card_type |
|--------|-----------------|
| `rules.json` | 所有 `rule` 卡片 |
| `entity.json` | 所有 `entity` 卡片 |
| `ui.json` | 所有 `ui` 卡片 |
| `phase.json` | 所有 `phase` 卡片 |

每个类型文件中保留完整的 `fields` 和元数据。

### 6. 生成拆分清单

输出 `manifest.json`，包含：

- `source_file`：原始 cards.json 路径
- `total_cards`：卡片总数
- `split_strategy`：使用的拆分策略
- `files`：每个拆分文件的清单

清单中每个文件条目包含：

- `filename`：文件名
- `card_count`：包含的卡片数量
- `card_types`：包含的卡片类型及各自数量
- `dependencies`：该文件依赖的其他拆分文件（通过卡片 ID 交叉引用推断）

详见 [reference.md](reference.md) 中的 manifest Schema。

### 7. 增量更新处理

当输入的 cards.json 包含新增模块或功能时：

1. 读取已有 manifest.json（若存在）
2. 对比新旧卡片 ID，识别新增、变更、删除
3. 仅重新生成受影响的拆分文件，未变更的文件跳过
4. 更新 manifest.json

增量更新规则：

| 变更类型 | 处理方式 |
|---------|---------|
| 新增 module 卡片 | 生成新的 `module_<id>.json` |
| 新增 feature 卡片 | 生成新的 `feature_<id>.json`，更新所属 module 文件 |
| 变更任意卡片 | 重新生成该卡片所在的所有拆分文件 |
| 删除卡片 | 重新生成受影响的拆分文件，删除空文件 |
| 无变更 | 跳过 |

### 8. 一致性校验

- 每个拆分文件可被标准 JSON 解析器解析
- 所有拆分文件中的卡片总数 = 原 cards.json 中的卡片总数
- 每张卡片只出现在其应归属的拆分文件中（module 文件中可重复出现在 type 文件中，这是允许的）
- `external_ref: true` 标记的依赖确实不在当前文件中
- manifest 中的 `card_count` 与实际文件一致

## 边界与非目标

- **不做**卡片生成 — 这是 knowledge-card-generator 的职责
- **不做**卡片内容修改或合并 — 只拆分，不改变卡片内容
- **不做**JSON 以外格式的拆分 — YAML 拆分不在本 skill 范围
- **不做**向量索引或图谱索引的拆分 — 只处理卡片集本身
- **不做**AI 上下文窗口计算或自动选择加载策略 — 只提供拆分文件和清单，加载策略由调用方决定
- 依赖输入 cards.json 的质量，若 cards.json 字段不完整则拆分结果同样不完整

## 验收标准

- 每个拆分文件可被标准 JSON 解析器直接解析，无语法错误
- 拆分文件中的卡片保持原有字段完整性（所有 fields 子字段存在）
- 跨文件依赖保留 `id` 引用并标记 `external_ref: true`
- `source` 字段指向原 cards.json
- manifest.json 中的 card_count 与实际文件一致
- 所有拆分文件的卡片总数（去重后）= 原 cards.json 的卡片数量
- 增量更新时未变更的文件不被重写
- 文件命名遵循 `module_<id>.json`、`feature_<id>.json`、`<type>.json` 规则
