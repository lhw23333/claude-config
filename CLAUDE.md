# 全局开发工作流规范

## 五层工具链架构

优先使用内置能力，避免不必要的 MCP 依赖。

### 第一层: MCP (仅外部工具集成)

- **Playwright**: 浏览器自动化、E2E 测试、网页截图
- **GitHub**: PR/Issue/Actions 管理、代码搜索、安全扫描
- 其他需求优先用 Agent/Skill/Hooks/Bash 解决

### 第二层: Agent 使用规范

**内置 Agent:**

| 场景 | Agent 类型 | 说明 |
|------|-----------|------|
| 代码搜索、文件定位 | `Explore` | 替代 Context7 等文档 MCP |
| 架构设计、复杂推理 | `Plan` | 替代 Sequential Thinking MCP |
| 代码 Review、多步重构 | `general-purpose` | 替代 Code Review MCP |
| Claude API/SDK 问题 | `claude-code-guide` | 替代文档类 MCP |

**自定义校验 Agent (`~/.claude/agents/`):**

| Agent | 用途 | 使用时机 |
|-------|------|---------|
| `feature-validator` | 需求匹配 + TypeCheck + Lint + Test + Build 全链路校验 | 功能开发完成后 |
| `code-reviewer` | 独立上下文代码 review (安全/逻辑/质量/性能) | commit 前 |
| `test-runner` | 自动检测测试框架并运行全部测试 | 任何时候验证代码 |

### 第三层: Skill 使用规范

| Skill | 用途 | 替代 |
|-------|------|------|
| `/simplify` | 3 并行 Agent 代码质量审查 + 自动修复 | ESLint/SonarQube MCP |
| `/validate` | 完整校验流水线 (lint→type→test→review) | CI/CD MCP |
| `/commit` | 生成规范化 commit | Auto-Commit MCP |
| `/claude-api` | AI API 开发指导 | 无 |

### 第四层: Hooks (自动触发)

| Hook | 事件 | 作用 |
|------|------|------|
| lint 检查 | `PreToolUse` (git commit) | commit 前自动运行 lint |
| commitlint | `PostToolUse` (git commit) | commit 后校验消息格式，不合规则提示修正 |
| 桌面通知 | `Notification` | Windows Toast 通知任务完成 |

### 第五层: Bash CLI 工具

- `git rebase/merge/worktree/flow` — 分支管理
- `npm run lint` — 代码校验
- `commitlint` — commit 消息规范校验（已全局安装）
- 项目级: Husky + lint-staged（在各项目中按需初始化）

## 功能校验流程

完成功能开发后，按以下顺序执行：

1. `/validate` — 运行完整校验流水线
2. 修复所有 blocking issues
3. `/simplify` — 3 Agent 并行审查 + 自动修复代码质量
4. `/commit` — 生成规范 commit (Hook 自动运行 lint)

快速校验（小改动）：直接 `/commit`，Hook 会自动拦截 lint 问题。

## Git 工作流

- Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- Parent rebase: `git fetch upstream && git rebase upstream/main`
- 分支策略: main (稳定) / dev (开发) / feature/* (功能)

## 代码风格

- TypeScript strict mode
- 使用项目已有的 ESLint 配置，不额外引入
- 优先编辑现有文件，避免创建不必要的新文件
