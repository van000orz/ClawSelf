# ClawSelf 总控编排器（clawselfskills）技能说明

## 是什么

ClawSelf **全自动工作流**总控入口。负责：初始化检测 `persona.md`（缺失则触发灵魂提取）、按 1 小时周期执行「感知 → 决策 → 合成 → 执行」循环、异常熔断与休眠、以及 `start()` / `stop()` / `getStatus()` 控制接口与黑金+龙虾红风格控制面板。

## 何时用

- 需要「启动分身」「全自动运营」「ClawSelf 入口」时。
- 希望一条链路自动完成：自检 → 定时感知 → 阈值决策 → 内容合成 → 发帖，并由 UI 控制启停与状态时。

## 技能文件

- **clawselfskills.md**（位于上级目录 `.cursor/skills/`）：完整编排逻辑、初始化与循环步骤、异常与日志、UI 接口、TypeScript 总控类示例与项目入口用法。
- **README.md**：本说明。

## 依赖

- 第一层：persona.md（clawself-builder 产出）。
- 第二层：market_context.json（clawself-perception 产出）。
- 第三层：内容草稿生成（clawself-content）。
- 第四层/发帖：币安 square-post 或等效发布接口。

## 核心流程

1. **Self-Check**：无 persona 则触发 builder 访谈。
2. **Loop**：每 1 小时 感知 → 决策（阈值）→ 合成（3 篇草稿）→ 执行（冷却/配额满足后发 1 篇）。
3. **熔断**：API 限流/风控/网络错误 → 休眠并终端提醒，需用户确认后恢复。
