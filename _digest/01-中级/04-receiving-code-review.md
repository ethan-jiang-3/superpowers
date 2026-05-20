# 04 — Receiving Code Review（接收代码审查）

## 这个技能干什么

教你（和你的 agent）如何**正确地接收代码审查反馈**——用技术评估替代情绪反应，用验证替代盲从，用技术正确性替代社交舒适。

**核心原则：实施前先验证。假设前先问清楚。技术正确性优先于社交舒适。**

## 什么时候触发

收到任何代码审查反馈时——不管是来自 human partner、code reviewer subagent、还是外部 reviewer。

## 硬门禁

**绝对禁止的反应：**
- "You're absolutely right!"（表演式同意）
- "Great point!" / "Excellent feedback!"（表演式赞美）
- "Let me implement that now"（验证之前就动手）

**替代做法：**
- 用自己的话重述技术要求
- 问澄清性问题
- 如果 reviewer 错了，用技术理由 push back
- 直接开始干活（行动 > 语言）

## 怎么用 — 接收模式 (The Response Pattern)

```
收到 code review 反馈时：

1. 读（READ）：完整阅读反馈，不要边读边反应
2. 理解（UNDERSTAND）：用自己的话重述要求（或提问）
3. 验证（VERIFY）：对照 codebase 实际情况检查
4. 评估（EVALUATE）：在这个 codebase 中，这个建议在技术上合理吗？
5. 回应（RESPOND）：技术性确认或有理由的 push back
6. 实施（IMPLEMENT）：一次一项，每项独立测试
```

### 处理不清晰的反馈

```
如果有任何一项不清楚：
  停下来 — 先不要实施任何东西
  对不清楚的项请求澄清

为什么？这些项可能是相关的。部分理解 = 错误实施。
```

**示例：**
```
human partner: "修一下 1-6"
你理解了 1、2、3、6。4、5 不清楚。

❌ 错误：先修 1、2、3、6，之后再问 4、5
✅ 正确："我理解了 1、2、3、6。需要 4 和 5 的澄清再开始。"
```

## 核心概念

### 为什么禁止表演式同意

"Great point!" "You're absolutely right!"——这些看起来无害的社交礼仪实际上是问题：

1. **它们替代了真正的理解**。说"You're right"比理解 reviewer 在说什么并正确实施容易得多。Agent 说"You're right"，然后修错了——因为它在表演同意，不是在技术评估
2. **它们建立了错误的互动模式**。如果每次 review 都以赞美开始，agent 学到的模式是"先赞美，再做事"，而不是"先验证，再做事"
3. **它们浪费 token**。"Thanks for catching that!" 对代码质量没有任何贡献

Superpowers 的态度：**行动说话。直接修。代码本身显示你听到了反馈。**

如果你发现自己想写"Thanks"——删掉它，改为描述修复了什么。

### 按来源处理反馈

**来自 Human Partner 的反馈：**
- **可信的** — 理解后实施
- **仍然可以问** — 如果范围不清楚
- **不要表演式同意**
- **直接跳到行动**或技术性确认

**来自外部 Reviewer 的反馈：**

```
实施之前：
  1. 检查：在这个 codebase 中技术上正确吗？
  2. 检查：会破坏现有功能吗？
  3. 检查：当前实现有原因吗？
  4. 检查：在所有平台/版本上都能工作吗？
  5. 检查：reviewer 理解完整上下文吗？

如果建议看起来不对：
  用技术理由 push back

如果不能轻易验证：
  说出来："没有 [X] 我无法验证这个。我应该 [调查/问/继续] 吗？"

如果和 human partner 之前的决定冲突：
  停下来先和 human partner 讨论
```

**Human partner 的规则：** "外部反馈——保持怀疑，但仔细检查。"

外部反馈 ≠ 命令。它是**要评估的建议，不是要服从的命令。**

### YAGNI 检查

当 reviewer 建议"实现得专业一点/完善一点"时：

```
如果 reviewer 建议 "implement properly"：
  grep codebase 查找实际使用

  如果没被用："这个 endpoint 没被调用。删掉它（YAGNI）？"
  如果被用了：那就好好实现
```

**Human partner 的规则：** "你和 reviewer 都向我汇报。如果我们不需要这个功能，不要加。"

### 实施顺序

多项目反馈的实施顺序：

```
1. 先澄清所有不清楚的项
2. 然后按顺序实施：
   - 阻塞性问题（破坏性变更、安全问题）
   - 简单修复（typo、import、命名）
   - 复杂修复（重构、逻辑变更、架构调整）
3. 每个修复单独测试
4. 验证没有回归
```

### 什么时候 Push Back

**Push back 当：**
- 建议会破坏现有功能
- Reviewer 缺乏完整上下文
- 违反 YAGNI（不需要的功能）
- 在这个技术栈中技术上不正确
- 有遗留/兼容性原因
- 和 human partner 的架构决定冲突

**怎么 Push Back：**
- 用技术理由，不是防御
- 问具体问题
- 引用可工作的测试/代码
- 如果是架构级别的分歧，引入 human partner

### 优雅地纠正自己的 Push Back

如果你 push back 了然后发现自己是错的：

```
✅ "你是对的——我检查了 X，它确实做 Y。现在实施。"
✅ "验证了这一点，你是对的。我最初的理解有误因为 [原因]。修复中。"

❌ 长篇道歉
❌ 解释你为什么 push back
❌ 过度解释
```

事实陈述，然后继续。不要为 push back 道歉——有理由地质疑反馈是正确的行为。

### 当你错了的时候

```
✅ "Fixed. [简短描述什么改了]"
✅ "Good catch — [具体问题]. Fixed in [位置]."
✅ [直接修，展示在代码中]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ 任何感谢表达
```

## 常见错误

| 错误 | 改正 |
|------|------|
| 表演式同意 | 重述技术要求，或直接行动 |
| 盲从实施 | 先对照 codebase 验证 |
| 一批修改不测试 | 一次一个，每项测试 |
| 假设 reviewer 一定对 | 检查是否破坏现有功能 |
| 避免 push back | 技术正确性 > 舒适 |
| 部分实施 | 先澄清所有项 |
| 不能验证也继续 | 说出限制，请求方向 |

## 实际案例

**表演式同意（坏）：**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**技术验证（好）：**
```
Reviewer: "Remove legacy code"
✅ "检查中... build target 是 10.15+，这个 API 需要 13+。遗留代码需要保留以兼容。
    当前实现有错误的 bundle ID——修复它还是放弃 pre-13 支持？"
```

**YAGNI（好）：**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grep 了 codebase——没有东西调用这个 endpoint。删掉它（YAGNI）？
    还是我遗漏了什么使用场景？"
```

**不清晰的项（好）：**
```
human partner: "Fix items 1-6"
理解 1、2、3、6。不清楚 4、5。
✅ "理解了 1、2、3、6。需要 4 和 5 的澄清再开始实施。"
```

## GitHub 回复规范

当在 GitHub PR 上回复 inline review 评论时，在评论线程中回复（通过 `gh api`），而不是作为顶级 PR 评论。

## 注意事项

- **验证在实施之前**——这是铁律。不管反馈来自谁，先验证它对不对
- **外部反馈 = 建议，不是命令**——评估，然后决定
- **行动 > 语言**——修复代码比说"你说得对"更好地展示了你的态度
- **不清晰的反馈 → 停下来问**——部分理解 = 错误实施
- **Push back 是正确行为**——如果你有技术理由质疑，质疑是你的责任

## 与其他技能的关系

```
receiving-code-review
    ├── 处理 requesting-code-review 产出的 review 报告
    ├── 处理 SDD 中 reviewer subagent 的反馈
    ├── 处理 human partner 的任何技术反馈
    └── 价值观与 verification-before-completion 一致：验证先行，声称在后
```
