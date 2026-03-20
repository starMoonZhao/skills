---
name: skill-design-patterns
description: |
  Skill 设计模式规范与质量检查。在创建或审查 SKILL.md 文件时，使用此 skill 确保遵循五大设计模式和结构化最佳实践。
  当用户提到"规范化 skill"、"检查 skill 质量"、"skill 设计模式"、"优化 skill"、"skill 写得对不对"、
  "skill best practices"、"review skill"、"skill 模式" 时使用。
  也适用于：用户新建 skill 时作为设计指导，或对已有 skill 进行模式匹配和改进建议。
---

# Skill 设计模式规范

> 基于 Google Cloud Tech "5 Agent Skill design patterns every ADK developer should know"

## 核心原则

**格式不是重点，内容设计才是关键。** SKILL.md 的 frontmatter 和目录结构已经标准化，真正区分 skill 质量的是内部逻辑的结构化程度。不要把复杂且易出错的指令硬塞进单一提示，而要分解工作流程，运用合适的结构化模式。

## 五大设计模式

### Pattern 1: Tool Wrapper（工具封装器）

**适用场景：** 需要为 agent 提供特定库/框架/技术的按需上下文。

**核心思路：** 将 API 约定、编码规范、最佳实践打包成 skill，agent 仅在实际使用该技术时加载。

**结构要点：**
- 使用 `references/` 目录存放具体的约定文档（如 `conventions.md`）
- SKILL.md 监听用户提示中的特定关键词
- 指令中明确区分"审查代码时"和"编写代码时"的不同行为
- 规则作为绝对真理应用，不允许 agent 自由发挥

**模板：**
```markdown
---
name: {tech}-expert
description: {技术名} development best practices and conventions. Use when...
metadata:
  pattern: tool-wrapper
  domain: {tech}
---

You are an expert in {技术名} development. Apply these conventions to the user's code.

## Core Conventions
Load 'references/conventions.md' for the complete list of best practices.

## When Reviewing Code
1. Load the conventions reference
2. Check the user's code against each convention
3. For each violation, cite the specific rule and suggest the fix

## When Writing Code
1. Load the conventions reference
2. Follow every convention exactly
3. [技术特定的附加要求]
```

---

### Pattern 2: Generator（生成器）

**适用场景：** 需要确保每次输出结构一致的文档/代码/报告生成。

**核心思路：** 编排一个"填空"流程——加载模板、读取风格指南、询问缺失变量、填充文档。

**结构要点：**
- `assets/` 目录存放输出模板
- `references/` 目录存放风格指南
- 指令充当"项目经理"角色，协调资源检索
- 强制 agent 逐步执行，不允许跳步

**模板：**
```markdown
---
name: {output}-generator
description: Generates structured {output type} in {format}. Use when...
metadata:
  pattern: generator
  output-format: {format}
---

You are a {output type} generator. Follow these steps exactly:

Step 1: Load 'references/style-guide.md' for tone and formatting rules.
Step 2: Load 'assets/{template-name}.md' for the required output structure.
Step 3: Ask the user for any missing information needed to fill the template:
- [变量1]
- [变量2]
- [变量3]
Step 4: Fill the template following the style guide rules. Every section must be completed.
Step 5: Return the completed document as a single {format} document.
```

---

### Pattern 3: Reviewer（审核者）

**适用场景：** 需要按清单系统化评审代码/文档/设计。

**核心思路：** 将"检查什么"与"如何检查"分离——评分标准存储在外部文件中，skill 指令定义评审流程。

**结构要点：**
- `references/review-checklist.md` 存放模块化评分标准
- 指令保持静态，动态加载评审标准
- 强制按严重程度分组输出（error > warning > info）
- 替换清单即可得到完全不同的专业审计

**模板：**
```markdown
---
name: {domain}-reviewer
description: Reviews {target} for {aspects}. Use when...
metadata:
  pattern: reviewer
  severity-levels: error,warning,info
---

You are a {domain} reviewer. Follow this review protocol exactly:

Step 1: Load 'references/review-checklist.md' for the complete review criteria.
Step 2: Read the user's {target} carefully. Understand its purpose before critiquing.
Step 3: Apply each rule from the checklist. For every violation:
- Note the location
- Classify severity: error (must fix), warning (should fix), info (consider)
- Explain WHY it's a problem, not just WHAT is wrong
- Suggest a specific fix
Step 4: Produce a structured review:
- **Summary**: What it does, overall quality assessment
- **Findings**: Grouped by severity (errors first)
- **Score**: Rate 1-10 with justification
- **Top 3 Recommendations**: The most impactful improvements
```

---

### Pattern 4: Inversion（反转）

**适用场景：** 需要 agent 先充分了解需求再行动，而非猜测后立即生成。

**核心思路：** 颠覆"用户驱动提示、agent 执行"的默认动态——agent 充当面试官，按阶段提问，收集完整信息后才生成输出。

**结构要点：**
- 使用明确的、不可跳过的阶段门控指令（如 "DO NOT start building until all phases are complete"）
- 按阶段顺序提问，每次一个问题，等待回答
- 所有问题回答完毕后才进入合成阶段
- 合成后请求用户确认并迭代

**模板：**
```markdown
---
name: {task}-planner
description: Plans {task} by gathering requirements through structured interview. Use when...
metadata:
  pattern: inversion
  interaction: multi-turn
---

You are conducting a structured requirements interview. DO NOT start building until all phases are complete.

## Phase 1 — Discovery (ask one question at a time, wait for each answer)
Ask these questions in order. Do not skip any.
- Q1: "[核心问题]"
- Q2: "[用户/场景问题]"
- Q3: "[规模/约束问题]"

## Phase 2 — Technical Constraints (only after Phase 1 is fully answered)
- Q4: "[技术环境]"
- Q5: "[技术偏好]"
- Q6: "[硬性要求]"

## Phase 3 — Synthesis (only after all questions are answered)
1. Load 'assets/{template}.md' for the output format
2. Fill every section using the gathered requirements
3. Present to the user
4. Ask: "Does this accurately capture your requirements? What would you change?"
5. Iterate on feedback until the user confirms
```

---

### Pattern 5: Pipeline（流水线）

**适用场景：** 复杂多步任务，不能容忍跳步或忽略指令。

**核心思路：** 强制严格的顺序工作流，在关键节点设置硬性检查点（diamond gate），agent 必须获得用户确认后才能继续。

**结构要点：**
- 每一步在需要时才加载对应的 `references/` 和 `assets/` 文件，保持上下文窗口整洁
- 使用明确的门控条件（如 "Do NOT proceed to Step N until the user confirms"）
- 最后一步执行质量检查，发现问题需修复后才呈现最终结果

**模板：**
```markdown
---
name: {task}-pipeline
description: Executes {task} through a multi-step pipeline with checkpoints. Use when...
metadata:
  pattern: pipeline
  steps: "{N}"
---

You are running a {task} pipeline. Execute each step in order. Do NOT skip steps.

## Step 1 — [阶段名]
[具体指令]

## Step 2 — [阶段名]
[具体指令]
- Load 'references/{ref}.md' for [用途]
- [操作步骤]
- Present results for user approval

Do NOT proceed to Step 3 until the user confirms.

## Step 3 — [阶段名]
Load 'assets/{template}.md' for the output structure. [操作步骤]

## Step 4 — Quality Check
Review against 'references/quality-checklist.md':
- [检查项1]
- [检查项2]
- [检查项3]
Report results. Fix issues before presenting the final output.
```

---

## 模式组合

这五种模式并非互斥，可以自由组合：

| 组合 | 说明 |
|------|------|
| Pipeline + Reviewer | 流水线末尾加审核步骤，双重检查产出质量 |
| Generator + Inversion | 生成器开头加反转，先收集变量再填模板 |
| Pipeline + Tool Wrapper | 流水线某步加载特定技术的约定文件 |
| Reviewer + Tool Wrapper | 审核时动态加载领域特定的检查清单 |

## 模式选择决策树

创建 skill 时，按以下问题选择模式：

1. **是否在为特定技术/库提供上下文？** → Tool Wrapper
2. **是否需要每次生成结构一致的输出？** → Generator
3. **是否需要按标准评分/审查？** → Reviewer
4. **是否需要先充分了解需求再行动？** → Inversion
5. **是否有严格的多步流程且不容跳步？** → Pipeline
6. **以上多个条件都满足？** → 组合使用

## Skill 质量检查清单

对任何 SKILL.md 进行规范化审查时，检查以下项目：

### 结构规范
- [ ] frontmatter 包含 `name` 和 `description`
- [ ] `description` 包含明确的触发条件（关键词列表）
- [ ] 正文使用 Markdown 格式，层次清晰
- [ ] 指令具体、可执行，不含模糊表述

### 模式合规
- [ ] 已识别并标注所使用的设计模式
- [ ] 如使用 references/ 或 assets/，有明确的加载时机指令
- [ ] 步骤编号清晰，agent 不会跳步或合并步骤
- [ ] 需要用户确认的节点有明确的门控条件

### 内容设计
- [ ] 逻辑与格式分离（具体规则在外部文件，指令定义流程）
- [ ] agent 角色定义明确（"You are a..."）
- [ ] 输出格式有明确要求
- [ ] 避免将复杂指令硬塞进单一提示

### 常见问题
- [ ] 没有过度依赖 agent 自由发挥（关键约定有强制指令）
- [ ] 没有遗漏错误处理路径
- [ ] description 中的触发词覆盖了常见的中英文表述

## 使用方式

### 创建新 skill 时
1. 根据决策树选择合适的模式（或组合）
2. 使用对应模板作为起点
3. 按质量检查清单逐项验证
4. 如需外部资源，创建 `references/` 和 `assets/` 目录

### 审查已有 skill 时
1. 识别该 skill 实际使用的模式
2. 对照该模式的结构要点检查完整性
3. 运行质量检查清单
4. 输出改进建议，按优先级排序
