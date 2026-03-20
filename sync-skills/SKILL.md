---
name: sync-skills
description: |
  Sync skills between project development directory and Claude global skills directory (~/.claude/skills/).
  Use when user says: "同步skill", "sync skill", "同步一下", "推送skill", "把skill同步过去",
  "同步所有skill", "同步某个skill", "拉取skill", "pull skill", "从全局拉取", "更新本地skill".
  Also use when user wants to push skills to global, pull from global to project, or check sync status.
---

# Sync Skills

将项目开发目录中的 skill 同步到 Claude 全局目录（或反向同步）。

## 路径约定

| 用途 | 路径 |
|------|------|
| 项目 skill 开发目录 | 当前工作目录下的 `skills/` 子目录，或 `.claude/skills/` |
| Claude 全局生效目录 | `~/.claude/skills/` |

## 第一步：确定源目录

运行以下命令检测当前项目的 skill 目录：

```bash
# 检测可能的 skill 目录
ls skills/ 2>/dev/null && echo "FOUND: skills/" || \
ls .claude/skills/ 2>/dev/null && echo "FOUND: .claude/skills/" || \
echo "NOT FOUND"
```

如果用户没有明确说明目录，优先查找：
1. `./skills/`（扁平结构，直接包含 skill 子目录）
2. `./.claude/skills/`（标准 Claude 项目结构）

## 第二步：根据用户意图选择操作

### 推送（项目 → 全局）：同步 / 推送 / push

**同步所有 skill：**
```bash
SOURCE_DIR="skills"  # 或 .claude/skills
DEST_DIR="$HOME/.claude/skills"

for skill_dir in "$SOURCE_DIR"/*/; do
  name=$(basename "$skill_dir")
  mkdir -p "$DEST_DIR/$name"
  cp "$skill_dir/SKILL.md" "$DEST_DIR/$name/SKILL.md"
  echo "✓ 已同步: $name"
done
echo "同步完成"
```

**同步单个 skill（用户指定了名称）：**
```bash
SOURCE_DIR="skills"  # 或 .claude/skills
name="skill-name"   # 替换为实际 skill 名
mkdir -p "$HOME/.claude/skills/$name"
cp "$SOURCE_DIR/$name/SKILL.md" "$HOME/.claude/skills/$name/SKILL.md"
echo "✓ 已同步: $name"
```

### 拉取（全局 → 项目）：拉取 / pull / 从全局同步

**将全局 skill 拉取到项目（覆盖本地）：**
```bash
SOURCE_DIR="skills"  # 或 .claude/skills
GLOBAL_DIR="$HOME/.claude/skills"

for skill_dir in "$GLOBAL_DIR"/*/; do
  name=$(basename "$skill_dir")
  if [ -d "$SOURCE_DIR/$name" ]; then
    cp "$GLOBAL_DIR/$name/SKILL.md" "$SOURCE_DIR/$name/SKILL.md"
    echo "✓ 已拉取: $name"
  fi
done
echo "拉取完成（仅更新本地已有的 skill）"
```

### 查看状态：哪些 skill 有差异

```bash
SOURCE_DIR="skills"
GLOBAL_DIR="$HOME/.claude/skills"

echo "=== Skill 同步状态 ==="
for skill_dir in "$SOURCE_DIR"/*/; do
  name=$(basename "$skill_dir")
  local_file="$skill_dir/SKILL.md"
  global_file="$GLOBAL_DIR/$name/SKILL.md"

  if [ ! -f "$global_file" ]; then
    echo "[ 仅本地 ] $name"
  elif ! diff -q "$local_file" "$global_file" > /dev/null 2>&1; then
    echo "[ 有差异 ] $name"
  else
    echo "[ 已同步 ] $name"
  fi
done

# 检查全局有但本地没有的
for skill_dir in "$GLOBAL_DIR"/*/; do
  name=$(basename "$skill_dir")
  if [ ! -d "$SOURCE_DIR/$name" ]; then
    echo "[ 仅全局 ] $name"
  fi
done
```

## 执行步骤

1. **先用 Bash 检测源目录**（`ls skills/` 或 `ls .claude/skills/`），确认路径
2. **根据用户意图**（同步/拉取/状态）选择对应命令
3. **执行命令**，逐行输出同步结果
4. **汇报结果**：同步了几个 skill，哪些成功，哪些跳过

## 注意

- 操作前不需要确认，直接执行（用户说同步就同步）
- 如果源文件不存在，跳过并提示，不报错中断
- 同步后 skill 立即在新对话中生效
