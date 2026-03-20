---
name: project-skills-creator
description: |
  skills 项目上下文。这是一个 Claude Code Skill 开发项目，用于开发、维护和版本管理 skills。
  在此项目中开发或修改 skill 时，使用此 skill 了解项目结构和工作流程。
  当用户提到"skill 开发"、"修改 skill"、"更新 skill"、"同步 skill" 时主动使用。
---

# Skills 项目上下文

> 由 project-onboard 生成于 2026-03-18

## 项目概述

Claude Code Skill 开发项目。在本地开发、编辑 skill 文件，然后同步到 `~/.claude/skills/` 使其生效。

## 目录结构

```
skills/
├── .claude/
│   └── skills/              ← skill 源文件（在这里编辑）
│       ├── skills/          ← 本项目的 project skill
│       ├── project-onboard/ ← 项目接入分析 skill
│       ├── code-review/     ← 代码 CR skill
│       └── dev-flow/        ← 完整需求开发流水线 skill
└── src/                     ← 预留（暂未使用）
```

## Skill 文件结构

每个 skill 是一个目录，包含 `SKILL.md`：

```
SKILL.md 结构：
---
name: skill-name
description: |
  触发描述（影响 AI 何时自动使用此 skill）
---

# Skill 正文内容（Markdown，指导 AI 的行为）
```

## 工作流程

### 开发 skill

1. 在 `.claude/skills/{skill-name}/SKILL.md` 中编辑
2. 测试：在新对话中触发 skill，验证行为
3. 同步到全局生效（见下方命令）

### 同步命令

```bash
# 同步单个 skill 到全局
cp .claude/skills/code-review/SKILL.md ~/.claude/skills/code-review/SKILL.md

# 同步所有 skill 到全局
for skill in .claude/skills/*/; do
  name=$(basename "$skill")
  mkdir -p ~/.claude/skills/$name
  cp "$skill/SKILL.md" ~/.claude/skills/$name/SKILL.md
done

# 从全局拉取最新（覆盖本地）
for skill in project-onboard code-review dev-flow; do
  cp ~/.claude/skills/$skill/SKILL.md .claude/skills/$skill/SKILL.md
done
```

### 新增 skill

1. 在 `.claude/skills/` 下创建新目录
2. 创建 `SKILL.md`，填写 frontmatter（name + description）和正文
3. 同步到 `~/.claude/skills/`

## 关键路径

| 用途 | 路径 |
|------|------|
| 本地 skill 源文件 | `.claude/skills/{name}/SKILL.md` |
| 全局 skill（生效位置） | `~/.claude/skills/{name}/SKILL.md` |
| project-onboard skill | `.claude/skills/project-onboard/SKILL.md` |
| code-review skill | `.claude/skills/code-review/SKILL.md` |
| dev-flow skill | `.claude/skills/dev-flow/SKILL.md` |