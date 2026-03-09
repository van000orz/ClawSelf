# ClawSelf 使用教程

> 本仓库是一套**基于 OpenClaw 的币安广场数字分身**技能包。所有技能放在 `.cursor/skills/` 下，可直接交给你的 **OpenClaw**（或其它支持 Skill 的 Agent）使用，实现从「灵魂提取 → 热点感知 → 内容生成 → 自动发帖」的全链路。

---

## 一、ClawSelf 是什么

ClawSelf 把你在币安广场的发言风格和投资观，沉淀成一份 **persona（人设）**，再通过**感知层**抓市场热点、**内容层**按人设写稿、**执行层**按规则发帖，形成一条可自动运转的「数字分身」流水线。

| 层级 | 作用 | 产出 |
|------|------|------|
| **第一层 · 灵魂提取** | 100 问访谈，提炼你的投资观、黑话、情绪反应 | `persona.md`、`raw_interview_data.log` |
| **第二层 · 感知层** | 拉取社交热度、聪明钱流入、叙事话题 | `market_context.json` |
| **第三层 · 内容层** | 用人设 + 市场数据写贴文草稿 | 贴文正文（可发币安广场） |
| **第四层 · 执行层** | 按阈值与冷却/配额发帖，异常熔断 | 调用发帖 API |
| **总控** | 全自动调度：自检 → 每小时感知→决策→合成→执行 | 可选，需自行实现或接 OpenClaw |

---

## 二、技能在哪儿（目录结构）

所有给 OpenClaw 用的技能都在 **`.cursor/skills/`** 里：

```
.cursor/skills/
├── 使用教程.md              ← 你正在看的这份
├── clawselfskills.md        ← 总控编排器（全自动工作流说明）
├── clawself-builder/        ← 第一层：灵魂提取（100 问）
│   ├── clawself-builder.md  ← 技能正文
│   └── README.md
├── clawself-perception/     ← 第二层：感知层
│   ├── clawself-perception.md
│   └── README.md
├── clawself-content/        ← 第三层：内容层
│   ├── clawself-content.md
│   └── README.md
├── clawself-executor/       ← 第四层：执行层调度
│   ├── clawself-executor.md
│   └── README.md
├── clawselfskills/          ← 总控说明
│   └── README.md
└── Binance Skills/          ← 币安官方技能（感知层依赖）
    ├── crypto-market-rank/
    └── meme-rush/
```

- 给 OpenClaw 用：把对应 **`.md` 技能文件**（或整个 `skills` 目录）交给 OpenClaw，按「技能文档」里的步骤执行即可。
- 网页下载：仓库根目录的 **`web/`** 下有 `index.html`，打开后可**直接下载**各层 Skill 的 `.md` 或 **`skills.zip`** 整包。

---

## 三、给 OpenClaw 用的推荐顺序

### 1. 先做人设（第一层）

- **技能**：`clawself-builder/clawself-builder.md`
- **做法**：让 OpenClaw 按文档对你做 **100 问灵魂提取**，或你打开项目里的 **`web/interview.html`** 在浏览器里答完 100 题。
- **得到**：  
  - `persona.md`（人设，后续层都要用）  
  - `raw_interview_data.log`（原始问答，可追溯）

把 `persona.md` 放到 OpenClaw 能读到的位置（如工作区根目录或 `~/.openclaw/persona.md`）。

### 2. 再做感知（第二层）

- **技能**：`clawself-perception/clawself-perception.md`
- **依赖**：币安相关接口（见文档里的 crypto-market-rank、meme-rush 等）。
- **做法**：让 OpenClaw 按文档调接口，执行「查热度 → 查资金 → 查叙事 → 合成 JSON」。
- **得到**：`market_context.json`（社交热度 Top5、聪明钱 Top3、叙事话题、高优先级素材等）。

建议用 Cron 每 2 小时跑一次，保证数据新鲜。

### 3. 再写内容（第三层）

- **技能**：`clawself-content/clawself-content.md`
- **输入**：`persona.md` + `market_context.json`
- **做法**：让 OpenClaw 按文档把人设与市场数据合成贴文（犀利短评 / 深度长文 / 互动提问等）。
- **得到**：可直接用于发帖的正文（含 Hashtags）。

### 4. 最后执行发帖（第四层）

- **技能**：`clawself-executor/clawself-executor.md`
- **作用**：不是「有稿就发」，而是按**阈值**（如聪明钱流入 > 某值、热度 Top）与**冷却、配额、同币种限次**等规则，决定何时调用内容层、何时调发帖 API。
- **依赖**：内容层草稿 + 币安发帖接口（如 square-post）。文档里写了熔断与错误处理。

若要做**全自动一条龙**，再让 OpenClaw 按 **`clawselfskills.md`**（总控）做：自检 persona → 每小时 感知→决策→合成→执行，并处理熔断与启停。

---

## 四、用网页快速跑通第一层

不想先配 OpenClaw，也可以先用网页把人设跑通：

1. 用本地 HTTP 打开项目里的 `web/`（例如：`cd web && python -m http.server 8000`）。
2. 浏览器打开 `http://localhost:8000`，进首页。
3. 点击 **「开始灵魂访谈」**，在 `interview.html` 里答完 100 题。
4. 完成后在页面上下载 **`persona.md`** 和 **`raw_interview_data.log`**，放到你的工作区或 OpenClaw 配置路径。
5. 之后在 OpenClaw 里加载第二、三、四层技能时，都会用到这份 `persona.md`。

---

## 五、一句话对照表（给 OpenClaw 用）

| 你想让 OpenClaw 做… | 用这个技能 |
|---------------------|------------|
| 帮我做 100 问访谈并生成 persona | `clawself-builder/clawself-builder.md` |
| 拉取市场热度/聪明钱/叙事，产出 market_context | `clawself-perception/clawself-perception.md` |
| 根据 persona + market_context 写一篇贴文 | `clawself-content/clawself-content.md` |
| 按规则决定何时发帖、冷却与熔断 | `clawself-executor/clawself-executor.md` |
| 全自动：自检 → 每小时感知→写稿→发帖 | `clawselfskills.md`（总控，需按文档实现或对接） |

---

## 六、注意事项

- **persona 是根**：没有 `persona.md`，感知层的高优先级匹配、内容层的人设注入都做不好，建议先完成第一层。
- **币安接口**：感知层和发帖依赖币安相关 API，需自行配置网络与权限；执行层文档中有错误码与熔断说明。
- **技能即文档**：每个 `.md` 里都是「步骤 + 规则」，OpenClaw 按文档执行即可，无需改代码（除非你要自己实现总控或 UI）。

把 **`.cursor/skills/`** 拷给 OpenClaw 用，或从仓库 **`web/`** 下载单份 Skill、整包 **skills.zip** 均可；优先看各技能目录下的 **README.md** 和技能正文，再按本教程顺序跑即可。
