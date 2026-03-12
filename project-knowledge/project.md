# skills CLI — 项目知识库总览

## 项目简介

`skills` 是一个开放式 Agent Skills 生态的命令行工具（CLI），允许用户从 GitHub 仓库、URL 或本地路径安装、管理和更新供 AI Agent（如 Claude Code、Cursor、Codex、Amp 等）使用的技能包（SKILL.md 文件）。技能安装方式支持符号链接（symlink）和文件复制（copy）两种模式，全局安装（~/.agents/skills）或项目级安装（.agents/skills）。

## 命令总览

| 命令 | 别名 | 说明 | 文档 |
|------|------|------|------|
| `skills add <pkg>` | `a`, `i`, `install` | 安装技能（从 GitHub、URL 或本地路径） | [cmd-add](./cmd-add/page.md) |
| `skills remove [skills]` | `rm`, `r` | 移除已安装的技能 | [cmd-remove](./cmd-remove/page.md) |
| `skills list` | `ls` | 列出已安装的技能 | [cmd-list](./cmd-list/page.md) |
| `skills find [query]` | `search`, `f`, `s` | 搜索并安装技能 | [cmd-find](./cmd-find/page.md) |
| `skills check` | — | 检查技能是否有更新 | [cmd-check-update](./cmd-check-update/page.md) |
| `skills update` | `upgrade` | 更新所有技能到最新版本 | [cmd-check-update](./cmd-check-update/page.md) |
| `skills init [name]` | — | 创建新的 SKILL.md 模板 | [cmd-init](./cmd-init/page.md) |
| `skills experimental_install` | — | 从 skills-lock.json 恢复技能 | [cmd-experimental-install](./cmd-experimental-install/page.md) |
| `skills experimental_sync` | — | 从 node_modules 同步技能 | [cmd-experimental-sync](./cmd-experimental-sync/page.md) |

## 核心模块索引

| 模块 | 文件 | 说明 |
|------|------|------|
| 安装器 | `src/installer.ts` | 技能安装核心逻辑（symlink/copy/路径安全） | [core-installer](./core-installer/page.md) |
| Agent 定义 | `src/agents.ts` | 所有支持的 Agent 配置与检测 | [core-agents](./core-agents/page.md) |
| 全局锁文件 | `src/skill-lock.ts` | `~/.agents/.skill-lock.json` 管理 | [core-lock](./core-lock/page.md) |
| 本地锁文件 | `src/local-lock.ts` | `skills-lock.json`（提交到仓库）管理 | [core-lock](./core-lock/page.md) |
| 技能发现 | `src/skills.ts` | 扫描目录、解析 SKILL.md | — |
| 源解析 | `src/source-parser.ts` | 解析 GitHub 短链、完整 URL、本地路径 | — |
| Git 操作 | `src/git.ts` | clone 仓库 | — |
| 遥测 | `src/telemetry.ts` | 匿名使用统计上报 | — |
| Providers | `src/providers/` | 远程技能源（GitHub、HuggingFace、Mintlify、WellKnown） | — |

## 跨模块业务链路索引

### 技能安装链路（add 命令）

```
用户执行 skills add <source>
  → source-parser.ts 解析 source 类型
  → git.ts clone 仓库（远程）/ 直接使用本地路径
  → skills.ts 发现所有 SKILL.md
  → 用户交互选择：skill / agent / scope / mode
  → installer.ts 执行安装（symlink 或 copy）
  → skill-lock.ts 写入全局锁（global 安装）
  → local-lock.ts 写入本地锁（project 安装）
  → telemetry.ts 上报遥测
```

### 技能更新链路（check/update 命令）

```
用户执行 skills check / skills update
  → 读取 ~/.agents/.skill-lock.json
  → 对每个 skill 调用 skill-lock.ts fetchSkillFolderHash（GitHub Trees API）
  → 比对 skillFolderHash 判断是否有更新
  → update 命令：调用 spawnSync npx skills add <url> -g -y 重装
```

### 技能恢复链路（experimental_install）

```
用户执行 skills experimental_install
  → install.ts 读取 skills-lock.json
  → 对每个 skill 调用 runAdd 重新安装
```

### 从 node_modules 同步链路（experimental_sync）

```
用户执行 skills experimental_sync
  → sync.ts 遍历 node_modules 查找 SKILL.md
  → 交互选择 skill / agent
  → installer.ts 安装
  → local-lock.ts 写入本地锁
```

## 支持的 Agent 列表（截至当前）

| Agent | 本地 skillsDir | 全局 globalSkillsDir |
|-------|---------------|---------------------|
| Amp | `.agents/skills` | `~/.config/agents/skills` |
| Antigravity | `.agent/skills` | `~/.gemini/antigravity/skills` |
| Augment | `.augment/skills` | `~/.augment/skills` |
| Claude Code | `.claude/skills` | `~/.claude/skills` |
| Cline | `.cline/skills` 或 `.cursor/skills` | — |
| Codex | `.codex/skills` 或 `~/.codex/skills` | — |
| Cursor | `.cursor/skills` | — |
| OpenCode | `.agents/skills` | `~/.config/agents/skills`（Universal） |
| Replit | — | — |
| ... | ... | ... |

> Universal Agent（如 OpenCode）始终安装到规范目录 `.agents/skills`，其他 agent 通过 symlink 指向此目录。

## 锁文件说明

| 文件 | 路径 | 用途 |
|------|------|------|
| 全局锁文件 | `~/.agents/.skill-lock.json` | 记录全局安装的 skill，用于 check/update |
| 本地锁文件 | `<project>/skills-lock.json` | 记录项目级安装的 skill，可提交到 git |

锁文件当前版本为 **v3**，关键字段 `skillFolderHash`（GitHub Trees API 的 SHA）用于检测更新。旧版本会被清空，需重新安装。
