# ClawSelf Perception（感知层）技能说明

## 是什么

ClawSelf 第二层：**感知层**。整合社交热度（Social Hype）、聪明钱流入（Smart Money）与叙事话题（Topic Rush），产出 `market_context.json`，并对照 `persona.md` 标出高优先级素材，以「下滑」语气向用户汇报。建议挂载到 Cron 每 2 小时执行一次，保证分身始终处于共识最前线。

## 何时用

- 需要「扫一眼市场」「感知层」「热点捕捉」「共识在哪」时。
- 需要为分身提供发帖素材、产出 `market_context.json` 供内容层与执行层使用时。

## 技能文件

- **clawself-perception.md**：完整执行流程（查热度 → 查资金 → 查叙事 → 合成 JSON → 高优先级匹配 → 汇报）。
- **README.md**：本说明。

## 依赖

- **crypto-market-rank**：Social Hype Leaderboard、Smart Money Inflow Rank 的请求/响应格式。
- **meme-rush**：Topic Rush Rank List 的请求/响应格式。

## 产出

- **market_context.json**：含 `socialHypeTop5`、`smartMoneyTop3`、`topicRushRising`、`highPriority`（与人设匹配的币种/话题），供第三层内容合成与第四层执行调度使用。
