# agent-skills

可复用的 AI Agent 技能集合。每个技能独立目录，可直接安装到 Claude Code、Cursor、Codex 等 agent 环境。

## 技能列表

| 技能 | 说明 |
|------|------|
| [knowledge-consolidation](skills/knowledge-consolidation/) | 任务完成后回顾工作，判断哪些内容应沉淀到 AGENTS.md、memory、项目文档，输出具体补丁 |
| [project-doc-generator](skills/project-doc-generator/) | 基于规格化输入生成项目文档体系，关注依赖关系、生成顺序、一致性、增量更新 |
| [agents-md-maintainer](skills/agents-md-maintainer/) | 维护 AGENTS.md 的结构和内容，确保只有长期规则写入，避免临时上下文污染 |
| [memory-governance](skills/memory-governance/) | 管理 agent 记忆文件：分类、去重、验证、治理，防止记忆膨胀和过期 |
| [hooks-designer](skills/hooks-designer/) | 设计 agent hooks：触发时机、执行动作、失败策略、回退方案 |
| [project-docs-governance](project-docs-governance/) | 企业级项目文档治理：规划、生成、补全与维护完整文档体系 |
| [code-review-checklist](skills/code-review-checklist/) | 生成结构化 code review 检查清单，按语言/框架/变更类型定制 |
| [error-pattern-library](skills/error-pattern-library/) | 积累常见错误模式和解决方案，支持按错误特征检索 |
| [api-design-reviewer](skills/api-design-reviewer/) | 审查 API 设计的一致性、向后兼容、安全性和可扩展性 |
| [refactoring-planner](skills/refactoring-planner/) | 分析代码坏味道，生成重构计划，评估影响范围和风险 |
| [dependency-analyzer](skills/dependency-analyzer/) | 分析依赖关系图谱，识别循环依赖、过时依赖、安全风险 |
| [test-strategy-designer](skills/test-strategy-designer/) | 根据代码变更分析测试影响面，设计测试策略 |
| [commit-message-craftsman](skills/commit-message-craftsman/) | 基于变更内容生成规范的 commit message |
| [onboarding-guide-generator](skills/onboarding-guide-generator/) | 从项目结构和配置自动生成新成员上手指南 |

## 安装

### 前置条件

安装 [skills CLI](https://github.com/nicepkg/skills)（如果尚未安装）：

```bash
npm install -g @nicepkg/skills
```

### 安装全部技能

```bash
npx skills add bingooyong/agent-skills
```

### 列出可用技能

```bash
npx skills add bingooyong/agent-skills --list
```

### 安装单个技能

```bash
npx skills add bingooyong/agent-skills --skill knowledge-consolidation
```

### 安装多个指定技能

```bash
npx skills add bingooyong/agent-skills --skill knowledge-consolidation --skill memory-governance
```

### 手动安装（不使用 skills CLI）

将目标技能目录复制到 agent 的 skills 目录：

```bash
# Claude Code
cp -R skills/<skill-name> ~/.claude/skills/

# Cursor
cp -R skills/<skill-name> ~/.cursor/skills/

# 项目级安装（所有 agent 通用）
cp -R skills/<skill-name> .claude/skills/    # Claude Code
cp -R skills/<skill-name> .cursor/skills/    # Cursor
```

## 技能之间的协作

这些技能设计为可协同工作：

```
knowledge-consolidation（知识沉淀）
  ↓ 判断落点
  ├→ agents-md-maintainer（写入 AGENTS.md）
  ├→ memory-governance（写入 memory）
  └→ project-doc-generator（写入项目文档）

hooks-designer（设计 hooks）← 可与所有技能配合

code-quality 生态（代码质量相关技能互相配合）:
code-review-checklist ← 输入给 → api-design-reviewer, refactoring-planner
error-pattern-library ← 输入给 → code-review-checklist, dependency-analyzer, refactoring-planner
test-strategy-designer ← 输入给 → refactoring-planner, api-design-reviewer

refactoring-planner ← 依赖 → dependency-analyzer（耦合分析驱动重构）
commit-message-craftsman（独立，与 code-review-checklist 轻度关联）
onboarding-guide-generator ← 引用 → dependency-analyzer, project-doc-generator
```

## 目录结构

```
agent-skills/
├── README.md                        # 本文件
├── AGENTS.md                        # 仓库级规范
├── .gitignore
├── skills/                          # 所有技能
│   ├── knowledge-consolidation/
│   │   └── SKILL.md
│   ├── project-doc-generator/
│   │   └── SKILL.md
│   ├── agents-md-maintainer/
│   │   └── SKILL.md
│   ├── memory-governance/
│   │   └── SKILL.md
│   └── hooks-designer/
│       └── SKILL.md
│   ├── code-review-checklist/
│   │   └── SKILL.md
│   ├── error-pattern-library/
│   │   └── SKILL.md
│   ├── api-design-reviewer/
│   │   └── SKILL.md
│   ├── refactoring-planner/
│   │   └── SKILL.md
│   ├── dependency-analyzer/
│   │   └── SKILL.md
│   ├── test-strategy-designer/
│   │   └── SKILL.md
│   ├── commit-message-craftsman/
│   │   └── SKILL.md
│   └── onboarding-guide-generator/
│       └── SKILL.md
├── project-docs-governance/         # 独立 skill（含完整 references/templates）
│   └── SKILL.md
├── references/                      # 跨 skill 共享参考资料
├── templates/                       # 跨 skill 共享模板
│   └── skill-template.md            # 新 skill 模板
└── scripts/                         # 辅助脚本
```

## 贡献

1. Fork 本仓库
2. 基于 `templates/skill-template.md` 创建新技能
3. 遵循 `AGENTS.md` 中的规范
4. 提交 PR

### 新增技能前请确认

- 该技能有长期复用价值（不仅用于一次性任务）
- 与现有技能不重叠
- 命名使用小写短横线格式
- 包含完整的 SKILL.md（有 frontmatter、触发条件、执行步骤、边界说明）

## 版本演进

- **v0.x** — 初始技能集合，持续迭代
- **v1.0** — 技能接口稳定，新增技能不再破坏已有技能
- **v1.x** — 增量新增技能，已有技能优化但不改变核心接口

## 许可

MIT
