# agent-skills

本仓库收集可在 Cursor（及其他兼容 Agent Skills 的环境）中使用的技能包，每个技能为独立目录，根文件为 `SKILL.md`。

## 仓库地址

```text
git@github.com:bingooyong/agent-skills.git
```

```bash
git clone git@github.com:bingooyong/agent-skills.git
cd agent-skills
```

## 技能列表

| 技能目录 | 说明 |
|----------|------|
| [`project-docs-governance/`](project-docs-governance/) | 企业级项目文档治理：规划、生成、补全与维护完整文档体系；与 Memory ROOT 对齐；文档缺口与质量检查。 |

详细安装与验证步骤见各技能目录下的 [`INSTALL.md`](project-docs-governance/INSTALL.md)。

## 快速安装（用户级，全项目可用）

将某一技能复制到 Cursor 用户 skills 目录后，所有项目均可自动加载该技能：

```bash
mkdir -p ~/.cursor/skills
cp -R /path/to/agent-skills/project-docs-governance ~/.cursor/skills/
```

将 `/path/to/agent-skills` 换为你本机克隆本仓库的路径。

## 项目级安装

在目标项目根目录：

```bash
mkdir -p .cursor/skills
cp -R /path/to/agent-skills/project-docs-governance .cursor/skills/
```

## 许可与贡献

各技能包内如有单独许可或说明，以该目录内文件为准。
