# 全局开发工作流规范

## 六层工具链架构

优先使用内置能力，避免不必要的 MCP 依赖。

### 第一层: MCP (仅外部工具集成)

- **Playwright**: 浏览器自动化、E2E 测试、网页截图
- **GitHub**: PR/Issue/Actions 管理、代码搜索、安全扫描
- **SSH**: 远程服务器管理 (classfang/ssh-mcp-server, MCP 官方收录)
- 其他需求优先用 Plugin/Agent/Skill/Hooks/Bash 解决

### 第二层: Plugin (第三方技能插件)

| Plugin | 来源 | 说明 |
|--------|------|------|
| `superpowers` | obra/superpowers-marketplace | 敏捷开发全流程框架 (brainstorming, TDD, debugging, code-review, git-worktrees) |

**Superpowers 核心技能:**

| 技能 | 用途 | 触发方式 |
|------|------|---------|
| `brainstorming` | Socratic 需求细化，产出设计文档 | 功能开发前 |
| `writing-plans` | 分解为 2-5 分钟任务，含精确文件路径 | 设计通过后 |
| `executing-plans` | 批量执行计划任务，含检查点 | 计划就绪后 |
| `test-driven-development` | RED-GREEN-REFACTOR 严格 TDD | 实现阶段 |
| `systematic-debugging` | 4 阶段根因分析 | 遇到 bug 时 |
| `requesting-code-review` | 5 维度代码审查 (质量/架构/测试/需求/生产就绪) | 实现完成后 |
| `using-git-worktrees` | 创建隔离工作树，并行开发 | 功能开发前 |
| `finishing-a-development-branch` | 合并/PR/清理工作流 | 分支完成时 |
| `subagent-driven-development` | 并行子代理开发 | 复杂功能 |
| `verification-before-completion` | 证据优先的完成确认 | 声明完成前 |

### 第三层: Agent 使用规范

**内置 Agent:**

| 场景 | Agent 类型 | 说明 |
|------|-----------|------|
| 代码搜索、文件定位 | `Explore` | 替代 Context7 等文档 MCP |
| 架构设计、复杂推理 | `Plan` | 替代 Sequential Thinking MCP |
| 代码 Review、多步重构 | `general-purpose` | 通用任务 |
| Claude API/SDK 问题 | `claude-code-guide` | 替代文档类 MCP |

**自定义校验 Agent (`~/.claude/agents/`):**

| Agent | 用途 | 使用时机 |
|-------|------|---------|
| `feature-validator` | 需求匹配 + TypeCheck + Lint + Test + Build 全链路校验 | 功能开发完成后 |
| `test-runner` | 自动检测测试框架并运行全部测试 | 任何时候验证代码 |
| `deployment-engineer` | CI/CD、GitOps、Docker/K8s 部署自动化 (来源: wshobson/agents) | 远程部署任务 |

### 第四层: Skill 使用规范

| Skill | 用途 | 替代 |
|-------|------|------|
| `/simplify` | 3 并行 Agent 代码质量审查 + 自动修复 | ESLint/SonarQube MCP |
| `/validate` | 完整校验流水线 (lint→type→test→superpowers review) | CI/CD MCP |
| `/commit` | 生成规范化 commit | Auto-Commit MCP |
| `/claude-api` | AI API 开发指导 | 无 |

### 第五层: Hooks (自动触发)

| Hook | 事件 | 作用 |
|------|------|------|
| lint 检查 | `PreToolUse` (git commit) | commit 前自动运行 lint |
| commitlint | `PostToolUse` (git commit) | commit 后校验消息格式 |
| 桌面通知 | `Notification` | Windows Toast 通知任务完成 |

### 第六层: Bash CLI 工具

- `git rebase/merge/worktree/flow` — 分支管理
- `npm run lint` — 代码校验
- `commitlint` — commit 消息规范校验（已全局安装）
- 项目级: Husky + lint-staged（在各项目中按需初始化）

## 功能开发全流程

### 完整流程（中大型功能）

1. `/brainstorm` — Socratic 需求细化，产出设计文档
2. `/write-plan` — 分解为精确任务列表
3. 创建 worktree (superpowers 自动触发)
4. TDD 实现 (superpowers RED-GREEN-REFACTOR)
5. `/validate` — 完整校验流水线
6. 修复所有 blocking issues
7. `/simplify` — 3 Agent 并行审查 + 自动修复代码质量
8. `/commit` — 生成规范 commit (Hook 自动运行 lint)
9. finishing-a-development-branch — 合并/PR/清理

### 快速流程（小改动）

直接 `/commit`，Hook 会自动拦截 lint 问题。

### 调试流程

遇到 bug 时，superpowers:systematic-debugging 自动触发 4 阶段根因分析。

## Git 工作流

- Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- Parent rebase: `git fetch upstream && git rebase upstream/main`
- 分支策略: main (稳定) / feature/* (功能) / worktrees (隔离开发)

## 代码风格

- TypeScript strict mode
- 使用项目已有的 ESLint 配置，不额外引入
- 优先编辑现有文件，避免创建不必要的新文件
