# ClawSelf Content（内容层）技能说明

## 是什么

ClawSelf 第三层：**内容层**。把第一层的 **persona.md**（人设）和第二层的 **market_context.json**（社交热度、聪明钱、叙事话题）合在一起，生成符合人设、适合币安广场的贴文草稿。

- **禁止 AI 八股文**：不用「综上所述」「首先其次」等套话。
- **性格注入**：按 persona 的语气、用词、交易观来写，像「你」在说话。
- **逻辑分支**：根据「热度高但聪明钱在跑」「新叙事 + 人设匹配」等信号，自动选警示类 / 挖掘类 / 中性等发帖逻辑。
- **格式统一**：Hook + 数据支撑 + 互动结尾，文末自动加 3–5 个 Hashtags（如 #BNB #OpenClaw #Meme）。

## 何时用

- 用户/系统说「写一篇贴文」「根据感知结果发帖」「内容层生成」时。
- 已有 `persona.md` 和 `market_context.json`，需要产出可直接发布或先预览的草稿时。

## 技能文件

- **SKILL.md**：完整流程与规则（数据解析 → 内容建模 → 输出格式 → 预览/发布），Agent 按此执行即可。
- **README.md**：本说明，仅作介绍与快速参考。

## 实现方式

- 可按 SKILL.md 的步骤用任意语言实现（如 TypeScript/Next.js + MiniMax 或 GPT-4o）。
- 项目中另有一份 TypeScript 参考实现：`src/content/ContentSynthesizer.ts`，可与本技能对照使用或替换为自研实现。

## 依赖

- 第一层：persona.md（clawself-builder 产出）。
- 第二层：market_context.json（clawself-perception 产出）。
