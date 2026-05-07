---
name: hooks-designer
description: "Design agent hooks: identify hook-worthy moments, define trigger timing, actions, failure handling, and rollback. Use when (1) deciding if a workflow step should be a hook vs inline logic, (2) designing pre/post tool hooks, (3) defining failure strategies and fallback behavior, (4) auditing existing hooks for correctness. Distinguishes hook-worthy capabilities from non-hook capabilities."
---

# Hooks 设计 Skill

识别哪些环节适合增加 hooks，设计触发时机、执行动作、失败处理、回退策略。区分应该做成 hooks 的能力和不应该 hook 化的能力。

## 何时使用

- 想在 agent 工作流的特定环节自动执行某些操作
- 需要设计自动化检查、通知、转换等"旁路"逻辑
- 审计现有 hooks 是否合理、是否有遗漏
- 团队讨论"这个能力应该做成 hook 还是别的方式"

**不适用**：核心业务逻辑设计、用户界面设计、与 agent 工作流无关的自动化。

## Hooks 适用性判断

### 应该做成 hook 的场景

| 特征 | 示例 |
|------|------|
| 在固定时机触发 | 每次提交前、每次文件修改后、每次测试运行后 |
| 动作可独立于主流程 | lint 检查不影响代码生成逻辑 |
| 需要一致地应用于所有场景 | 所有 PR 都需要格式化 |
| 失败时有明确的处理策略 | lint 失败 → 阻止提交 |
| 跨项目可复用 | 同一个 hook 可用于多个项目 |

### 不应该做成 hook 的场景

| 特征 | 示例 | 应该怎么做 |
|------|------|------------|
| 只在特定上下文触发 | 只在某个项目的某次任务中需要 | 内联逻辑 |
| 依赖复杂的运行时状态 | 需要知道用户当前的心理模型 | 对话中手动执行 |
| 动作本身是核心流程 | 代码生成、文件编辑 | 主流程的一部分 |
| 无法定义明确的失败策略 | "感觉不对就停下来" | 人工判断 |
| 触发频率过高导致干扰 | 每次字符输入都触发 | 调整触发时机 |

## 输入

| 输入 | 说明 | 必需 |
|------|------|------|
| 工作流描述 | agent 的典型工作流程 | 是 |
| 痛点或需求 | 希望自动化的环节 | 是 |
| 现有 hooks | 已有的 hook 配置（若存在） | 否 |
| 技术约束 | 使用的 agent 平台、可用的 hook 类型 | 否 |

## 输出

- Hook 设计方案：每个 hook 的完整定义
- 触发时机与条件
- 执行动作与参数
- 失败处理策略
- 回退方案
- 适用性分析：为什么这个能力适合/不适合做成 hook
- 配置文件内容（可直接使用的 JSON/YAML）

## 执行步骤

### 1. 梳理工作流

- 列出 agent 的典型工作流步骤
- 标注每个步骤的输入、输出、可能的失败点
- 识别用户提到的痛点和期望自动化的环节

### 2. 识别 hook 候选点

对工作流中的每个环节，判断是否需要 hook：

```
1. 这个环节是否有固定的触发时机？ → 是：继续；否：不适合 hook
2. 触发时的动作是否明确可定义？ → 是：继续；否：不适合 hook
3. 动作是否独立于主流程？ → 是：继续；否：考虑内联
4. 失败时是否有明确处理策略？ → 是：继续；否：需补充策略
5. 这个 hook 是否跨项目可复用？ → 是：通用 hook；否：项目级 hook
```

### 3. 设计每个 hook

对每个确认的 hook，定义：

```yaml
hook-name: string              # 小写短横线命名
trigger:
  event: string                # 触发事件（如 pre-commit, post-edit）
  condition: string            # 触发条件（可选，如 "仅对 .ts 文件"）
action:
  command: string              # 执行的命令或动作
  timeout: number              # 超时时间（秒）
  params: object               # 参数（可选）
failure:
  strategy: block | warn | skip  # 失败策略
  message: string              # 失败时的提示消息
  retry: number                # 重试次数（可选）
rollback:
  enabled: boolean             # 是否有回退操作
  action: string               # 回退动作（可选）
metadata:
  scope: global | project      # 作用范围
  priority: number             # 执行优先级
  description: string          # 一句话说明
```

### 4. 失败策略设计

| 策略 | 适用场景 | 行为 |
|------|----------|------|
| **block** | 必须通过的检查（lint、安全扫描） | 阻止后续操作，显示错误 |
| **warn** | 建议但不强制（性能建议、风格提醒） | 显示警告，允许继续 |
| **skip** | 尽力而为的操作（通知、日志记录） | 失败时跳过，不影响主流程 |

### 5. 输出配置

生成可直接使用的配置内容：

```json
{
  "hooks": {
    "pre-commit": [
      {
        "command": "pnpm lint",
        "timeout": 30,
        "failure": "block",
        "message": "Lint 检查未通过，请修复后再提交"
      }
    ],
    "post-edit": [
      {
        "matcher": "*.ts",
        "command": "pnpm typecheck",
        "timeout": 60,
        "failure": "warn",
        "message": "类型检查发现问题"
      }
    ]
  }
}
```

## 边界与非目标

- **不做**具体的 shell 脚本编写（只设计 hook 的配置和策略）
- **不做**替代 agent 的核心工作流设计
- **不做**CI/CD pipeline 设计（这是不同的关注点，虽然概念相似）
- **不做**hooks 的运行时实现
- 专注于设计，实现交给具体的 agent 平台

## 验收标准

- 每个 hook 都有明确的触发时机、动作和失败策略
- 适用性判断有理有据，不是所有东西都设计成 hook
- 失败策略与 hook 的重要性匹配（关键检查用 block，建议性用 warn）
- 配置内容可直接复制到目标平台使用
- 设计方案中区分了通用 hook 和项目级 hook
