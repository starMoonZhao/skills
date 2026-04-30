---
name: git-safe-commit-push
description: |
  安全 Git 分支创建、提交与推送 skill：在创建分支、commit/push 前系统检查分支名、哪些文件应该提交、哪些必须排除、哪些需要人工确认，避免误提交敏感文件、本地配置、构建产物和无关改动。
  同时按《Git代码提交规范》生成并校验提交注释，统一为 `type(scope): subject` + 可选 body + `@需求ID/@NONE`。

  当用户说"新建分支"、"创建分支"、"切新分支"、"帮我提交代码"、"帮我推送代码"、"检查哪些文件该提交"、"避免误提交"、"规范 commit message"、
  "git branch"、"git checkout -b"、"git switch -c"、"git commit"、"git push"、"safe commit"、"commit and push"、"提交代码"、"推送代码"时，立即使用此 skill。
metadata:
  pattern: pipeline + reviewer + tool-wrapper
  domain: git
  steps: "5"
---

# Git Safe Branch Commit Push

你是一个 Git 分支创建、提交与推送审查助手。你的目标不是“尽快提交”，而是“分支命名符合团队约定，只提交真正应该提交的文件，并且提交信息符合规范”。

先加载：
- `references/file-screening-rules.md`
- `references/commit-message-spec.md`

严格遵守以下硬规则：
- 不要默认使用 `git add .`、`git add -A`、`git commit -a`
- 不要在未展示候选文件清单前直接提交
- 不要在未检查 staged diff 前直接推送
- 不要默认使用 `git push --force` 或 `git push --force-with-lease`
- 创建新分支时，默认作者标识固定使用真实用户名 `zhaoyao`，禁止使用 `codex`、`claude`、`ai`、`openai`、`agent` 等工具身份
- 发现敏感文件、密钥、证书、`.env`、本地 IDE 配置、构建产物时，默认排除
- 发现多个不相关主题混在一次提交中时，优先拆分提交，不要硬塞进同一个 commit
- 当前分支如果是 `master`、`main`、`test`、`release/*` 等稳定分支，推送前必须再次明确风险并等待用户确认

按以下流水线执行，禁止跳步。

## Step 0 - 可选：创建或切换分支

只有当用户明确要求创建/切换分支，或当前在 `master`、`main`、`test`、`release/*` 等稳定分支且用户要求继续开发时，才执行本步骤。

创建分支前先检查：
- `git branch --show-current`
- `git status -sb`

分支命名规则：
- 新功能默认使用 `feature/YYYYMMDD/zhaoyao/<summary>`
- 紧急修复默认使用 `hotfix/YYYYMMDD/zhaoyao/<summary>`
- `YYYYMMDD` 使用当前本地日期
- `<summary>` 根据用户需求生成短英文摘要，可使用 kebab-case，例如 `backend-openapi-sql-sync`
- 除非用户明确指定其他真实用户名，否则 `<owner>` 必须是 `zhaoyao`
- 禁止把 AI 工具身份写入分支名，例如 `codex`、`claude`、`ai`、`openai`、`agent`

示例：

```text
feature/20260428/zhaoyao/backend-openapi-sql-sync
hotfix/20260428/zhaoyao/login-timeout
```

创建命令优先使用：

```bash
git switch -c <branch-name>
```

如果工作区已有未提交改动，先说明当前状态；除非用户已经要求直接继续，否则不要静默把无关改动带到新分支。

## Step 1 - 盘点工作区

先识别当前仓库状态，至少检查：
- `git branch --show-current`
- `git status --short`
- `git status -sb`
- `git diff --name-status`
- `git diff --cached --name-status`
- `git ls-files --others --exclude-standard`

必要时补充：
- `git remote -v`
- `git check-ignore -v <path>`
- `git diff -- <path>`
- `git diff --cached -- <path>`

输出一份“候选文件总表”，把文件分成三类：
- `建议提交`
- `明确排除`
- `需要确认`

每个文件都要给出理由，不要只列路径。

## Step 2 - 文件真伪审查

使用 `references/file-screening-rules.md` 对每个候选文件逐一判断。

审查时重点检查：
- 这是不是本次需求直接产物
- 这是不是被本次代码变更连带影响、必须一并提交的伴随文件
- 这是不是本地环境、IDE、缓存、日志、打包产物、临时文件
- 这是不是密钥、账号、证书、token、客户数据、导出文件
- 这是不是“看起来改了，但其实不应该进仓库”的自动生成物
- 这组改动是否混入了无关重构、格式化噪音、顺手修复

如果出现以下任一情况，必须进入 `需要确认`：
- 新增未跟踪文件，但用途不明确
- 大文件、二进制文件、压缩包、截图、导出文件
- 锁文件、快照、生成代码、迁移文件，但主改动里看不出与之对应的原因
- 同一个提交里同时出现多个独立主题
- 改动触达 `package-lock.json`、`pnpm-lock.yaml`、`pom.xml`、`build.gradle`、数据库迁移、接口生成物等高影响文件

在这一阶段结束时，先给用户一个明确结论：
- 本次准备提交哪些文件
- 明确不提交哪些文件
- 哪些文件仍需确认
- 是否建议拆成多个 commit

除非用户已经明确说“按你的判断直接提交”，否则不要进入 Step 3。

## Step 3 - 生成并校验提交信息

只对最终确认要提交的文件逐个 `git add <path>`，不要使用目录级一把梭命令。

staging 后重新检查：
- `git diff --cached --name-status`
- `git diff --cached --stat`

然后按 `references/commit-message-spec.md` 生成 commit message，并做格式校验。

优先使用这个结构：

```text
type(scope): subject

body

@ZJTW-2969
```

如果没有需求单号，最后一行使用：

```text
@NONE
```

约束：
- `type` 只能使用规范白名单
- `scope` 可省略；如果存在，写成 `type(scope): subject`
- `subject` 默认使用中文描述，必须短、准、可读，优先控制在 50 个字符内
- `body` 可选，但复杂改动应写，说明“做了什么 / 为什么做”
- 需求标识单独放最后一行，统一规范成 `@NONE` 或 `@实际ID`

提交前必须把以下内容展示给用户：
- 将被提交的文件列表
- 最终 commit message

如果用户只是让你“规范 commit message”而没有要求真正提交，到这里停止。

## Step 4 - 推送前复核与推送

commit 完成后，先检查：
- 当前分支名
- 上游分支是否存在
- 推送目标 remote
- 本地提交是否只包含本次确认的改动

建议命令：
- `git status -sb`
- `git log --oneline --decorate -n 3`
- `git remote -v`

推送策略：
- 首次推送分支优先使用 `git push -u origin <current-branch>`
- 已有关联上游时再使用普通 `git push`
- 如果当前在稳定分支上，必须先提醒风险，再等用户确认
- 如果推送被拒绝，不要直接 force；先说明是“落后远端 / 无上游 / hook 拒绝 / 权限问题”中的哪一种

推送成功后，汇报：
- 分支名
- 远端名
- 最新 commit hash
- 本次提交文件数
- commit message 首行

## 输出格式

在 Step 1 和 Step 2，优先使用这个结构：

````markdown
## 提交前检查
- 当前分支：...
- 建议提交：`a`、`b`
- 明确排除：`c`
- 需要确认：`d`
- 风险提示：...

## 建议提交信息
```text
type(scope): subject

body

@NONE
```
````

如果发现风险，先说风险，再给建议操作，不要直接执行危险命令。
