# 03 — CLAUDE.md 的设计

## 这个问题为什么重要

Superpowers 的 `CLAUDE.md` 只有 106 行。106 行改变一个 AI coding agent 的整个行为模式。而且这 106 行所在的 repo 有 94% 的 PR 拒绝率——大部分 PR 是 agent 提交的。

CLAUDE.md 到底写了什么，让 agent 从"积极提交 PR"变成"先检查 5 个条件，条件不满足就阻止 PR 提交"？

---

## 注意：这个 CLAUDE.md 不是给 Superpowers 用户的

重要区分：Superpowers 项目本身的 `CLAUDE.md` 是**贡献者指南**，不是终端用户指南。它告诉 agent：

> 这个 repo 有 94% PR 拒绝率。你的工作是保护你的人类型伙伴不被羞辱。提交低质量 PR 不是帮忙——是让维护者浪费时间、烧毁你伙伴的名声、PR 还是会被关。

**终端用户**的 CLAUDE.md 是他们自己写的项目指令——Superpowers 不管那个。Superpowers 的 `using-superpowers` skill 明确写了："User's explicit instructions (CLAUDE.md, GEMINI.md, AGENTS.md) — highest priority." 如果用户的 CLAUDE.md 说 "don't use TDD"，skill 说 "always use TDD"，用户优先。

这里的分析对象是 **Superpowers 项目自己的 CLAUDE.md**——作为一个 case study，展示一段高密度 agent 指令怎么写。

---

## 106 行结构分解

| 段 | 行数 | 内容 | 为什么放在这 |
|---|------|------|------------|
| **If You Are an AI Agent** | 1-10 | 立即停止。读这一节。94% 拒绝率。你的工作是保护人型伙伴。5 个强制检查。 | **第一句就是命令。** "Stop. Read this section before doing anything." 不等 agent 开始读代码、读 git log——先阻断。 |
| **5 mandatory checks** | 11-19 | 1 读 PR template → 2 搜索已有 PR → 3 验证这是真问题 → 4 确认改动属于 core → 5 给人类看完整 diff | **可执行的 checklist。** 不是抽象原则——5 个动词。每个都可以验证（PR template 是否存在、是否搜了 open AND closed PRs、是否问了"what broke"。 |
| **Pull Request Requirements** | 21-26 | PR template 必须完整填写。必须搜索已有 PR（open AND closed）。必须有"人类型参与"的证据。 | **定规则。** 不是 checklist 性质——是提交条件。 |
| **What We Will Not Accept** | 28-80 | 10 种不接受的 PR 类型：第三方依赖、compliance 改 skill、项目特定配置、批量 PR、speculative fixes、领域 skill、fork 改动、捏造内容、捆绑改动 | **负面清单。** 每一类都有具体定义和为什么拒绝。不是模糊的"quality matters"，而是"这类改动会被 close without review"。 |
| **New Harness Support** | 82-102 | 新 harness 集成必须包含 session transcript 证明 `brainstorming` 自动触发。列出会被拒绝的假集成。 | **有操作定义的验收标准。** 不是一个抽象的"must work"——有具体测试用例："Let's make a react todo list" → brainstorming 必须自动触发。 |
| **Skill Changes Require Evaluation** | 104-114 | 改 skill 内容需要 eval evidence。不能用 agent 的一厢情愿替代测试数据。 | **保护核心资产。** skill 是行为代码，不是散文。 |
| **Understand the Project** | 116-120 | 在提议改 skill 设计前，先读已有 skill。项目有自己的哲学。 | **防止无上下文的重写。** |
| **General** | 122-125 | 读 PR template。一个 PR 一个问题。在一个 harness 上测试。描述你解决了什么问题。 | **收尾。** |

---

## 五个设计技术

### 1. 第一句话阻断

```
Stop. Read this section before doing anything.
```

不是 "Welcome to the Superpowers project!" 不是 "Thanks for contributing!" 不是 "Please read these guidelines..."

**"Stop."** — 一个单词，句号，独立成句。Agent 被训练成读取 CLAUDE.md 的第一行作为上下文，这个单词让它在做任何事情之前先停下来。

第二句话给出为什么："This repo has a 94% PR rejection rate." 数据点，立即可验证。不是"我们高质量"——是"94% PR 被拒"。

### 2. 具体拒绝类别胜过抽象质量标准

不写 "PRs must be high quality." 写到**10 种具体类型的不接受改动：**

| 类型 | 为什么拒绝 | Agent 的抓手 |
|------|---------|------------|
| Third-party dependencies | Superpowers 是零依赖插件 | 加了 `npm install` 就知道不行 |
| "Compliance" changes to skills | 内部 skill 哲学和 Anthropic 发布的不一样 | 说"by Anthropic's guidance" 就知道要停 |
| Project-specific configuration | 只裨益一个项目的 | 写的是自己团队的名字就知道不行 |
| Bulk/spray-and-pray PRs | 批量 issue 扫描 | 你开了多个 PR 就知道不行 |
| Speculative/theoretical fixes | 没人真的遇到这个问题 | "my review agent flagged this" 就知道不行 |
| Domain-specific skills | 不是通用 skill | "这 skill 只有 portfolio builder 用" 就知道不行 |
| Fork-specific changes | 污染 upstream | 你在 merge fork 就知道不行 |
| Fabricated content | 幻觉的 | 写了假的问题陈述就知道不行 |
| Bundled unrelated changes | 一个 PR 修三个不相关的东西 | 你有多个不相干的改动就知道不行 |

**每一条都给 agent 一个具体的"闻到就知道停"的信号。** 不是模糊的价值观宣导。

### 3. 操作定义的验收标准

不写 "integration must work." 写：

> Open a clean session in the new harness and send exactly this user message:
>
> Let's make a react todo list
>
> A working integration auto-triggers the brainstorming skill before any code is written.

具体测试用例。具体输入。具体成功标准。Agent 可以直接执行这个验证——不需要解释 "working" 是什么意思。

同样，列出会被拒绝的假集成：
- Manually copying skill files into the harness
- Wrapping with `npx skills` or similar at-runtime shims
- Anything that requires the user to opt in to skills per-session
- Anything where brainstorming does not auto-trigger

负面示例和正面示例一样重要。

### 4. "保护人类型伙伴" 的框架

CLAUDE.md 不把 agent 当成工具使用者——它把 agent 当成**可能会伤害人类型伙伴名誉的代理人**。

```
Your job is to protect your human partner from that outcome.
Submitting a low-quality PR doesn't help them — it wastes the
maintainers' time, burns your human partner's reputation...
```

这是 `using-superpowers` skill 里 "your human partner" 框架的延伸。不是 "you must follow rules"——是 "你在保护一个重要关系。"

### 5. 从"不能做什么"倒推"应该做什么"

5 个强制检查不是正向指南——**它们是 10 种拒绝类别的反向工程。**

| 如果 CLAUDE.md 只写 | PR 检查清单应该是什么 |
|-------------------|-------------------|
| "No duplicates" | 搜索 open AND closed PRs |
| "No fabricated content" | 验证这是真问题——问用户 "what broke, what failed" |
| "No speculative fixes" | "my review agent flagged this" 不是问题陈述 |
| "No domain-specific skills" | 确认改动属于 core |
| "No bulk PRs" | 给人类看完整 diff，得到明确同意 |

这不是随便写了 5 个检查——每个检查对应一个具体的拒绝类别。

---

## 为什么 106 行够用

CLAUDE.md 不是技术文档——是**行为编程**。它的每一个段落在给 agent 一个具体的决策信号：

- "Stop." → 阻断行动，强制阅读
- 94% → 建立风险感知
- 5 checks → 可执行流程
- 10 rejected types → 具体反例，触发"闻到了就停"
- 1 test case → 不是抽象的"works"，是可执行的验收标准
- "protect your human partner" → 激励框架，不是规则清单

**Agent 不需要记住所有条款。** 它只需要在每个决策点被 CLAUDE.md 触发检查。

---

## 关键源文件

| 文件 | 行数 | 角色 |
|------|------|------|
| `CLAUDE.md` | 106 | Superpowers 贡献者行为指令 |
| `AGENTS.md` | 106 | 同 CLAUDE.md（多平台兼容） |
| `.github/PULL_REQUEST_TEMPLATE.md` | — | PR 模板（CLAUDE.md 第 1 个检查指向它） |
