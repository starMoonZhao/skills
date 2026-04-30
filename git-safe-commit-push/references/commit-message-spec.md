# Commit Message 规范

本规范以用户提供的《Git代码提交规范》PDF 为主，并做了少量保守收敛，目的是让提交信息更稳定、更容易校验。

## 一、基础格式

提交信息使用以下结构：

```text
type(scope): subject

body

@ZJTW-2969
```

说明：
- 第一行是标题行，必填
- `body` 可选
- 最后一行是需求单/任务单标识，使用 `@实际ID` 或 `@NONE`
- `subject` 默认使用中文描述，例如 `feat:新增代码提交检查`

如果没有需求单号，使用：

```text
@NONE
```

PDF 示例里出现了 `@None`、`@none`、`@NONE`。执行时统一规范成 `@NONE`。

## 二、type 白名单

只允许以下类型：

- `fix`：修复 bug
- `feat`：新增功能
- `docs`：文档改动
- `style`：纯格式调整，不改变逻辑
- `refactor`：重构，不新增功能、不修 bug
- `perf`：性能优化
- `test`：测试相关
- `build`：构建、依赖、CI、打包流程
- `revert`：回滚提交
- `chore`：杂项维护

如果用户给的 type 不在白名单里，先规范化再提交。

## 三、scope 规则

推荐格式：

```text
type(scope): subject
```

补充说明：
- PDF 标题格式写的是 `type(scope): subject`
- PDF 示例同时出现了带 scope 和不带 scope 的写法，所以 `scope` 视为可选
- 如果使用 scope，必须紧贴类型，不加多余空格
- scope 应该指向模块、子系统或业务域，不要写成过长句子

示例：

```text
feat(auth):新增登录重试限制
fix:处理用户信息接口空响应
test(SO):补充用例
```

## 四、subject 规则

PDF 中可确认 subject 长度目标为 `50`。执行时按以下标准：

- 优先控制在 50 个字符内
- 默认使用中文描述，不使用英文短句作为默认输出
- 只写本次提交的核心动作
- 不写流水账，不把多个主题并列塞进同一行
- 避免模糊词，如“修改一下”“优化代码”“更新内容”

推荐写法：

- `feat:新增代码提交检查`
- `fix(order):修复重复支付回调问题`
- `feat(user):新增企业邀请流程`

不推荐写法：

- `fix:调整代码`
- `feat:更新很多内容`
- `feat:add commit check`

## 五、body 规则

`body` 不是强制项，但以下情况建议填写：

- 改动跨多个文件或多个层级
- 标题不足以解释“为什么这样改”
- 涉及兼容性、迁移、风险控制

body 建议回答两件事：

- 做了什么
- 为什么要这么做

## 六、需求标识规则

最后一行统一写成：

- `@实际需求ID`
- `@NONE`

示例：

```text
feat(report):新增导出重试保护

当 OSS 回调延迟时自动重试导出任务。

@ZJTW-2969
```

```text
chore:同步本地 skill 引用

@NONE
```

## 七、分支命名建议

PDF 中给出了如下命名风格，推送前可以用来校验当前分支是否合理：

```text
feature/YYYYMMDD/<owner>/<summary>
hotfix/YYYYMMDD/<owner>/<summary>
```

执行时对 `<owner>` 做强约束：

- 默认固定使用真实用户名 `zhaoyao`
- 除非用户明确指定其他真实用户名，否则不要替换成别的值
- 禁止使用 AI 工具身份，例如 `codex`、`claude`、`ai`、`openai`、`agent`

示例：

```text
feature/20260428/zhaoyao/backend-openapi-sql-sync
hotfix/20260428/zhaoyao/login-timeout
```

如果当前分支明显不符合团队约定，推送前先提醒用户。

## 八、历史修正规则

如果只是最后一次 commit message 写错，优先：

```bash
git commit --amend
```

如果多个本地 commit 需要整理，再考虑：

```bash
git rebase -i <base>
```

没有用户明确要求时，不要主动改写已经推送到远端的历史。
