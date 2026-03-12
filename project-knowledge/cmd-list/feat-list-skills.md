# 技能列表查询模块

- **所属命令**: `skills list`
- **主要职责**: 读取文件系统中已安装的技能，结合锁文件 source 信息，格式化展示
- **关键入口**: `runList(args)` / `src/list.ts`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  LS01["用户执行 skills list [options]"] --> LS02["parseListOptions() 解析参数"]
  LS02 --> LS03["listInstalledSkills(options) 读取文件系统"]
  LS03 --> LS04["getAllLockedSkills() 读取全局锁文件"]
  LS04 --> LS05{有已安装技能？}
  LS05 -- 否 --> LS06["输出空提示信息"]
  LS05 -- 是 --> LS07["按 agent 分组格式化输出"]
  LS07 --> LS08["展示 skill 名称、路径、来源、更新时间"]
```

## 涉及代码映射

- **组件与文件**：
  - `runList(args)` / `src/list.ts`
  - `parseListOptions(args)` / `src/list.ts`
  - `listInstalledSkills(options)` / `src/installer.ts`
  - `getAllLockedSkills()` / `src/skill-lock.ts`
- **关键状态字段**：
  - `options.global`：是否查看全局安装
  - `options.agent`：按 agent 过滤
