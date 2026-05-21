# 01 — 项目架构：Superpowers 本身怎么运作的

## 这个目录研究什么

`00-学习路径/` 研究的是"**怎么用** Superpowers"——它提供了哪些技能、怎么组合这些技能来完成软件开发。

这个目录研究的是"**Superpowers 本身怎么运作的**"——作为一套运行在 AI coding agent 上的"操作系统"，它的技术架构是怎样的。

具体来说，回答以下问题：

1. **Bootstrap 链路**：一个 session 启动时，Superpowers 是怎么"激活"的？从 hooks 到 using-superpowers 调度中心，完整链路是什么？
2. **多平台适配**：Superpowers 声称支持 Claude Code、Codex、Cursor、OpenCode、Gemini CLI 等多个平台。它是怎么做到的？适配层长什么样？
3. **CLAUDE.md 的设计**：为什么仅仅 106 行就能彻底改变 agent 的行为？它用了什么信息密度和结构设计？
4. **测试体系**：怎么测试"agent 行为"？headless session 是什么？怎么验证一个技能在真实 agent 上是否生效？

---

## 为什么需要这个维度

学会使用一套工具，和理解这套工具本身的架构，是两个层次的能力。

理解 Superpowers 的技术架构有实际价值：
- **调试技能不生效的问题**：知道 bootstrap 链路后才能定位是 hook 没加载、using-superpowers 没匹配、还是技能本身有 bug
- **给自己的项目设计类似的指令体系**：CLAUDE.md 的 106 行设计、多平台适配模式，都是可借鉴的工程实践
- **为 Superpowers 做贡献**：理解测试体系后才能提交有质量的 PR（94% 拒绝率的反面——那 6% 是怎么通过的）

---

## 源文件索引

| 源文件 | 用途 | 关键内容 |
|--------|------|---------|
| `hooks/session-start` | 57 行 bootstrap 脚本 | session 启动时的初始化逻辑 |
| `hooks/hooks.json` | Hook 配置文件 | 定义什么时候触发什么 hook |
| `CLAUDE.md` | 106 行项目指令 | 核心行为规则、贡献指南 |
| `AGENTS.md` | 指向 CLAUDE.md 的符号链接 | 多平台兼容 |
| `skills/using-superpowers/SKILL.md` | 调度中心 | 技能发现和触发逻辑 |
| `.claude-plugin/plugin.json` | Claude Code 插件清单 | 文件注册、hook 声明 |
| `.codex-plugin/` | Codex CLI 适配 | Codex 平台插件配置 |
| `.cursor-plugin/` | Cursor 适配 | Cursor 平台插件配置 |
| `.opencode/` | OpenCode 适配 | OpenCode 平台配置 |
| `docs/testing.md` | 303 行测试文档 | headless session 测试方法 |
| `tests/` | 集成测试 | 各技能的实际测试用例 |
| `scripts/` | 工具脚本 | 版本管理、插件同步 |

---

## 文件规划

| 序号 | 文件 | 研究问题 |
|------|------|---------|
| 00 | `00-目录结构与工程约定.md` | Skill 对用户项目目录有什么假设？三层约定（Superpowers 路由/Plan 结构/用户覆盖）各负责什么？真实 spec/plan 的信息密度对比。Worktree 为什么难、Superpowers 怎么解的 |
| 01 | `01-bootstrap与技能加载.md` | 从 `hooks/session-start` 到 `using-superpowers` 的完整 7 阶段链路：hook 触发 → polyglot wrapper → JSON 注入 → 平台特定路径 → skill 匹配 → 贡献者指令 |
| 02 | `02-多平台适配.md` | 6 个平台的适配层逐一分析：Claude Code、Cursor、Codex、Gemini CLI、OpenCode、Copilot CLI。三层适配架构（注入/发现/翻译）。工具名映射表 |
| 03 | `03-CLAUDE.md的设计.md` | 106 行的行为编程分析：5 个设计技术（第一句阻断、具体拒绝类别、操作定义验收、保护 partner 框架、倒推 checklist）。为什么 106 行够用 |
| 04 | `04-测试体系.md` | 三层测试架构：快速测试 → 集成测试 → GATE+对抗测试。Headless session 原理。RED-GREEN-REFACTOR for skills。测试基础设施总览 |

---

## 与其他目录的关系

```
_digest/
├── 00-学习路径/    ← 怎么用 Superpowers 做项目
├── 01-项目架构/    ← ★ 当前目录：Superpowers 本身的技术架构
├── 02-设计哲学/    ← 为什么这么设计（思想层）
├── 03-可借鉴模式/  ← 能拿出来用到别处的东西
└── 04-演化与背景/  ← 历史、决策、教训
```

01 和 02 的区别：01 回答"**怎么运作的**"（技术机制），02 回答"**为什么这么设计**"（设计思想）。01 是 02 的基础——先理解机制，再理解思想。
