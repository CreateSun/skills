# 技能搜索与安装模块

- **所属命令**: `skills find`
- **主要职责**: 调用 skills.sh 搜索 API 获取技能列表，提供交互式 TUI 让用户浏览并安装
- **关键入口**: `runFind(args)` / `src/find.ts`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  FD01["用户执行 skills find [query]"] --> FD02{有 query 参数？}
  FD02 -- 有 --> FD03["searchSkillsAPI(query) 调用搜索 API"]
  FD02 -- 无 --> FD04["展示交互式 TUI（实时搜索）"]
  FD03 --> FD05{有结果？}
  FD05 -- 否 --> FD06["输出未找到提示"]
  FD05 -- 是 --> FD07["格式化展示搜索结果"]
  FD07 --> FD08["用户选择（Enter 安装 / q 退出）"]
  FD04 --> FD08
  FD08 --> FD09{用户操作}
  FD09 -- 选择技能 --> FD10["runAdd([skill.source]) 触发安装流程"]
  FD09 -- 退出 --> FD11["退出"]
  FD10 --> FD12["上报遥测 event: find"]
```

## 关键依赖

- `src/find.ts` → `searchSkillsAPI(query)`：GET `https://skills.sh/api/search?q=<query>&limit=10`
- `src/add.ts` → `runAdd()`, `parseAddOptions()`

## 涉及代码映射

- **组件与文件**：
  - `runFind(args)` / `src/find.ts`
  - `searchSkillsAPI(query)` / `src/find.ts`
- **关键函数**：
  - `formatInstalls(count)` — 格式化安装次数（K/M 后缀）
  - `runAdd()` — 触发安装
- **关键状态字段**：
  - `SEARCH_API_BASE`：可通过 `SKILLS_API_URL` 环境变量覆盖
