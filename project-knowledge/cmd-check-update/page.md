# cmd-check / cmd-update 命令说明

- **命令**:
  - `skills check` — 检查技能是否有更新（只读）
  - `skills update` / `skills upgrade` — 检查并自动更新所有技能
- **入口文件**: `src/cli.ts` → `runCheck()` / `runUpdate()`（均在 `cli.ts` 中实现）
- **命令角色**: 读取全局锁文件，对每个 skill 通过 GitHub Trees API 获取最新 `skillFolderHash`，与已记录的 hash 对比，`update` 命令自动重装有更新的技能

## 功能模块一览

- **锁文件读取**：读取 `~/.agents/.skill-lock.json` 获取所有已安装技能的 `skillFolderHash`
- **可检查性过滤**：跳过本地路径、git URL 或无 hash 的技能，输出"无法自动检查"提示
- **Hash 对比**：调用 `fetchSkillFolderHash()` 获取 GitHub Trees API 最新 hash，与记录值对比
- **更新展示**（check）：列出有更新的技能名称和来源
- **自动重装**（update）：对有更新的技能调用 `spawnSync npx skills add <url> -g -y` 重装

## 关键流程列表

- [检查更新流程](./feat-check-updates.md)
- [执行更新流程](./feat-run-update.md)
