# core-installer 核心模块说明

- **文件**: `src/installer.ts`
- **模块角色**: 技能安装的底层执行层，负责路径计算、安全校验、symlink/copy 安装、技能枚举等所有与文件系统操作直接相关的逻辑

## 功能模块一览

- **路径安全**：`sanitizeName(name)` 防止路径穿越攻击，`isPathSafe()` 验证目标路径在预期目录内
- **路径计算**：`getCanonicalSkillsDir()`, `getCanonicalPath()`, `getInstallPath()`, `getAgentBaseDir()` 计算各种安装路径
- **Symlink 安装**：为 Universal Agent 安装到 canonical 目录，为其他 agent 创建符号链接
- **Copy 安装**：将技能目录文件复制到各 agent 的 skillsDir
- **已安装检测**：`isSkillInstalled()` 检查 skill 是否已在某 agent 中安装
- **技能枚举**：`listInstalledSkills()` 遍历安装目录返回已安装技能列表
- **WellKnown 安装**：`installWellKnownSkillForAgent()` 安装来自 well-known 端点的技能（含多文件支持）

## 关键流程列表

- [Symlink 安装流程](./feat-symlink-install.md)
- [Copy 安装流程](./feat-copy-install.md)
