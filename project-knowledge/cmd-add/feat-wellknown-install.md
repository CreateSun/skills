# Well-Known 端点安装模块

- **所属命令**: `skills add`
- **主要职责**: 处理来自任意域名 `/.well-known/skills/index.json` 端点发现的技能，与普通 GitHub 安装流程并行但独立
- **关键入口**: `handleWellKnownSkills(source, url, options, spinner)` — 在 `parseSource` 返回 `type='well-known'` 时触发

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  WK01["parseSource 识别 well-known 类型"] --> WK02["wellKnownProvider.fetchAllSkills(url)"]
  WK02 --> WK03{找到技能？}
  WK03 -- 0 个 --> WK04["报错退出"]
  WK03 -- 有 --> WK05{options.list？}
  WK05 -- 是 --> WK06["打印技能列表后退出"]
  WK05 -- 否 --> WK07{options.skill 指定？}
  WK07 -- "*" --> WK08["全选"]
  WK07 -- 指定名称 --> WK09["过滤匹配"]
  WK07 -- 未指定，单个 --> WK10["自动选唯一"]
  WK07 -- 未指定，多个，-y --> WK11["全选"]
  WK07 -- 未指定，多个 --> WK12["展示 multiselect UI"]
  WK08 --> WK13["进入 agent/scope/mode 选择流程（同 add 主流程）"]
  WK09 --> WK13
  WK10 --> WK13
  WK11 --> WK13
  WK12 --> WK13
  WK13 --> WK14["installWellKnownSkillForAgent() 执行安装"]
  WK14 --> WK15["写入 skill-lock / local-lock"]
  WK15 --> WK16["promptForFindSkills()"]
```

## 关键依赖

- `src/providers/index.ts` → `wellKnownProvider.fetchAllSkills(url)`
- `src/installer.ts` → `installWellKnownSkillForAgent()`

## 涉及代码映射

- **组件与文件**：
  - `handleWellKnownSkills` / `src/add.ts`
  - `wellKnownProvider` / `src/providers/index.ts`
- **关键函数**：
  - `wellKnownProvider.fetchAllSkills(url)` — 从 `/.well-known/skills/index.json` 拉取
  - `installWellKnownSkillForAgent(skill, agent, options)` — Well-Known 专用安装方法
  - `wellKnownProvider.getSourceIdentifier(url)` — 获取规范化 source 标识符
