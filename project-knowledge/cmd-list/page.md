# cmd-list 命令说明

- **命令**: `skills list` （别名：`ls`）
- **入口文件**: `src/list.ts` → `runList(args)`，由 `src/cli.ts` 路由
- **命令角色**: 列出当前已安装的技能，支持按作用域（全局/项目）和 agent 过滤

## 功能模块一览

- **参数解析**：`parseListOptions(args)` 解析 `-g`/`--global`、`-a`/`--agent` 选项
- **技能列举**：`listInstalledSkills(options)` 从安装目录读取已安装技能列表（来自 `installer.ts`）
- **锁文件对比**：`getAllLockedSkills()` 从全局锁文件读取 source 信息，与文件系统结果合并展示
- **格式化输出**：按 agent 分组展示，显示 skill 名称、安装路径、来源等信息

## 关键流程列表

- [技能列表查询流程](./feat-list-skills.md)
