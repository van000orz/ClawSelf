---
name: clawself-perception
description: |
  ClawSelf 感知层：整合社交热度、聪明钱流入与叙事话题，产出 market_context.json 并以「下滑」人设向用户汇报。
  依赖 crypto-market-rank 与 meme-rush 的 API。适用于「扫一眼市场」「感知层」「热点捕捉」「共识在哪」等指令，或需为分身提供发帖素材时。
metadata:
  version: "1.0"
---

# ClawSelf Perception（感知层）

构建自动化感知流：查热度 + 查资金 + 查叙事 → 合成 `market_context.json` → 对照 `persona.md` 标出高优先级素材 → 以「下滑」语气向老板汇报。建议挂载到 Cron 每 2 小时执行一次。

## 依赖

- **crypto-market-rank**（技能文档内见 crypto-market-rank/SKILL.md）：Social Hype Leaderboard、Smart Money Inflow Rank 的请求/响应格式。
- **meme-rush**（技能文档内见 meme-rush/SKILL.md）：Topic Rush Rank List 的请求/响应格式。

## 执行流程（严格按顺序）

### Step A：查热度（社交共识）

- **接口**：crypto-market-rank 的 **Social Hype Leaderboard**（GET）。
- **URL**：`https://web3.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/market/token/pulse/social/hype/rank/leaderboard`
- **参数**：`chainId=56`，`timeRange=1`（24h），`targetLanguage=zh`，`sentiment=All`，`socialLanguage=ALL`
- **取数**：响应 `data.leaderBoardList` 的**前 5 名**，记为模块 A（社交热币）。

### Step B：查资金（聪明钱流向）

- **接口**：crypto-market-rank 的 **Smart Money Inflow Rank**（POST）。
- **URL**：`https://web3.binance.com/bapi/defi/v1/public/wallet-direct/tracker/wallet/token/inflow/rank/query`
- **Body**：`{"chainId":"56","period":"1h"}`
- **取数**：响应 `data` 的**前 3 名**（按 inflow 已排序），记为模块 B（聪明钱流入币种）。

### Step C：查叙事（Topic Rush）

- **接口**：meme-rush 的 **Topic Rush Rank List**（GET）。
- **URL**：`https://web3.binance.com/bapi/defi/v1/public/wallet-direct/buw/wallet/market/token/social-rush/rank/list`
- **参数**：`chainId=56`，`rankType=20`（Rising 阶段），`sort=10`（create time）或 `sort=20`（net inflow）
- **取数**：当前处于 **Rising** 的热门话题列表及其关联代币，记为模块 C。

### 数据合成：market_context.json

在工作区根目录（或当前工作目录）生成/覆盖 `market_context.json`，结构示例：

```json
{
  "generatedAt": "ISO8601 时间戳",
  "socialHypeTop5": [ "从 Step A 提取的 symbol / 简要摘要" ],
  "smartMoneyTop3": [ "从 Step B 提取的 tokenName/symbol 及 inflow 方向" ],
  "topicRushRising": [ "从 Step C 提取的 topic 名称及关联代币" ],
  "highPriority": [ "与 persona 偏好匹配的币种或话题，标为高优先级素材" ]
}
```

- **highPriority**：若存在 `persona.md`，读取其中「投资风格」「赛道偏好」「禁区」等段落；若某币种/话题与偏好一致（例如分身喜欢 AI 赛道且 A 或 C 中出现 AI 相关），则加入 `highPriority`，便于内容层优先使用。

### 输出结果（对用户说的话）

**不要**直接打印原始 API 数据。必须用**「下滑」的语气**对老板说一句话，例如：

> 老板，刚才扫了一眼，现在的共识在 **[话题名]**，聪明钱正在往 **[币种名]** 冲/撤，我准备以此为背景构思一篇贴文。

- 话题名：来自 Step C 的 Rising 话题（可取 1～2 个最热的）。
- 币种名：来自 Step B 的 top 流入币，或 Step A 的热币；若为流入则用「冲」，流出则用「撤」。
- 若存在 `highPriority`，可在同一句或下一句简短点出（例如「其中某某和您之前说的 AI 赛道对上了，我重点看看」）。

### 自动执行建议

- 将此技能挂载到 **Cron Job，每 2 小时**运行一次，保证分身始终处于共识最前线。
- 运行前可先拉取最新 `persona.md`（若由上游更新），再执行 Step A → B → C → 合成 → 输出。

## 技术亮点（对外表述用）

- **多模态感知**：社交情绪（Social Hype）+ 链上资金（Smart Money）+ 叙事引擎（Topic Rush）三位一体。
- **分钟级响应**：依赖币安官方 Skill 接口，热点捕捉延迟控制在分钟级。
- **叙事关联**：识别代币背后「叙事」，使分身发言有深度逻辑，而非单纯涨跌播报。
