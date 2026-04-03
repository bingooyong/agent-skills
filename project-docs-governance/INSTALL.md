# 安装到 Cursor 本地 Skills

## 方式一：复制到 Cursor 用户目录（推荐）

Cursor 会自动加载 **用户级** skills 目录下的所有技能，在所有项目中可用：

```bash
# 确保目录存在
mkdir -p ~/.cursor/skills

# 从 agent-skills 仓库根目录复制（把路径换成你本机克隆位置）
cp -R /path/to/agent-skills/project-docs-governance ~/.cursor/skills/
```

若你已在包含本 skill 的 monorepo 中，且 skill 位于 `.cursor/skills/project-docs-governance/`，也可：

```bash
cp -R .cursor/skills/project-docs-governance ~/.cursor/skills/
```

## 方式二：使用项目内 skill（无需安装）

将本目录放到 **目标项目** 的 `.cursor/skills/project-docs-governance/` 下，在该项目内打开 Cursor 时，该 skill 作为**项目 skill** 可用，无需复制到 `~/.cursor/skills/`。

## Cursor 如何发现 Skills

| 位置 | 作用域 |
|------|--------|
| `~/.cursor/skills/<skill-name>/` | **用户级**：所有项目均可使用 |
| `<项目根>/.cursor/skills/<skill-name>/` | **项目级**：仅在该项目中可用 |

无需在 Cursor 设置里手动启用；只要目录存在且内含 `SKILL.md`，Cursor 会在对话中根据描述自动匹配合适的 skill。

## 验证是否生效

重启 Cursor 或新开一个对话，在涉及「文档审计、生成/补全项目文档、文档缺口分析」等场景下 @ 本 skill 或直接提问，Agent 会优先使用 project-docs-governance 的规则与流程。
