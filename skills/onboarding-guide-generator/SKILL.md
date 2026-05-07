---
name: onboarding-guide-generator
description: "Generate onboarding guides for new team members from project structure, configuration, and conventions. Use when (1) creating getting-started guides from project artifacts, (2) generating environment setup instructions, (3) documenting team workflows and toolchain, (4) keeping onboarding docs in sync. Outputs structured onboarding documents with prerequisites, setup steps, and common tasks."
---

# 上手指南生成器 Skill

从项目结构、配置文件和团队约定自动生成新成员上手指南，确保新成员能快速搭建环境并理解项目。

## 何时使用

- 项目初始化时创建上手指南
- 新成员加入团队，需要快速了解项目
- 项目环境或工具链发生重大变更
- 定期更新上手文档，保持与项目同步

**不适用**：API 文档、架构设计文档、与项目无关的培训材料。

## 上手指南章节结构

| 章节 | 内容来源 | 必需 |
|------|----------|------|
| 项目简介 | README / AGENTS.md | 是 |
| 前置条件 | engines 配置 / Dockerfile / 环境要求 | 是 |
| 环境搭建 | 安装脚本 / 配置文件 / Makefile | 是 |
| 项目结构说明 | 目录扫描 | 是 |
| 开发工作流 | scripts 配置 / CI 配置 / AGENTS.md | 是 |
| 常见任务 | Makefile targets / npm scripts | 否 |
| 故障排查 | error-pattern-library / FAQ | 否 |
| 进一步阅读 | 文档索引 / AGENTS.md | 否 |

## 输入

| 输入类型 | 说明 | 必需 |
|----------|------|------|
| 项目根目录路径 | 要生成指南的项目 | 是 |
| 目标读者角色 | 后端 / 前端 / 全栈 / 运维 | 否（默认全栈） |
| 已有上手指南 | 用于增量更新 | 否 |
| 特定关注点 | 用户要求强调的内容 | 否 |

## 输出

- 完整上手指南文档（Markdown）
- 与已有指南的变更摘要（增量更新时）
- 需要人工确认的占位符列表

## 执行步骤

### 1. 扫描项目结构

```
识别项目类型和工具链:
  IF 存在 package.json → Node.js 项目
    → 提取 scripts (dev/build/test/lint)
    → 提取 engines (node/npm 版本)
    → 提取 dependencies 和 devDependencies
    → 提取 packageManager 字段
  IF 存在 pyproject.toml / setup.py / requirements.txt → Python 项目
    → 提取 Python 版本要求
    → 提取依赖管理工具 (pip/poetry/pipenv)
  IF 存在 go.mod → Go 项目
    → 提取 Go 版本要求
  IF 存在 Cargo.toml → Rust 项目
  IF 存在 pom.xml / build.gradle → Java 项目

识别容器化和部署:
  IF 存在 Dockerfile → 容器化项目
    → 提取基础镜像
  IF 存在 docker-compose.yml → 多服务编排
    → 提取服务列表

识别 CI/CD:
  IF 存在 .github/workflows → GitHub Actions
  IF 存在 .gitlab-ci.yml → GitLab CI
  IF 存在 Jenkinsfile → Jenkins

识别代码质量工具:
  IF 存在 .eslintrc / eslint.config → ESLint
  IF 存在 .prettierrc / prettier.config → Prettier
  IF 存在 tsconfig.json → TypeScript
```

### 2. 提取关键配置

- 运行时版本要求（Node / Python / Go / Java）
- 包管理器（npm / yarn / pnpm / pip / poetry）
- 构建、测试、lint 命令
- 环境变量模板（.env.example）
- 数据库/缓存等外部服务依赖
- 证书/密钥等安全相关配置

### 3. 推断开发工作流

```
从项目配置推断工作流:
  IF 存在分支保护规则 → 推断分支策略
  IF 存在 commitlint 配置 → 推断 commit 规范
  IF 存在 .github/PULL_REQUEST_TEMPLATE → 推断 PR 流程
  IF 存在 AGENTS.md → 提取协作原则
  IF 存在 Makefile → 提取常用 make target
  IF 存在 CONTRIBUTING.md → 提取贡献流程
```

### 4. 生成指南内容

```markdown
# [项目名] 上手指南

## 项目简介
（从 README / AGENTS.md 提取，1-2 段）

## 前置条件
| 工具 | 版本要求 | 安装方式 |
|------|----------|----------|
| Node.js | >= 18.x | nvm install 18 |
| pnpm | >= 8.x | npm install -g pnpm |
| Docker | >= 20.x | [官网下载] |

## 环境搭建
### 1. 克隆项目
git clone <repo-url>
cd <project>

### 2. 安装依赖
pnpm install

### 3. 配置环境变量
cp .env.example .env
（说明需要修改的关键变量）

### 4. 启动开发服务
pnpm dev
（说明服务地址和验证方式）

## 项目结构
├── src/
│   ├── modules/       # 业务模块
│   ├── shared/        # 共享工具和组件
│   └── config/        # 配置文件
├── tests/             # 测试文件
├── docs/              # 项目文档
└── scripts/           # 辅助脚本

## 开发工作流
### 日常开发
1. 创建 feature branch
2. 开发 + 测试
3. 提交代码（commit 规范: ...）
4. 创建 PR

### 运行测试
pnpm test

### 代码检查
pnpm lint

## 常见任务
### 添加新功能
（项目特定的功能开发流程）

### 修复 Bug
（项目特定的 bug 修复流程）

## 故障排查
| 问题 | 解决方案 |
|------|----------|
| 安装依赖失败 | 检查 Node 版本，清除缓存重试 |
| 端口被占用 | 修改 .env 中的 PORT 配置 |

## 进一步阅读
- [架构文档](docs/architecture.md)
- [API 文档](docs/api.md)
- [贡献指南](CONTRIBUTING.md)
```

### 5. 输出文档

- 生成的完整指南内容
- 标记需要人工确认的占位符（无法从项目配置推断的内容）
- 与已有指南的 diff（增量更新时）

```
需要人工确认:
- [ ] Git 仓库地址
- [ ] 是否需要 VPN 访问
- [ ] 团队代码 review 流程
- [ ] 部署权限申请方式
```

## 边界与非目标

- **不做**项目文档体系生成 — 这是 project-doc-generator 的职责
- **不做**架构或设计文档 — 只生成上手指南
- **不做**API 文档 — API 设计审查是 api-design-reviewer 的职责
- **不做**团队管理流程文档
- 只生成上手指南，不做深度技术文档
- 不自动更新指南 — 只提供内容，更新由用户确认

## 验收标准

- 前置条件中的版本要求与项目配置文件一致
- 环境搭建步骤可以直接复制执行
- 项目结构说明覆盖了主要目录
- 无法自动推断的内容标记为占位符而非编造
- 指南适合目标读者角色（后端/前端/全栈/运维）
