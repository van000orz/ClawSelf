---
name: clawselfskills
description: |
  ClawSelf 总控编排器：全自动工作流入口。初始化检测 persona.md、循环执行感知→决策→合成→执行，含异常熔断与 UI 控制面板（start/stop/getStatus）。适用于「启动分身」「全自动运营」「ClawSelf 入口」等指令。
metadata:
  version: "1.0"
---

# ClawSelf 总控编排器（clawselfskills）

基于 OpenClaw 的币安广场自动化分身**全自动工作流**总控。负责初始化检测、定时感知、决策触发、内容合成与执行发帖的整条链路，并提供 `start()` / `stop()` / `getStatus()` 控制接口与黑金+龙虾红风格控制面板。

---

## 1. 初始化检测（Self-Check）

- **检查本地是否存在 `persona.md`**  
  - 路径：`~/.openclaw/persona.md` 或项目根目录 `persona.md`。  
- **若不存在**：  
  - 触发 **clawself-builder**（灵魂提取层）技能，启动访谈流程。  
  - 等待用户完成访谈并生成 `persona.md` 后，再进入循环调度。  
- **若存在**：  
  - 直接进入「循环调度逻辑」，不重复访谈。

---

## 2. 循环调度逻辑（The Loop）

### 2.1 感知步（Perception）

- **周期**：每隔 **1 小时** 自动执行一次。  
- **动作**：调用 **clawself-perception** 技能（内部使用 crypto-market-rank、meme-rush 等 API）。  
- **产出**：更新 `market_context.json`（含 `socialHypeTop5`、`smartMoneyTop3`、`topicRushRising`、`highPriority`）。  
- **日志**：记录「感知步完成时间、数据摘要」。

### 2.2 决策步（Decision）

- **输入**：当前 `market_context.json`。  
- **规则示例**：  
  - Smart Money 某币种净流入 > 设定阈值（如 50 万 USD）；或  
  - 社交热度 Top 3 出现新人/新叙事；或  
  - `highPriority` 非空（与人设匹配的素材）。  
- **输出**：布尔值「是否触发本周期发帖」；若触发，可附带本次主推 symbol/话题。  
- **与执行层一致**：决策逻辑与 **clawself-executor** 中的阈值、冷却、配额、多样性保持一致，避免重复发帖或超限。

### 2.3 合成步（Synthesize）

- **触发条件**：决策步输出「触发」。  
- **动作**：将当前 `persona.md` + `market_context.json` + 本次选定模板类型，交给 **ContentSynthesizer**（第三层 clawself-content）。  
- **产出**：根据当前情绪/热度生成 **3 篇推文草稿**（可覆盖犀利短评、深度长文、互动提问三种类型，或按轮换策略各一）。  
- **日志**：记录「合成步完成时间、草稿条数」。

### 2.4 执行步（Execution）

- **前置条件**：  
  - 执行准则满足：冷却时间 **45 分钟**（模拟真人作息）、当日配额未满、同币种 4h 限次等（见 clawself-executor）。  
- **动作**：从 3 篇草稿中选 1 篇（或按策略），调用币安官方发布 Skill（square-post）发送内容。  
- **若接口报错或风控**：进入异常熔断，停止自动化并在终端提醒用户（见第 3 节）。  
- **日志**：记录「执行步完成时间、发布结果/错误码」。

---

## 3. 异常处理与日志

- **链路日志**：  
  - 每一次「感知 → 决策 → 合成 → 执行」的完整链路写入日志（时间戳、步骤名、摘要、错误码（若有））。  
  - 建议存储为 `clawself_orchestrator.log` 或通过 OpenClaw 日志接口输出。  
- **休眠模式**：  
  - 当遇到 **API 额度限制**、**网络错误**、**风控/封禁类错误码** 时：  
    - 自动进入「休眠模式」：停止本轮及后续自动发帖与合成触发。  
    - 在**终端**（或 OpenClaw 提供的通知通道）提醒用户：「ClawSelf 已进入休眠，请检查 API/网络/账号后手动恢复」。  
  - 恢复方式：用户显式调用 `start()` 或「恢复运行」指令后，重新自检并继续循环。

---

## 4. UI 控制面板接口

- **start()**：  
  - 启动总控：执行初始化检测（无 persona 则先走 builder），通过后启动 1 小时周期的循环调度。  
- **stop()**：  
  - 停止总控：暂停定时器与调度，不再执行感知/决策/合成/执行，直至再次 `start()`。  
- **getStatus()**：  
  - 实时返回当前状态字符串，供 UI 展示，例如：  
    - `"正在监听市场..."`（感知步中）  
    - `"正在撰写推文..."`（合成步中）  
    - `"冷却中"`（处于 45 分钟冷却）  
    - `"休眠中（已暂停，请检查 API/网络）"`  
    - `"等待 persona（请先完成灵魂访谈）"`  
    - `"运行中"`（空闲等待下一周期）

**UI 风格**：  
- 主色：币安黑金 `#FCD535`、强调/危险：龙虾红 `#E24A4A`。  
- 控制面板至少包含：启动 / 停止按钮、当前状态展示（调用 `getStatus()`）、最近一条链路日志摘要。

---

## 5. 技术栈与实现要点

- **语言/框架**：TypeScript + Next.js（或纯 Node 脚本，由项目选择）。  
- **与 OpenClaw 集成**：  
  - 通过 OpenClaw 的 `exec` 或 `skills` 接口调用各层技能（clawself-builder、clawself-perception、clawself-content、square-post）。  
  - 具体调用方式以 OpenClaw 文档为准（如 `skills.run('clawself-perception')` 或等价 API）。  
- **定时**：  
  - 使用 `setInterval`（1 小时）或 Cron 表达式驱动「感知步」，其余步骤在同一次调度内顺序执行。  
- **状态持久化**：  
  - 与 clawself-executor 共用或复用同一份状态（上次发帖时间、当日计数、熔断标志、4h 内各 symbol 计数），避免冲突。

---

## 6. 代码实现（总控核心）

以下为总控编排器的 TypeScript 核心逻辑，可作为项目入口模块实现参考。

```typescript
// ========== 类型与配置 ==========
const CONFIG = {
  PERCEPTION_INTERVAL_MS: 60 * 60 * 1000,  // 1 小时
  COOL_DOWN_MINUTES: 45,
  PERSONA_PATH: process.env.PERSONA_PATH || 'persona.md',
};

type Status =
  | 'idle'           // 运行中，等待下一周期
  | 'checking_persona'
  | 'waiting_persona' // 等待用户完成访谈
  | 'perceiving'      // 正在监听市场
  | 'deciding'
  | 'synthesizing'    // 正在撰写推文
  | 'executing'
  | 'cooldown'        // 冷却中
  | 'sleeping';      // 休眠（异常暂停）

interface OrchestratorState {
  status: Status;
  lastPerceptionAt: number | null;
  lastPostAt: number | null;
  sleeping: boolean;
  log: Array<{ ts: number; step: string; msg: string }>;
}

// ========== 总控类 ==========
class ClawSelfOrchestrator {
  private state: OrchestratorState = {
    status: 'idle',
    lastPerceptionAt: null,
    lastPostAt: null,
    sleeping: false,
    log: [],
  };
  private timerId: ReturnType<typeof setInterval> | null = null;

  private log(step: string, msg: string) {
    const entry = { ts: Date.now(), step, msg };
    this.state.log.push(entry);
    // 可同时写入 clawself_orchestrator.log 或 OpenClaw 日志接口
    console.log(`[ClawSelf] ${step}: ${msg}`);
  }

  private async checkPersona(): Promise<boolean> {
    this.state.status = 'checking_persona';
    // 检查 persona.md 是否存在（~/.openclaw/persona.md 或 根目录 persona.md）
    const fs = await import('fs/promises');
    const path = await import('path');
    const possiblePaths = [
      process.env.HOME ? path.join(process.env.HOME, '.openclaw', 'persona.md') : '',
      path.join(process.cwd(), 'persona.md'),
    ].filter(Boolean);
    for (const p of possiblePaths) {
      try {
        await fs.access(p);
        this.log('self-check', `persona found: ${p}`);
        return true;
      } catch {
        continue;
      }
    }
    this.log('self-check', 'persona not found, trigger clawself-builder');
    return false;
  }

  private async runPerception() {
    this.state.status = 'perceiving';
    this.log('perception', 'calling clawself-perception');
    try {
      // 调用 OpenClaw skills: clawself-perception → 更新 market_context.json
      // await openclaw.skills.run('clawself-perception');
      this.state.lastPerceptionAt = Date.now();
      this.log('perception', 'perception done');
    } catch (e: any) {
      this.log('perception', `error: ${e?.message || e}`);
      this.enterSleep('API/网络异常，已进入休眠');
      return;
    }
  }

  private async runDecision(): Promise<boolean> {
    this.state.status = 'deciding';
    // 读取 market_context.json，按 clawself-executor 阈值判断是否触发
    // 若 冷却中 / 配额满 / 多样性不通过 → return false
    const now = Date.now();
    if (this.state.lastPostAt && (now - this.state.lastPostAt) < CONFIG.COOL_DOWN_MINUTES * 60 * 1000) {
      this.state.status = 'cooldown';
      this.log('decision', 'in cooldown, skip');
      return false;
    }
    // TODO: 读 market_context.json，判断 inflow / hype / highPriority
    const triggered = true; // 示例
    this.log('decision', triggered ? 'triggered' : 'not triggered');
    return triggered;
  }

  private async runSynthesize() {
    this.state.status = 'synthesizing';
    this.log('synthesize', 'calling ContentSynthesizer, 3 drafts');
    try {
      // 调用 clawself-content，生成 3 篇草稿（或按轮换 1 篇）
      // await openclaw.skills.run('clawself-content', { count: 3, ... });
      this.log('synthesize', 'synthesize done');
    } catch (e: any) {
      this.log('synthesize', `error: ${e?.message || e}`);
    }
  }

  private async runExecution() {
    this.state.status = 'executing';
    this.log('execution', 'calling square-post');
    try {
      // 从草稿选 1 篇，调用 square-post 发布
      // await openclaw.skills.run('square-post', { bodyTextOnly: draft });
      this.state.lastPostAt = Date.now();
      this.log('execution', 'post success');
    } catch (e: any) {
      this.log('execution', `error: ${e?.message || e}`);
      this.enterSleep('发布接口异常或风控，已进入休眠');
    }
    this.state.status = 'cooldown';
  }

  private enterSleep(reason: string) {
    this.state.sleeping = true;
    this.state.status = 'sleeping';
    this.log('meltdown', reason);
    console.warn('[ClawSelf] 已进入休眠，请检查 API/网络/账号后手动 start() 恢复。');
  }

  private async tick() {
    if (this.state.sleeping) return;
    await this.runPerception();
    if (this.state.sleeping) return;
    const triggered = await this.runDecision();
    if (!triggered) {
      this.state.status = 'idle';
      return;
    }
    await this.runSynthesize();
    if (this.state.sleeping) return;
    await this.runExecution();
    this.state.status = 'idle';
  }

  // ========== 对外接口 ==========
  async start() {
    if (this.timerId) return;
    const hasPersona = await this.checkPersona();
    if (!hasPersona) {
      this.state.status = 'waiting_persona';
      this.log('start', 'trigger clawself-builder, waiting for persona');
      // 触发 clawself-builder 访谈；实际实现中可打开访谈页或调用 builder skill
      return;
    }
    this.state.sleeping = false;
    this.state.status = 'idle';
    this.timerId = setInterval(() => this.tick(), CONFIG.PERCEPTION_INTERVAL_MS);
    this.log('start', 'orchestrator started, interval 1h');
    await this.tick(); // 立即执行一次
  }

  stop() {
    if (this.timerId) {
      clearInterval(this.timerId);
      this.timerId = null;
    }
    this.state.status = 'idle';
    this.log('stop', 'orchestrator stopped');
  }

  getStatus(): string {
    const statusToText: Record<Status, string> = {
      idle: '运行中',
      checking_persona: '正在检查 persona...',
      waiting_persona: '等待 persona（请先完成灵魂访谈）',
      perceiving: '正在监听市场...',
      deciding: '决策中',
      synthesizing: '正在撰写推文...',
      executing: '正在发布...',
      cooldown: '冷却中',
      sleeping: '休眠中（已暂停，请检查 API/网络）',
    };
    return statusToText[this.state.status] ?? this.state.status;
  }

  getLastLog(count = 5) {
    return this.state.log.slice(-count);
  }
}

export const orchestrator = new ClawSelfOrchestrator();
export { orchestrator as default };
```

---

## 7. 作为项目入口的用法

- **方式 A：Next.js API Route**  
  - 在 `pages/api/clawself/control.ts`（或 App Router 的 `app/api/clawself/control/route.ts`）中：  
    - `POST body: { action: 'start' | 'stop' }` 调用 `orchestrator.start()` / `orchestrator.stop()`。  
    - `GET` 返回 `getStatus()` 与最近日志 `getLastLog()`，供控制面板前端轮询。  

- **方式 B：控制面板页面**  
  - 在 `pages/clawself.tsx`（或 `app/clawself/page.tsx`）中：  
    - 使用主色 `#FCD535`、强调色 `#E24A4A`。  
    - 按钮「启动」→ 请求 `POST /api/clawself/control` with `action: 'start'`。  
    - 按钮「停止」→ `action: 'stop'`。  
    - 状态区域定时请求 `GET /api/clawself/control` 显示 `getStatus()` 与最近链路日志。  

- **方式 C：纯 Node 入口**  
  - 新建 `scripts/run-clawself.ts`，在进程内调用 `orchestrator.start()`，通过信号量或 HTTP 暴露 `stop` / `getStatus`（如简单 HTTP 服务或 OpenClaw 插件），供外部控制。  

将 **clawselfskills.md** 放在 `.cursor/skills/` 下，即作为 ClawSelf 项目的**总控编排器技能文档**；实际运行入口为上述 TypeScript 模块（或等价实现）并挂载到 Next.js API / 控制面板或 OpenClaw 的 exec/skills 调用链中。
