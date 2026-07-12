# Claude Code 配置与技能集

为 Vibe Coding 打造的 Claude Code 设置、技能（Skills）和子代理（Sub-agents）合集，涵盖功能开发（规格驱动工作流）、代码分析、GitHub 集成和知识管理等增强型开发工作流。

> OpenAI Codex 的配置和自定义 prompt 请参考 [feiskyer/codex-settings](https://github.com/feiskyer/codex-settings)。

## 目录

- [安装](#安装)
- [技能列表](#技能列表)
- [子代理](#子代理)
- [配置模板](#配置模板)
- [脚本工具](#脚本工具)
- [已知限制](#已知限制)
- [常见问题](#常见问题)
- [参考指南](#参考指南)
- [参考资源](#参考资源)
- [许可证](#许可证)

## 安装

### 方式一：Claude Code Plugin（推荐）

```sh
/plugin marketplace add feiskyer/claude-code-settings

# 安装主插件（包含所有技能和代理）
/plugin install claude-code-settings

# 或者按需安装单个技能
/plugin install codex-skill               # Codex 自动化
/plugin install autonomous-skill          # 长时间任务自动化
/plugin install nanobanana-skill          # 图片生成
/plugin install youtube-transcribe-skill  # YouTube 字幕提取
```

**注意：** [~/.claude/settings.json](settings.json) 不通过 Plugin 配置，需要手动设置。

### 方式二：npx skills

`npx skills` 可以为你的 AI 编程工具安装技能：

```sh
# 列出可用技能
npx -y skills add -l feiskyer/claude-code-settings

# 安装全部技能
npx -y skills add --all feiskyer/claude-code-settings

# 手动选择要安装的技能
npx -y skills add feiskyer/claude-code-settings
```

### 方式三：手动安装

```sh
# 备份原有配置
mv ~/.claude ~/.claude.bak

# 克隆本仓库
git clone https://github.com/feiskyer/claude-code-settings.git ~/.claude

# 安装 LiteLLM 代理
pip install -U 'litellm[proxy]'

# 启动 litellm 代理（监听 http://0.0.0.0:4000）
litellm -c ~/.claude/guidances/litellm_config.yaml

# 也可以用 tmux 在后台运行
# tmux new-session -d -s copilot 'litellm -c ~/.claude/guidances/litellm_config.yaml'
```

启动后会看到：

```
Please visit https://github.com/login/device and enter code XXXX-XXXX to authenticate.
```

打开链接，登录并授权你的 GitHub Copilot 账户。

**注意：**

1. 默认配置使用 [LiteLLM Proxy Server](https://docs.litellm.ai/docs/simple_proxy) 作为 LLM 网关连接 GitHub Copilot。也可以使用 [copilot-api](https://github.com/ericc-ch/copilot-api) 作为代理（需将端口改为 4141）。
2. 确保以下模型在你的账户中可用，否则需替换为你自己的模型名：

   - ANTHROPIC_DEFAULT_SONNET_MODEL: claude-sonnet-4.6
   - ANTHROPIC_DEFAULT_OPUS_MODEL: claude-opus-4.6
   - ANTHROPIC_DEFAULT_HAIKU_MODEL: gpt-5-mini

## 技能列表

技能是[可复用的能力模块](https://docs.anthropic.com/en/docs/claude-code/skills)，教会 Claude 如何完成特定任务。可通过 `/skill-name [参数]` 手动调用，或根据上下文自动触发。

<details>
<summary><b>brainstorming</b> — 头脑风暴：从想法到设计</summary>

### [brainstorming](./skills/brainstorming)

在构建新功能、创建新组件或设计新系统之前使用。通过协作对话探索用户意图、需求和设计方案，再进入实现阶段。

**触发词：** 用户描述想要构建的东西且涉及设计决策

**核心特性：**

- 逐个提问，渐进式理解需求
- 提出 2-3 个方案并推荐最优解
- 分节呈现设计，每节获得用户认可
- 编写规格文档并自审
- 可选的浏览器可视化伴侣（展示 mockup、图表、布局对比）

</details>

<details>
<summary><b>codex-skill</b> — 将任务交给 Codex CLI</summary>

### [codex-skill](plugins/codex-skill)

非交互式自动化模式，使用 OpenAI Codex 完成 Claude 设计好的功能或计划。

**触发词：** "codex", "use gpt", "gpt-5", "let openai", "full-auto", "用codex", "让gpt实现"

**核心特性：**

- 多种执行模式（只读、工作区写入、完全访问）
- 模型选择（gpt-5, gpt-5.1, gpt-5.1-codex 等）
- 自主执行无需审批
- JSON 结构化输出
- 可恢复会话

**依赖：** Codex CLI（`npm i -g @openai/codex` 或 `brew install codex`）

</details>

<details>
<summary><b>autonomous-skill</b> — 长时间任务自动化</summary>

### [autonomous-skill](plugins/autonomous-skill)

跨多个会话执行复杂的长时间任务，采用双代理模式（Initializer + Executor），支持自动会话续接。

**核心特性：**

- 双代理模式（Initializer 创建任务列表，Executor 逐步执行）
- 跨会话自动续接与进度追踪
- 任务隔离，每个任务有独立目录（`.autonomous/<task-name>/`）
- 通过 `task_list.md` 和 `progress.md` 持久化进度
- 使用 Claude CLI headless 模式执行

**依赖：** Claude CLI

</details>

<details>
<summary><b>nanobanana-skill</b> — 使用 Gemini 生成图片</summary>

### [nanobanana-skill](plugins/nanobanana-skill)

通过 Google Gemini API 生成或编辑图片。

**触发词：** "nanobanana", "generate image", "create image", "edit image", "图片生成", "AI绘图"

**核心特性：**

- 多种宽高比和分辨率（1K、2K、4K）
- 图片编辑功能
- 多模型选择（gemini-3-pro-image-preview, gemini-2.5-flash-image）

**依赖：**

- `GEMINI_API_KEY`（配置在 `~/.nanobanana.env`）
- Python3 + google-genai, Pillow, python-dotenv

</details>

<details>
<summary><b>gpt-image-skill</b> — 使用 OpenAI GPT Image 生成图片</summary>

### [gpt-image-skill](./skills/gpt-image-skill)

使用 OpenAI GPT Image 模型（gpt-image-2, gpt-image-1 等）生成或编辑图片。

**触发词：** "gpt image", "openai image", "draw image", "AI绘图", "画图"

**核心特性：**

- 多模型支持（gpt-image-2, gpt-image-1.5, gpt-image-1, gpt-image-1-mini）
- 最多 3 张参考图编辑
- 多种尺寸和质量等级
- 透明背景、多输出格式（png, jpeg, webp）

**依赖：**

- `OPENAI_API_KEY`（配置在 `~/.gpt-image.env` 或环境变量）
- Python3 + openai, Pillow

</details>

<details>
<summary><b>youtube-transcribe-skill</b> — YouTube 字幕提取</summary>

### [youtube-transcribe-skill](plugins/youtube-transcribe-skill)

从 YouTube 视频链接提取字幕/转录文本。

**核心特性：**

- 双提取方式：CLI（快速）和浏览器自动化（兜底）
- 自动选择字幕语言（zh-Hans, zh-Hant, en）
- 保存转录文本到本地文件

**依赖：** `yt-dlp`（CLI 方式）或 `chrome-devtools-mcp`（浏览器方式）

</details>

<details>
<summary><b>deep-research</b> — 多 Agent 深度调研</summary>

### [deep-research](./skills/deep-research)

多 Agent 编排工作流：将调研目标拆分为可并行子目标，通过 `claude -p` 子进程执行，聚合结果并交付精修报告。

**触发词：** "深度调研", "deep research", "wide research", "多Agent并行调研"

**核心特性：**

- 多 Agent 编排：拆分目标为并行子任务
- Skills 优先策略：已安装技能 → MCP 工具 → WebFetch/WebSearch
- 结构化交付：生成文件级报告，含摘要和结论
- 逐章精修与来源验证
- 可扩展执行：从微型（1-2 任务）到大型（15+）

**产出目录结构：**

```
.research/<name>/
├── prompts/           # 子任务 prompt
├── child_outputs/     # 子进程输出
├── logs/              # 执行日志
├── raw/               # 原始数据缓存
└── final_report.md    # 最终报告
```

</details>

<details>
<summary><b>translate</b> — 技术文章中文翻译</summary>

### [translate](./skills/translate)

将英文或日文技术文章翻译为自然流畅的中文。采用三步法（直译、问题识别、意译），保留 Markdown 格式和技术术语原文。

**用法：** `/translate [粘贴文本或提供文件路径]`

</details>

<details>
<summary><b>github-fix-issue</b> — 端到端修复 GitHub Issue</summary>

### [github-fix-issue](./skills/github-fix-issue)

从分析到创建分支、实现、测试、提交 PR 的完整 Issue 修复流程。

</details>

<details>
<summary><b>github-review-pr</b> — 审查 GitHub Pull Request</summary>

### [github-review-pr](./skills/github-review-pr)

使用并行子代理进行多角度代码分析，支持置信度评分和误报过滤。

</details>

<details>
<summary><b>skill-creator</b> — 创建和评测 Agent 技能</summary>

### [skill-creator](./skills/skill-creator)

创建、迭代优化和基准测试 Agent 技能，包含定量评估循环和描述优化。

</details>

## 子代理

`agents/` 目录包含扩展 Claude Code 能力的专业化[子代理](https://docs.anthropic.com/en/docs/claude-code/sub-agents)。

| 代理 | 功能 |
|------|------|
| **pr-reviewer** | GitHub PR 代码审查专家 |
| **github-issue-fixer** | GitHub Issue 修复专家 |
| **instruction-reflector** | 分析和改进 Claude Code 指令 |
| **deep-reflector** | 会话分析和学习捕获 |
| **insight-documenter** | 技术突破文档化 |
| **ui-engineer** | UI/UX 开发专家 |

## 配置模板

[配置模板目录](settings/README.md) — 为各种模型提供商预配置的 settings.json。

| 配置文件 | 说明 |
|----------|------|
| [copilot-settings.json](settings/copilot-settings.json) | GitHub Copilot 代理 |
| [litellm-settings.json](settings/litellm-settings.json) | LiteLLM 网关 |
| [deepseek-settings.json](settings/deepseek-settings.json) | DeepSeek v3.1 |
| [qwen-settings.json](settings/qwen-settings.json) | Qwen3-Coder-Plus（阿里云 DashScope） |
| [siliconflow-settings.json](settings/siliconflow-settings.json) | SiliconFlow（Kimi-K2-Instruct） |
| [vertex-settings.json](settings/vertex-settings.json) | Google Cloud Vertex AI |
| [azure-settings.json](settings/azure-settings.json) | Azure AI（Anthropic 兼容端点） |
| [azure-foundry-settings.json](settings/azure-foundry-settings.json) | Azure AI Foundry 原生模式 |
| [minimax.json](settings/minimax.json) | MiniMax-M2 |
| [openrouter-settings.json](settings/openrouter-settings.json) | OpenRouter（多模型统一 API） |

## 脚本工具

[`scripts/`](scripts/) 目录包含维护 Claude Code 环境的实用脚本。

| 脚本 | 功能 |
|------|------|
| [`update-cc-plugins.sh`](scripts/update-cc-plugins.sh) | 一键更新所有已安装的 marketplace 和插件/技能 |

```sh
bash ~/.claude/scripts/update-cc-plugins.sh
```

## 已知限制

**WebSearch** 是 [Anthropic 专有工具](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/web-search-tool)，在非官方 Anthropic API 环境下不可用。如需网页搜索，请接入外部 MCP 服务器：

- [Tavily MCP](https://docs.tavily.com/documentation/mcp)
- [Brave MCP](https://github.com/brave/brave-search-mcp-server)
- [Firecrawl MCP](https://docs.firecrawl.dev/mcp-server)
- [DuckDuckGo Search MCP](https://github.com/nickclyde/duckduckgo-mcp-server)

## 常见问题

<details>
<summary>VSCode 中 Claude Code 2.0+ 扩展的登录问题</summary>

如果不使用 Claude.ai 订阅，需在 VSCode settings.json 中手动配置环境变量：

```json
{
  "claude-code.environmentVariables": [
    { "name": "ANTHROPIC_BASE_URL", "value": "http://localhost:4000" },
    { "name": "ANTHROPIC_AUTH_TOKEN", "value": "sk-dummy" },
    { "name": "ANTHROPIC_MODEL", "value": "opusplan" },
    { "name": "ANTHROPIC_DEFAULT_SONNET_MODEL", "value": "claude-sonnet-4.6" },
    { "name": "ANTHROPIC_DEFAULT_OPUS_MODEL", "value": "claude-opus-4.6" },
    { "name": "ANTHROPIC_DEFAULT_HAIKU_MODEL", "value": "gpt-5-mini" },
    { "name": "DISABLE_TELEMETRY", "value": "1" }
  ]
}
```

同时需要 [~/.claude/config.json](config.json) 的内容来跳过 claude.ai 登录。

</details>

<details>
<summary>API Key 缺失或无效</summary>

确保 `ANTHROPIC_AUTH_TOKEN` 中配置的 API key 已添加到 `~/.claude.json` 的已批准列表：

```json
{
  "customApiKeyResponses": {
    "approved": ["sk-dummy"],
    "rejected": []
  }
}
```

</details>

## 参考指南

- [使用 GitHub Copilot 作为模型提供商](guidances/github-copilot.md)
- [使用 LLM 网关（LiteLLM）作为模型提供商](guidances/llm-gateway-litellm.md)

## 参考资源

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/overview) — 必读
- [anthropics/skills](https://github.com/anthropics/skills) — 官方技能列表
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) — Anthropic 官方插件列表
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) — 社区精选资源
- [wshobson/agents](https://github.com/wshobson/agents) — Claude Code 专业子代理合集

## 许可证

本项目基于 MIT 许可证发布 — 详见 [LICENSE](LICENSE)。
