# Symlink 安装模块

- **所属模块**: `src/installer.ts`
- **主要职责**: 以符号链接方式安装技能，Universal Agent 直接使用 canonical 目录（`.agents/skills/<name>`），其他 agent 在各自的 skillsDir 中创建指向 canonical 目录的 symlink
- **关键入口**: `installSkillForAgent(skill, agentType, options)` 当 `mode === 'symlink'`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  SL01["installSkillForAgent(skill, agent, options)"] --> SL02["sanitizeName(skill.name)"]
  SL02 --> SL03["getCanonicalPath(name, options) 计算 canonical 路径"]
  SL03 --> SL04["isPathSafe(canonicalBase, canonicalPath) 路径安全校验"]
  SL04 -- 不安全 --> SL05["报错返回"]
  SL04 -- 安全 --> SL06["cp -r skill.path → canonical 目录"]
  SL06 --> SL07{isUniversalAgent(agent)？}
  SL07 -- 是 --> SL08["完成（Universal Agent 直接用 canonical 目录）"]
  SL07 -- 否 --> SL09["getInstallPath(name, agent, options) 计算 agent 安装路径"]
  SL09 --> SL10["isPathSafe() 校验 agent 安装路径"]
  SL10 -- 不安全 --> SL05
  SL10 -- 安全 --> SL11["symlink(canonicalPath, agentInstallPath)"]
  SL11 --> SL12{symlink 失败？}
  SL12 -- EPERM/ENOTSUP（Windows） --> SL13["降级 copy 并设 symlinkFailed=true"]
  SL12 -- 成功 --> SL14["返回 InstallResult success=true"]
  SL13 --> SL14
```

## 关键依赖

- `sanitizeName(name)` — 防路径穿越，转换为 kebab-case
- `isPathSafe(base, target)` — 验证目标路径在预期目录内
- `isUniversalAgent(agentType)` — 判断是否为 Universal Agent

## 涉及代码映射

- **关键函数**：
  - `installSkillForAgent(skill, agentType, { global, mode })` / `src/installer.ts`
  - `sanitizeName(name)` / `src/installer.ts`
  - `isPathSafe(basePath, targetPath)` / `src/installer.ts`
  - `getCanonicalPath(name, options)` / `src/installer.ts`
  - `getInstallPath(name, agentType, options)` / `src/installer.ts`
- **关键状态字段**：
  - `InstallResult.symlinkFailed`：symlink 失败时降级为 copy
  - `InstallResult.canonicalPath`：canonical 安装目录
  - `InstallResult.mode`：实际使用的安装模式

## 节点索引表

| ID | 节点说明 | 类型 |
|----|---------|------|
| SL01 | 开始安装 | 开始节点 |
| SL02 | 路径名称清洗 | 处理节点 |
| SL04 | 路径安全校验 | 决策节点 |
| SL06 | 复制技能到 canonical 目录 | 处理节点 |
| SL07 | 判断是否为 Universal Agent | 决策节点 |
| SL11 | 创建 symlink | 处理节点 |
| SL12 | symlink 是否成功 | 决策节点 |
| SL13 | 降级为 copy（Windows 兼容） | 异常节点 |
