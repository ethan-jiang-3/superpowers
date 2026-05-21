# 04 — CSO 描述陷阱

## 这个问题为什么重要

写 skill 的人想在 description 里总结 skill 做了什么——这样 agent 读 description 就能快速了解。**这恰恰是 bug 的根源。**

Superpowers 发现：如果 skill 的 description 包含流程摘要，agent 会**跳过 skill body 直接按摘要行动**。摘要永远不如全文准确——于是一个本意是"帮助"的设计变成了 skill 失效的原因。

`writing-skills` 把这个问题称为 CSO（Claude Search Optimization），模式是："description = when to use, NOT what it does."

---

## 真实案例：subagent-driven-development

### 坏的 description

```yaml
description: Use when executing plans - dispatches subagent per task
             with code review between tasks
```

流程摘要："code review between tasks."

### Agent 实际做了什么

Agent 读了 description 的摘要。知道要做 code review。**但摘要说的是"一次 review"，而 skill body 的 flowchart 明确写了两次 review（spec compliance 先，然后 code quality）。** Agent 按摘要做了——一次 review。第二次 review 被跳过了。Skill body 从未被读。

### 好的 description（现在的版本）

```yaml
description: Use when executing implementation plans with independent tasks
             in the current session
```

只有触发条件。零流程信息。Agent 必须读 skill body 才能知道怎么做。

### 修复效果

Agent 正确读了 flowchart，做了两阶段 review。

---

## 为什么 agent 会这样？

### 闭合需求

LLM 在得到一个"足够好"的行动计划后（哪怕是从 description 摘要中提取的不完整版本），寻找更多信息的动力下降。Description 的摘要满足了 agent 的方向需求，full body 成了不需要的"冗余"。

### Description 的双重角色

Description 本应只回答一个问题："我应该 invoke 这个 skill 吗？" 当 description 还回答了"这个 skill 让我做什么？"——agent 从 description 得到了足够的指令，不需要调 skill body。

### 具体失败模式：步骤遗漏

Subagent-driven-development 是典型：摘要说了"code review"但跳过了"两阶段 review"。问题不是摘要错了——是不完整。不完整版本的存在阻止了完整（correct）版本被加载。

---

## 好和坏的例子

`writing-skills/SKILL.md` 举了四个例子：

```yaml
# BAD: Summarizes workflow
description: "Guides the design process before implementation — explores user intent, conducts research, writes spec"
# Agent: skips skill body, invents its own design process

# GOOD: Trigger conditions only
description: "Use before any creative work — establishes design, requirements and constraints before building"
# Agent: invokes skill to know the design process
```

```yaml
# BAD: Summarizes workflow
description: "Handles finishing work — merges, creates PRs, cleans up branches"
# Agent: skips skill body, improvises its own finishing steps

# GOOD: Trigger conditions only
description: "Use when implementation is complete, all tests pass, and you need to integrate"
# Agent: invokes skill to learn the structured completion process
```

---

## 实际 description 的 CSO 审计

| Skill | Description | CSO 违规？ |
|-------|------------|---------|
| TDD | "Use when implementing any feature or bugfix, before writing implementation code" | **否**——纯触发 |
| Systematic Debugging | "Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes" | **否**——纯触发 |
| Verification | "Use when about to claim work is complete... — requires running verification commands and confirming output before making any success claims; evidence before assertions always" | **中度**——破折号后描述了流程 |
| Writing Skills | "Use when creating new skills, editing existing skills, or verifying skills work before deployment" | **否**——纯触发 |
| Brainstorming | "You MUST use this before any creative work... Explores user intent, requirements and design before implementation." | **边缘**——"Explores..."暗示流程 |
| Dispatching Parallel Agents | "Use when facing 2+ independent tasks..." | **否**——纯触发 |
| Executing Plans | "Use when you have a written implementation plan to execute in a separate session with review checkpoints" | **轻**——"with review checkpoints"是流程暗示 |
| Finishing a Dev Branch | "Use when implementation is complete... — guides completion of development work by presenting structured options for merge, PR, or cleanup" | **有**——破折号后总结了整个流程 |
| Receiving Code Review | "Use when receiving code review feedback... — requires technical rigor and verification, not performative agreement or blind implementation" | **中度**——告诉 agent 该做什么 |
| Requesting Code Review | "Use when completing tasks, implementing major features, or before merging to verify work meets requirements" | **否**——纯触发 |
| Subagent-Driven Dev | "Use when executing implementation plans with independent tasks in the current session" | **否**——这是修过的 |
| Using Git Worktrees | "Use when starting feature work that needs isolation... — ensures an isolated workspace exists via native tools or git worktree fallback" | **轻**——后半描述结果 |
| Using Superpowers | "Use when starting any conversation — establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions" | **有**——告诉 agent 具体做什么 |
| Writing Plans | "Use when you have a spec or requirements for a multi-step task, before touching code" | **否**——纯触发 |

注意：using-superpowers 有 CSO 违规但这个违规可能是故意的——因为它的内容是通过 bootstrap 注入的，不依赖 Skill 工具调用。

---

## 规则

`writing-skills/SKILL.md:99-102`:

```
description: Third-person, describes ONLY when to use (NOT what it does)
Start with "Use when..." to focus on triggering conditions
NEVER summarize the skill's process or workflow
```

`writing-skills/SKILL.md:156`:

> "Descriptions that summarize workflow create a shortcut Claude will take.
> The skill body becomes documentation Claude skips."

---

## 修复

规则很简单：**description 只回答一个问题——"我该 invoke 这个 skill 吗？"** 不能回答"invoke 之后我该做什么？"——那个答案只在 skill body 里。

实现方法：description 写"Use when [条件]"。不写"Use when [条件] — does [X], [Y], [Z] to achieve [result]"。破折号之后的内容全是流程——全删掉。

---

## 关键源文件

| 文件 | 内容 |
|------|------|
| `skills/writing-skills/SKILL.md:99-102` | CSO 规则：description = when, not what |
| `skills/writing-skills/SKILL.md:140-197` | 好/坏例子 |
| `skills/writing-skills/SKILL.md:154-158` | Subagent-driven-dev 真实案例 |
| `skills/subagent-driven-development/SKILL.md` | 修好的 description（现在的版本） |
