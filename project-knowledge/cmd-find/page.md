# cmd-find 命令说明

- **命令**: `skills find [query]` （别名：`search`, `f`, `s`）
- **入口文件**: `src/find.ts` → `runFind(args)`，由 `src/cli.ts` 路由
- **命令角色**: 在 skills.sh 注册表中搜索技能，支持关键词搜索（非交互式）和交互式键盘导航（TUI），用户可直接安装搜索结果中的技能

## 功能模块一览

- **API 搜索**：`searchSkillsAPI(query)` 调用 `https://skills.sh/api/search` 搜索技能
- **交互式 TUI**：使用 readline 实现实时搜索、上下键导航、Enter 安装的终端 UI
- **结果展示**：显示技能名称、slug、来源、安装次数
- **安装触发**：选中技能后调用 `runAdd()` 执行安装流程

## 关键流程列表

- [技能搜索与安装流程](./feat-find-and-install.md)
