# core-agents Agent 定义模块说明

- **文件**: `src/agents.ts`
- **模块角色**: 定义所有支持的 AI Agent 的配置，提供 Agent 检测、Universal Agent 判断等工具函数

## 功能模块一览

- **Agent 配置注册**：`agents` 对象包含所有支持的 agent 的 name、displayName、skillsDir、globalSkillsDir、detectInstalled 函数
- **Agent 检测**：`detectInstalledAgents()` 并行检测本机已安装的 agent
- **Universal Agent**：`getUniversalAgents()` 返回"通用"agent 列表（使用 `.agents/skills` 规范目录），`isUniversalAgent()` 判断
- **分组工具**：`getNonUniversalAgents()` 获取非通用 agent 列表（需要 symlink 指向规范目录）

## 已支持 Agent 列表

| Agent Key | displayName | skillsDir（项目级） | 是否 Universal |
|-----------|-------------|---------------------|----------------|
| `amp` | Amp | `.agents/skills` | 是 |
| `antigravity` | Antigravity | `.agent/skills` | — |
| `augment` | Augment | `.augment/skills` | — |
| `claude-code` | Claude Code | `.claude/skills` | — |
| `openclaw` | OpenClaw | `skills` | — |
| `cline` | Cline | `.cline/skills` 或 `.cursor/skills` | — |
| `codex` | Codex | `.codex/skills` | — |
| `cursor` | Cursor | `.cursor/skills` | — |
| `opencode` | OpenCode | `.agents/skills` | 是 |
| `replit` | Replit | — | — |
| ... | ... | ... | ... |

> Universal Agent 的 skillsDir 与 canonical 目录（`.agents/skills`）一致，安装时不需额外创建 symlink。

## 涉及代码映射

- **关键函数**：
  - `detectInstalledAgents()` — 并行调用每个 agent 的 `detectInstalled()` 函数，返回已安装列表
  - `getUniversalAgents()` — 过滤出 Universal Agent（skillsDir === `.agents/skills`）
  - `getNonUniversalAgents()` — 过滤出非 Universal Agent
  - `isUniversalAgent(agentType)` — 判断某 agent 是否为 Universal
  - `getOpenClawGlobalSkillsDir()` — OpenClaw 全局目录的特殊逻辑（多路径检测）
- **关键配置字段**：
  - `AgentConfig.skillsDir`：项目级安装目录（相对路径）
  - `AgentConfig.globalSkillsDir`：全局安装目录（绝对路径）
  - `AgentConfig.detectInstalled`：检测函数，检查特定配置目录是否存在
