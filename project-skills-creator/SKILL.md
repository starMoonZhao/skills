---
name: project-skills-creator
description: |
  skills 项目上下文。这是一个 Claude Code Skill 开发项目，用于开发、维护和版本管理 skills。
  在此项目中开发或修改 skill 时，使用此 skill 了解项目结构和工作流程。
  当用户提到"skill 开发"、"修改 skill"、"更新 skill"、"skill development"、"edit skill" 时主动使用。
metadata:
  pattern: tool-wrapper
  domain: skill-development
---

# Skills 项目上下文

你是此 skill 开发项目的上下文助手。当在此项目中工作时，严格遵循以下项目结构和规范。

> 由 project-onboard 生成于 2026-03-18，更新于 2026-03-20

## 项目概述

Claude Code Skill 开发项目。在项目根目录下开发、编辑 skill 文件，然后通过 sync-skills 同步到 `~/.claude/skills/` 使其生效。

## 目录结构

```
skills/                          ← 项目根目录
├── .gitignore
├── code-review/SKILL.md         ← 代码 CR skill
├── dev-flow/SKILL.md            ← 完整需求开发流水线 skill
├── project-onboard/SKILL.md     ← 项目接入分析 skill
├── project-skills-creator/SKILL.md ← 本项目的 project skill（本文件）
├── skill-design-patterns/SKILL.md  ← Skill 设计模式规范
└── sync-skills/SKILL.md         ← Skill 同步工具
```

每个 skill 是一个目录，包含 `SKILL.md`。

## Skill 文件结构规范

```markdown
---
name: skill-name
description: |
  触发描述（影响 AI 何时自动使用此 skill，包含中英文触发词）
metadata:
  pattern: tool-wrapper | generator | reviewer | inversion | pipeline
  [其他 metadata 字段]
---

# Skill 标题

角色定义（"你是..." / "You are..."）

## 正文内容（Markdown，指导 AI 的行为）
```

关键规范：
- frontmatter 必须包含 `name`、`description`、`metadata`（含 pattern）
- 正文开头必须有角色定义
- description 中的触发词应覆盖中英文常见表述
- 参考 `skill-design-patterns` skill 选择合适的设计模式

## 工作流程

### 开发 skill

1. 在项目根目录下创建 `{skill-name}/SKILL.md`
2. 按 Skill 文件结构规范编写内容
3. 测试：在新对话中触发 skill，验证行为
4. 同步：使用 `/sync-skills` 或手动同步到全局

### 同步

使用 `sync-skills` skill 进行同步操作，不要手动 cp。直接说"同步 skill"即可。

### 新增 skill

1. 在项目根目录下创建新目录 `{skill-name}/`
2. 创建 `SKILL.md`，按规范填写 frontmatter 和正文
3. 使用 sync-skills 同步到 `~/.claude/skills/`

## 行为指令

当在此项目中工作时：
- **创建新 skill**：始终参考 `skill-design-patterns` skill 选择合适的设计模式
- **修改现有 skill**：先读取当前内容，理解其设计模式，保持模式一致性
- **同步操作**：优先使用 sync-skills skill，不要手动拼命令
- **质量检查**：完成编写后，对照 skill-design-patterns 中的质量检查清单自查

## 关键路径

| 用途 | 路径 |
|------|------|
| 本地 skill 源文件 | `{项目根目录}/{name}/SKILL.md` |
| 全局 skill（生效位置） | `~/.claude/skills/{name}/SKILL.md` |
