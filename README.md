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
