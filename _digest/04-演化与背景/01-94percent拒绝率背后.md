# 01 — 94% PR 拒绝率背后

## 这个数字从哪来

RELEASE-NOTES.md v5.1.0 (2026-04-30)：

> "An audit of the last 100 closed PRs against this repo showed a 94% rejection rate driven by AI-generated slop: agents that didn't read the PR template, opened duplicates, fabricated problem descriptions, or pushed fork- or domain-specific changes upstream."

随后 CLAUDE.md 被重写，顶部加了 "If You Are an AI Agent" 段。94% 出现在 CLAUDE.md 的两次——第 7 行（开场）和第 61 行（"Fabricated content" 段作为加强）。

---

## 9 种被拒类型

CLAUDE.md 把被拒的 PR 分成 9 类——每一类都有具体定义和拒绝理由：

| 类型 | 为什么被拒 | Agent 的"闻到就停"信号 |
|------|---------|---------------------|
| **第三方依赖** | Superpowers 是零依赖插件。加了外部工具 = 应该自己发 plugin | 写了 `npm install` 指令 |
| **Compliance 改 skill** | Anthropic 公开 skill 指南和 Superpowers 内部哲学不同。没 eval evidence 的"合规"改动不接受 | "by Anthropic's guidance" |
| **项目特定配置** | 只裨益一个项目/团队/领域的 | 写了自己团队/项目的名字 |
| **批量/spray-and-pray PR** | 从 issue tracker 抓多个 issue 一次开多个 PR | 开了多个 PR |
| **Speculative/theoretical fix** | 没人真的遇到这个问题。"my review agent flagged this"不是问题陈述 | 受害者是 review agent |
| **领域特定 skill** | 不属于通用 skill | "portfolio builder", "prediction market" |
| **Fork 特殊改** | 污染 upstream | merge fork 分支 |
| **捏造内容** | 幻觉的问题描述、编造的功能 | "This pull request is slop that's made of lies"（真实维护者评语） |
| **捆绑无关改动** | 一个 PR 三个不相关的事 | 多个不相干的 diff |

---

## PR Template 的过滤机制

`.github/PULL_REQUEST_TEMPLATE.md`（126 行）的 7 个必填 section 每个对应一种失败模式：

| Section | 过滤什么 |
|---------|--------|
| "What problem are you trying to solve?" | Speculative fix——必须描述具体失败，不是 "improving" |
| "Is this change appropriate for core?" | 领域 skill、项目配置、第三方集成 |
| "What alternatives did you consider?" | 表面分析（空 or "none" = 红旗） |
| "Does this PR contain multiple unrelated changes?" | 捆绑 PR |
| "Existing PRs"（checkbox + 搜索 open AND closed） | 重复 PR |
| "Environment tested" 表 | 未测试改动 |
| "Human review" checkbox | 人类没看过的 PR |

特殊场景：
- **新 harness**：需 session transcript 证明 `brainstorming` auto-triggers
- **Skill 改动**：需 eval evidence + 对抗性压力测试结果

---

## 那 6% 通过的 PR 做了什么

从 RELEASE-NOTES 里可追踪的社区贡献：
- v5.1.0: @shaanmajid（删除遗留 CHANGELOG.md）、@arittr（README 修复、Codex 安装指导）
- v5.0.5: @sarbojitrana（brainstorm server ESM fix）
- v5.0.1: @karuturi（marketplace 安装指导）、@mvanhorn（dual-emit fix）
- v4.3.1: 多个 Windows bug fix

模式：**解决的是一个具体的、真实的、有 issue number 的问题，包含具体测试，范围是一个东西。**

---

## CLAUDE.md 的防御设计

第 1-10 行是直接针对 AI agent 的行为中断：

1. "Stop. Read this section before doing anything." → **阻断**（在 agent 读代码/git log 前）
2. "94% PR rejection rate" → **数据**（不是观点，是事实）
3. "maintainers close slop PRs within hours, often with public comments" → **社会后果**
4. "Your job is to protect your human partner from that outcome" → **防御性动机框架**
5. 5 个可执行的强制检查 → **checklist**

这个 CLAUDE.md 在每次 agent session 启动时都被读。一个文件塑造所有 agent 的行为。

---

## 关键源文件

| 文件 | 内容 |
|------|------|
| `CLAUDE.md` | 贡献者防御指南（106 行） |
| `.github/PULL_REQUEST_TEMPLATE.md` | 7 段过滤机制（126 行） |
| `RELEASE-NOTES.md` v5.1.0 | 94% audit 首次发布 |
