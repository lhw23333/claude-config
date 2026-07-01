# 本地配置快照 — Current Local Skill/Agent Config

> 快照日期:2026-07-01 · 来源:`~/.claude/`(本机实际安装状态)
> 本文只**总结配置结构**,不含任何 token / cookie / 密钥;个人 skill 的内部标识(如飞书 wiki node token、真实姓名)已刻意省略。

本仓库的 `README.md` / `CLAUDE.md` 描述的是**目标配置(七层工具链、10 个 agent、6 个 skill)**;
本文记录的是**本机 `~/.claude/` 当前实际装了什么**。两者存在明显漂移,详见 §3。

---

## 1. 当前本地 Skills(2 个)

| Skill | 层级 | 作用 | 后端 / 依赖 |
|-------|------|------|-------------|
| **`agent-reach`** | L4 Skill | 互联网能力路由器:15 平台多后端搜索/阅读(全网调研、平台链接) | Exa / GitHub / YouTube / RSS / Jina(零配置);Twitter·Reddit·Facebook·Instagram·小红书 走 **OpenCLI**(复用 Chrome 登录态);B站走 **bili-cli** |
| **`perf-self-review`** | L4 Skill | 从飞书「周报」汇总生成个人绩效自评(半年/季度/年度) | `lark-cli`(飞书 app,需授权 token) |

- `agent-reach` 结构:`SKILL.md` + `references/{search,social,career,dev,web,video}.md`(按分类路由)。
- 两个 skill **均不在本仓库 `skills/` 目录中**(仓库收录的是 spec/contract/e2e/perf/preview/validate)。

## 2. 当前本地 Agents / Plugins / Settings

| 项 | 现状 |
|----|------|
| **自定义 Agents**（`~/.claude/agents/`) | **空** —— 未安装任何自定义 agent(仅内置 Explore / Plan / general-purpose / claude-code-guide 可用) |
| **启用的 Plugin** | `tikhub-plugin@tikhub-plugins` —— 社交媒体数据(抖音/TikTok/Instagram/小红书/YouTube/Twitter/Threads 等一系列数据获取 skill) |
| **settings.json 顶层键** | `env` · `enabledPlugins` · `extraKnownMarketplaces` · `theme` · `skipWorkflowUsageWarning` |
| **settings.json 未含** | 无 `mcpServers`、无 `hooks`、无 `permissions`(本地 settings 精简;MCP 如 claude-in-chrome 由运行时提供) |

## 3. 与仓库文档的漂移对账(Documented vs Installed)

| 维度 | 仓库文档(目标) | 本地实际(当前) | 状态 |
|------|----------------|----------------|------|
| 自定义 Agents | 10 个(spec-writer、type-coverage、a11y-auditor…) | 0 个 | ⚠️ 未安装 |
| Skills | 6 个(/spec /contract /e2e /perf /preview /validate) | 0 个(实为 agent-reach、perf-self-review) | ⚠️ 完全不同 |
| Plugin | `superpowers`(obra/superpowers-marketplace) | `tikhub-plugin` | ⚠️ 不一致 |
| MCP | Playwright / GitHub / chrome-devtools | settings 无 mcpServers | ⚠️ 未在 settings 固化 |
| Hooks | biome fix / commitlint / type-coverage 等 | settings 无 hooks | ⚠️ 未固化 |

**结论**:仓库当前是一份"理想配置蓝图",尚未在本机落地;而本机实际运行的是一套面向**联网调研 + 社媒数据 + 飞书绩效**的轻量配置。二者需要对账收敛。

## 4. 建议(择一)

1. **以本地为准**:把 `agent-reach`、`perf-self-review` 纳入仓库 `skills/`(需先脱敏 `perf-self-review` 的飞书 token/姓名),并更新 README 的分层表以反映真实安装。
2. **以仓库为准**:按 `README.md` 的安装步骤把 10 个 agent + 6 个 skill 复制进 `~/.claude/`,让本机向蓝图收敛。
3. **双轨并存**:蓝图配置与实用 skill 分区维护,在 README 标注"目标 vs 已装"。

## 5. 安全说明

- 本快照不含任何 GitHub PAT、飞书 token、平台 Cookie 或 API Key。
- `perf-self-review` 的原始文件含飞书内部 wiki 标识与个人信息,**未**上传;如需纳入仓库请先脱敏。
- `agent-reach` 为公开工具的路由文档,可按需完整纳入 `skills/`。
