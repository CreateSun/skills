# cmd-add 命令说明

- **命令**: `skills add <source>` （别名：`a`, `i`, `install`）
- **入口文件**: `src/add.ts` → `runAdd(args, options)`，由 `src/cli.ts` 中 `main()` 路由分发
- **命令角色**: 技能安装的核心命令，支持从 GitHub、HTTPS URL、本地路径安装一个或多个 skill

## 功能模块一览

- **源解析**：解析用户输入的 source 字符串，识别 GitHub 短链/完整 URL/本地路径/well-known 端点
- **仓库克隆**：远程源调用 `git.ts` 克隆到临时目录
- **技能发现**：扫描目录下所有 SKILL.md，过滤内部技能，支持按 plugin 分组
- **安全审计**：异步拉取 skills.sh 安全评估数据（非阻塞）
- **交互选择**：提示用户选择 skill、agent、安装 scope（global/project）、安装 mode（symlink/copy）
- **执行安装**：调用 `installer.ts` 为每个 agent 执行安装
- **锁文件写入**：全局安装写 `~/.agents/.skill-lock.json`，项目安装写 `skills-lock.json`
- **遥测上报**：成功安装后上报匿名统计
- **find-skills 提示**：首次安装后提示用户是否安装 find-skills 辅助技能

## 关键流程列表

- [源解析与仓库克隆流程](./feat-source-and-clone.md)
- [技能选择与安装执行流程](./feat-skill-select-and-install.md)
- [Well-Known 端点安装流程](./feat-wellknown-install.md)
