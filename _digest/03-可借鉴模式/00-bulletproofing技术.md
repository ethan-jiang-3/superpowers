# 00 — Bulletproofing 技术

## 这是什么

Bulletproofing 是 Superpowers 里最值得独立拿出来用的方法论：**四招把一段 agent 指令从"agent 会在压力下合理化跳过"变成"agent 在最大压力下仍然遵守"。**

它来自 `writing-skills/SKILL.md:459-525`，基于 13 个 skill 的对抗性测试经验，底层是 Cialdini (2021) 说服心理学 + Meyke et al. (2025) 28,000 次 LLM 对话的验证数据。

---

## 四招

### 第 1 招：显式堵漏洞

不说"不要做 X"。列出"X 的所有变体"并逐一禁止。

```
# 坏的（只写规则）
"写代码在前？删掉。"

# 好的（堵所有已知 workaround）
"写代码在前？删掉。重来。
 不允许：
 - 不要保留为'reference'
 - 不要'adapt'同时写 test
 - 不要看它
 - 删除就是删除"
```

每一条"不要"对应一个在对抗性测试中观察到的 agent 行为。

### 第 2 招：spirit-vs-letter 声明

```
Violating the letter of the rules is violating the spirit of the rules.
```

这句话必须出现在 skill 文档的**最前面**（约第 14 行的位置），在任何具体规则之前。它关闭了一整类合理化——agent 不能说"我理解精神，形式上没那么重要"因为规则声明了形式就是精神。

这句话是 TDD 第 2 次 REFACTOR 中加入的。迭代 1 的 agent 用"tests after achieve the same goals"绕过了规则。迭代 2 的 agent 换了策略——"spirit not letter"。迭代 3 加了这句话后 agent 才停止。

### 第 3 招：借口反驳表

```
| 借口 | 现实 |
|------|------|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "Already manually tested" | Manual ≠ repeatable. |
| "Tests after achieve same" | Tests-after answer "what does this do?" Tests-first answer "what should this do?" |
```

左列是 agent 在压力下真实说出的借口（逐字转录），右列是简短、事实性的反驳。

**五条设计原则：**
1. 一句话反驳——长了给 agent 继续辩论的空间
2. 针对性强——agent 看到自己刚想的借口就在表里，自我识别率极高
3. 行动导向——告诉 agent 做什么（"RUN the verification"），不只说不对
4. 事实不是观点——"Tests passing immediately prove nothing" 是事实，无法二次合理化
5. 覆盖不同类别——简单性、效率性、成本性、哲学性、探索性各一条

### 第 4 招：Red Flags 列表

```
## Red Flags — STOP and Start Over
- Code before test
- "It's just a simple change"
- "I already manually tested it"
- "Tests after achieve the same purpose"
- "It's about spirit not ritual"
- "This is different because..."

All of these mean: Delete code. Start over with TDD.
```

每条 Red Flag 是 agent 可能在脑子里闪过的想法。看到它在表里 = 触发 STOP。最后一条 "This is different because..." 是万能捕捞器，捕获所有未被单独列出的变体。

---

## TDD 的 6 次迭代：完整案例

| 迭代 | 加什么 | Agent 找到的新借口 |
|------|--------|-----------------|
| 1 | 基础规则："先写 test" | "Tests after achieve same goals" → 加"Why Order Matters" |
| 2 | Why Order Matters 章节 | "spirit not letter" → 加"违反字就是违反精神" |
| 3 | Spirit-vs-letter 声明 | 遵守！但在其他地方找漏洞 |
| 4-6 | 关闭新漏洞 | 6 次后才 bulletproof——最大压力下 100% 合规 |

**关键经验：没有任何防御是一次写对的。每个都是对 agent 真实借口的回应。**

---

## 饱和标准

怎么知道已经 bulletproof？

1. Agent 在最大压力（3+ 压力组合）下选择正确选项
2. Agent 引用技能中的具体条目作为决策理由
3. Agent 承认被诱惑过但最终遵守了
4. Meta-testing 中 agent 说 "the skill was clear, I should follow it"

---

## 理论基础

基于 Cialdini (2021) 七说服原则 + Meyke et al. (2025) N=28,000 LLM 验证。最有效的组合：
- **Authority**: "YOU MUST", "Never", "No exceptions"
- **Commitment**: TodoWrite 追踪、公开宣布使用 skill
- **Social Proof**: "Every time", "Always", "X without Y = failure"
- **不要用 Liking/Reciprocity**: 产生 sycophancy

数据：说服技术让 LLM 合规率从 33% → 72%（p < .001）。

---

## 关键源文件

| 文件 | 内容 |
|------|------|
| `skills/writing-skills/SKILL.md:459-525` | 四招完整方法论 |
| `skills/writing-skills/testing-skills-with-subagents.md` | 对抗性测试完整流程 |
| `skills/writing-skills/persuasion-principles.md` | 心理学基础 |
| `skills/test-driven-development/SKILL.md` | 参考实现 |
