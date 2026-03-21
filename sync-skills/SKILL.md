---
name: sync-skills
description: |
  Sync skills between current project root directory and project's .claude/skills/ directory.
  Use when user says: "同步skill", "sync skill", "同步一下", "推送skill",
  "同步所有skill", "同步某个skill", "拉取skill", "pull skill", "更新本地skill".
  Also use when user wants to sync skills to .claude/skills/, pull from .claude/skills/ to project, or check sync status.
metadata:
  pattern: pipeline
  steps: "3"
---

# Sync Skills

你是一个 skill 同步工具，负责在当前项目根目录下的 skill 开发目录和项目的 `.claude/skills/` 目录之间同步。严格按以下步骤执行。

## 路径约定

| 用途 | 路径 |
|------|------|
| skill 开发目录（源） | 当前项目根目录下的各 `{skill-name}/`（含 SKILL.md、references/、assets/ 等） |
| Claude 生效目录（目标） | 当前项目根目录下的 `.claude/skills/{skill-name}/` |

## 第一步：确定目录

检测当前项目根目录下是否有 skill 子目录，以及 `.claude/skills/` 是否存在：

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
```

## 第二步：根据用户意图选择操作

### 推送（开发目录 → .claude/skills/）：同步 / 推送 / push

**同步所有 skill：**
```bash
DEST_DIR=".claude/skills"

for skill_dir in */; do
  name=$(basename "$skill_dir")
  [ "$name" = ".claude" ] && continue
  [ "$name" = ".git" ] && continue
  [ "$name" = ".idea" ] && continue
  if [ -f "$skill_dir/SKILL.md" ]; then
    rm -rf "$DEST_DIR/$name"
    cp -r "$skill_dir" "$DEST_DIR/$name"
    echo "✓ 已同步: $name"
  fi
done
echo "同步完成"
```

**同步单个 skill（用户指定了名称）：**
```bash
name="skill-name"   # 替换为实际 skill 名
rm -rf ".claude/skills/$name"
cp -r "$name" ".claude/skills/$name"
echo "✓ 已同步: $name"
```

### 拉取（.claude/skills/ → 开发目录）：拉取 / pull

**⚠️ 安全门控：拉取操作会覆盖开发目录文件。执行前先运行"查看状态"命令展示差异，让用户确认后再执行。**

```bash
SOURCE_DIR=".claude/skills"

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

```bash
DEST_DIR=".claude/skills"

echo "=== Skill 同步状态 ==="
for skill_dir in */; do
  name=$(basename "$skill_dir")
  [ "$name" = ".claude" ] && continue
  [ "$name" = ".git" ] && continue
  [ "$name" = ".idea" ] && continue
  [ -f "$skill_dir/SKILL.md" ] || continue

  dev_file="$skill_dir/SKILL.md"
  target_file="$DEST_DIR/$name/SKILL.md"

  if [ ! -d "$DEST_DIR/$name" ]; then
    echo "[ 未同步 ] $name"
  elif ! diff -rq "$skill_dir" "$DEST_DIR/$name" > /dev/null 2>&1; then
    echo "[ 有差异 ] $name"
  else
    echo "[ 已同步 ] $name"
  fi
done

# 检查 .claude/skills/ 有但开发目录没有的
for skill_dir in "$DEST_DIR"/*/; do
  [ -d "$skill_dir" ] || continue
  name=$(basename "$skill_dir")
  if [ ! -d "$name" ]; then
    echo "[ 仅.claude ] $name"
  fi
done
```

## 执行步骤

1. **检测目录**：确认开发目录中有哪些 skill，`.claude/skills/` 不存在则自动创建
2. **根据用户意图**（同步/拉取/状态）选择对应命令
3. **执行命令**，逐行输出同步结果
4. **汇报结果**：同步了几个 skill，哪些成功，哪些跳过

## 注意

- 推送操作前不需要确认，直接执行（用户说同步就同步）
- 拉取操作前必须先展示差异状态，等用户确认
- 跳过 `.claude`、`.git`、`.idea` 等非 skill 目录
- 如果源文件不存在，跳过并提示，不报错中断
- 同步整个 skill 目录（含 SKILL.md、references/、assets/ 等所有文件）
- 同步到 `.claude/skills/` 后，skill 在当前项目中立即生效
