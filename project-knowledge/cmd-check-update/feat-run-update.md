# 执行更新模块（skills update）

- **所属命令**: `skills update` / `skills upgrade`
- **主要职责**: 检测到有更新的技能后，自动调用 `npx skills add <url> -g -y` 重新安装最新版本
- **关键入口**: `runUpdate()` / `src/cli.ts`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  UP01["用户执行 skills update"] --> UP02["readSkillLock() 读取全局锁"]
  UP02 --> UP03{有已安装技能？}
  UP03 -- 否 --> UP04["提示安装，退出"]
  UP03 -- 是 --> UP05["遍历所有 skill，过滤可检查项"]
  UP05 --> UP06["fetchSkillFolderHash() 获取最新 hash"]
  UP06 --> UP07{hash 变化？}
  UP07 -- 否 --> UP08["跳过"]
  UP07 -- 是 --> UP09["加入 updates 列表"]
  UP09 --> UP10["构造安装 URL（含 skillPath subpath）"]
  UP10 --> UP11["spawnSync npx skills add <url> -g -y"]
  UP11 --> UP12{安装成功？}
  UP12 -- 是 --> UP13["successCount++"]
  UP12 -- 否 --> UP14["failCount++"]
  UP13 --> UP15["汇总输出结果"]
  UP14 --> UP15
  UP15 --> UP16["上报遥测 event: update"]
```

## 关键实现细节

- 更新 URL 构造规则：
  ```
  sourceUrl（去掉 .git）+ /tree/main/ + skillFolder
  例：https://github.com/owner/repo/tree/main/skills/my-skill
  ```
- 始终使用 `forceRefresh: true`（直接用 GitHub Trees API）确保 hash 准确

## 涉及代码映射

- **组件与文件**：
  - `runUpdate()` / `src/cli.ts`
- **关键函数**：
  - `spawnSync('npx', ['-y', 'skills', 'add', installUrl, '-g', '-y'])` — 子进程重装

## 节点索引表

| ID | 节点说明 | 类型 |
|----|---------|------|
| UP01 | 用户执行 `skills update` | 开始节点 |
| UP06 | GitHub Trees API 获取最新 hash | API 节点 |
| UP10 | 构造重装 URL | 处理节点 |
| UP11 | `spawnSync` 调用 `npx skills add` 重装 | API 节点 |
| UP16 | 上报遥测 | API 节点 |
