# 全局开发工作流规范（Web / TypeScript 栈）

面向 Web/TS 开发的七层工具链，强调 **Spec 可执行化** — 业务、接口、行为三层规格都要能被机器验证，而不是自然语言 TODO。

## 七层工具链架构

优先使用内置能力，避免不必要的 MCP 依赖。

### 第零层: Spec (规格层)

在 brainstorm 之后、write-plan 之前强制产出 **可验证的规格**。

| 层级 | 形态 | 载体 | 可验证性 |
|------|------|------|---------|
| 业务规格 | Gherkin `Given-When-Then` | `specs/NNN-*.spec.md` | 人工 + LLM review |
| 接口规格 | 代码即规格 | **Zod** schema + TS types + tRPC/OpenAPI | 编译器 + 运行时 |
| 行为规格 | 测试即规格 | Vitest + **fast-check** + Playwright + Storybook | CI 自动化 |

**核心原则:** 越靠下约束力越强。Spec 编程成熟度 = 下沉到可执行层的比例。

**强制顺序**: 业务 spec → Zod schema → 类型 → 测试骨架 → 实现。`spec-writer` agent 保证这个顺序不被跳过。

### 第一层: MCP (仅外部集成)

| MCP | 用途 |
|------|------|
| **Playwright** | 浏览器自动化、E2E 测试、视觉回归 |
| **GitHub** | PR/Issue/Actions、代码搜索、安全扫描 |
| **chrome-devtools** | 性能 profiling、网络分析、Lighthouse |
| ~~SSH~~ | 按需启用，Web 栈本地开发极少用 |

其他需求优先用 Plugin/Agent/Skill/Hooks/Bash 解决。

### 第二层: Plugin

| Plugin | 来源 | 说明 |
|--------|------|------|
| `superpowers` | obra/superpowers-marketplace | 敏捷开发全流程框架 |

**Superpowers 核心技能:**

| 技能 | 用途 | 触发时机 |
|------|------|---------|
| `brainstorming` | Socratic 需求细化 | 功能开发第 1 步 |
| `writing-plans` | 分解为 2-5 分钟任务 | spec 产出后 |
| `executing-plans` | 批量执行任务含检查点 | 计划就绪后 |
| `test-driven-development` | RED-GREEN-REFACTOR | 实现阶段 |
| `systematic-debugging` | 4 阶段根因分析 | 遇到 bug 时 |
| `requesting-code-review` | 5 维度代码审查 | 实现完成后 |
| `using-git-worktrees` | 创建隔离工作树 | 功能开发前 |
| `finishing-a-development-branch` | 合并/PR/清理 | 分支完成时 |
| `subagent-driven-development` | 并行子代理开发 | 复杂功能 |
| `verification-before-completion` | 证据优先完成确认 | 声明完成前 |

### 第三层: Agent

**内置 Agent:**

| 场景 | Agent | 说明 |
|------|-------|------|
| 代码搜索、文件定位 | `Explore` | 替代 Context7 等文档 MCP |
| 架构设计、复杂推理 | `Plan` | 替代 Sequential Thinking MCP |
| 通用多步任务 | `general-purpose` | 代码 Review、重构 |
| Claude API/SDK 问题 | `claude-code-guide` | 替代文档类 MCP |

**自定义 Agent (`~/.claude/agents/`):**

| Agent | 用途 | 使用时机 |
|-------|------|---------|
| **`spec-writer`** | 从 brainstorm 产出业务/接口/行为三层 spec，强制 Zod-first | 写计划之前 |
| `feature-validator` | 需求匹配 + TypeCheck + Lint + Test + Build 全链路校验 | 功能开发完成后 |
| `test-runner` | 自动检测测试框架并运行全部测试 | 任何时候验证代码 |
| **`type-coverage`** | type-coverage + ts-prune + `any` 泄漏审计 | commit 前 |
| **`bundle-analyzer`** | vite/webpack bundle 尺寸、tree-shaking、依赖膨胀 | 发布前 |
| **`a11y-auditor`** | axe-core 可访问性扫描 + WCAG 合规 | UI 改动后 |
| **`visual-regression`** | Playwright 截图 diff，检测 UI 非预期变更 | UI 改动后 |
| **`dead-code-hunter`** | knip + ts-prune 死代码/未用依赖清理 | 定期清理 |
| `deployment-engineer` | CI/CD、GitOps、Docker/K8s 部署自动化 | 远程部署任务 |
| `github-analyzer` | 九维度开源项目评分与相似项目推荐 | 调研参考项目 |

### 第四层: Skill

| Skill | 用途 | 替代 |
|-------|------|------|
| **`/spec`** | 创建/更新可执行 spec 文件（业务 + Zod + 测试骨架） | Kiro/Spec Kit |
| **`/contract`** | 从 spec 生成 Zod/tRPC/OpenAPI 契约 | 手写 schema |
| **`/e2e`** | 从 spec 自动生成 Playwright 测试 | 手写 E2E |
| **`/perf`** | Lighthouse + bundle 尺寸 + Web Vitals 审计 | 独立 Lighthouse CI |
| **`/preview`** | 部署到 Vercel/Netlify preview 并回贴 PR | Preview Deploy MCP |
| `/validate` | 完整校验流水线 (lint→type→test→review) | CI/CD MCP |
| `/simplify` | 3 并行 Agent 代码质量审查 + 自动修复 | ESLint/SonarQube MCP |
| `/commit` | 生成规范化 commit | Auto-Commit MCP |
| `/claude-api` | AI API 开发指导 | 无 |

### 第五层: Hooks

| Hook | 事件 | 作用 |
|------|------|------|
| biome 自动 fix | `PostToolUse` (Write/Edit `*.ts`,`*.tsx`,`*.js`,`*.jsx`) | 每次写入后自动 format + lint fix |
| lint 检查 | `PreToolUse` (git commit) | commit 前运行 `npm run lint` |
| type-coverage 阈值 | `PreToolUse` (git commit) | `type-coverage --at-least 98` 兜底 |
| commitlint | `PostToolUse` (git commit) | 校验消息格式 |
| 桌面通知 | `Notification` | Windows Toast |

**哲学**: Hook = 防护网，不是约束。自动化越多，人工纪律要求越少。

### 第六层: Bash CLI 工具链

- **`biome`** — 替代 ESLint + Prettier（快 10×，一条命令）
- `tsx` / `bun` — 快速执行 .ts 文件
- `type-coverage` — 类型覆盖率指标
- `knip` / `ts-prune` — 死代码检测
- `syncpack` — monorepo 依赖版本对齐
- `fast-check` — Property-Based Testing
- `changeset` — 版本与 changelog 自动化
- `commitlint` — commit 消息规范校验（全局安装）
- `git rebase/merge/worktree` — 分支管理

项目级: Husky + lint-staged（按需初始化）。

---

## 功能开发全流程（7 段式）

```
brainstorm → spec → write-plan → contract → TDD → validate+perf+a11y → commit+preview
```

### 完整流程

1. **`/brainstorm`** — Socratic 需求细化，产出设计决策记录
2. **`/spec`** — 产出可执行规格三件套：
   - `specs/NNN-*.spec.md`（业务：Given-When-Then + 验收标准）
   - Zod schema 骨架（接口契约）
   - 测试骨架（行为 spec）
3. **`/write-plan`** — 分解为精确任务列表（引用 spec ID）
4. 创建 worktree（`using-git-worktrees` 自动触发）
5. **`/contract`** — 从 spec 生成 Zod / tRPC / OpenAPI 类型
6. **TDD 实现**（RED-GREEN-REFACTOR，spec 中的测试先失败）
7. **三合一校验**：
   - `/validate` — lint + type + test + review
   - `/perf` — Lighthouse + bundle
   - `a11y-auditor` agent — 可访问性
8. 修复所有 blocking issues
9. **`/simplify`** — 3 Agent 并行审查 + 自动修复
10. **`/commit`** — 规范 commit（Hook 自动跑 lint + type-coverage）
11. **`/preview`** — 部署 preview 并贴 PR
12. `finishing-a-development-branch` — 合并/PR/清理

### 快速流程（小改动）

直接 `/commit`，Hook 会自动拦截 lint / type-coverage / commitlint 问题。

### 调试流程

遇到 bug 时，`superpowers:systematic-debugging` 自动触发 4 阶段根因分析。

---

## Spec 编程能力提升指南

**核心观点**：Spec 编程的瓶颈不是写不写 spec，而是 spec 能否被机器验证。自然语言 spec 对 LLM 没有约束力，可执行的 spec 才有。

### 五个铁律

1. **Zod-first 数据流** — 所有跨边界数据（API、localStorage、URL params、form、env）**必须**先有 Zod schema，再 `z.infer<typeof Schema>` 得类型。把 TS 的"编译时幻觉"变成"运行时保证"。

2. **Spec ID 强制追溯** — 每个 commit message 必须含 `spec:NNN` 引用；废弃的 spec 加 `@deprecated` frontmatter，不删除。

3. **Property-Based Testing 常规化** — 纯函数 / 状态机 / 序列化逻辑必须有 `fast-check` 属性测试，作为 TDD GREEN 之后的额外 gate。

4. **Spec 漂移检测** — `spec-drift-detector` 定期扫描 spec 引用的符号是否还存在、类型是否还匹配。超阈值阻断 commit。

5. **Design by Contract 落地** — 用 `tiny-invariant` / `ts-assert` 在函数入口出口写 pre/post 条件，失败直接崩而非默默错。

### 进阶方向

- **状态机 Spec**：复杂 UI 状态用 `xstate` 显式建模，状态机本身就是 spec
- **TLA+ 轻量用**：核心并发/分布式逻辑写 TLA+ spec，平时 90% 代码不需要
- **参考范式**：GitHub Spec Kit、AWS Kiro 的 spec-driven development

---

## Git 工作流

- **Conventional Commits**: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`, `perf:`, `build:`, `ci:`
- **Spec 引用**: commit body 必须含 `spec:NNN`（spec 相关改动）
- **Parent rebase**: `git fetch upstream && git rebase upstream/main`
- **分支策略**: main (稳定) / feature/* (功能) / worktrees (隔离开发)
- **Changeset**: 版本 bump 用 `changeset` 而非手改 package.json

---

## 代码风格

- **TypeScript strict mode** 必开，`noUncheckedIndexedAccess` 推荐
- **Biome** 作为 format + lint 唯一入口（替代 ESLint+Prettier）
- **type-coverage ≥ 98%**，禁止 `any` 无注释逃逸
- **Zod first**：跨边界数据必有 schema
- 优先编辑现有文件，避免创建不必要的新文件
- 导入路径用 `@/` alias，避免深层相对路径
