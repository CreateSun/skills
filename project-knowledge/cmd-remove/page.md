# cmd-remove 命令说明

- **命令**: `skills remove [skills...]` （别名：`rm`, `r`）
- **入口文件**: `src/remove.ts` → `removeCommand(skillNames, options)`，由 `src/cli.ts` 路由
- **命令角色**: 移除已安装的技能，支持交互式选择或按名称直接移除，可作用于全局或项目级

## 功能模块一览

- **技能扫描**：扫描安装目录（canonical 目录 + 各 agent 目录）获取已安装技能名称集合
- **交互式选择**：无参数时展示 multiselect UI 让用户勾选要删除的技能
- **按名称删除**：指定名称时直接匹配并删除
- **文件删除**：从 canonical 目录和各 agent 目录删除技能目录或符号链接
- **锁文件清理**：从 `~/.agents/.skill-lock.json` 移除对应条目
- **遥测上报**

## 关键流程列表

- [技能删除执行流程](./feat-remove-skill.md)
