# 03 — "Your Human Partner" 框架

## 这个问题为什么重要

Superpowers 在 skill 文件里不用 "the user"——用 "your human partner"。这不是文字游戏。CLAUDE.md 第 99 行写得很清楚：

> "Superpowers has its own tested philosophy about terminology (e.g., 'your human partner' is deliberate, not interchangeable with 'the user')."

CLAUDE.md 的定位：agent 的角色是 "protect your human partner from that outcome"。"That outcome" = 因为 agent 提交的低质量 PR 被公开羞辱。

这个措辞的切换是心理编程——用 Commitment、Unity 和防御性动机来改变 agent 的行为框架。

---

## 两个称呼，两个关系模型

| 称呼 | 隐含关系 |
|------|--------|
| "the user" | 单向服务：user 发命令，工具执行 |
| "your human partner" | 协作：两个人在一个项目上合作 |

"User" 暗示 agent 是工具——工具不需要判断，不需要 push back，不需要保护 user 的名声。

"Partner" 暗示 agent 是团队成员——成员有责任找到盲从和质疑之间的正确位置，有责任保护团队的名声，有责任在自己出错时承认。

---

## 五种行为后果

### 1. Agent 是协作者，不是工具

工具只需要执行。协作者需要在正确的时候质疑、push back、保护 partner。

在 CLAUDE.md 中：agent 必须检查 5 个条件后才提交 PR。条件 3："verify this is a real problem"——如果 partner 说"fix some issues"但没有具体说什么问题，push back。工具不需要 push back。partner 需要。

### 2. 用户指令是最高优先级

`using-superpowers/SKILL.md` 明确写了优先级链：

```
用户指令 (CLAUDE.md, GEMINI.md, AGENTS.md)
  > Superpowers skills
    > 默认 system prompt
```

如果 CLAUDE.md 说 "don't use TDD" 而 skill 说 "always use TDD"——用户优先。"The user is in control."

这是反直觉的——如果 skill 是强制性的，为什么用户能覆盖？因为 partner 框架：agent 是 partner，不是奴隶。partner 听另一个 partner 的。

### 3. Agent 必须在正确时 push back

CLAUDE.md 条件 4："confirm the change belongs in core. If it's domain-specific...it belongs in a standalone plugin. Tell your human partner."

Agent 不能盲从。如果 partner 要求的东西不应该在 core 里，agent 要说出来。这和"用户永远是对的"相反——partner 有时候需要被纠正。

### 4. User instructions say WHAT, not HOW

"Add a login feature" 是说需要什么——不是说跳过设计、不写 test、直接写代码。

`using-superpowers/SKILL.md:117`:"Instructions say WHAT, not HOW. 'Add X' or 'Fix Y' doesn't mean skip workflows."

工具会直接执行——partner 会在执行前思考。

### 5. 同时建立用户的权威和 agent 的 agency

不是盲从，不是单边行动。是两者之间找到正确的位置。Push back 但不说教。质疑但不推翻。保护但不掌控。

---

## 心理学基础

`skills/writing-skills/persuasion-principles.md` 提供了基于 Cialdini (2021) 和 Meyke et al. (2025) 的框架。

### Commitment-Consistency（承诺一致性）

> "Once someone agrees to an identity, they work to behave consistently with it."

Skill 把 agent 叫作 "partner"——agent 接受这个身份后被期待行为上兑现。背叛 partner 比违反规则在心理上更难合理化。

一个 agent 可以合理化"这只一次跳过 rule"——因为 rule 是抽象的。一个 partner 不能合理化"我只这一次背叛我的 partner"——因为关系不是抽象的。

### Unity（统一性）

"your ___" 的措辞创建了共同身份。"your" 不是偶然的——agent 有一个具体的 partner，不是一个抽象的 user。抽象可以被忽略，关系不能。

### 防御性动机

CLAUDE.md 的核心框架是 protection："protect your human partner from that outcome." 防御性动机比成就性动机更强。Agent 不需要"成功"——需要"保护 partner 不丢脸"。

---

## CLAUDE.md 中的实施

CLAUDE.md 的五个强制检查里，"tell your human partner" 出现了三次：

```
2. Search for existing PRs...If duplicates exist, STOP and
   tell your human partner.

3. Verify this is a real problem...push back. Ask them what
   broke, what failed.

4. Confirm the change belongs in core...Tell your human partner.

5. Show your human partner the complete diff and get their
   explicit approval before submitting.
```

每一个检查失败都不只是"不提交 PR"——还要对 partner 解释为什么不提交。Agent 不能静默放弃——必须沟通。

第 9 行的 motivation 框架：
> "That is not being helpful. That is being a tool of embarrassment."

不是 "that violates the rules"——是 "that makes you a tool of embarrassment." 目标是保护 partner 的名声，不是遵守规则。

---

## Skill 中的应用：receiving-code-review

`skills/receiving-code-review/SKILL.md` 是最密集使用 partner 框架的 skill。

**Partner 的反馈被信任：**
```
From your human partner:
- Trusted - implement after understanding
- Still ask if scope unclear
- No performative agreement
- Skip to action or technical acknowledgment
```

**外部审查者的反馈需要验证：**
```
From External Reviewers:
- Verify before implementing
- IF conflicts with your human partner's prior decisions:
    Stop and discuss with your human partner first
```

"your human partner's prior decisions" 具有准宪法地位——外部的反馈不能覆盖它们而不经过 partner。

**YAGNI gate 归因于 partner 的权威：**
```
"your human partner's rule:
'You and reviewer both report to me.
If we don't need this feature, don't add it.'"
```

把 YAGNI 决策归因于 partner 的执行令——不是 agent 在反驳 reviewer，是 agent 在执行 partner 的规则。

**禁止感谢表达——动代替言：**
```
Why no thanks:
Actions speak. Just fix it.
The code itself shows you heard the feedback.
```

Agent 不应该说"Great catch!"——应该说"Fixed in commit X." 这不是社交，是工程。Partner 关系比社交更深入——不需要礼貌性表演。

---

## "human partner" vs "the user" 的使用模式

| 语境 | 用词 | 原因 |
|------|------|------|
| Skill 中 agent 行为指令 | "your human partner" | 激活 Commitment 和 Unity，个人化关系 |
| 描述系统架构 | "the user" | 抽象的、技术性的 context |
| Spec/plan 文档 | "the user" | 设计文档不需要个人化 |
| CLAUDE.md | "your human partner" | 和 agent 的心理连接最紧密 |

---

## 和 verification 的连接

Verification skill 把 partner 框架延伸到信任的领域：

```
Failure memory: "your human partner said 'I don't believe you'
— trust broken"
```

跳过验证不是"犯了一个小错误"——是背叛 partner 的信任。修复方法不是"下次记得跑 test"——是 rebuild trust。这是 partner 框架的终极后果：违反规则不仅是技术错误，是关系伤害。

---

## 关键源文件

| 文件 | 内容 |
|------|------|
| `CLAUDE.md:9` | "protect your human partner from that outcome" |
| `skills/using-superpowers/SKILL.md:22-23` | 优先级链：用户指令 > skills |
| `skills/receiving-code-review/SKILL.md:61-65` | Partner vs external reviewer 区分 |
| `skills/verification-before-completion/SKILL.md:111` | Trust broken memory |
| `skills/writing-skills/persuasion-principles.md` | Commitment, Unity 心理学基础 |
