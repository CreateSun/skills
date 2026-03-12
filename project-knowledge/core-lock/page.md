# core-lock 锁文件管理模块说明

- **文件**:
  - `src/skill-lock.ts` — 全局锁文件管理
  - `src/local-lock.ts` — 本地（项目级）锁文件管理
- **模块角色**: 维护两套锁文件体系，追踪已安装技能的来源、版本 hash 等信息，支持 check/update/restore 功能

## 全局锁文件（skill-lock.ts）

- **路径**: `~/.agents/.skill-lock.json`
- **版本**: v3（字段 `skillFolderHash` 为 GitHub Trees API SHA）
- **用途**: 全局安装的 skill 记录，供 `skills check` 和 `skills update` 使用

### 关键接口

| 函数 | 说明 |
|------|------|
| `addSkillToLock(name, entry)` | 添加/更新 skill 条目 |
| `removeSkillFromLock(name)` | 删除 skill 条目 |
| `getSkillFromLock(name)` | 读取指定 skill 的锁条目 |
| `getAllLockedSkills()` | 返回所有锁定的 skill 列表 |
| `fetchSkillFolderHash(source, path, token)` | 调用 GitHub Trees API 获取 skill 目录最新 hash |
| `getGitHubToken()` | 从环境变量或 gitconfig 获取 GitHub Token |
| `isPromptDismissed(key)` | 检查某个提示是否已被用户忽略 |
| `dismissPrompt(key)` | 标记某个提示为已忽略 |
| `getLastSelectedAgents()` | 获取上次安装时选择的 agent 列表（持久化） |
| `saveSelectedAgents(agents)` | 保存本次选择的 agent 列表 |

### 锁文件数据结构

```typescript
interface SkillLockFile {
  version: number;           // 当前为 3
  skills: Record<string, SkillLockEntry>;
  dismissed?: DismissedPrompts;  // 已忽略的提示
  lastSelectedAgents?: string[]; // 上次选择的 agent
}

interface SkillLockEntry {
  source: string;           // 规范化来源标识（如 "owner/repo"）
  sourceType: string;       // "github" | "local" | "git" | "well-known"
  sourceUrl: string;        // 原始安装 URL
  skillPath?: string;       // 技能在仓库中的相对路径
  skillFolderHash: string;  // GitHub tree SHA（用于更新检测）
  installedAt: string;      // ISO 时间戳
  updatedAt: string;        // ISO 时间戳
  pluginName?: string;      // 所属 plugin 名称
}
```

## 本地锁文件（local-lock.ts）

- **路径**: `<project>/skills-lock.json`（提交到 git）
- **用途**: 项目级安装记录，供 `skills experimental_install` 恢复使用

### 关键接口

| 函数 | 说明 |
|------|------|
| `addSkillToLocalLock(name, entry, cwd)` | 添加/更新本地锁条目 |
| `readLocalLock(cwd)` | 读取本地锁文件 |
| `computeSkillFolderHash(dir)` | 计算技能目录的内容 hash（用于变更检测） |

## 关键流程列表

- [Hash 检测更新流程](./feat-hash-check.md)
