# 05 — Verification Before Completion（完成前验证）

## 这个技能干什么

一个硬门禁：**在声称任何工作"完成"之前，必须运行验证命令并拿到证据。** "证据先行，再说完成。"

自称"测试通过了"但你根本没跑过测试 = 说谎，不是效率。

## 什么时候触发

任何声称完成之前：
- 说 "Done" / "Fix applied" / "Tests pass" / "Build succeeds"
- commit 之前
- 创建 PR 之前
- 进入下一个 task 之前
- 委托 subagent 之前

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
（没有刚跑的验证证据 = 不能说完成）
```

## 门禁函数 — 每次声称完成前的 5 步

```
1. 识别 — 什么命令能证明这个声称？
2. 执行 — 跑完整命令（不是部分、不是"应该可以"）
3. 阅读 — 看完整输出，检查 exit code，数失败数
4. 验证 — 输出真的证实了声称吗？
   - 否 → 报告实际状态，附证据
   - 是 → 声称完成，附证据
5. 然后 — 才能说你完成了

跳过任何一步 = 撒谎，不是验证
```

## 常见翻车现场

| 你声称 | 需要 | 不够的 |
|--------|------|--------|
| "测试过了" | 测试命令输出：0 failures | "上次跑过"、"应该过了" |
| "Lint 干净" | Lint 输出：0 errors | 部分检查 |
| "编译成功" | 编译命令：exit 0 | 只看 lint 没看编译 |
| "Bug 修好了" | 原始症状复现测试：通过 | "代码改了，应该好了" |
| "需求满足了" | 逐行对照 checklist | 只看测试 |

## Red Flags — 赶紧停

- 用了 "should" / "probably" / "seems to"
- 验证前就说 "Great!" / "Perfect!" / "Done!"
- 准备 commit/push/PR 但没验证
- 相信 agent 报告 "success" 而不验证
- **任何暗示成功的措辞 — 如果你没跑过验证命令**

## 为什么不验证 = 不可饶恕

来自 24 个失败记忆的教训：
- 用户说 "我不信你" — 信任破裂
- 没定义的函数被 ship — 线上 crash
- 遗漏需求被 ship — 不完整的功能
- 假完成 → 改方向 → 返工 → 浪费时间

## 正确做法示例

```
✅ [跑 npm test] [输出: 34/34 pass] "34 个测试全部通过"
❌ "测试应该没问题了"

✅ [跑 cargo build] [exit 0] "编译通过"
❌ "lint 过了"（lint 不等于编译）

✅ 重读 plan → 建 checklist → 逐项验证 → 报告
❌ "测试过了一个阶段完成"
```

## 与其他技能的关系

- 独立技能，不依赖任何其他技能
- 被所有执行流程调用（SDD 的每个 task 完成后、executing-plans 的每个 task 完成后）
- `finishing-a-development-branch` 的第一步就是验证测试
