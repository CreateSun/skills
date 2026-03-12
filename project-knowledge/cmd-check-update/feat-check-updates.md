# 检查更新模块（skills check）

- **所属命令**: `skills check`
- **主要职责**: 读取全局锁文件，对每个可检查的技能通过 GitHub Trees API 获取最新 hash，对比后输出是否有更新，不执行实际更新操作
- **关键入口**: `runCheck(args)` / `src/cli.ts`

## 逻辑流程（Mermaid）

```mermaid
flowchart TD
  CK01["用户执行 skills check"] --> CK02["readSkillLock() 读取全局锁文件"]
  CK02 --> CK03{有已安装技能？}
  CK03 -- 否 --> CK04["输出空提示，退出"]
  CK03 -- 是 --> CK05["getGitHubToken() 获取 GitHub Token"]
  CK05 --> CK06["遍历所有 skill 条目"]
  CK06 --> CK07{skill 可检查？}
  CK07 -- 本地/git/无hash --> CK08["加入 skipped 列表"]
  CK07 -- 可检查 --> CK09["fetchSkillFolderHash(source, skillPath, token)"]
  CK09 --> CK10{获取 hash 失败？}
  CK10 -- 是 --> CK11["加入 errors 列表"]
  CK10 -- 否 --> CK12{hash 与记录不同？}
  CK12 -- 相同 --> CK13["无更新"]
  CK12 -- 不同 --> CK14["加入 updates 列表"]
  CK13 --> CK15["汇总输出结果"]
  CK14 --> CK15
  CK08 --> CK15
  CK11 --> CK15
  CK15 --> CK16["printSkippedSkills() 输出无法检查列表"]
  CK16 --> CK17["上报遥测 event: check"]
```

## 关键依赖

- `src/skill-lock.ts` → `fetchSkillFolderHash(source, skillPath, token)`：调用 GitHub Trees API
- `getGitHubToken()`：从 `GITHUB_TOKEN` 或 `~/.gitconfig` 中读取 Token

## 涉及代码映射

- **组件与文件**：
  - `runCheck(args)` / `src/cli.ts`
  - `readSkillLock()` / `src/cli.ts`
  - `getSkipReason(entry)` / `src/cli.ts`
  - `printSkippedSkills(skipped)` / `src/cli.ts`
- **关键函数**：
  - `fetchSkillFolderHash(source, skillPath, token)` / `src/skill-lock.ts` — GitHub Trees API 调用
  - `getGitHubToken()` — 获取 Token 提高 API 速率限制
- **关键状态字段**：
  - `entry.skillFolderHash`：记录的 GitHub tree SHA
  - `entry.skillPath`：技能在仓库中的相对路径
  - `updates`：有更新的技能列表
  - `skipped`：无法自动检查的技能列表

## 节点索引表

| ID | 节点说明 | 类型 |
|----|---------|------|
| CK01 | 用户执行 `skills check` | 开始节点 |
| CK02 | 读取全局锁文件 | 处理节点 |
| CK09 | GitHub Trees API 获取最新 hash | API 节点 |
| CK12 | 比对 hash 判断是否有更新 | 决策节点 |
| CK14 | 确认有更新 | 处理节点 |
| CK17 | 上报遥测 | API 节点 |
