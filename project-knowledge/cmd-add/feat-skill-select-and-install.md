# 技能选择与安装执行模块

- **所属命令**: `skills add`
- **主要职责**: 基于已发现的 skill 列表，通过交互式 UI 让用户选择 skill、目标 agent、安装 scope 和 mode，然后执行实际安装并写入锁文件
- **关键入口**: `runAdd()` 中 skill 选择 → agent 选择 → scope 选择 → mode 选择 → `installSkillForAgent()`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  SEL01["扫描完成，得到 skills 列表"] --> SEL02{options.skill 指定？}
  SEL02 -- "*" --> SEL03["全选所有 skill"]
  SEL02 -- 指定名称 --> SEL04["filterSkills() 过滤匹配"]
  SEL02 -- 未指定，单个 --> SEL05["自动选择唯一 skill"]
  SEL02 -- 未指定，多个 --> SEL06["展示 multiselect UI 选择"]
  SEL03 --> SEL07["异步拉取安全审计数据（非阻塞）"]
  SEL04 --> SEL07
  SEL05 --> SEL07
  SEL06 --> SEL07
  SEL07 --> SEL08{options.agent 指定？}
  SEL08 -- "*" --> SEL09["目标 = 所有 agent"]
  SEL08 -- 指定名称 --> SEL10["验证 agent 合法性"]
  SEL08 -- 未指定 --> SEL11["detectInstalledAgents()"]
  SEL11 --> SEL12{已检测到的 agent 数量}
  SEL12 -- 0 个 --> SEL13["展示 agent 选择 UI"]
  SEL12 -- 1 个 --> SEL14["自动选择 + 加入 Universal Agents"]
  SEL12 -- 多个 --> SEL15["展示搜索式 multiselect UI"]
  SEL09 --> SEL16{是否询问 scope？}
  SEL10 --> SEL16
  SEL13 --> SEL16
  SEL14 --> SEL16
  SEL15 --> SEL16
  SEL16 -- options.global 已定 --> SEL17["跳过 scope 选择"]
  SEL16 -- 未定，且支持 global --> SEL18["展示 scope 选择（Project/Global）"]
  SEL17 --> SEL19{是否询问 mode？}
  SEL18 --> SEL19
  SEL19 -- options.copy 或 options.yes --> SEL20["使用 symlink/copy"]
  SEL19 -- 未指定 --> SEL21["展示 mode 选择 UI"]
  SEL20 --> SEL22["展示 Installation Summary + 安全审计"]
  SEL21 --> SEL22
  SEL22 --> SEL23{options.yes？}
  SEL23 -- 否 --> SEL24["p.confirm 确认安装"]
  SEL23 -- 是 --> SEL25["installSkillForAgent() 执行安装"]
  SEL24 -- 确认 --> SEL25
  SEL24 -- 取消 --> SEL26["退出"]
  SEL25 --> SEL27["写入 skill-lock / local-lock"]
  SEL27 --> SEL28["上报遥测"]
  SEL28 --> SEL29["提示安装 find-skills（首次）"]
```

## 关键依赖

- `src/installer.ts` → `installSkillForAgent()`, `isSkillInstalled()`, `getCanonicalPath()`
- `src/agents.ts` → `detectInstalledAgents()`, `agents`, `getUniversalAgents()`
- `src/skill-lock.ts` → `addSkillToLock()`, `fetchSkillFolderHash()`
- `src/local-lock.ts` → `addSkillToLocalLock()`, `computeSkillFolderHash()`
- `src/telemetry.ts` → `fetchAuditData()`, `track()`

## 涉及代码映射

- **组件与文件**：
  - `runAdd` / `src/add.ts`（主流程，约 900 行）
  - `handleWellKnownSkills` / `src/add.ts`（well-known 分支）
  - `selectAgentsInteractive` / `src/add.ts`（agent 选择 UI）
  - `promptForFindSkills` / `src/add.ts`（安装后提示）
- **关键函数**：
  - `filterSkills(skills, names)` — 模糊匹配 skill 名称
  - `detectInstalledAgents()` — 检测本机已安装的 agent
  - `installSkillForAgent(skill, agent, options)` — 核心安装
  - `ensureUniversalAgents(agents)` — 保证 Universal Agent 始终包含
- **关键状态字段**：
  - `selectedSkills`：用户最终选择的 skill 列表
  - `targetAgents`：目标 agent 列表
  - `installGlobally`：是否全局安装
  - `installMode`：`'symlink' | 'copy'`
  - `tempDir`：临时目录（finally 清理）

## 节点索引表

| ID | 节点说明 | 类型 |
|----|---------|------|
| SEL01 | 技能发现完成 | 开始节点 |
| SEL07 | 异步拉取安全审计（非阻塞） | API 节点 |
| SEL11 | `detectInstalledAgents()` 检测本机 agent | API 节点 |
| SEL22 | 展示安装摘要（含安全评级） | 处理节点 |
| SEL25 | `installSkillForAgent()` 执行安装 | API 节点 |
| SEL27 | 写入 skill-lock / local-lock | 处理节点 |
| SEL28 | 上报遥测 | API 节点 |
| SEL29 | 提示安装 find-skills | 处理节点 |
