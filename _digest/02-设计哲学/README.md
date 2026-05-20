# 02 — 设计哲学：核心设计理念的深度剖析

## 这个目录研究什么

`00-学习路径/` 研究的是"**做什么**"和"**怎么做**"。这个目录研究的是"**为什么这么做**"。

Superpowers 有很多反直觉的设计决策：

- 为什么每个技能里都有"借口 vs 真相"表格？这不是浪费 token 吗？
- 为什么有些技能是"刚性"（Iron Law），有些是"柔性"？划分标准是什么？
- 为什么"先写测试"被规定为不可商量的铁律，而不是"推荐做法"？
- 为什么 CLAUDE.md 里反复强调"your human partner"而不是"the user"？
- 为什么 writing-skills 强调 description 只能写触发条件，不能总结工作流？

这些不是随意设计的——背后有对 agent 行为的深刻理解、说服心理学的研究支撑、以及大量对抗测试的实证数据。

这个目录就是挖掘这些"为什么"。

---

## 为什么需要这个维度

理解设计哲学有 3 个层次的价值：

**层次 1：更好地使用 Superpowers**
知道为什么某项规则存在，你（和你的 agent）更不可能在压力下绕过它。当你理解"借口 vs 真相"表格背后的对抗测试过程，你就不太可能觉得"这次不一样"。

**层次 2：设计自己的指令体系**
如果你要给自己的项目写 CLAUDE.md 或技能文档，Superpowers 的设计哲学是可以迁移的。你会学到：怎么设计防绕过的规则、什么时候需要 Iron Law、怎么用"证据先行"替代"看起来完成了"。

**层次 3：理解 agent 行为的深层规律**
Superpowers 的设计哲学本质上是关于"怎么让 LLM agent 在压力下仍然遵守规则"的知识体系。这套知识不只适用于 Superpowers——它适用于任何需要 agent 可靠执行复杂指令的场景。

---

## 5 个核心设计问题

| # | 设计问题 | 核心张力 | 对应文件 |
|---|---------|---------|---------|
| 1 | 怎么防止 agent 在压力下找借口绕过规则？ | 灵活性 vs 纪律 | `00-防御心理合理化` |
| 2 | 为什么有些规则是"Iron Law"，有些只是"推荐"？ | 刚性 vs 柔性的划分逻辑 | `01-铁律与刚性层级` |
| 3 | 为什么"看起来完成了"不等于"完成了"？ | 信任 vs 验证 | `02-证据先行` |
| 4 | "your human partner" 不只是话术——它背后的行为设计是什么？ | 工具关系 vs 合作关系 | `03-human-partner框架` |
| 5 | 为什么 description 写工作流会导致 agent 跳过技能？ | 搜索优化 vs 行为引导 | `04-CSO描述陷阱` |

---

## 源文件索引

| 源文件 | 提供了什么 |
|--------|-----------|
| `writing-skills/persuasion-principles.md` | 说服心理学研究（Cialdini 2021, Meincke et al. 2025）：33%→72% 合规率提升的实证基础 |
| `writing-skills/testing-skills-with-subagents.md` | 对抗测试方法论：怎么用压力场景测试 agent 行为 |
| `systematic-debugging/CREATION-LOG.md` | 一个技能从无到有的完整创建日志——展示 TDD 方法论在实际技能创建中的应用 |
| `writing-skills/SKILL.md` | Bulletproofing 技术、CSO 原理、说服心理学的技能化应用 |
| `skills/test-driven-development/SKILL.md` | 13 条 Red Flags + 借口反驳表——discipline-enforcing 技能的设计范本 |
| `skills/systematic-debugging/SKILL.md` | Iron Law 设计范本 + 11 条 Red Flags |
| `CLAUDE.md` | "your human partner" 的实际应用、PR 标准的哲学基础 |
| `skills/verification-before-completion/SKILL.md` | "证据先行"哲学的最直接体现——24 个失败案例 |

---

## 文件规划

| 序号 | 文件 | 核心内容 |
|------|------|---------|
| 00 | `00-防御心理合理化.md` | Superpowers 最独特的设计贡献：借口 vs 真相表格、Red Flags 列表的心理学基础；为什么 agent 会在压力下自我合理化；Cialdini 的说服 5 原则如何被转化为 prompt 设计技术；对抗测试如何发现并堵上每个借口 |
| 01 | `01-铁律与刚性层级.md` | 刚性技能 vs 柔性技能的划分标准：为什么 TDD 是 Iron Law 而 SDD 是柔性？Iron Law 的设计要素（不可商量的表述、违反即重置、没有例外条款）；"Violating the letter of the rules is violating the spirit of the rules" 为什么是必要的 |
| 02 | `02-证据先行.md` | Verificationism 的设计哲学：为什么"agent 说完成了"不可信；24 个真实失败案例的分类学；verification-before-completion 的 4 条验证原则；这和 TDD 的"先看测试失败"是同构的 |
| 03 | `03-human-partner框架.md` | "your human partner" 的行为设计意图：建立合作关系而非工具使用关系；承诺一致性原理（agent 同意了"partner"身份 = 更难合理化背弃行为）；统一性原理（Unity）在 prompt 设计中的应用；为什么 CLAUDE.md 用"protect your human partner from that outcome"来激活防御性动机 |
| 04 | `04-CSO描述陷阱.md` | Claude Search Optimization 的核心发现：description 写工作流 = agent 只读 description 跳过技能正文；SDD 的真实案例（原本 description 含工作流 → agent 做了单阶段 review）；这个陷阱的深层原因（agent 的 attention 机制和信息搜寻行为）；怎么设计一个"好的" description |

---

## 与其他目录的关系

```
_digest/
├── 00-学习路径/    ← 怎么用 Superpowers
├── 01-项目架构/    ← 怎么运作的（技术机制）
├── 02-设计哲学/    ← ★ 当前目录：为什么这么设计（设计思想）
├── 03-可借鉴模式/  ← 设计哲学的工程化产出
└── 04-演化与背景/  ← 这些设计是怎么演化出来的
```

02 和 03 的区别：02 回答"**为什么**"（思想层），03 回答"**怎么拿出来用**"（工程层）。02 是 03 的理论基础。
