---
name: clawself-content
description: |
  ClawSelf 第三层内容层：将 persona.md 与 market_context.json 合成为符合人设的币安广场贴文草稿。
  禁止 AI 八股文，注入性格，支持 Hook + 数据支撑 + 互动结尾，自动匹配 Hashtags。适用于「写一篇贴文」「内容层」「根据感知结果发帖」等指令。
metadata:
  version: "1.0"
---

# ClawSelf Content（内容层）

将第一层产出的 **persona.md** 与第二层产出的 **market_context.json** 合成为符合人设的贴文草稿，支持预览多类型（犀利短评 / 深度长文 / 互动提问）或直接发布。

## 输入依赖

- **persona.md**：路径通常为 `~/.openclaw/persona.md` 或工作区根目录 `persona.md`，须包含至少 [语言风格指南] 与 [交易策略底座]。
- **market_context.json**：由 clawself-perception 技能产出，包含 `socialHypeTop5`、`smartMoneyTop3`、`topicRushRising`、`highPriority`。

---

## 1. 数据解析 (Data Parsing)

### 1.1 解析 persona.md

- 读取 persona.md 全文。
- 提取 ** [语言风格指南] ** 段落中的：
  - 句子长短偏好、结构偏好（要点式/叙事式/先结论后论据）
  - 常用句式/口头禅
  - 总体气质（带头大哥/邻家交易员/冷静研究员/毒舌吐槽）
  - 对外强度（温和↔强硬）、幽默/自嘲倾向
  - 黑话 vs 专业术语倾向、Emoji 使用习惯（最常用 3 个）
- 提取 ** [交易策略底座] ** 段落中的：
  - 对加密的总体看法、更信技术指标还是社区情绪
  - HODL vs 旋转、持仓周期、冲锋意愿与触发条件
  - 止损态度、最痛恨的损失类型（被套/踏空）
  - 明确禁区/雷区、偏好赛道/叙事

以上字段用于后续「人设注入」与逻辑分支，缺失时使用默认或留空。

### 1.2 解析 market_context.json

- 读取 JSON，提取：
  - **Social Hype**：`socialHypeTop5`（字符串数组，表示社交热度前 5 的币/摘要）
  - **Smart Money Inflow**：`smartMoneyTop3`（数组，每项含 symbol/tokenName、inflow、direction 为 in/out）
  - **Rising Topics**：`topicRushRising`（数组，每项含 name、可选 tokenList）
  - **高优先级**：`highPriority`（与人设匹配的币种或话题）

若字段为简化格式（如 `smartMoneyTop3` 为字符串数组），则做兼容：symbol 取字符串本身，direction 默认 in，inflow 默认 0。

---

## 2. 内容建模 (Content Modeling)

### 2.1 禁止 AI 八股文

生成正文时**严禁**使用以下词汇或结构：

- 综上所述、首先、其次、最后、总之
- 一方面、另一方面、不可否认、众所周知

若大模型产出中包含以上任一，须在输出前替换或删除。

### 2.2 性格注入

- 将 1.1 解析出的语言风格与交易策略，整理成「人设块」注入给大模型系统提示。
- 要求大模型：用该人设的口气写「Hot Take」，不卑不亢，有数据有观点，可带口头禅与 Emoji 习惯。

### 2.3 逻辑链条（发帖类型）

根据数据信号决定本次发帖的逻辑类型，用于系统提示中的语气与侧重点：

| 条件 | 逻辑类型 | 发帖倾向 |
|------|----------|----------|
| [社交热度高] 且 [聪明钱流出] | **alert（警示类）** | 警示/冷静提醒，不要无脑唱多 |
| [新叙事出现] 且 [高优先级有匹配] | **dig（挖掘类）** | 挖掘/发现机会，可带一点 FOMO 但要有逻辑 |
| [聪明钱流入] 且 [高优先级有匹配] | **fomo** | 可适度「冲」，但要有数据支撑 |
| 其余 | **neutral（中性）** | 结合数据给出有态度的 Hot Take 即可 |

---

## 3. 输出格式化 (Output Generation)

### 3.1 结构要求（符合币安广场算法）

生成内容须包含三部分，但**大模型只输出正文**，不输出「Hook:」「Body:」等标签：

- **Hook（钩子）**：首段或首句，吸引点击与停留。
- **数据支撑段落**：引用 socialHypeTop5、smartMoneyTop3、topicRushRising 中的具体信号，用自然语言融入，不堆砌原始数据。
- **引导互动的结尾（CTA）**：金句、提问或邀请评论区讨论。

程序侧可将正文按段落拆分为：第一段 = Hook，中间 = Body，最后一段 = CTA。

### 3.2 Hashtags

- 正文中**不要**包含 Hashtag，由程序在正文末尾另行拼接。
- 从池中自动匹配 3–5 个：必含 `#BNB`、`#OpenClaw`；若有 Rising 叙事则加入 `#Meme`、`#Narrative`；若有高优先级则加入 `#Alpha`；其余从 `#Web3`、`#Crypto`、`#DeFi`、`#SmartMoney`、`#BinanceSquare` 等中选取，总数不超过 5 个。

### 3.3 帖子类型（输出变体）

- **犀利短评 (sharp_short)**：2–4 句话，结论先行，一句数据支撑，结尾可留钩子或金句。
- **深度长文 (deep_long)**：多段，有 Hook、有数据段、有个人判断、有互动结尾。
- **互动提问 (interactive_question)**：用数据或现象引出一个问题，邀请评论区讨论，保持人设。

---

## 4. 预览与发布

### 4.1 预览

- 可一次生成多种类型草稿（如犀利短评、深度长文、互动提问），将每篇的 **正文 + Hashtags** 作为草稿发给用户预览。
- 用户选择其中一篇后，再调用发布流程（或人工复制到币安广场发布）。

### 4.2 直接发布

- 不经过预览，直接按指定类型（默认犀利短评）生成一篇草稿，将 `fullText = 正文 + \n\n + Hashtags` 交给调用方，由调用方通过币安广场 API 或人工发布。

---

## 5. 大模型调用约定

- **系统提示**：人设块 + 当前逻辑类型说明 + 当前帖子类型说明 + 禁止词列表 + 输出格式（只输出正文、不含 Hashtag）。
- **用户提示**：当前 market_context 关键信号摘要（社交热度前 5、聪明钱 1h 前三及流入/流出、Rising 叙事、高优先级素材）。
- 推荐引擎：MiniMax M2.5 或 GPT-4o；temperature 建议 0.7–0.9，max_tokens 建议 512–1024。

---

## 6. 输出产物

- **ContentDraft**（概念结构）：
  - variant：sharp_short / deep_long / interactive_question
  - logicType：alert / dig / neutral / fomo
  - hook、body、cta：从正文拆分
  - hashtags：字符串数组
  - fullText：正文 + 换行 + Hashtags，可直接用于发布
