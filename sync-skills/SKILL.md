---
name: sync-skills
description: |
  Sync skills between current project root directory, project's .claude/skills/ directory,
  and Codex global ~/.codex/skills/ directory.
  Use when user says: "同步skill", "sync skill", "同步一下", "推送skill",
  "同步所有skill", "同步某个skill", "拉取skill", "pull skill", "更新本地skill".
  Also use when user wants to sync skills to .claude/skills/ or ~/.codex/skills/,
  pull from either target back to project, or check sync status.
metadata:
  pattern: pipeline
  steps: "3"
---

# Sync Skills

你是一个 skill 同步工具，负责在当前项目根目录下的 skill 开发目录、项目的 `.claude/skills/` 目录、以及 Codex 全局 `~/.codex/skills/` 目录之间同步。严格按以下步骤执行。

## 路径约定

| 用途 | 路径 |
|------|------|
| skill 开发目录（源） | 当前项目根目录下的各 `{skill-name}/`（含 SKILL.md、references/、assets/ 等） |
| Claude 生效目录（目标） | 当前项目根目录下的 `.claude/skills/{skill-name}/` |
| Codex 生效目录（目标） | 当前用户目录下的 `~/.codex/skills/{skill-name}/` |

默认规则：
- 用户只说"同步 skill / sync skill"而未指定目标时，默认同时同步到 `.claude/skills/` 和 `~/.codex/skills/`
- 用户明确说"同步到 claude"时，只操作 `.claude/skills/`
- 用户明确说"同步到 codex"时，只操作 `~/.codex/skills/`
- 用户说"查看状态"时，默认同时展示两个目标的状态
- 用户说"拉取"时，必须明确来源目录；如果用户没说清楚，先展示状态并询问是从 `.claude/skills/` 还是 `~/.codex/skills/` 拉回开发目录

## 第一步：确定目录

检测当前项目根目录下是否有 skill 子目录，以及 `.claude/skills/`、`~/.codex/skills/` 是否存在：

```bash
# 检测开发目录中的 skill
echo "=== 开发目录中的 skill ==="
for d in */; do
  [ -f "$d/SKILL.md" ] && echo "  $d"
done

# 检测 .claude/skills/ 目录
if [ -d ".claude/skills" ]; then
  echo "=== .claude/skills/ 已存在 ==="
  ls .claude/skills/
else
  echo ".claude/skills/ 不存在，将自动创建"
  mkdir -p .claude/skills
fi

# 检测 ~/.codex/skills/ 目录
if [ -d "$HOME/.codex/skills" ]; then
  echo "=== ~/.codex/skills/ 已存在 ==="
  ls "$HOME/.codex/skills/"
else
  echo "~/.codex/skills/ 不存在，将自动创建"
  mkdir -p "$HOME/.codex/skills"
fi
```

## 第二步：根据用户意图选择操作

### 推送（开发目录 → 目标目录）：同步 / 推送 / push

先确定目标目录：

```bash
TARGETS=()

# 默认：同时同步到 .claude 和 ~/.codex
TARGETS+=(".claude/skills")
TARGETS+=("$HOME/.codex/skills")

# 如果用户明确指定单一目标，就只保留对应目录
# Claude only:
TARGETS=(".claude/skills")
# Codex only:
TARGETS=("$HOME/.codex/skills")
```

**同步所有 skill：**
```bash
for DEST_DIR in "${TARGETS[@]}"; do
  mkdir -p "$DEST_DIR"
  echo "=== 同步到 $DEST_DIR ==="
  for skill_dir in */; do
    name=$(basename "$skill_dir")
    [ "$name" = ".claude" ] && continue
    [ "$name" = ".codex" ] && continue
    [ "$name" = ".git" ] && continue
    [ "$name" = ".idea" ] && continue
    if [ -f "$skill_dir/SKILL.md" ]; then
      rm -rf "$DEST_DIR/$name"
      cp -r "$skill_dir" "$DEST_DIR/$name"
      echo "✓ 已同步: $name -> $DEST_DIR"
    fi
  done
done
echo "同步完成"
```

**同步单个 skill（用户指定了名称）：**
```bash
name="skill-name"   # 替换为实际 skill 名
for DEST_DIR in "${TARGETS[@]}"; do
  mkdir -p "$DEST_DIR"
  rm -rf "$DEST_DIR/$name"
  cp -r "$name" "$DEST_DIR/$name"
  echo "✓ 已同步: $name -> $DEST_DIR"
done
```

### 拉取（目标目录 → 开发目录）：拉取 / pull

**⚠️ 安全门控：拉取操作会覆盖开发目录文件。执行前先运行"查看状态"命令展示差异，让用户确认后再执行。**

先确定来源目录：

```bash
# Claude 来源
SOURCE_DIR=".claude/skills"

# Codex 来源
SOURCE_DIR="$HOME/.codex/skills"
```

```bash
for skill_dir in "$SOURCE_DIR"/*/; do
  [ -d "$skill_dir" ] || continue
  name=$(basename "$skill_dir")
  if [ -d "$name" ]; then
    rm -rf "$name"
    cp -r "$SOURCE_DIR/$name" "$name"
    echo "✓ 已拉取: $name"
  fi
done
echo "拉取完成（仅更新开发目录中已有的 skill）"
```

### 查看状态：哪些 skill 有差异

默认同时查看 `.claude/skills/` 和 `~/.codex/skills/` 两个目标：

```bash
for DEST_DIR in ".claude/skills" "$HOME/.codex/skills"; do
  echo "=== Skill 同步状态: $DEST_DIR ==="
  mkdir -p "$DEST_DIR"

  for skill_dir in */; do
    name=$(basename "$skill_dir")
    [ "$name" = ".claude" ] && continue
    [ "$name" = ".codex" ] && continue
    [ "$name" = ".git" ] && continue
    [ "$name" = ".idea" ] && continue
    [ -f "$skill_dir/SKILL.md" ] || continue

    if [ ! -d "$DEST_DIR/$name" ]; then
      echo "[ 未同步 ] $name"
    elif ! diff -rq "$skill_dir" "$DEST_DIR/$name" > /dev/null 2>&1; then
      echo "[ 有差异 ] $name"
    else
      echo "[ 已同步 ] $name"
    fi
  done

  for skill_dir in "$DEST_DIR"/*/; do
    [ -d "$skill_dir" ] || continue
    name=$(basename "$skill_dir")
    if [ ! -d "$name" ]; then
      echo "[ 仅目标目录 ] $name"
    fi
  done
done
```

## 执行步骤

1. **检测目录**：确认开发目录中有哪些 skill，`.claude/skills/` 和 `~/.codex/skills/` 不存在则自动创建
2. **解析目标**：识别用户要操作 `.claude`、`.codex` 还是两个都处理；如果是拉取且来源不明确，先展示状态再确认来源
3. **根据用户意图**（同步/拉取/状态）选择对应命令
4. **执行命令**，逐行输出同步结果
5. **汇报结果**：同步了几个 skill，分别落到哪些目录，哪些成功，哪些跳过

## 注意

- 推送操作前不需要确认，直接执行；如果用户没指定目标，默认同时同步到 `.claude/skills/` 和 `~/.codex/skills/`
- 拉取操作前必须先展示差异状态，等用户确认；并且必须明确来源是 `.claude/skills/` 还是 `~/.codex/skills/`
- 跳过 `.claude`、`.codex`、`.git`、`.idea` 等非 skill 目录
- 如果源文件不存在，跳过并提示，不报错中断
- 同步整个 skill 目录（含 SKILL.md、references/、assets/ 等所有文件）
- 同步到 `.claude/skills/` 后，skill 在当前项目中立即生效
- 同步到 `~/.codex/skills/` 后，skill 对 Codex 全局生效
