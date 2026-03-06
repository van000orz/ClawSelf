## ClawSelf 前端（Streamlit）

这个目录下的 `app.py` 是一个基于 Streamlit 的前端页面，用于：

- 下载人格构建 Skill：`SKILL_ClawSelf_builder.md`
- 通过 31 个问题的访谈流程，收集用户回答
- 在本地生成：
  - `raw_interview_data.log`：原始问答日志
  - `persona.md`：结构化数字人格画像（基于 `.cursor/skills/clawself-builder/persona.template.md`）

### 启动方式

```bash
pip install -r requirements.txt
streamlit run app.py
```

## ClawSelf 前端（纯 HTML 网站）

在 `web/` 目录下提供了一个 **纯 HTML/CSS/JS** 的前端网站版本（无后端）：

- 用户在网页回答 31 个问题
- 网页在浏览器本地生成并下载：
  - `persona.md`
  - `raw_interview_data.log`
- 支持下载 `SKILL_ClawSelf_builder.md`（需要通过本地 http server 打开网页）

### 启动方式（推荐）

```bash
cd web
python -m http.server 8000
```

然后打开 `http://localhost:8000`。

### 说明

- 目前 `persona.md` 为**模板 + 原始问答附录**的版本，已经可以直接作为其他 Agent 的 persona 底座。
- 若要接入 MiniMax / OpenClaw 等大模型服务，可以在 `app.py` 的 `build_persona_md` 中替换为真正的 API 调用逻辑。 

