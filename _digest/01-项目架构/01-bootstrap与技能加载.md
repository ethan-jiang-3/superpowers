# 01 — Bootstrap 与技能加载链路

## 这个问题为什么重要

你打开一个 AI coding 会话，Superpowers 怎么"醒过来"的？从什么都没有到 using-superpowers 调度中心在线、14 个 skill 随时待命——这中间有一条完整的加载链。

如果不理解这条链路，出了问题你只能猜："skill 为什么不触发？""using-superpowers 到底加载了没有？"

---

## 完整链路概览

```
session 启动
  → hook 触发（hooks.json / hooks-cursor.json / gemini-extension.json）
    → run-hook.cmd（polyglot wrapper，解决 Windows 不支持无扩展名脚本）
      → session-start（bash 脚本，读取 SKILL.md，JSON 注入）
        → 模型上下文里出现 using-superpowers 全文
          → agent 收到任意用户消息时，检查是否有 skill 匹配
            → Skill 工具调用 → 平台加载 skill 文件 → agent 执行
```

---

## Stage 0: 插件安装和清单注册

每个平台的插件清单告诉 harness：skills 在哪、hooks 在哪、bootstrap 指令是什么。

| 平台 | 清单文件 | 声明了什么 |
|------|---------|---------|
| Claude Code | `.claude-plugin/plugin.json` | `name: "superpowers"`, `version: "5.1.0"`（Claude Code 自动发现 `skills/` 目录） |
| Cursor | `.cursor-plugin/plugin.json` | `"skills": "./skills/"`, `"hooks": "./hooks/hooks-cursor.json"`（显式路径） |
| Gemini CLI | `gemini-extension.json` | `"contextFileName": "GEMINI.md"`（不执行脚本，用文件引用注入） |
| OpenCode | `.opencode/plugins/superpowers.js` | JS 插件：`config` hook 注册 skills 路径，`messages.transform` hook 注入 bootstrap |
| Codex App | `.codex-plugin/plugin.json` | `"skills": "./skills/"`（不执行脚本，靠 skill description 自动触发） |
| Copilot CLI | 同 Claude Code 的 marketplace | 同 Claude Code 的 `hooks.json` |

---

## Stage 1: Hook 配置——什么触发 bootstrap

两套 hook 配置文件：

### Claude Code / Copilot CLI: `hooks/hooks.json`

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [{
          "type": "command",
          "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
          "async": false
        }]
      }
    ]
  }
}
```

**触发条件：** `SessionStart` 事件 + matcher `startup|clear|compact`——新 session、`/clear`、上下文压缩时都触发。`async: false` 阻塞 session 初始化，必须等 hook 执行完。

### Cursor: `hooks/hooks-cursor.json`

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [{
      "command": "./hooks/run-hook.cmd session-start"
    }]
  }
}
```

注意 cas: Cursor 用 `sessionStart`（camelCase），Claude Code 用 `SessionStart`（PascalCase）。Cursor 没有 matcher——每次 session 启动都触发。

### Gemini CLI: `gemini-extension.json`

不执行脚本。`"contextFileName": "GEMINI.md"` 让 Gemini 在 session 启动时加载 `GEMINI.md` 的内容。

### OpenCode: `.opencode/plugins/superpowers.js`

没有 JSON hook。JS 插件用两个程序化 hook：
- `config`——把 skills 目录路径注册进 OpenCode 配置
- `experimental.chat.messages.transform`——在每个 agent step 时检查第一条用户消息是否需要注入 bootstrap

---

## Stage 2: Polyglot Wrapper——`hooks/run-hook.cmd`

这是一个**批处理/bash 双语法文件**。存在的唯一原因：Windows CMD 不能直接执行无扩展名的 shell 脚本。

- **Unix（Linux/macOS）：** heredoc `: << 'CMDBLOCK'` 跳过批处理段，直接 `exec bash "${SCRIPT_DIR}/$1" "$@"` 执行目标脚本
- **Windows：** 在 Git for Windows 路径和 PATH 中搜索 `bash.exe`，找到后用它执行。找不到则静默退出（skills 仍然可用，但没有 bootstrap context 注入）

**触发：** `hooks.json` 或 `hooks-cursor.json` 在 session 启动时
**输出：** 目标脚本（`session-start`）的 stdout

---

## Stage 3: Bootstrap 脚本——`hooks/session-start`

这是整个 bootstrap 链的**核心引擎**。57 行 bash。

### Step by step

**1. 解析路径**（line 7-8）
```bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"
```

**2. Legacy 迁移提醒**（line 12-15）
检查 `~/.config/superpowers/skills` 是否存在。存在就生成警告，提醒用户移到 `~/.claude/skills`。

**3. 读取核心 payload**（line 18）
```bash
using_superpowers_content=$(cat "${PLUGIN_ROOT}/skills/using-superpowers/SKILL.md")
```
**把 using-superpowers/SKILL.md 的全文读进变量。** 这是整个 bootstrap 传输的唯一 payload。

**4. JSON 转义**（line 23-31）
纯 bash 参数替换实现 `escape_for_json()`——转义反斜杠、双引号、换行、回车、制表符。比逐字符循环快一个数量级。

**5. 构建 session_context**（line 35）
```
<EXTREMELY_IMPORTANT>
You have superpowers.
**Below is the full content of your 'superpowers:using-superpowers' skill...**
[using-superpowers/SKILL.md 全文]
[可选的 legacy 警告]
</EXTREMELY_IMPORTANT>
```

**6. 平台分发**（line 46-55）——根据环境变量输出不同 JSON 格式：

| 检测条件 | 平台 | JSON 格式 |
|---------|------|---------|
| `CURSOR_PLUGIN_ROOT` 已设置 | Cursor | `{"additional_context": "..."}` (snake_case) |
| `CLAUDE_PLUGIN_ROOT` 已设置且 `COPILOT_CLI` 未设置 | Claude Code | `{"hookSpecificOutput": {"hookEventName": "SessionStart", "additionalContext": "..."}}` (嵌套格式) |
| 其他（包括 `COPILOT_CLI=1`） | Copilot CLI / 未知 | `{"additionalContext": "..."}` (SDK 标准) |

第 41 行的注释说明了为什么必须只输出一种格式：Claude Code 会同时读取 `additional_context` 和 `hookSpecificOutput`，不做去重。如果两个字段都输出，内容会被注入两次。

---

## Stage 4: 调度中心——`using-superpowers/SKILL.md`

这就是被注入的内容。它告诉 agent 三件核心事情：

### 1. 强制 skill 检查（非协商）

```
If you think there is even a 1% chance a skill might apply to what
you are doing, you ABSOLUTELY MUST invoke the skill.

This is not negotiable. This is not optional.
```

### 2. 优先级链

```
用户指令（CLAUDE.md, GEMINI.md, AGENTS.md）
  > Superpowers skills
    > 默认系统 prompt
```

用户永远可以覆盖 skill 行为。

### 3. 平台差异指引

| 平台 | Skill 工具名 |
|------|------------|
| Claude Code | `Skill` |
| Copilot CLI | `skill` |
| Gemini CLI | `activate_skill` |
| 其他 | 查看平台文档 |

### 4. 决策流程（GraphViz 图）

```
User message received → Might any skill apply?
  → [yes, even 1%] → Invoke Skill tool → Announce → Follow skill
  → [definitely not] → Respond
```

### 5. 12 条 Red Flags 表

覆盖 agent 最常见的自欺借口："This is just a simple question""I need more context first""The skill is overkill"——每条都有具体反驳。

---

## Stage 5: 平台特定的 Bootstrap 路径

### Gemini CLI：文件引用，不执行脚本

`GEMINI.md` 内容只有两行：
```
@./skills/using-superpowers/SKILL.md
@./skills/using-superpowers/references/gemini-tools.md
```

`@` 是 Gemini CLI 的文件引用语法。Gemini 把两个文件都注入上下文——包括**其他平台不默认加载的工具映射表**。

### OpenCode：JS 注入，不是 JSON hook

`superpowers.js` 的 `messages.transform` hook：
1. 检查第一条用户消息是否已包含 `EXTREMELY_IMPORTANT`
2. 如未，读取 `SKILL.md`，去掉 frontmatter
3. 把 skill body + OpenCode 专用工具映射包装进 `<EXTREMELY_IMPORTANT>...</EXTREMELY_IMPORTANT>`
4. **插入到第一条用户消息的开头**（不是 system prompt 的 additional context）——避免 token 膨胀和多 system message 问题

因为 `messages.transform` 在每个 agent step（不是每个 turn）都触发，插件做了模块级缓存和双重注入保护。

### Codex App：无脚本，靠 description 自动触发

`plugin.json` 声明 `"skills": "./skills/"`，Codex 自动发现 skills。`using-superpowers` 的 description 里有 `Use when starting any conversation`，Codex 的 skill 系统读到这个 description 后在 session 启动时自动加载。

**没有 hook 脚本，没有 JSON 注入，没有文件引用。** 最轻量的 bootstrap。

---

## Stage 6: Skill 匹配和调用（Bootstrap 之后）

Bootstrap 完成后，using-superpowers 的内容在模型的上下文里。之后的链路是**行为性的**——agent 遵循指令，不是文件系统机制：

1. 用户发任意消息
2. Agent 检查："有 skill 适用吗？"
3. 有（哪怕 1% 概率）→ 调用 `Skill("skill-name")`
4. 平台加载该 skill 的 `SKILL.md`
5. Agent 按 skill 指令执行
6. Skill 引用其他 skill → 形成链（brainstorming → writing-plans → SDD → TDD → code-review → finishing）

---

## Stage 7: 贡献者指令——CLAUDE.md 和 AGENTS.md

这两个文件**不参与**终端用户的 bootstrap 链。它们是给 Superpowers 项目本身的贡献者看的——告诉 agent：这个 repo 有 94% 的 PR 拒绝率、提交前必须做 5 项检查、什么不会被接受。

它们的角色和 `using-superpowers/SKILL.md` 完全不同：一个是给 Superpowers 的用户（调度 skill），一个是给 Superpowers 的贡献者（防止 slop PR）。

---

## 完整 Bootstrap 目录图

```
session 启动（任任平台）
    │
    ├── [Claude Code / Copilot CLI]
    │     hooks/hooks.json → matcher: startup|clear|compact
    │       → run-hook.cmd session-start
    │         → session-start 读取 using-superpowers/SKILL.md
    │           → JSON additional context 注入
    │             → agent 上下文里有了 using-superpowers
    │               → 每条用户消息都检查 skill
    │
    ├── [Cursor]
    │     hooks/hooks-cursor.json → sessionStart（camelCase）
    │       → 同 Claude Code 链条（不同 JSON 输出格式）
    │
    ├── [Gemini CLI]
    │     gemini-extension.json → contextFileName: GEMINI.md
    │       → GEMINI.md: @using-superpowers/SKILL.md + @gemini-tools.md
    │         → agent 上下文里有了 using-superpowers + 工具映射
    │
    ├── [OpenCode]
    │     .opencode/plugins/superpowers.js
    │       → config hook: 注册 skills 路径
    │       → messages.transform hook: 读取 SKILL.md → 注入第一条用户消息
    │         → agent 上下文里有了 using-superpowers + OpenCode 工具映射
    │
    └── [Codex App]
          .codex-plugin/plugin.json → skills: "./skills/"
            → Codex 自动发现 skills
            → using-superpowers description: "Use when starting any conversation"
              → 自动触发
```

---

## 关键设计属性

### 1. 内容是注入的，不是引用的

`session-start` 脚本和 OpenCode JS 插件**读取 SKILL.md 的全文**并嵌入。agent 不需要先调工具才能读——内容已经在上下文里了。

### 2. 平台检测是环境变量，不是配置

`session-start` 用 `CURSOR_PLUGIN_ROOT`、`CLAUDE_PLUGIN_ROOT`、`COPILOT_CLI` 三个环境变量判断平台。不需要配置文件，不需要用户选择。

### 3. Gemini 的分裂方案

Gemini 不执行任何脚本。它用 `@` 文件引用加载两个文件——包括工具映射表。其他平台只在需要时才读工具映射。

### 4. Codex 的极简方案

没有 hook。没有脚本。没有注入。全靠 skill description 里的 `Use when starting any conversation` 触发。

### 5. OpenCode 的缓存

`messages.transform` 在每个 agent step 都触发，所以 JS 插件做了模块级缓存——只读一次文件。同时检查 `EXTREMELY_IMPORTANT` 是否已存在，防止重复注入。

### 6. Hook 格式兼容是精细活

`session-start` 脚本有 10 行注释解释为什么必须只输出一个 JSON 格式：Claude Code 同时读两个字段。如果两个字段都输出，内容重复注入。这是一个具体平台 bug 的 workaround。

---

## 关键源文件索引

| 文件 | 行数 | 角色 |
|------|------|------|
| `hooks/session-start` | 57 | Bootstrap 核心引擎：读取 SKILL.md → JSON 注入 |
| `hooks/hooks.json` | 16 | Claude Code/Copilot CLI hook 配置 |
| `hooks/hooks-cursor.json` | 10 | Cursor hook 配置（不同格式） |
| `hooks/run-hook.cmd` | ~40 | Windows/Unix polyglot wrapper |
| `.claude-plugin/plugin.json` | 19 | Claude Code 插件清单 |
| `.cursor-plugin/plugin.json` | ~15 | Cursor 插件清单 |
| `.codex-plugin/plugin.json` | ~50 | Codex 插件清单（含 marketplace 元数据） |
| `.opencode/plugins/superpowers.js` | ~140 | OpenCode JS 插件（config + messages.transform） |
| `gemini-extension.json` | ~5 | Gemini 扩展配置 |
| `GEMINI.md` | 2 | Gemini 文件引用：`@using-superpowers/SKILL.md` |
| `skills/using-superpowers/SKILL.md` | 118 | 调度中心：skill 检查规则、优先级、Red Flags |
| `CLAUDE.md` | 106 | 贡献者指令（不参与用户 bootstrap） |
| `AGENTS.md` | 106 | 同 CLAUDE.md |
