# Antigravity 全景指南：App、CLI、IDE、SDK 与自定义实践

> 🧭 **其他指南**：[Codex 全景指南](README.md) · [Claude Code 全景指南](Claude%20Code%20%E5%85%A8%E6%99%AF%E6%8C%87%E5%8D%97.md)

> 面向 Google Antigravity 新手与开发者：从入口选型、登录与模型，到权限与沙箱、Rules、Skills、MCP、Hooks、Plugins、多智能体、`agy` 命令行、IDE 扩展和 Python SDK，提供属性说明、操作流程与示例。
>
> 内容依据 Antigravity 官方英文文档整理并翻译，产品名、命令、设置键保持英文原样。文档里没有写明的内容本指南不做推断，并在对应位置标注“文档未写明”。
>
> 最后核对：2026-11。Antigravity 迭代很快，版本号、模型名和套餐细节请以当前客户端和官方页面为准。

## 阅读导航

| 我想…… | 去这里 |
|---|---|
| 搞清 App / CLI / IDE / 扩展 / SDK 该用哪个 | Part I →“Antigravity 是什么与入口选型” |
| 安装、登录、选账号类型 | Part I →“安装与第一次使用”“登录与认证” |
| 选模型、调推理强度、看配额与积分 | Part I →“模型与推理强度”“套餐、配额与积分” |
| 日常使用 App：@ 提及、语音、浏览器、项目 | Part II（App 核心） |
| 权限、沙箱、产物审阅、远程运行 | Part II（权限、沙箱、产物与远程） |
| 写 Rules / Skills、接 MCP、装插件、配 Hooks | Part III（扩展机制） |
| 并行多个智能体、定时与自动化任务 | Part III（多智能体与自动化） |
| 在终端里用 `agy`、脚本和 CI 里无人值守运行 | Part IV：CLI |
| 用 Python 写自己的智能体，或在各类编辑器里使用 | Part V：SDK 与 IDE |
| 从 Firebase Studio、Gemini CLI、workflows 迁移 | Part I 与 Part IV 的迁移小节 |
| 配置不生效 | 文末“常见问题” |

## Antigravity 全景图

```text
Google Antigravity
│
├─ 入口（同一套智能体能力，不同界面）
│  ├─ Antigravity App    桌面“指挥中心”，管理并行与后台智能体
│  ├─ Antigravity CLI    终端客户端，命令 agy
│  ├─ Antigravity IDE    带智能体与产物审阅的编辑器
│  ├─ IDE 扩展           VS Code / JetBrains / Visual Studio / Zed / Xcode 等
│  └─ Antigravity SDK    Python SDK，编程式构建智能体
│
├─ 账号与计费
│  └─ 个人套餐 / Workspace / Pay As You Go / Enterprise
│
├─ 安全边界
│  └─ 权限规则（Deny / Ask / Allow）+ 终端沙箱 + 浏览器白/黑名单
│
├─ 上下文与交付
│  ├─ Rules / Skills / Customization discovery
│  └─ Artifacts：Plan、Walkthrough、Screenshots、Review changes
│
├─ 扩展
│  ├─ MCP / MCP Apps / Plugins / Marketplace
│  └─ Hooks / Sidecars
│
├─ 多智能体与自动化
│  └─ subagents / Swarm / Teamwork / Dynamic Workflows / /schedule / /automation
│
└─ 远程
   └─ SSH / Remote Control / Remote Agents
```

## 小白先走这条路线

```text
选入口：日常交互用 App，终端党用 CLI，已有编辑器用 IDE 扩展
      │
      ▼
安装并登录（先确认账号类型：个人账号最省事）
      │
      ▼
在一个项目里先问一个只读问题，熟悉界面与权限提示
      │
      ▼
了解权限预设与终端沙箱，再决定是否放宽
      │
      ▼
写第一条 Rule（项目约定）和第一个 Skill（固定流程）
      │
      ▼
需要外部系统再接 MCP；任务成熟后再用 subagent / swarm / 定时任务
```

完成后你应该看到：智能体按项目 Rules 工作；需要批准的操作会停下来问你；改动可以通过 Review changes 审阅；任务产出以 Artifact 形式可评论、可追溯。

## Antigravity ↔ Claude Code 对照表

如果你读过本仓库的《Claude Code 全景指南》，可以用下表对应概念。两者设计并不相同，“对应”只表示解决的问题相近，路径和字段不能互相照抄。

| 需求 | Claude Code | Antigravity |
|---|---|---|
| 命令行入口 | `claude` | `agy` |
| 非交互运行 | `claude -p` | `agy -p`（headless 模式） |
| 项目长期规则 | `CLAUDE.md` | `.agents/rules/*.md`（带 `trigger` frontmatter） |
| 可复用工作流 | Skills（`.claude/skills`） | Skills（`.agents/skills`） |
| MCP 配置 | `.mcp.json` / `claude mcp add` | `mcp_config.json`（全局在 `~/.gemini/config/`，工作区在 `.agents/`） |
| 钩子 | `settings.json` 的 `hooks` | 独立的 `hooks.json`（不写进 `settings.json`） |
| 子代理 | `.claude/agents/*.md` | 自定义 subagents（Markdown + frontmatter） |
| 并行 / 团队 | agent view、agent teams、workflows | `/swarm`、`/teamwork`、`/run-workflow` |
| 计划模式 | plan 权限模式 | `/plan` |
| 权限 | allow / ask / deny 规则 + 权限模式 | Deny / Ask / Allow 规则 + 权限预设 |
| 命令隔离 | 沙箱化 Bash | 终端沙箱 |
| 分发单元 | 插件 + marketplace | Plugins + Marketplace |
| 程序化 | Agent SDK（TS / Python） | Antigravity SDK（Python） |
| 接入其他编辑器 | IDE 扩展 | IDE 扩展 + ACP（Agent Client Protocol） |

---

## Part I：入门、账号与模型

### Antigravity 是什么与入口选型

Google Antigravity 是一个智能体（agent）编码平台。它从早先的 Agent Manager 演进而来，现在有五类入口，共用同一套 agent harness（与 Gemini 模型联合训练），既能做同步的结对编程，也能跑异步后台智能体。

#### 入口一览

| 入口 | 是什么 | 适合谁 | 版本 / 平台 |
|---|---|---|---|
| Antigravity App | 独立桌面应用，用于启动、编排和检查智能体团队，带可视化 diff 与计划检查 | 想同时管多个项目、多个会话的人 | 概览页标 `v2.5+`；macOS 13+、Windows 10/11、Linux、Googlebook |
| Antigravity CLI（`agy`） | 键盘优先的终端客户端，可在本地 shell、远程云环境和 SSH 中运行 | 习惯终端、要 SSH / 脚本 / CI 的人 | 概览页标 `v1.2+`；macOS / Linux / Googlebook / Windows |
| IDE 扩展 | 把智能体嵌进编辑器，侧边栏对话、行内 diff | 不想离开现有编辑器的人 | VS Code、JetBrains、Visual Studio、Zed、Xcode、Eclipse；Android Studio 企业支持为 Preview |
| Antigravity IDE | 独立的编辑器（非扩展） | 想要一体化编辑器的个人用户 | macOS 12+、Windows 10（64 位）+、Linux / Googlebook；**不支持企业部署** |
| Antigravity SDK | Python 包 `google-antigravity`，用代码构建和部署自己的智能体 | 要把智能体嵌进自己应用或 CI/CD 的开发者 | Python 3.10+ |

> 说明：概览页的入口卡片写“6 IDEs”，安装页实际列了 7 个编辑器（含 Android Studio 和独立的 Antigravity IDE）；以安装页列表为准。

#### 决策树

```text
你想怎么用 Antigravity？
│
├─ 要在自己的程序 / CI 里嵌入智能体 ──────────────▶ SDK（Python 3.10+）
│
├─ 主要在终端、SSH 或脚本里工作
│     └─ agy（交互 TUI；agy -p 做一次性非交互运行）
│
├─ 不想离开现有编辑器
│     ├─ VS Code / JetBrains / VS / Zed / Xcode / Eclipse ─▶ IDE 扩展
│     └─ 要企业账号（Gemini Enterprise）？ ─▶ 用扩展，不要用独立 Antigravity IDE
│
└─ 想要独立的“指挥中心”：多项目、多会话、子智能体、计划与 diff 审查
      └─ Antigravity App
```

### 安装与第一次使用

#### 安装各入口

| 入口 | 安装方式 |
|---|---|
| App（macOS） | 下载 `Antigravity.dmg`，拖入 Applications；升级时选 **Replace** 而不是 **Keep Both**。要求 macOS 13（Ventura）或更高，Apple Silicon 或 64 位 Intel |
| App（Windows / Linux） | Windows 10（64 位）或 11，运行 `.exe` 向导；Linux 解压 `Antigravity.tar.gz` 后运行 `./Antigravity`，需 glibc >= 2.28、glibcxx >= 3.4.25（Ubuntu 20+、Debian 10+、Fedora 36+、RHEL 8+）。Googlebook 预装，经 Google Play 更新 |
| CLI | 见下方脚本；默认二进制 `~/.local/bin/agy`（Windows 为 `C:\Users\<username>\AppData\Local\agy\bin`） |
| SDK | `pip install google-antigravity`（Poetry：`poetry add google-antigravity`；uv：`uv add google-antigravity`） |

CLI 用官方提供的安装脚本安装（macOS / Linux / Googlebook 用 shell 脚本，Windows 分 PowerShell 与 CMD 两种脚本；脚本地址见官方 CLI 下载页），装完用 `agy --version` 验证。

安装脚本可追加两个标志：`--skip-aliases`（不清理或更新旧的 `agy` / `antigravity` shell 别名）、`--skip-path`（不修改 shell 配置里的 `PATH`）。

IDE 扩展的版本要求：

| 编辑器 | 要求 | 安装位置 |
|---|---|---|
| VS Code | 1.90 或更高，macOS / Linux / Windows | 扩展侧栏搜索 **Google Antigravity** |
| JetBrains | IntelliJ 系 IDE（IDEA、PyCharm、WebStorm、GoLand、CLion、Rider）2026.2.1 或更高 | **JetBrains AI** > **Settings** > **Agents**，搜索 Antigravity |
| Android Studio | 需支持 Bring Your Own Agent（ACP）的版本 | **Agent** 工具窗口的 agent 选择器，或 **Settings** > **Tools** > **AI** > **Agents** |
| Visual Studio | Visual Studio 2026（17.8 或更高），Windows 10/11 | **Extensions** > **Manage Extensions**，装完需重启 |
| Zed | 0.140.0 或更高，macOS / Linux | 命令面板运行 `agent: open settings`，**External Agents** > **Add** > **Install from Registry** |
| Xcode | Xcode 27 beta 6 或更高，macOS | **Settings** > **Intelligence**，点 Antigravity 旁的 **Get** |
| Eclipse | Eclipse 2026-06 或更高 | **Help** > **Eclipse Marketplace…**，装完重启 |

> 边界提醒：Xcode 集成要求测试版 IDE（Xcode 27 beta 6+）。

#### 操作流程：App

1. 安装并启动，点 **Sign in** 在浏览器完成登录（账号类型见下一节）。
2. 个人 Google 账号首次登录（`v2.30.0+`）会问“你的工作类型”：**Software Engineering**、**Knowledge Work**、**Data & Analytics**、**Science**、**Design**、**Other**；可点 **Skip**，默认是 Software Engineering。该选择决定初始界面布局（Interface Layout），并确认 **Terms & Data Use** 后在 Home 显示最多三条可关闭的起始提示。
3. 左侧栏点 **New Project**（文件夹加号图标）> **Add Folder** 选择一个或多个本地 Git 仓库或目录 > **Create**。
4. 输入指令（例如 `Add unit tests for the authentication handlers`）回车，在弹窗里选 **Mode**：**Local mode**（直接在当前文件夹工作）或 **New worktree mode**（在隔离的 Git worktree 中工作）。
5. Knowledge Work 布局下不提示 worktree，也可不建项目、直接在 **Home** 聊天；选 Software Engineering 时还可导入 Claude Code、Codex、Cursor、Gemini CLI 的 MCP、skills、rules 和指令。

#### 操作流程：CLI

```bash
mkdir agy-demo && cd agy-demo
agy --version      # 验证安装
agy                # 启动 TUI
```

1. 首次启动要选配色（Dark、Solarized Dark、Solarized Light 或终端默认）、渲染模式（**Alt-Screen** 全屏缓冲，或 **Inline** 顺序输出）、并确认信任当前仓库目录（工作区信任，之后才开始索引文件）。
2. 输入第一个任务，例如 `Write a simple python script to fetch web page text`。智能体会创建 `main.py`，提示时按 Y 批准（在 `auto` 模式下，工作区编辑和沙箱内命令自动执行）。
3. 随时用 `/diff` 打开交互式 diff 查看器；收到通知时按 Ctrl + R 查看计划等 artifacts（Esc 返回输入框）。
4. 在输入框按 Shift + Tab 切换执行模式：`auto`、`plan`、`default`、`accept-edits`。
5. 让它运行脚本验证输出；结束时 Ctrl + D 或 `/exit`。

#### 操作流程：IDE 扩展与 SDK

IDE：打开工作区，打开 Antigravity 面板（VS Code 点活动栏图标；Visual Studio 按 Ctrl + \, Ctrl + Alt + L；Eclipse 选 **Antigravity** > **Toggle Chat**；JetBrains / Zed / Xcode 在各自 Agent 入口选 Antigravity），输入任务。独立 Antigravity IDE 中，Cmd + L（Ctrl + L）跳到右侧 **Agent** 面板并把选中代码加入上下文。

SDK 最小示例，含默认全拒绝、仅放行读文件的声明式策略（`policies` 支持 `allow`、`deny`、`ask_user`）：

```python
import asyncio
from google.antigravity import Agent, LocalAgentConfig
from google.antigravity.hooks.policy import allow, deny

async def main():
    config = LocalAgentConfig(policies=[deny("*"), allow("view_file")])
    async with Agent(config) as agent:
        response = await agent.chat("What files are in the current directory?")
        print(await response.text())

asyncio.run(main())
```

#### 常用命令与快捷键

| 入口 | 命令 / 快捷键 | 作用 |
|---|---|---|
| CLI | `agy` / `agy -i "prompt"` / `agy -p "prompt"` | 启动 TUI / 带初始提示启动 / 非交互运行一次并打印 |
| CLI | `/agents`、`/statusline`、`/usage`、`/exit` | 切换后台子智能体 / 状态栏 / 查看额度与积分消耗 / 退出 |
| App | `/goal`、`/grill-me`、`/schedule`、`/browser`、`/plugin` | 自主跑到完成 / 先被访谈需求 / 定时或 cron 任务 / 浏览器子智能体 / 管理插件 |

完成后你应该看到：CLI 里 `agy --version` 打印版本号；首次任务后工作区出现新文件，并能用 `/diff` 看到改动；App 的项目列表里出现该项目和会话。

### 登录与认证

Antigravity 在 App、CLI、IDE 扩展和 SDK 之间使用统一的认证层。

#### 账号类型选型

| 你是谁 | 怎么登录 | 计费 / 条款 |
|---|---|---|
| 个人 Google 账号（如 `@gmail.com`） | App 点 **Sign in**；CLI 选 **Google OAuth**；IDE 点 **Sign In** | 免费计划或 Google AI Plus / Pro / Ultra；适用 Antigravity Terms of Service |
| Google Workspace 账号（自有域名，含单人公司） | **Continue with Google** 只拿到有限的 starter quota；要更多额度见“Workspace 账号”一节 | Workspace 套餐不含 Antigravity 套餐 |
| 有 Gemini Enterprise 许可证 | App 选 **Use Business Account**；CLI 选 **Use a Google Cloud project** > **Continue with Google Cloud**；再选 **Subscription** | Google Cloud Terms of Service |
| 无许可证、按 token 付费 | 同上，在 **Select Billing Model** 选 **Pay As You Go** | 按 Agent Platform 价格计费，无内含额度 |
| 外部身份提供方（如 Okta） | **Use Advanced SSO Config**，填 WIF Configuration String | BYOID，经 Workforce Identity Federation |
| 无头 / CI | ADC，或（仅 CLI）Gemini API key | 见下文 |

#### 各入口登录要点

- App / CLI 企业登录：App 选 **Use Business Account**（登录过的先点 **Other sign-in options**），CLI 选 **Use a Google Cloud project**；换项目或地区要先登出再登录。
- CLI 本地启动先尝试读系统密钥环（Apple Keychain、Linux Secret Service/dbus、Windows Credential Manager），有有效令牌就静默登录，否则打开浏览器；`/logout` 会清除密钥环中的认证信息。
- SSH 远程：CLI 检测到 SSH 后打印授权 URL，在本机浏览器登录，再把页面显示的授权码贴回远程终端。
- IDE：个人用户点 Antigravity 图标 > **Sign In**；企业用户选 **Sign in with Gemini Enterprise**，再选项目和地区（`global`、`us`、`eu`）。

#### CLI 使用 Gemini API key

适合无浏览器的无头和 CI 场景，请求直接发往 Gemini API，不建立账号会话。必须**同时**设置 provider 和环境变量，只设 `GEMINI_API_KEY` 没有效果。

1. 在 `~/.gemini/antigravity-cli/settings.json` 中设置：

```json
{
  "modelProvider": "gemini"
}
```

2. 导出密钥（占位符，写进 `~/.zshrc` 才能跨会话保留）：

```bash
export GEMINI_API_KEY="<your-api-key>"
```

3. 运行 `agy`，头部会显示 **Gemini API key** 而不是邮箱。

排查：`modelProvider` 为 `gemini` 但没导出密钥会启动即退出；`modelProvider` 只接受 `gemini`；`GOOGLE_API_KEY` 和 `.env` 不会被读取；启动只检查密钥非空，无效密钥到第一次对话才报错。

自定义 Gemini 兼容端点：`export GOOGLE_GEMINI_BASE_URL="https://your-endpoint.example.com"`。恢复默认认证：删除 `modelProvider` 并重启。使用 API key 时 `/logout` 无效。

#### ADC（Application Default Credentials）

```bash
gcloud auth application-default login --project {GCP_PROJECT}
export AGY_ADC_AUTH=true      # CLI；退出 ADC 用 unset AGY_ADC_AUTH
```

App 和 IDE：从设置了 `AGY_ADC_AUTH=true` 的终端启动，或在 `~/.gemini/config/config.json` 写入 `{"userSettings": {"enableAdc": true}}`。使用 ADC 时不支持早于 Gemini 3 Flash 的模型，`eu` 和 `us` 地区没有图像生成。

#### IDE 的 `auth.type`

JetBrains、Zed、Xcode 通过 JSON 配置外部 agent：

| `auth.type` | 账号 | 说明 |
|---|---|---|
| `"oauth-personal"` | 个人 Google 账号 | Free、Pro、Ultra |
| `"oauth-business"` | Gemini Enterprise（表中标 Preview） | 另需 `gcp.project`、`gcp.location` |
| `"gemini-api-key"` | Gemini API | 使用 Gemini API key |
| `"agent-platform"` | Agent Platform（Preview） | API key 或 ADC；ADC 时需 `gcp.project`、`gcp.location` |

#### SDK 认证

- Gemini API key：`export GEMINI_API_KEY="<your-api-key>"`。
- Gemini Enterprise / Vertex AI：`LocalAgentConfig(vertex=True, project="your-gcp-project", location="us-central1")`，或设置 `GOOGLE_GENAI_USE_VERTEXAI=True`、`GOOGLE_CLOUD_PROJECT`、`GOOGLE_CLOUD_LOCATION` 并运行 `gcloud auth application-default login`。
- 本地模型（离线、无需 API key）：`LiteRTAgentConfig`，或用 `LocalOpenAIAgentConfig` 连接 `Ollama`、`LM Studio`、`vLLM` 等本地 OpenAI 兼容服务。

### 模型与推理强度

模型选择器在对话输入框下方，可随时切换，选择在同一会话内保持。如果在智能体运行中途切换，它会用原模型跑完当前这一轮（或直到你取消）。

#### 可选模型与套餐

| 模型 | Free / Plus | Pro | Ultra | Enterprise |
|---|---|---|---|---|
| Gemini 4 Neon | 否 | 是（付费，或合格高端设备试用） | 是 | 是 |
| Gemini 4 Carbon | 否 | 否 | 是 | 是 |
| Gemini 4 Argon | 否 | 是（同上） | 是 | 是 |
| Gemini 4 Hydrogen | 否 | 是（仅非试用的付费订阅） | 是 | 是 |
| Orchestrator | 否 | 是（付费，或合格设备试用） | 是 | 是（受限，见下） |
| Gemini 3.8 Flash | 是 | 是 | 是 | 是 |
| Gemini 3.7 Flash / 3.6 Flash / 3.1 Pro | 否 | 否 | 否 | 是 |
| Claude Sonnet 5.5 / Opus 5.5（thinking） | 否 | 是（仅非试用的付费订阅） | 是 | 是（需管理员开启） |
| Claude Sonnet 4.6 / Opus 4.6（thinking） | 是 | 是 | 否 | 否 |
| GPT-OSS-120b | 是 | 是 | 是 | 否 |

> 边界提醒：Claude Sonnet 4.6、Opus 4.6 和 GPT-OSS-120b 在原文中标注“将于 2026 年 11 月 30 日移除”。Pro 档中带“非试用”的模型，试用用户需先取消试用并转为付费 Google AI Pro。

**Gemini 4 Neon** 是未选过模型时新会话的默认模型：思考级别 `Low`、`Medium`（默认）、`High`、`Extra`，上下文窗口最高 512K tokens，Free 和 Plus 不可用。

#### 推理强度

- 多数可调的 Gemini 模型提供 `Low`、`Medium`（默认）、`High` 三档。
- **Gemini 4 Neon** 和 **Gemini 4 Hydrogen** 多一档，在选择器里叫 **`Extra`**，对应 Extra High 推理预算，CLI 里写作 `xhigh`。
- Enterprise 里的 Claude Opus 5.5 / Sonnet 5.5 有四档：`Low`、`Medium`（默认）、`High`、`Max`，每档在选择器里是单独条目（如 **Claude Opus 5.5 (High)**）。

CLI 切换方式：

```bash
agy models                                   # 列出你账号可用的全部模型标识
agy --model gemini-4-neon-xhigh              # 模型名里带强度
agy --model claude-opus-5-5 --effort high    # 基础模型 + --effort（或会话内 /effort）
```

`/model` 在会话内切换；`--effort` / `/effort` 的取值为 `low`、`medium`、`high`、`xhigh`、`max`（视模型而定），其中 `/effort` 需要 `agy` `v1.2.11` 或更高。

#### Orchestrator：规划者加执行者

Orchestrator 是模型选择器里的一个选项，在同一对话里配对两个模型：大模型当规划者（探索代码库、写计划、审查进度），快模型当执行者（搜索、改文件、构建、测试）。它不是“先分类再路由到单个模型”，而是两者在同一任务里来回交接：

```text
你的消息
   │
   ▼
规划者：只读规划阶段（探索 → 消歧 → 写计划与重新规划条件）
   │
   ▼
执行者：改文件、跑构建和测试、对照计划验证
   │        ▲
   │        └─ 遇到意外（验证连续失败两次等）→ 交还规划者修计划
   ▼
规划者：在长任务中或你中途发消息时审查进度 → 收尾 / 继续 / 更新计划
```

| 角色 | 支持的模型 |
|---|---|
| Orchestrator Model（规划者） | Gemini 4 Neon（默认）、Gemini 4 Carbon、Gemini 4 Argon、Claude Opus 5.5 |
| Executor Model（执行者） | Gemini 4 Hydrogen（默认）、Claude Sonnet 5.5 |

- 仅 Google AI Ultra、Google AI Pro（付费或合格设备试用）和 Gemini Enterprise 可用。
- App：**Settings** > **Models** > **Orchestrator Settings** 设默认规划者与执行者；悬停 **Orchestrator** 可仅为当前对话改规划者。自定义规划者 / 执行者只适用于本地会话。
- CLI：`/model` 里选 **Orchestrator**，用 ← / → 或 Tab 选 Planner（`Neon`；Ultra 有 `Carbon`；或 `Argon`），搭配 Gemini 4 Hydrogen。也可 `agy --model "Orchestrator (Neon)"`。Orchestrator 在 CLI 使用固定推理预算，`--effort` 和 `/effort` 不适用。
- Gemini Enterprise 里 Orchestrator 固定为 Gemini 4 Carbon（High）规划加 Gemini 4 Hydrogen（High）执行，需要两个模型都可用；标识为 `gemini-pro-orchestrator`，没有 Orchestrator Settings 和 `--effort`。

原文建议：日常开发（功能、修 bug、重构、多文件调试）用 Orchestrator（推荐默认）；每步都需最大推理深度时用 Neon、Carbon 或 Argon 单模型；快速日常维护用 Gemini 4 Hydrogen。要多智能体并行或互查，用 subagents、Swarm、Boost 或 Teamwork，而不是 Orchestrator。

另外，生成式图像工具（UI 草图、配图、架构图）用的是 Nano Banana 2，不可自定义。

### 套餐、配额与积分

本节适用于个人 Google 账号；Workspace 套餐不含 Antigravity 套餐或更高限额。原文没有给出具体价格或额度数字，只有下列定性描述（概览页仅提到 Free 计划起价 `$0/month`）。

| 档位 | 配额 | 其他 |
|---|---|---|
| Free、Google AI Plus 等非 Pro/Ultra | 有意义的额度，每周刷新；有每周速率限制 | 基线功能全部可用 |
| Google AI Pro | 较高额度，每五小时刷新，直到触及每周上限；每周限制更高 | 可用积分超额使用 |
| Google AI Ultra | 最高额度，每五小时刷新；每周限制最高 | 可用第三方模型；可用积分超额使用 |

- 所有档位的基线：以 Gemini 3.8 Flash 等模型作为核心智能体模型、无限 Tab 补全、全部产品功能（如定时任务和 CLI）。
- 限额主要由可用容量决定，且与智能体做的工作量挂钩：简单任务能发更多提示，复杂任务消耗更多额度。限额可能调整。
- **积分超额**：Pro / Ultra 用户可以用已购买的 AI credits（或一次性促销积分）在基线额度之外继续使用，按标准 Gemini Enterprise 消耗定价计费。基线额度用完后由用户设置 **AI Credit Overages** 控制：**Never**（等额度刷新）或 **Always**（始终用积分，额度刷新后自动回到基线）。
- 在 **Settings** 页面查看各模型的基线额度用量；CLI 里用 `/usage` 查看额度和积分消耗。
- 暂不支持：自带密钥（BYOK）或自带端点以获得额外限额；通过自定义合同的组织级套餐。

### Workspace 账号

“Workspace 账号”指自有域名的邮箱（如 `you@yourcompany.com`），由 Google Workspace 或 Cloud Identity 管理，包括单人公司。判断方式：你的组织为该域名注册了 Workspace / Cloud Identity，或你在 Admin console 管理用户，或 Google 账号页写着“由你的域名管理”。如果只是用非 Gmail 邮箱注册的个人 Google 账号且域名不用 Workspace，那是个人账号，不用看本节。

核心事实：Workspace 套餐目前不含 Antigravity 套餐、更高限额或 AI 积分，即使组织订阅了带 Gemini 功能的 Workspace 套餐。用 **Continue with Google** 默认获得有限的 **Antigravity Starter Quota**；如果管理员关闭了个人使用，则显示账号不符合条件。

| 选项 | 如何付费 | 条款与控制 | 准备 |
|---|---|---|---|
| 个人 Google 账号 | 免费，或 Google AI Pro / Ultra 订阅（仅此选项可用这两档） | Antigravity Terms of Service；无组织管理员控制；可自行关闭遥测 | 几分钟 |
| 连接 Google Cloud 项目（Pay As You Go） | 按 token，按 Google Cloud API 价格 | Google Cloud Terms of Service；遥测始终关闭 | 开通计费的 Google Cloud 项目 |
| Gemini Enterprise | 席位许可证或按量版 | Google Cloud Terms of Service，集中管理员控制 | 管理员配置 |

- 关闭个人账号遥测：App 的 **Settings** > **Account** 关闭 **Enable Telemetry**；CLI 运行 `/config` 把 **Enable Telemetry** 设为 off。
- 切换账号：先登出（CLI 为 `/logout`），再选 **Continue with Google**（CLI 为 **Google OAuth**）用个人账号登录。
- 以上登录步骤覆盖 App 和 CLI；独立 Antigravity IDE 不支持商业账号；IDE 扩展走 Gemini Enterprise 文档。

### Pay As You Go

**Pay As You Go** 计费模型：Antigravity 把请求发到你自己 Google Cloud 项目里的 Gemini Enterprise Agent Platform，只为用掉的 token 付费，计入项目的 Cloud Billing 账号，不需要 Gemini Enterprise 许可证。没有内含额度。Anthropic Claude 模型仅在项目已启用时可用，项目访问不到的合作方模型会被隐藏。遥测不收集。

> 边界提醒：不要与“Gemini Enterprise Pay-as-you-go 版”混淆。后者是一种 Gemini Enterprise 许可证，登录时在 **Subscription** 下选择，且每个用户需要 `roles/discoveryengine.agentspaceUser` 角色；前者在 **Select Billing Model** 里选 **Pay As You Go**，不需要许可证。

#### 操作流程：准备项目（管理员做一次）

1. 选择或创建 Google Cloud 项目。
2. 确保项目关联了有效的 Cloud Billing 账号。
3. 启用 Gemini Enterprise API（`aiplatform.googleapis.com`）。
4. 给每位开发者授予 **Agent Platform User**（`roles/aiplatform.user`）角色，或包含 `aiplatform.endpoints.predict` 权限的其他角色。
5. 可选：要用 Claude 模型，在 Model Garden 为项目启用。

#### 操作流程：登录

版本要求：App `v2.31.0` 或更高，CLI `v1.4.11` 或更高。

- App：**Use Business Account** > **Continue with Google Cloud** > **Select Billing Model** 选 **Pay As You Go** > **Provide Google Cloud Project** 填项目 ID > **Choose Region** 选 **Global**、**United States** 或 **Europe**。
- CLI：`agy`（已登录先 `/logout`）> **Use a Google Cloud project** > **Continue with Google Cloud** > **Pay As You Go** > 输入项目 ID，选 `global`、`us` 或 `eu`。
- 换项目或地区：登出再登录。无头环境用 ADC。只用 CLI 时还可以用 Gemini API key（按 Gemini API 价格计费）。

常见报错：`aiplatform.endpoints.predict` denied 表示缺 `roles/aiplatform.user` 或计费 / API 未开；“Selected model is not supported in the selected location”要换地区重新登录或换模型；看不到 Claude 模型要在 Model Garden 为项目启用。

### Enterprise：Gemini Enterprise 部署

通过 Gemini Enterprise，Antigravity 可使用你组织 Google Cloud 基础设施中托管的模型，会话受 Google Cloud 企业安全控制、数据驻留和 Google Cloud Terms of Service 约束。

> 边界提醒：企业集成支持 App、CLI 和 IDE 扩展。**独立 Antigravity IDE 不支持企业部署**；旧的 Gemini Code Assist IDE 扩展只支持第一方 Gemini 模型。

#### 两种接入方式

| 方式 | 要点 |
|---|---|
| Gemini Enterprise Pay-as-you-go | 无席位许可证；按量计入已关联项目的合格 Cloud Billing 账号（发票账号，或带预付积分的自助账号）；用户需 `roles/discoveryengine.agentspaceUser`；登录时在 **Subscription** 下选为许可证 |
| Gemini Enterprise Standard 或 Plus 许可证 | 含第一方额度、托管超额和集中管理；跨 App、CLI、IDE 扩展，无需单独的 Antigravity 许可证 |

第一方 Gemini 模型默认启用，从按版本汇总的席位积分中扣除（`$10/month` Standard、`$15/month` Plus）。第三方合作方模型（Claude Opus 5.5 / Sonnet 5.5）需管理员在控制台 **Gemini Enterprise > Settings > AI developer tools** 里开启，按 token 计费。

#### 开发者登录

1. 启动 App，或运行 `agy`（已登录先 `/logout`），或打开 IDE 扩展。
2. App 选 **Use Business Account**；CLI 选 **Use a Google Cloud project**；IDE 扩展用其 Gemini Enterprise 登录选项。
3. 选 **Continue with Google Cloud**，浏览器完成认证。
4. **Select Billing Model** 选 **Subscription**（CLI 为 **Subscriptions**）。
5. 选择与项目关联的许可证；找不到时选 **Other License**（CLI 为 **Can't find your subscription? Enter a Google Cloud project**），输入项目 ID 和位置（`global`、`us`、`eu`）自行指派。CLI 会把项目和位置存进 `~/.gemini/antigravity-cli/settings.json` 的 `gcp` 以预填后续登录。

用 BYOID（如 Okta）：选 **Use Advanced SSO Config**，填入管理员提供的 WIF Configuration String，形如 `locations/global/workforcePools/POOL_ID/providers/PROVIDER_ID`。

> 边界提醒：你的遥测和模型交互会记录到所选许可证对应的 Google Cloud 项目；每个项目和位置只能维持一个许可证。同一邮箱存在于多个身份提供方时，要用与许可证匹配的那个身份登录。

#### 管理员清单

1. 购买并配置 Gemini Enterprise，分配许可证。管理员用 `roles/discoveryengine.agentspaceAdmin`；包括管理员在内凡使用 Antigravity 的人都需要 `roles/discoveryengine.agentspaceUser`。
2. **AI developer tools 可能默认关闭**（例如已有订阅或其他地区）：在 **Gemini Enterprise > Settings > AI developer tools** 选择订阅所在地区（`global`、`us`、`eu`）并打开开关（会同时启用 `discoveryengine.googleapis.com` 和 `businessaicode.googleapis.com`）。
3. 选择或新建 Google Cloud 项目，确认 Cloud Billing 有效，启用 `aiplatform.googleapis.com`。
4. 可选：启用第三方合作方模型。
5. 按需配置网络放行、VPC-SC、请求与响应日志。

> 边界提醒：购买许可证前先启用 `aiplatform.googleapis.com` 并等待约五分钟生效，否则新购许可证可能失败或无法开通。

#### 地区与网络

地区 `global`、`us`（多区域）、`eu`（多区域）：三者都支持文本生成、代码推理、多模态。图像生成目前仅在 `global` 端点可用。地区数据驻留约束适用于第一方 Gemini 模型；第三方模型推理在 `global`、`us`、`eu` 运行，不支持国家级区域。

出站流量受限时，需放行 TCP 443 的 `businessaicode.googleapis.com`、`aiplatform.googleapis.com`、`discoveryengine.googleapis.com`、`cloudcode-pa.googleapis.com`（us / eu 另有 `*.us.rep.googleapis.com`、`*.eu.rep.googleapis.com` 区域域名；双向 TLS 用 `*.mtls.googleapis.com`）。即使用 `us` 或 `eu`，登录时仍会连全局的 `businessaicode.googleapis.com` 和 `cloudcode-pa.googleapis.com`，两者必须保持可达。经 PSC / Private Google Access / VPC-SC 私网访问 us / eu 时，需为 `*.<location>.rep.googleapis.com` 建 PSC 区域端点和私有 DNS 区域。

#### 安全与权限控制

- 把 `businessaicode.googleapis.com` 和 `aiplatform.googleapis.com` 加入 VPC Service Controls 边界；可配置请求与响应日志。
- 人在回路：App 保持 **Default**（隔离终端沙箱）或 **Request Review**（**Settings > General > Permission Settings**），使非沙箱 shell 命令、外部文件修改、MCP 工具调用都需批准；**Turbo** / **Always proceed** 仅用于隔离容器或一次性分支。
- CLI：保持交互式权限和 `--sandbox=true`（macOS 为 `seatbelt`，Linux 为 `bubblewrap`），或先用 `--mode plan`；用 `/permissions` 或 `/config` 配置策略。
- 选 Claude 时的“混合执行”：所选模型负责主要推理、规划、工具编排和代码生成；轻量第一方 Gemini 模型仍在后台处理上下文管理、摘要等高频协调，这些消耗第一方 token，可能产生费用。

企业里的 CLI / API 标识形如 `gemini-4-neon`、`gemini-4-carbon`、`gemini-4-argon`、`gemini-4-hydrogen`、`gemini-pro-orchestrator`、`gemini-3.8-flash`、`gemini-3.1-pro-preview`、`claude-opus-5-5-<level>`、`claude-sonnet-5-5-<level>`（level 为 low、medium、high、max）。

向支持团队提供诊断日志：CLI（Linux / macOS）`~/.gemini/antigravity-cli/cli.log`；App（macOS）`~/Library/Logs/Antigravity/language_server.log`。

### Build with Google

Build with Google 是 App 中的一组精选插件包，把 skills、MCP 服务器和编辑器扩展预配置给特定的 Google 技术栈，免去逐个找工具。

操作流程：左侧栏 **Customizations** > **Marketplace** 标签 > 安装所需平台的插件；之后在 **Customizations** > **Installed** 启用、停用、配置或卸载。

| 插件包 | 内容 |
|---|---|
| Modern Web Guidance、Chrome DevTools | 现代 Web 最佳实践；Chrome DevTools 与 Puppeteer（无障碍、Core Web Vitals、内存泄漏） |
| Firebase Bundle、Google Cloud Developer、Data Agent Kit Starter Pack | Firebase 各服务；Cloud Run、GKE、IAM、Terraform 等；BigQuery、dbt、Spark、Dataflow 等及数据 MCP 服务器 |
| Android CLI、Dart and Flutter、Google Maps Platform | Android 开发；Dart / Flutter 加 Dart MCP 服务器；Maps、Places、Routes 等（可用 Maps Demo Key 免计费起步） |
| Google Antigravity SDK、Science | `google-antigravity` 的架构与安全策略参考；科学计算 skills |

想让智能体用最新官方文档回答任何 Google 开发者产品的问题，可再配合 Google Developer Knowledge MCP 服务器（在 MCP Store 一键安装，使用你的 Google 凭证）。

### 常见问题（FAQ）

#### 为什么个人 Google 账号登录失败？

个人账号（如 `@gmail.com`）在任何套餐（含免费）都能用。失败时查三项：账号关联的国家或地区是否在支持名单；是否满 18 岁（不满 18 岁不可用，可到 Google 账号做年龄验证）；是否其实是 Workspace 等由组织管理的工作 / 学校账号。

#### 哪些地区可用？

只有支持名单上的国家和地区可用（名单按美洲、欧洲、非洲、亚洲、大洋洲列出）。我核对的亚洲名单里没有中国大陆。判断依据是 Google Terms of Service 页面上关联的国家，若有误可提交修改地区的申请；开始前以官方名单为准。

#### 企业账号为什么登录不了？

依次检查：登录时选了 **Use Business Account**，且管理员给你分配了 Gemini Enterprise 许可证或所需 IAM 角色；管理员已在订阅所在地区开启 AI developer tools；邮箱存在多个身份提供方时用与许可证匹配的身份；企业登录只在 App、CLI 和 IDE 扩展可用，独立 Antigravity IDE 不行。

#### Workspace 账号能用吗？

能登录，但默认只有有限的 starter quota。要更多用量，用个人账号订阅 Pro / Ultra，或用 Pay As You Go 连自己的 Google Cloud 项目，或由管理员配置 Gemini Enterprise。

#### 可以用第三方软件（如 Claude Code、OpenClaw、OpenCode）调用我的 Antigravity 登录吗？

不可以。用第三方软件、工具或服务访问 Antigravity 违反服务条款，可能导致账号被暂停或终止。要在第三方 coding agent 里用 Gemini，官方建议使用 Gemini Enterprise 或 Google AI Studio API key。

#### 怎么升级到付费 Google AI 套餐？

部分模型需要付费 Google AI Pro 或 Ultra，免费试用不含。Free 或 Plus 用户在 Google One 订阅 Pro 或 Ultra；Pro 试用用户需提交请求取消试用并转为付费 Pro，或直接升级 Ultra。

#### `agy -p` 无头模式能包进第三方 agent 或自动轮换账号吗？

不能。`agy -p` 面向直接脚本和 CI；在消费者套餐下，把 `agy` 包进第三方编码智能体或应用，或自动轮换账号，不受支持。

#### 怎么关闭数据收集？

可随时在设置里退出。App：**Settings** > **Account** 关闭 **Enable Telemetry**；CLI：`/config`。

#### 其他

App 支持 worktree；支持任何公开的文本类编程语言；智能体运行时 Antigravity 会阻止电脑休眠。

### 从 Firebase Studio 迁移

Antigravity 是本地运行、智能体优先的环境，取代 Firebase Studio 基于网页的 Code 视图。你仍可用 Firebase CLI 部署项目并在本地测试函数。

对应关系：Cloud IDE Code 视图变为本地的 VS Code 风格界面；Agentic chat 原生集成在 IDE 里；App Hosting 部署走 Firebase CLI 或内置集成；浏览器端模拟器变为本机运行 Firebase Local Emulator Suite。

#### 前置条件

均需安装并保持最新：Google Antigravity IDE、Node.js（20 或更高）、Firebase CLI（15.10.0 或更高）。

#### 操作流程：自动迁移（方案 1）

1. 在 Firebase Studio 点工作区顶部的 **Move now**。
2. 若出现 **Zip and Download** 按钮就点它；否则打开命令面板运行 **Firebase Studio: Zip & Download**。
3. 本地解压并用 Antigravity 打开。
4. 在 Agent 面板输入下面的提示。为节省 token，官方建议选 **Gemini Flash** 模型：

```text
@fbs-to-agy-export
```

5. 按智能体的引导完成迁移；报错时让它重试。下载窗口没出现时，检查浏览器是否拦截了弹窗。

#### 操作流程：手动导出（方案 2）

不使用 AI token，用 Firebase CLI 直接导出，`<path>` 为解压后的项目目录或原 `.zip`：

```bash
npx firebase-tools@latest studio:export <path>
```

> 边界提醒：`studio:export` 目前针对 Next.js、Flutter、Angular 工作区优化；其他类型可以试，但迁移可能不完全成功。

#### 预览与发布

- 预览：左侧栏 **Run and Debug** > 点播放按钮启动本地开发服务器 > 按终端提示预览。Agent 面板被隐藏时点标题栏 **Toggle Agent**。
- 发布：在对话里输入 `Publish my app`，提示运行 `firebase deploy` 时选 **Yes**。已发布过 App Hosting 会发布到原 URL，首次发布会被带着走完流程；以后同样说 `publish my app`。
- 部署出问题：先用 Firebase CLI 重新认证，或检查项目 secrets。迁移的 bug 提交到 `firebase-tools` 的 GitHub Issues。

### 从 workflows 迁移到 skills

Workflows（旧版的单个 Markdown 文件，用于可重复的提示序列）已**弃用**，新的 workflows 系统在路上（即 Dynamic Workflows：智能体编写并运行 Python 脚本，在并行和串行阶段编排多个智能体）。现有 workflows 目前仍可用。2026 年 5 月 Antigravity 采用了开放的 Agent Skills 标准。

#### Workflow 与 Skill 对照

| 项目 | 旧 Workflow | Agent Skill |
|---|---|---|
| 规范 | Antigravity 专有格式 | 开放的 Agent Skills 标准 |
| 结构 | 单个 `.md` 文件 | 目录包（`<skill-name>/SKILL.md` 加资源） |
| 工作区路径 | `.agents/workflows/<name>.md` | `.agents/skills/<name>/SKILL.md` |
| 全局路径 | `~/.gemini/config/global_workflows/<name>.md` | `~/.gemini/config/skills/<name>/SKILL.md` |
| 上下文加载 | 整个文件载入提示 | 渐进式：先索引元数据，正文按需加载 |
| 附带资源 | 不支持 | 支持 `scripts/`、`references/`、`examples/` |
| 斜杠命令 | `/<workflow-name>` | `/<skill-name>` |
| 大小限制 | 12,000 字符 | 不限制，可多文件模块化 |

同名时 skill 优先于 workflow。

#### 操作流程：自动迁移

`/migrate-workflows` 仅在 Antigravity App 2.12.0 或更高版本可用，CLI 和 Antigravity IDE 里没有。

1. 打开 App 2.12.0 或更高版本。
2. 运行 `/migrate-workflows`。
3. 智能体会：扫描全局（`~/.gemini/config/global_workflows/`、`~/.gemini/config/workflows/`）和工作区（`.agents/workflows/`）目录；解析每个 workflow 的 YAML frontmatter 和正文；生成 `~/.gemini/config/skills/<workflow-name>/SKILL.md`（全局）或 `.agents/skills/<workflow-name>/SKILL.md`（工作区）；把原文件加 `.bak` 后缀归档（如 `deploy.md` 变 `deploy.md.bak`）；立即激活新 skills。

#### 操作流程：手动迁移

```bash
mkdir -p .agents/skills/deploy-staging                      # 工作区；全局则为 ~/.gemini/config/skills/deploy-staging
mv .agents/workflows/deploy-staging.md .agents/skills/deploy-staging/SKILL.md
```

然后让 `SKILL.md` 以标准 frontmatter 开头；`description` 要写清楚“做什么”和“何时使用”：

```markdown
---
name: deploy-staging
description: Deploys the current project build to the staging preview environment on App Engine. Use when testing staging releases or verifying PR previews.
---

# Deploy Staging

Follow these instructions to deploy the current build to staging...
```

可选：把内嵌脚本和长参考文档拆到 `scripts/`、`references/` 子目录，在 `SKILL.md` 里指示智能体按需运行或查看。

最后在 App 中输入 `/deploy-staging` 验证：智能体会加载 skill 元数据并执行步骤。

#### 迁移前后示例

迁移前是带 `name` / `description` frontmatter 的单文件（如 `build-and-test`：依次跑 `npm run check`、`npm run test`、`npm run build`）；迁移后放在 `.agents/skills/build-and-test/`，含 `SKILL.md` 和 `scripts/verify-bundle.js`，`description` 写成“Runs full validation suite ... Use before submitting changes.”。

> 边界提醒：workflows 目前仍能用，不必立刻迁移；但要继续使用旧的逐步指令，官方建议迁移成 skill，而不是等待新的 Dynamic Workflows。

---

## Part II：Antigravity App

### Antigravity App 功能总览

Antigravity App 与 Antigravity CLI 运行的是同一个智能体（agent），所以多数能力两边通用：Projects、权限、Subagents、Skills / Rules / MCP、Hooks、Plugins、Remote Control 等。App 额外提供的是“看得见”的界面：侧边栏、Side panel、内置浏览器、Review 面板、命令面板、语音模式、Automations。

```text
Antigravity App
│
├─ 左侧边栏
│  ├─ Projects（点 + 新建；可选 New Remote Project / SSH Host…）
│  ├─ Automations（按计划给智能体发消息）
│  ├─ Customizations（Plugins、Marketplace、Skills、Rules、MCP、Custom Agents）
│  └─ Settings（底部）
│
├─ 对话区（message box）
│  ├─ @ 提及：文件、文件夹、Rules、Conversation
│  ├─ Quote / Comment：引用选中文本
│  ├─ 麦克风（语音听写）与 Start Voice mode（语音模式）
│  └─ 模型选择器、权限选择器（permissions picker）
│
├─ Side panel（对话右侧，标签页）
│  ├─ Artifacts、文件、Diff、终端、Subagent 对话、网页（内置浏览器）、PDF、.ipynb
│  └─ Overview panel（右上角）：Review / Files / Terminal / Plugins + 产出清单
│
└─ 命令面板：Cmd/Ctrl + K（对话）、+ P（文件）、+ Shift + K（项目）、+ Shift + P（命令）
```

#### 典型工作流

1. 在左侧边栏的 **Projects** 旁点 **+** 建项目，选好文件夹（可多个）。
2. 新对话里选 **Local**（直接改当前目录）或 **New worktree**（隔离的 Git worktree，适合复杂任务和并行 subagent）。
3. 用 `@` 把相关文件、规则、旧对话指给智能体，再描述任务。
4. 智能体产出 implementation plan 等 artifact，在 Side panel 里阅读并用 **Comment** 反馈。
5. 点 Overview 里的 **Review** 查看 diff，按需 Stage / Commit / Push。
6. 需要验证页面效果时，让智能体用浏览器 subagent 打开本地服务检查。

#### App 与 CLI 的差异速览

| 能力 | App | CLI |
|---|---|---|
| 默认项目 | 手动创建项目；也可在项目之外开对话 | `default-cli-project`，用 `--project=<project_id>` / `--new-project` 切换 |
| 搜索 | 命令面板（全文搜索对话） | `/resume`（只搜标题、首条提示、ID）、`/codesearch` |
| 语音模式 | Start Voice mode 按钮 | `/voice live` 或 F6 |
| 语音听写 | 麦克风按钮或 Ctrl + M | `/voice dictate`、`/record` 或 F5 |
| 浏览器 | 内置浏览器（个人账号 `v2.30.0+`）或 Chrome | `/browser` 驱动本机 Chrome |
| 设置 | Settings 界面 | `~/.gemini/antigravity-cli/settings.json` 与 `/config` |
| Review | Review 面板 | `/diff` |

> 边界提醒：多个功能标注了版本（如 `v2.30.0+`）和账号类型（个人 Google Account / 商业与企业账号）。看不到某个入口时，先查版本和账号类型，不要先怀疑安装坏了。

---

### @ 提及与引用

`@` 让智能体直接从正确的上下文开始，而不是自己搜索。App 里除了文件，还能提及规则和历史对话，也能引用选中的文本。

#### 可提及 / 可引用的对象

| 对象 | App | CLI |
|---|---|---|
| 文件和文件夹 | 输入 `@` 加部分名称 | 输入 `@` 加部分路径 |
| Rules | `@` 后选 **Rules**，或输入 `@rules:` | 不支持 |
| 过去的对话 | `@` 后选 **Conversation**，或输入 `@conversation:` | 不支持 |
| 选中的文本 | 选中后点 **Quote** | 不支持 |
| 给 subagent 发消息 | 打开该 subagent，在它自己的输入框里写 | 提示以 `@` 加 subagent 名称开头 |

#### 智能体实际收到什么

提及文件不会把内容贴进提示，智能体只拿到路径，需要时再去读。

| 你添加的 | 智能体收到 |
|---|---|
| 文件或文件夹 | 完整路径 |
| 行范围 | 文件路径和行号，不含选中文本 |
| 规则 | 规则全文 |
| 过去的对话 | 对话标题和 ID，需要细节时读取保存的 transcript |
| 引用文本 | 引用的文本本身 |
| 终端输出 | 选中的输出（最多最后 8 KB）和终端名称 |

#### 操作流程（App）

1. 在 message box 输入 `@bkmk`，模糊匹配会找到 `bookmarks.js`。裸 `@` 只列出 **Rules** 和 **Conversation** 两个分类。
2. 用 ↑ / ↓ 高亮，Enter 或 Tab 选中（或直接点击），提及会变成带图标的 chip。
3. 不想敲 `@` 时，点 message box 里的 **Add Context**（**+**）并选 **Mentions**。
4. 引用回答：选中文本，点弹出的 **Quote**，或按 Cmd + L / Cmd + I（macOS）、Ctrl + L / Ctrl + I（Windows 和 Linux）。
5. 在文件或 diff 里引用，chip 显示形如 `bookmarks.js #L13-17`；在 artifact 里引用，chip 显示 artifact 名称和选中文字开头。
6. 想留评审意见而不是引用，点 **Comment**。终端里选中输出后点 **Quote**，会生成形如 `terminal:bash` 的 chip。

可照抄示例：

```text
@rules:release 按 @conversation:rate limiter 里的结论，给 @src/routes/bookmarks.js 补上限流测试
```

#### 常见坑

- 要应用 `manual` 触发的规则，只能靠 `@` 提及；Antigravity 不会自动加载它们。
- Subagent 不出现在 App 的 `@` 菜单里，被 kill 的 subagent 也无法再发消息。
- CLI 里要提及工作区外的路径，查询以 `~/` 或 `/` 开头，例如 `@~/Develop/`；提及不等于授权，权限设置仍然决定能否读取。
- CLI 里 subagent 名称格式为 `<type>:<role>`，必须精确匹配（不区分大小写），否则提示会发给父智能体。
- 在 CLI 里手动输入或改过的路径只是纯文本，不带“选中项”的完整路径信息。

---

### 语音模式

Voice mode 由 **Gemini Live** 驱动：你说话，智能体用语音回答，同时在对话里干活，你可以随时插话、纠正、停止，也能用语音回答权限提示。

#### 语音模式 vs 语音听写

| | Voice mode | Voice dictation |
|---|---|---|
| 作用 | 与智能体口头对话 | 把你说的话输入到 message box |
| App 怎么开 | 在**空**的 message box 点 **Start Voice mode** | 点麦克风按钮，或按 Ctrl + M |
| 智能体怎么答 | 语音回答，工作内容显示在对话里 | 发送后以文字回答 |
| 适合 | 免手操作、边说边调整任务 | 想先审阅、编辑提示再发送 |

#### 启用步骤（App）

1. 清空 message box（包括附件），点 **Start Voice mode**（带声波条的圆形按钮）。它仅在输入框为空时出现。
2. 系统询问麦克风权限时选允许。
3. 首次启动会弹出 **Choose Antigravity’s Voice**：用 ← / → 试听，点 **Use** 或按 Enter 确认。
4. 出现 **Connecting…**，就绪后有提示音；面板显示 **Listening. Ask anything.** 即可开口。

CLI：运行 `/voice live` 或按 F6；需要麦克风和扬声器，且版本为 1.5.0 及以上。SSH 会话里要在本机运行 `agy mic-serve`，并设置 `ANTIGRAVITY_MIC`。

#### 你可以对它说什么

| 目的 | 示例说法 |
|---|---|
| 给任务 | “Find the failing test and fix it.” |
| 中途调整 | “Just check it, don’t change anything yet.” |
| 询问进度 | “What are you doing?” |
| 停止任务 | “Stop” / “Cancel that” / “Never mind” |
| 让它别说话 | “Wait” / “Hold on” / “I’m reading” |
| 打开进度播报 | “Keep me posted” / “Update me.” |
| 设置间隔 | “Update me once a minute.” |
| 关闭播报 | “Work quietly” |
| 结束通话 | “Bye” / “End the call.” |

默认是安静工作；开启播报后约每 30 秒一次，语音模式结束后播报会关闭。

#### 用语音回答权限提示

智能体会朗读请求并等你回答：“Allow it just this once.”（仅一次）、“Allow it for this conversation.”、“Allow it for this project.”、“Always allow this.”（全局）、“No.”（拒绝，可附带替代方案）。有些高风险操作只能允许一次；权限提示等待期间说 “Stop” 会拒绝它。

> 边界提醒：删除文件（如 `rm`）、提权（如 `sudo`）、force-push 这类破坏性操作不能靠语音批准，必须在屏幕上点 **Allow**；但可以用语音拒绝。

#### 控制项与快捷键

| 动作 | App（macOS） | App（Windows / Linux） | CLI |
|---|---|---|---|
| 静音 / 取消静音麦克风 | Option + M | Alt + M | F6 或 Shift + F6，`/voice mute` |
| 静音智能体的声音 | Shift + Option + M | Alt + Shift + M | 无对应项 |
| 结束 | 点 **End**，或说 “Bye” | 同左 | F7、`/voice stop`，或空提示下 Esc |
| 改音色 / 音量 / 语速 | 面板或 Settings | 同左 | `/voice speaker <name>`、`/voice volume 0-100`、`/voice speed 0.5-3` |
| 允许插话 | 直接说话 | 直接说话 | `/voice duplex on`（建议戴耳机） |

App 里语速滑块范围是 0.75x 到 2x。语音相关设置在 **Settings** > **General** > **Voice mode**（Voice、Microphone、Volume）。Settings 的 **Shortcuts** 里这两个快捷键叫 **Mute Voice Mic** 和 **Mute Voice Speaker**。

#### 如何验证

- App 对话里出现 **Started voice chat** 标记，面板顶部的 call bar 显示谁在说话和已用时长。
- 结束后出现 **Stopped voice chat** 标记和 **Voice transcript** 卡片，点 **Open** 查看，**Copy transcript** 复制；transcript 保存在本机，删除对话时一并删除。
- CLI 提示下方的状态行显示 **Live**、时长，以及 listening / thinking / speaking。

#### 限制与常见坑

- App 里语音模式只支持**本地**对话，不支持 Remote Control 会话或云端对话。
- 同一时间只能在一个对话里开启；不能在 subagent 对话或 side question 里启动。
- CLI 不支持 print mode 和 remote agent 会话。
- **Start Voice mode** 不出现：先清空输入框和附件，确认没有在别的对话里开着、已登录、没超出用量上限；也可能是你的账号尚未开放。
- 麦克风被拦截：到 macOS 的 **System Settings** > **Privacy & Security** > **Microphone**，或 Windows 的 **Settings** > **Privacy & security** > **Microphone** 放行。
- 你的音频会流到 Google 服务器由 Gemini Live 识别；CLI 默认在智能体说话时静音麦克风，所以要先开 `/voice duplex on` 才能打断。
- CLI 的语音听写目前不对商业与企业账号开放。

---

### Browser use（浏览器智能体）

智能体可以打开网页、操作你正在运行的应用、检查控制台错误和失败的网络请求、截图，并在结束任务前验证视觉修复。

#### 两种后端，别搞混

| | 内置浏览器智能体 | Chrome DevTools 浏览器 subagent |
|---|---|---|
| 适用 | App `v2.30.0+`，个人 Google Account（Free、Plus、Google AI Pro、Google AI Ultra），正在 macOS、Windows、Linux 上逐步推出（rolling out） | CLI；App 的商业 / 企业账号（管理员未禁用时）；Antigravity IDE `v2.6.0+` 的本地工作区 |
| 浏览器 | Side panel 里的内置浏览器，无需安装 Chrome 或 Node.js | 本机 Google Chrome `v144+`，经 Chrome DevTools MCP |
| 启动 | 用自然语言描述，或 `/browser <task>` | `/browser <task>` |
| 并行 | 可多个 subagent，各占独立标签页 | 每个实例只有一个 Chrome 调试连接 |
| 登录态 | 每个 Antigravity 账号独立持久的 cookie 配置，与 Chrome 分开 | 你正在运行的 Chrome 配置 |
| 深度诊断 | 控制台错误、经过清洗的网络请求 | 性能 trace、Lighthouse、堆快照、限速、`evaluate_script` |

内置浏览器暂不支持商业 / 企业账号、Remote Agents 和 SSH 工作区窗口。手动使用的内置浏览器 side panel 在所有桌面 App 账号可用（`v2.26.0+`）。

#### 怎么开启与验证（App）

1. 个人账号 `v2.30.0+`：直接在 message box 描述任务，主智能体会自动启动 `browser` subagent。
2. 想明确指定时用 `/browser`。
3. 打开 **Settings** > **Browser**，在 **Browser Actuation Rules** 下确认后端：显示 “It runs in the built-in browser, so no separate browser install is needed.” 即内置浏览器；显示 “Requires Google Chrome” 则走 Chrome。

可照抄示例：

```text
/browser Open http://localhost:3000, switch to a 390px mobile viewport, verify that the navigation drawer opens and closes cleanly, and report any console errors.
```

简单的静态文档问答，智能体直接读页面、不开标签页；需要实时渲染、点击、表单输入、响应式检查、截图、控制台 / 网络检查或已登录页面时才委派给 `browser` subagent。

#### 配置 Chrome 后端（CLI）

1. 机器上装好 Node.js（`npx` 在 `PATH` 中）和 Google Chrome `v144+`。
2. Chrome 打开 `chrome://inspect/#remote-debugging`，开启 **Allow remote debugging for this browser instance**。
3. 在 `agy` 会话里运行 `/browser <任务和完成标准>`。
4. Chrome 弹窗时在 60 秒内点 **Allow**。
5. 用 `/agents` 或 Alt + J 监控、响应站点权限提示，Ctrl + K 在主对话里批准待处理请求。

#### 观察与接管（App）

- 智能体控制的标签页图标变成光标箭头，页面上有动画光标和状态提示（或 **Thinking…**）。
- 不抢键盘焦点；你正在看某个标签页时，新 subagent 在后台标签页打开。
- 每个并行 subagent 只能操作自己的标签页，不能碰其他标签页。
- 点标签页底部红色 **Stop** 立即接管（**Browser: you took over**）；停止主对话也会停止它的浏览器 subagent。
- 遇到 Google 登录、第三方 OAuth / SSO、MFA、passkey 或 CAPTCHA，智能体会暂停并停在登录页，你在该标签页里登录后回复 `Signed in, continue` 即可继续。

#### 权限与安全护栏

| 项目 | 行为 |
|---|---|
| 站点导航（`execute_url`） | 在 **Request Review**、**Default**、**Auto** 预设下，访问未批准站点会提示 **Allow executing actions on this URL?**，可选仅此次 / 本对话该域名 / 始终允许；跨域重定向每一跳都检查 |
| `localhost` | 按端口隔离：批准 `localhost:3000` 不等于批准 `localhost:8080`；要放开所有端口，在规则里写不带端口的 `localhost` |
| 敏感 Google 子域 | `mail.google.com`、`docs.google.com`、`drive.google.com` 等不被通用 `google.com` 授权覆盖，必须单独写规则 |
| 文件上传 | 只能选为任务暂存的文件（最多 20 个，每个 50 MiB，总计 100 MiB），每次上传都要确认，**Turbo** 下也一样 |
| 文件下载（`download_url`） | `v2.31.0+` 下载会暂停询问；`.exe`、`.dmg`、`.pkg`、`.msi`、`.bat`、`.cmd`、`.ps1`、`.sh`、`.dll`、`.so`、`.dylib`、`.html`、`.svg`、`.xml`、`.wasm` 等直接拦截 |
| **Turbo** | 自动批准一般站点导航和标准 HTTP(S) 下载（Deny 列表和管理员限制除外）；上传、`data:` / `blob:` 下载、登录页、支付页、被拦截文件类型仍受限 |
| 始终开启 | 身份提供方页面（如 `accounts.google.com`、Microsoft、Apple、Okta、Auth0）和支付处理方（如 Stripe、PayPal）禁止智能体操作，相关内容在截图中被遮蔽；内置浏览器不能任意执行页内 JavaScript，网络诊断会去掉 cookie、`Authorization` 头和 OAuth / JWT token |

#### 配置项（Settings > Browser，`v2.30.0+`）

| 项 | 作用 |
|---|---|
| **Open Links In** | 聊天里的链接用 **Built-in Browser**（默认）还是 **External Browser** 打开 |
| **Download Location** | 内置浏览器下载目录，默认 `~/Downloads`，可 **Change…** / **Reset** |
| **Browsing History** | **Clear** 清除访问记录和地址栏联想 |
| **Browser Actuation Rules** | **Edit** 后为 **Execute URLs** 与 **Download files** 添加 **Allow** / **Ask** / **Deny** 规则 |
| **Browser Javascript Execution Policy** | 仅 Chrome DevTools MCP 时显示：**Request Review**（默认）/ **Always Proceed** / **Disabled** |

#### 手动使用内置浏览器

- 点智能体回复里的链接即可打开；按住 Cmd（macOS）或 Ctrl（Windows / Linux）点击或中键点击，在后台标签页打开；再加 Shift 在新的前台标签页打开；按住 Shift（或 Option / Alt）点击则用系统默认浏览器（均为 `v2.30.0+`）。
- 新标签页：Cmd + T / Ctrl + T，或 side panel 的 **+** 然后选 **Browser**。本地 HTTP 服务要写全 `http://`。
- 选中页面文字后右键选 **Comment**（按 C）留下锚定的反馈，或 **Quote**（Cmd + L / Ctrl + L）把选段和页面 URL 插入输入框；工具栏的相机图标 **Add screenshot to conversation** 附上当前视口截图。

#### 常见坑

- 浏览器 subagent 默认跑在 Gemini Flash 模型上；Flash 配额用完时它会停下来问你，不会自动换模型。要指定模型，在提示里说明。
- 登录和支付页智能体不会代填，别指望它输入密码、验证码。
- 看不到 **Browser** 设置页：版本低于 `v2.30.0`，或账号类型不支持。

---

### Projects

**Project** 是一组文件夹加上智能体的作用域：决定智能体能访问哪些目录，以及用哪套设置和权限。一个项目可以含一个或多个文件夹（例如前端和后端两个仓库），每个项目有自己独立的智能体设置。

#### Workspace 与 Project 的区别

| 项 | 旧模型（Workspace） | 新模型（Project） |
|---|---|---|
| 范围 | 绑定单个本地仓库 | 一组文件夹和上下文 |
| 目录边界 | 限制在一个目录结构 | 可同时跨**多个文件夹** |
| 设置 | 从机器全局继承 | 项目有自己的设置 |
| 权限 | 宽泛的全局权限 | 继承全局权限，并可叠加项目级权限 |
| 自定义 | 全局或按工作区管理 Skills / MCP | Skills、MCP、Hooks 在全局管理 |

#### 核心概念

- **文件夹类型**：本地文件夹（没有 Git）和本地 Git checkout。远程 Linux 机器上的文件夹点 **+** 选 **SSH Host…**。
- **Local 与 New worktree**：新对话时用 worktree 选择器决定。**Local** 直接改当前文件夹，适合快速交互；**New worktree** 为对话新建 Git worktree，适合复杂任务，也避免并行 subagent 互相冲突。
- 混合项目里用 New worktree，会给所有 Git checkout 创建新 worktree，同时仍操作非 Git 的本地文件夹。
- 在 **Knowledge Work** 界面布局（`v2.30.0+`）里，新对话直接在项目文件夹里运行，不显示 worktree 选择器。
- 新建项目的权限预设为 **Inherit Global**，对所有项目文件夹有读写权限，可在项目里改。

#### 操作流程

1. 点 **Projects** 旁的 **+** 新建项目，添加文件夹。
2. 点项目旁的齿轮图标，进入项目设置，按需覆盖权限预设。
3. 新对话里选 Local 或 New worktree。
4. 不想归属任何项目时，用 Cmd/Ctrl + Shift + K 选 **Outside of Project**，它在本地 scratch 目录里运行，有自己的设置并继承全局权限。

| 场景 | 做法 |
|---|---|
| 单文件夹 | 建一个含该文件夹的项目并配置设置 |
| 前后端联动 | 把相关文件夹放进同一个项目 |
| 多个智能体共用同一目录 | 选 Local |
| 隔离并发智能体 | 选 New Worktree |

#### CLI 对应

```bash
agy                          # 默认使用 default-cli-project
agy --project=<project_id>   # 在指定项目里开会话
agy --new-project            # 新建项目并进入
```

用 `/resume` 或 `--conversation=<conv_id>` 恢复对话时，CLI 自动使用该对话所属的项目。会话中运行 `/fork <project_id>` 可把当前对话复制到另一个项目继续。

> 边界提醒：默认只能读写项目文件夹。选 **Turbo** 预设后，智能体对整个文件系统有读写权限。

---

### 搜索与命令面板

#### 四个入口

| 类别 | macOS | Windows / Linux | 用途 |
|---|---|---|---|
| **Conversations** | Cmd + K | Ctrl + K | 搜索过去的对话 |
| **Files** | Cmd + P | Ctrl + P | 按文件名 / 路径找文件，在 side panel 打开 |
| **Projects** | Cmd + Shift + K | Ctrl + Shift + K | 切换项目，或选 **Outside of Project** |
| **Commands** | Cmd + Shift + P | Ctrl + Shift + P | 运行 App 命令，在空搜索框输入 `>` 也可切换 |

打开后用 ↑ / ↓ 移动，Enter 打开，Esc 关闭。也可点窗口左上角前进后退按钮旁的 **Search Conversations**（放大镜）。

#### 对话搜索的规则

- 搜索范围：对话标题（含重命名）、项目名、首个文件夹名、你的提示、智能体的回复（含其中代码块）。
- 不包含：智能体的 thinking、artifacts、智能体运行的命令及输出、读取或修改过的文件内容。
- 不区分大小写；只有正在输入的最后一个词按前缀匹配（`pagin` 能找到 pagination），其余词必须整词匹配。
- 多个词必须出现在对话的同一部分（都在标题，或都在你的提示，或都在智能体回复），否则不匹配。
- 精确短语排在前面；标题和项目 / 文件夹名的权重高于正文；最近更新的更靠前。
- 已归档对话不在最近列表里，但会出现在搜索结果中且排序靠后（`2.26.0+` 带归档图标）。
- 排序与过滤（`2.26.0+`）：点搜索框右侧的筛选图标，可选 **Last Updated**（默认）/ **Last Prompt**，以及 **Only Unread** / **Only Archived** / **Only Automations**。排序只作用于空搜索框的列表。

#### 页内查找

Cmd + F / Ctrl + F 打开 **Find** 栏，作用于当前焦点所在的对话，或 side panel 里的文件、artifact、diff。可开 **Match Case (Aa)**、**Match Whole Word (ab)**、**Use Regular Expression (.\*)**；Enter 下一个，Shift + Enter 上一个。

#### 常见坑

- **Files** 只匹配文件名和路径，不搜文件内容；要找文件里的文字，打开文件后用 Cmd/Ctrl + F。
- 想找“智能体当时跑了什么命令”，命令面板搜不到。
- CLI 的 `/resume`（别名 `/switch`、`/conversation`）只按标题、首条提示首行、对话 ID 过滤，不搜全文；要导入 App 里的对话，按 Tab 切到 **Other**，选中后 Enter，再按 Y 导入副本。
- CLI 用 `/codesearch <query>`（别名 `/cs`、`/search`）搜工作区代码，例如 `/codesearch requireApiKey`。

---

### Side panel 与 Overview panel

Side panel 在对话右侧以标签页展示 artifact、文件、diff、终端、subagent 对话和网页；Overview panel 汇总这次对话产出了什么，一键打开。

```text
┌──────────────────────────┬──────────────────────────────┬───────────────┐
│ 对话                      │ Side panel（标签页）            │ Overview      │
│                          │ Walkthrough │ bookmarks.js │ + │ Review        │
│                          │                              │ Files         │
│                          │  artifact / 文件 / diff /      │ Terminal      │
│                          │  终端 / 浏览器 / PDF / .ipynb  │ Plugins       │
│                          │                              │ ── Artifacts  │
└──────────────────────────┴──────────────────────────────┴───────────────┘
```

#### Overview panel

显示 / 隐藏：点右上角 **Toggle Overview Panel**，或 Cmd + Option + O（macOS）/ Ctrl + Alt + O（Windows 和 Linux）。默认打开，并且按对话记住你的选择。窗口太窄时它会隐藏，点按钮会覆盖在对话上，Esc 或点外部关闭。

| 顶部行 | 作用 |
|---|---|
| **Review** | 打开项目 Git 仓库的变更，并显示新增 / 删除行数 |
| **Files** | 打开文件树，点文件打开，或点 **Open File** 搜索 |
| **Terminal** | 新开一个终端标签，每点一次一个 |
| **Plugins** | 列出已安装插件和 MCP Apps 添加的视图 |

| 分区 | 内容 |
|---|---|
| **Subagents** | 智能体启动的 subagent，点击后可跟踪其对话 |
| **Background Tasks** | 后台任务，运行中的可点 **Cancel Task** 停止 |
| **Goals** | 对话的 goals，运行中的排前面 |
| **Documents** | PDF、Office 文件、Google Workspace 链接 |
| **Artifacts** | implementation plan、walkthrough 等 |
| **Links** | 智能体以 artifact 保存的网页，点击在内置浏览器打开 |
| **Modified Files** | 变更文件，点击看 diff（默认隐藏） |
| **Uploads** | 你附加的非 PDF 文件（默认隐藏） |

空分区不显示；每个分区最多显示 4 项，点 **See all** 展开。用 **Customize Sections**（右上三个点）勾选要显示的分区，对所有对话生效。

> 边界提醒：在 **Knowledge Work** 布局（`v2.30.0+`）里，Overview 在对话有产出前保持隐藏，没有 **Review**、**Terminal** 行和 **Customize Sections**，文件树改名为 **Folders**。

#### 标签页与分屏

| 动作 | macOS | Windows / Linux |
|---|---|---|
| 显示 / 隐藏 side panel | Cmd + Shift + B | Ctrl + Shift + B |
| 新标签页 | Cmd + T | Ctrl + T |
| 查找并打开文件 | Cmd + P | Ctrl + P |
| 关闭标签页 | Cmd + W | Ctrl + W |
| 重新打开关闭的标签页 | Cmd + Shift + T | Ctrl + Shift + T |
| 下一个 / 上一个标签页 | Ctrl + Tab / Ctrl + Shift + Tab | 同左 |
| 最大化 / 还原 side panel | Cmd + Shift + M | Ctrl + Shift + M |
| 向右分屏 / 向下分屏 | Cmd + D / Cmd + Shift + D | Ctrl + D / Ctrl + Shift + D |

- 从搜索或文件树打开的文件是预览标签（名称为斜体），下一个文件会替换它；双击标签页可固定。
- 右键标签页可批量关闭；**Review** 和插件添加的标签页还可 **Pin Tab**。
- 分屏：对话顶部 **More actions** > **Split** > **Split Right** / **Split Down**；侧边栏里的对话也能右键分屏；**Replace with New** 在当前窗格新开对话；**Remove from Split**（**X**）关闭窗格。每个窗格有自己的 **Toggle Overview Panel**。
- New Tab 页的 **Search or enter a URL** 同时搜打开的标签页、项目文件、subagent、artifact、后台任务，也可输入网址在内置浏览器打开。

完整快捷键见 **Settings** > **Shortcuts**。

---

### Settings 与 Agent settings

#### 打开设置与设置层级

打开方式：Cmd + ,（macOS，Windows / Linux 为 Ctrl + ,）、点左侧边栏底部 **Settings**、或点项目旁的齿轮。有活动项目时，点 **Settings** 默认打开该项目的设置，否则打开全局设置。

```text
Settings
├─ 全局设置：General（含 Interface Layout、Global Permissions）、Account、Appearance、Browser、Models
├─ 项目设置：Folders、Agent settings（Permission Preset）、项目级权限、Customizations
├─ 独立对话（Standalone）：无项目，本地 scratch 目录，有自己的设置
└─ Miscellaneous：Shortcuts、Feedback
```

插件、MCP、Skills、Rules、Custom agents 的安装与管理在左侧栏 **Customizations**，不在 **Settings**。

#### App 设置项速查

| 项 | 位置 | 取值 / 说明 | 作用 |
|---|---|---|---|
| Interface Layout | General（个人账号，`v2.30.0+`） | **Knowledge Work** / **Software Engineering** | 切换界面布局，智能体和模型不变 |
| Permission Preset（全局） | General > Global Permissions | **Request Review** / **Default**（默认）/ **Auto** / **Turbo** | 所有项目的默认审批策略 |
| Permission Preset（项目） | Settings > Projects | **Inherit Global** / 同上四项 | 项目级覆盖 |
| 消息框权限选择器 | message box（`v2.23.0+`） | 同上；**Use Global Settings** 清除项目覆盖 | 快速切换预设 |
| Allow / Ask / Deny 规则 | Global Permissions / 项目权限 | 规则叠加在预设之上，始终优先 | 细粒度放行或禁止 |
| Enable Telemetry | Account | 开 / 关 | 是否共享交互用于改进模型 |
| Models | Settings > Models | 查看配额、AI credit 超额开关 | 模型与用量 |
| Orchestrator Settings | Models（仅 Google AI Pro / Ultra） | Orchestrator Model、Executor Model | 默认规划与执行模型 |
| Open Links In | Browser（`v2.30.0+`） | **Built-in Browser**（默认）/ **External Browser** | 链接打开方式 |
| Download Location | Browser | 默认 `~/Downloads` | 浏览器下载目录 |
| Browser Actuation Rules | Browser | Allow / Ask / Deny | 浏览器可访问和下载的站点 |
| Enable Remote Control、Device Name | Application | 开关、设备名 | 从浏览器远程监控 / 驱动 App 会话 |
| Voice / Microphone / Volume | General > Voice mode | 音色、麦克风（默认 System default）、音量 | 语音模式设置 |
| Shortcuts | Shortcuts | 自定义快捷键 | 查看和修改快捷键 |

#### 权限预设

| 预设 | 终端命令 | 文件访问 |
|---|---|---|
| **Request Review** | 沙箱关闭，每条命令都要批准 | 读写工作区，其余需批准 |
| **Default**（默认） | 沙箱内免提示，沙箱外需批准 | 读写工作区和临时目录，其余需批准 |
| **Auto** | 沙箱内免提示；沙箱外自动评估，安全且可回退的放行，高风险或不可逆的仍询问 | 读写工作区和临时目录 |
| **Turbo** | 全部免提示，无隔离、无限制 | 对整个文件系统读写 |

要点：

- **Default** 是默认值；终端沙箱不可用时默认变为 **Request Review**。
- **Auto** 在 `v2.23.0+`（macOS / Linux）和 `v2.26.0+`（Windows）可用，且需要沙箱；`2.29.0` 及更早版本显示为 **Auto (Preview)**。商业与企业账号（包括 Antigravity in Gemini Enterprise）不可用。
- 商业与企业账号使用“旧版权限设置”：**Terminal command auto execution**（**Require Review** / **Proceed in Sandbox** / **Always Proceed**）、**Agent non-workspace file access**、**Terminal sandbox mode**。默认只能访问项目文件夹和 `~/.gemini/antigravity/`。

#### 最小操作步骤（App）

1. 按 Cmd/Ctrl + , 打开 **Settings** > **General** > **Global Permissions**，把 **Permission Preset** 保持为 **Default**。
2. 对一个完全信任的项目：**Settings** > **Projects**，改为更宽松的预设；对不信任的目录保持 **Request Review**。
3. 验证：让智能体跑一条会访问沙箱外资源的命令，应看到审批提示；在 message box 的权限选择器里确认当前预设。

#### CLI 的 settings.json

CLI 配置在 `~/.gemini/antigravity-cli/settings.json`，只保存与默认值不同的键（sparse persistence）。用 `/config`（别名 `/settings`）改的值立即保存。启动时若文件有问题，会出现 **Settings Error** 界面，可选 **Exit and fix manually**、**Continue with defaults this session**、**Overwrite the file with defaults**（后者会把旧文件另存为 `settings.json.corrupted.<timestamp>`）。

| 键 | 取值 | 作用 |
|---|---|---|
| `agentMode` | `auto` / `default`（或 `""`）/ `accept-edits` / `plan` | 会话执行模式；也可 `--mode` 或 Shift + Tab 切换 |
| `toolPermission` | `request-review`（默认）/ `proceed-in-sandbox` / `strict` / `always-proceed` | `auto` 之外的工具审批基线 |
| `artifactReviewPolicy` | `asks-for-review`（默认）/ `agent-decides` / `always-proceed` | 智能体就 artifact 求反馈时是否等你 |
| `enableTerminalSandbox` | 布尔，默认 `false`（`auto` 模式自动开启） | 把终端命令限制在 OS 沙箱 |
| `allowNonWorkspaceAccess` | 布尔，默认 `false` | 是否允许读写工作区外的文件 |
| `permissions` | `allow` / `ask` / `deny` 数组，如 `command(git status)` | deny 优先于 ask，ask 优先于 allow |
| `altScreenMode` | `default` / `always` / `never`（仅 JSON） | TUI 渲染方式；SSH、`tmux` 建议 inline |
| `colorScheme` | `terminal`（默认）/ `light` / `dark` / `solarized light` / `solarized dark` / `tokyo night` 等 | 配色 |
| `runningLightSpeed` | `fast` / `medium`（默认）/ `slow` / `off` | 进度动画速度 |
| `verbosity` | `high` / `medium`（默认）/ `low` | 对话中展示多少工具活动 |
| `hideActiveItems` | 布尔 | 控制是否在提示框下显示活动 subagent 和后台任务 |
| `copyOnSelect` | 布尔，默认开 | alt-screen 下松开鼠标即复制 |
| `editor` | `auto`（默认）/ `vim` / `emacs` / 自定义命令 | 外部编辑器（Ctrl + G 编写提示） |
| `editorMode` | `default` / `vim` | 提示框内的编辑模型 |
| `vimInsertFirst` | 布尔，默认关 | Vim 模式下直接进入 Insert |
| `notifications` | 布尔，默认关 | 需要批准或回合结束时响铃并弹桌面通知 |
| `queuedMessages` | `queue`（默认）/ `send-immediately` | 智能体工作时发送的消息排队还是立即打断；Alt + M 单次反向 |
| `pickerGrouping` | `none`（默认）/ `grouped` | `/resume` 列表是否按工作区分组 |
| `useG1Credits` | 布尔，默认关 | 套餐额度用完后是否使用 AI credits（商业账号不显示） |
| `showTips` / `showFeedbackSurvey` | 布尔，默认开 | 使用提示 / 完成任务后的简短问卷 |
| `historySize` | 整数，默认 `2000` | 磁盘上保留的提示历史条数；`-1` 全部保留，`0` 清空（仅 JSON） |
| `model` / `modelProvider` | 字符串（仅 JSON） | `model` 为 `/model` 选择的模型；`modelProvider` 设为 `"gemini"` 并配合 `GEMINI_API_KEY` 直连 Gemini API |
| `trustedWorkspaces` | 字符串数组（仅 JSON） | 你选择信任的文件夹，由 CLI 自动添加 |
| `statusLine` / `title` | 对象：`command`、`enabled`、`padding` 等 | 自定义状态栏和终端标题脚本 |
| `sandboxProxy` | 对象：`http.address` + `certFile` | 沙箱命令的网络流量走自己的 HTTP 代理 |

可照抄的最小配置（CLI，自动批准工作区编辑，命令仍按 `toolPermission` 询问）：

```json
{
  "agentMode": "accept-edits",
  "toolPermission": "request-review",
  "permissions": {
    "allow": ["command(git status)", "read_url(github.com)"],
    "ask": ["command(git push)"],
    "deny": ["write_file(.env)"]
  }
}
```

要让编辑器对该文件做自动补全和校验，在文件顶部加 `$schema` 键（值为官方发布的 settings JSON Schema 地址，见官方 Settings 页）；CLI 会忽略并保留它。

#### 其他 CLI 配置文件

| 内容 | 路径 |
|---|---|
| 快捷键 | `~/.gemini/antigravity-cli/keybindings.json`（`/keybindings` 打开） |
| Hooks | `~/.gemini/config/hooks.json`（全局）或 `<workspace>/.agents/hooks.json` |
| MCP 服务器 | `~/.gemini/config/mcp_config.json` 或 `<workspace>/.agents/mcp_config.json` |
| 插件 | `~/.gemini/config/plugins/<plugin>/` 或 `<workspace>/.agents/plugins/<plugin>/` |

快捷键文件把动作映射到按键数组，映射为 `[]` 即禁用默认键；`cli.exit`（Ctrl + D）、`cli.escape`（Esc / Ctrl + C）、`cli.enter`（Enter）受保护，不能禁用：

```json
{
  "cli.clear_screen": ["ctrl+l"],
  "prompt.insert_newline": ["shift+enter", "ctrl+j"],
  "edit.open_editor": ["ctrl+g"]
}
```

恢复默认快捷键：删除 `keybindings.json`。

#### 如何验证

- App：设置 **Auto** 后，到消息框的权限选择器里确认；Settings 里看不到 **Auto** 通常是版本、系统、沙箱可用性或账号类型不满足。
- CLI：`agentMode` 为 `auto` 时，右下角状态栏显示 `auto`，提示框显示 `Auto mode: edits & safe commands auto-approved (shift+tab to cycle)`。
- CLI：`agy --sandbox --model="Gemini 3.8 Flash"` 这类命令行参数只对当前会话生效，`/config` 里会显示 `on (overridden by --sandbox)`，不会写进 `settings.json`。

#### 常见坑

- 在 App 里 Settings 默认打开的是项目设置（有活动项目时）；想改全局设置，先退出项目上下文再开。
- `hooks` 不能写进 `settings.json`：CLI 保留这个键但从不加载，要写在 `hooks.json`。
- 全局与工作区 MCP 同名时，已经运行的那个服务器保留名字，所以工作区的服务器要用不同名称。
- `statusLine` 和 `title` 不要写成 `true` / `false`；要关闭脚本请在对象里设 `"enabled": false`。
- 如果终端沙箱在你的机器上不可用（如 Windows 10、早期 Windows 11 构建、部分 Linux 容器），Sandbox Mode 无效，`proceed-in-sandbox` 和 `auto` 不会提供。
- `auto` 在 CLI 中 `v1.3.5+`（macOS / Linux）和 `v1.4.4+`（Windows）可用，商业与企业账号除外；`v1.5.7` 起为默认模式。

### 权限规则：Deny / Ask / Allow

适用：App、CLI、IDE 扩展（VS Code、Visual Studio、Eclipse）、IDE。每个敏感操作都表示成**权限资源** `action(target)`，由三张名单判断，冲突时固定 **Deny > Ask > Allow**。例如 Ask 里有 `command(*)`、Allow 里有 `command(git)`，每条命令仍会先问你。

```text
智能体发起敏感操作 → action(target)
   │
   ▼
Deny 命中？ ─ 是 ─► 阻止
   │ 否
Ask 命中？ ── 是 ─► 弹审批卡片
   │ 否
Allow 命中？ ─ 是 ─► 直接执行
   │ 否
权限预设（App）/ 执行模式 + toolPermission（CLI）的默认行为
```

你写的规则叠在预设之上，始终优先于预设。

#### 支持的 action

| Action | target 写法 | 默认回退 |
|---|---|---|
| `read_file` | `read_file(/path)`、`read_file(dir)`、`read_file(*)` | Ask（工作区内允许） |
| `write_file` | `write_file(/path)`、`write_file(*)` | Ask（工作区内允许） |
| `read_url` | `read_url(domain)`、`read_url(*)` | Ask |
| `execute_url` | `execute_url(domain)`、`execute_url(*)` | Ask |
| `download_url` | `download_url(domain)`、`download_url(*)` | Ask |
| `command` | `command(prefix)`、`command(regex:pattern)`、`command(*)` | Ask（Default / Auto 预设下沙箱内允许） |
| `mcp` | `mcp(server/tool)`、`mcp(server/*)`、`mcp(*)` | Ask |

文档的 CLI 表格没有列 `download_url`。另有仅限商业/企业账号的 `unsandboxed`（见下文）。匹配规则：

- `read_file` / `write_file`：绝对路径或相对工作区根的路径，目录递归生效；`read_file(*)` 匹配系统上所有文件。
- `read_url`：匹配主机名及子域（`google.com` 覆盖 `mail.google.com`），忽略 URL 路径。
- `execute_url`：管浏览器子智能体打开或跳转到某域名。`localhost` 按 `:port` 区分；`mail.google.com`、`docs.google.com`、`drive.google.com`、`console.cloud.google.com` 需要显式规则。**`read_url` 的授权不覆盖它。**
- `download_url`：管 App 内置浏览器智能体下载文件，匹配同 `execute_url`；`data:`、`blob:` 下载总是询问一次，可执行或带活动标记的文件类型被阻止（Browser Actuation Rules 需 `v2.30.0+`，交互式下载提示需 `v2.31.0+`）。
- `command`：默认按“单词/token 前缀”字面匹配；以 `regex:` 开头则每个 token 按锚定正则 `^(?:pattern)$` 求值，如 `command(regex:npm run (build.*))`。沙箱内外都管。
- `mcp`：具体工具或某服务器全部工具，本地和远程服务器都适用。
- 隐含规则：允许 `write_file` 即同时获得该路径的 `read_file`；拒绝 `read_file` 即同时阻止该路径的 `write_file`。

#### 必须完整匹配的命令

命令/进程替换（`$(...)`、反引号、`<(...)`）、算术上下文、花括号展开、非字面命令名、网络或文件描述符重定向、会执行子命令的参数（如 `git -c core.pager=<cmd>`、`tar --to-command`），或无法干净解析时，整行**不再做前缀匹配**：只有规则与整行逐字符一致才免提示，否则回到 Ask。管道、`&&` / `||` / `;`、带引号字面量、普通 `$VAR`、简单文件重定向、透明包装器（`timeout`、`nohup`、`nice`、`env`）仍按前缀匹配。

```text
Allow 里有 command(git) 时：
  git status && git log   → 免提示
  git log $(whoami)       → 要求审批
```

Windows 的 PowerShell / Command Prompt 里无法拆成单词的命令也要完整匹配；匹配命令及其子命令用 `regex:`，如 `command(regex:git .*)`。路径在 Windows 上先去掉盘符、反斜杠转成 `/` 再求值。

#### App：权限预设

| 预设 | 终端沙箱 | 终端命令 | 文件访问 | MCP 与网页 |
|---|---|---|---|---|
| **Request Review** | 关 | 总是询问 | 仅工作区 | 询问 |
| **Default**（默认） | 开 | 沙箱内允许，沙箱外询问 | 工作区 + 临时目录 | 询问 |
| **Auto** | 开 | 沙箱内允许，沙箱外自动判断 | 工作区 + 临时目录 | 询问 |
| **Turbo** | 关 | 允许（不受限） | 整个文件系统 | 允许 |

- **Auto**：沙箱外的命令由 Antigravity 自动评估，安全、可逆的免提示，有风险或不可逆的仍要批准。
- 限制：沙箱不可用时默认值回退为 Request Review；**Auto** 在 macOS、Linux 需 `v2.23.0+`，Windows 需 `v2.26.0+`，且要求沙箱可用（`v2.29.0` 及更早显示为 **Auto (Preview)**）；**商业和企业账号（含 Gemini Enterprise）不可用**。

切换位置：

| 范围 | 位置 |
|---|---|
| 所有项目 | **Settings** > **General** > **Global Permissions** > **Permission Preset**（有远程智能体权限时是 **Permissions** > **This Computer**） |
| 单个项目 | **Settings** > **Projects**；新项目默认 **Inherit Global** |
| 消息框 | `v2.23.0+` 的权限选择器，改全局预设（项目有覆盖则改项目）；取消项目覆盖选 **Use Global Settings** |

#### CLI：执行模式与 toolPermission

CLI 规则写在 `~/.gemini/antigravity-cli/settings.json` 的 `permissions.allow / deny / ask`。规则没覆盖的由**执行模式**（`--mode` 或 `agentMode`）和 `toolPermission` 决定：

| 模式 | 行为 |
|---|---|
| `auto` | 自动批准工作区文件编辑；命令默认在沙箱内跑，沙箱外命令自动评估 |
| `accept-edits` | 自动批准工作区文件编辑与创建；shell 命令仍按 `toolPermission` 与规则评估 |
| `default`（`/config` 选 `default`，或 `agentMode` 设为 `""`） | 改文件前交互式 diff 审阅；命令按 `toolPermission`（默认 `request-review`） |

- `strict` 与 `always-proceed`（即 `--dangerously-skip-permissions`）优先于 `auto`。`auto` 之外，`proceed-in-sandbox` 让沙箱内命令免提示、沙箱外先问；`v1.4.4` 起沙箱不可用时它等同 `request-review`。
- `auto` 需 `v1.3.5+`（macOS、Linux）或 `v1.4.4+`（Windows），且沙箱可用；**商业和企业账号不可用**。`v1.5.7` 起，`auto` 可用时 CLI 默认以 `auto` 启动，否则以 `default` 启动。Shift + Tab 切换模式。

#### 配置示例（CLI）

```json
{
  "permissions": {
    "allow": [
      "command(git)",
      "command(regex:npm run (build|lint|test))",
      "read_file(/var/log/app)",
      "write_file(src/)",
      "read_url(google.com)",
      "mcp(linter/*)"
    ],
    "deny": [
      "command(rm -rf)",
      "command(regex:curl .*)",
      "command(sudo)",
      "write_file(.git/)",
      "write_file(/home/user/.ssh)"
    ],
    "ask": ["command(*)", "execute_url(aws.amazon.com)", "mcp(sql/execute_mutation)"]
  }
}
```

注意：这份官方示例里 `ask` 的 `command(*)` 会压过 `allow` 里所有 `command(...)`，实际使用时先去掉它。`/permissions` 在 TUI 里交互管理规则。

#### 验证与审批卡片

1. 让智能体执行一条被 deny 的命令（如 `sudo`），应被直接阻止。
2. 让它执行 Ask 里的命令，应弹出审批卡片。
3. 出现 Ask 时，点 **Allow** 之前可以直接编辑文件、URL、MCP 的 target 来放宽范围（如把 `/project/file.txt` 改成 `/project`），该扩大授权在本轮剩余时间有效。**终端命令不支持范围编辑。**

规则没写时的默认：工作区内读写免提示，工作区外要批准；网页（`read_url`、`execute_url`）和未配置的 MCP 默认 Ask。`read_file` 授权的路径进沙箱只读白名单，`write_file` 进读写白名单，`read_url` 域名进沙箱出站白名单。

#### 商业与企业账号

App 使用文档标为 **Previous permission settings** 的另一套：没有上面的预设，而是 **Security Preset**（见沙箱一节）。多一个 `unsandboxed(prefix)` / `unsandboxed(regex:pattern)` / `unsandboxed(*)`：匹配的命令在沙箱启用时于容器隔离之外执行。CLI 里 `unsandboxed` **只对商业和企业账号有效**，其他账号会忽略并警告，应改用 `command`。

#### 常见坑

- Ask 与 Allow 冲突时 Ask 赢，不是“更具体者赢”。
- 允许了 `read_url(x.com)` 不代表浏览器子智能体能打开它，需另写 `execute_url`。
- `command(git)` 放行不了带 `$(...)` 的 git 命令，它需要整行逐字符匹配。
- Turbo / `always-proceed` 不受限，只在完全信任环境时使用。

---

### 终端沙箱

适用：App、CLI、IDE 扩展（VS Code、Eclipse）、IDE。终端沙箱把智能体的 shell 命令隔离在操作系统级容器边界里，基于原生机制，无需虚拟机或 Docker 镜像，没有启动延迟。

- 沙箱内可以：写项目文件夹、临时目录和常见构建缓存；读 `/usr`、`/etc` 等系统目录。
- 沙箱会挡住：`~/.ssh`、`.env` 等敏感文件；没被显式挂载的内容（macOS、Linux 上不可见）；网络，只放行你批准的域名。

| 系统 | 技术 | 说明 |
|---|---|---|
| Linux | Namespaces | 内核命名空间隔离文件系统、隐藏宿主进程、切断网络 |
| macOS | `sandbox-exec` | Seatbelt（SBPL）限制文件系统访问和 socket 连接 |
| Windows（文档只在 CLI 部分列出） | 进程安全环境 | 命令在 AppContainer 中运行；**需 Windows 11 24H2 或更高并装有 2026 年 8 月更新或更高**；用户配置目录之外的文件夹可读不可写 |

#### App：启用

沙箱由权限预设控制：**Default**、**Auto** 开启，**Request Review**、**Turbo** 关闭（对比表见上一节）。操作：

1. **Settings** > **General** > **Global Permissions** > **Permission Preset**，选 **Default** 或 **Auto**；
2. 只想改某项目：**Settings** > **Projects**；
3. 让智能体跑一次构建或测试，确认命令在沙箱内执行。

商业和企业账号使用旧版设置：**Settings** > **General** > **Agent Settings** 里有 **Enable Sandbox Mode (Preview)** 开关和 **Terminal Command Auto Execution**（**Require Review** / **Proceed in Sandbox** / **Always Proceed**）；项目里可选 **Inherit Global** / **Enabled** / **Disabled**。**Security Preset** 打包终端与文件策略：

| 预设 | Terminal Command Auto Execution | 文件夹外文件访问 |
|---|---|---|
| **Default** | Require Review | Always Ask |
| **Full machine** | Require Review | Allow |
| **Turbo mode** | Always Proceed | Allow |

三个预设都不开沙箱；打开 **Enable Sandbox Mode** 后预设变为 **Custom**，常见组合是沙箱 + **Proceed in Sandbox**。

#### CLI：启用

`auto` 模式自动启用沙箱。其他模式在 `~/.gemini/antigravity-cli/settings.json` 里设置，或用 `/config`：

```json
{
  "enableTerminalSandbox": true,
  "toolPermission": "proceed-in-sandbox"
}
```

| 设置键 | 类型 / 默认 | 作用 |
|---|---|---|
| `agentMode` | string；`v1.5.7+` 可用时默认 `"auto"`，否则 `""` | `"auto"` 自动启用沙箱并自动评估沙箱外命令 |
| `enableTerminalSandbox` | boolean，默认 `false` | 让命令在沙箱内运行；`auto` 自动打开 |
| `toolPermission` | string，默认 `"request-review"` | `auto` 之外设 `"proceed-in-sandbox"`：沙箱内自动运行，沙箱外仍提示 |

```bash
agy --sandbox        # 本次会话强制开启沙箱，覆盖 settings.json
agy --mode=auto      # auto 模式：开启沙箱并自动评估沙箱外命令
```

#### 沙箱不可用时

从 `v1.4.4` 起，若沙箱不可用（如 Windows 10、较早的 Windows 11、阻止非特权用户命名空间的 Linux 主机和部分容器）：`enableTerminalSandbox` 和 `--sandbox` 无效；`proceed-in-sandbox` 等同 `request-review`；不提供 `auto`；`/config` 隐藏 **Sandbox Mode** 等选项。已保存的设置不变，沙箱可用后生效。

#### 边界如何从权限推导

| 范围 | 规则 |
|---|---|
| 文件系统 | 项目/工作区文件夹读写；`read_file` 允许的路径只读，`write_file` 允许的读写，叠在默认系统挂载上；其余不可访问（CLI 里被 deny 的路径阻止；Windows 上用户配置目录外仍可读） |
| 网络 | 默认无网络；`read_url` 允许的域名进出站白名单 |

#### 沙箱外运行

需联网或访问系统服务的命令可申请在沙箱外运行，此时在宿主机上以**完整权限**执行：

| 场景 | 处理 |
|---|---|
| App Default | 总是暂停等批准，除非已被 `command` allow / deny 规则覆盖 |
| App Auto / CLI `auto` | 自动评估：安全、可逆的免提示，其余仍确认 |
| CLI 其他模式 | 总要批准，除非命中 `command` allow（商业/企业账号是 `unsandboxed`） |

免提示放行某命令（沙箱内外皆可）：

```text
command(git push)
command(regex:docker compose .*)
```

CLI 的审批提示有四项：Yes / “本会话内对以该命令开头的命令总是允许” / “总是允许（写入 settings.json）” / No。商业和企业账号请求绕过沙箱时提示会明确警告风险，选“总是允许”记录的是 `unsandboxed(...)`，其他账号记录 `command(...)`。

#### 局限

- 沙箱外的命令拥有宿主机完整权限，批准前看清命令。
- 沙箱只包住终端命令；文件、网页、MCP 仍由权限规则管。
- 不满足系统要求时不会报错，而是退回无沙箱的审批流程，别默认一定在沙箱里。文档没有给出官方的沙箱验证命令。

---

### Artifacts：智能体的交付物

适用：App、CLI、IDE 扩展、IDE。**Artifact** 是智能体为完成任务、向你说明进度和思路而生成的结构化交付物：Markdown 计划（implementation plan）、代码 diff、架构图、图片、浏览器录屏。任务越长，你越不必盯每个工具调用，只在关键节点审阅。

```text
Planning mode
   ├─ Implementation plan ─► 你审阅 / 评论 ─► Proceed
   │                                           ▼
   │                                        实施修改
   │                                           ▼
   └──────────────────────────── Walkthrough（总结 + 验证结果 + 截图）
                                               ▼
                                   Review changes（逐文件看 diff）
```

Artifact 主要在 **Planning mode** 下生成。App 有可视化侧边栏和审阅面板：检查交互式计划、看可视化 diff、回放浏览器录屏，也能在侧边栏使用智能体构建的交互式 HTML 控件（如用 `/generative_ui`）。CLI 是键盘驱动的审阅面板：智能体生成或修改的文件需要批准时，终端状态栏出现通知。计划与目标不符时，用行内文字反馈在它改本地文件前纠正。

#### Planning mode 与 Fast mode

| 模式 | 行为 | 适合 |
|---|---|---|
| **Planning mode** | 先规划：按任务组组织工作、产出 implementation plan、深入研究代码库 | 复杂任务 |
| **Fast mode** | 直接执行，无规划阶段 | 变量重命名、跑一条 Bash、小重构 |

#### Implementation plan

适用：App、IDE 扩展、IDE。智能体用它设计要做的修改。用 `/plan` 可随时显式触发规划。

- 不论审阅策略如何，智能体都会在改代码前请求你审阅计划。点卡片上的 **Proceed** 继续。
- 在 artifact 上选中文字写评论（缩小范围、换技术栈、纠错）。评论附在下一条消息上，消息框上方出现 **2 Comments** 这样的 chip，悬停可预览。可带评论点 **Proceed**，也可改发消息反馈。
- 提交后，智能体要么迭代计划再请你审阅，要么开始实施。

#### Artifact review policy

适用：App、IDE 扩展、IDE（该页未列 CLI）。在 **Settings** > **General** 的 **Plan Review Policy** 选择：

| 设置名 | 行为 |
|---|---|
| **Always Ask**（Request review，推荐） | 生成计划或 diff 后暂停并通知，你可审阅、加行内评论、在工作区验证，满意后批准 |
| **Always Proceed** | **不会**让智能体越过计划：用 `/plan` 要计划时总会停下等批准 |

用 **Always Proceed** 时若要它写完计划继续，需在提示词里说明；计划卡片可能没有 **Proceed** 按钮，此时在消息框回复，如 `proceed`。

#### Walkthrough

智能体完成实现后生成：对修改的简明总结，示例含 **Changes Made** 和 **Validation Results**（含测试输出），方便你没实时跟进时了解现状。浏览器任务可要求它嵌入截图；用 `/plan` 规划时，智能体会被要求在 walkthrough 里包含截图，有录屏时也包含。

#### Screenshots

适用：App、IDE 扩展、IDE。浏览器子智能体可截取整页可见区或单个元素：在提示里要求，或由它按任务需要自行截取。App 内置浏览器工具栏的 **Add screenshot to conversation**（相机图标）可把当前页截图附到提示上。App `v2.30.0+`（个人 Google 账号）里可直接用自然语言或 `/browser` 加任务，在内置浏览器里运行；商业/企业账号（和 CLI）的 `/browser` 连接已开启远程调试的 Google Chrome。

**截图本身不是 artifact。** 要评论它：先让智能体把它保存到会话 artifacts，从回复里打开图片，拖选区域、写评论、点 **Add Comment**。评论作为待发评论，随下一条消息发送，或从 artifact 标题栏 **Review** 提交。

---

### Review changes：审阅智能体的修改

适用：App、CLI。保留修改前先检查：逐文件看 diff、按行反馈，App 里还能直接提交到 Git。

#### App：Review 面板

面板显示项目 Git 仓库的变更（含智能体的编辑）。打开：在 **Overview** 面板点 **Review**（Cmd + Option + O / Ctrl + Alt + O 切换面板），或点智能体回复下方汇总卡片（如 **2 files changed**）的 **Review**，此时面板只显示那一轮。**Maximize Pane**（Cmd + Shift + M / Ctrl + Shift + M）展开，**Toggle sidebar** 列出文件。

文件状态字母：**M** Modified、**A** Added（已暂存新文件）、**U** Untracked、**D** Deleted、**R** Renamed。

用 **Review** 旁的菜单选比较范围：

| 选项 | 显示 |
|---|---|
| **Uncommitted**（默认） | 已暂存和未暂存变更，分在 **Staged Changes** 与 **Changes** |
| **Branch** | 当前分支自与基础分支（如 `origin/main`）分叉以来的全部变更；找不到基础分支时不出现 |
| **Agent Edits** | 本会话智能体改过的文件，不论是否提交 |

只有 **Uncommitted** 视图能暂存、取消暂存和丢弃。看单轮：点该轮卡片的 **Review**，面板头显示 **For Turn**，点 **Clear filter** 返回。diff 显示可用 **⋮** 切 **View Split Diff** / **View Stacked Diff**、**Hide Whitespace Changes**、**Collapse All**，另有 **Search**。

#### 按行反馈

1. 悬停行首点 **Add inline comment**（**+**）；多行则选中后点 **Comment**。
2. 输入反馈，点 **Add Comment**。

评论附在下一条消息上（chip 如 **1 Comment**），发送后智能体收到每条评论及其文件和行号。只引用不评论：选中后点 **Quote** 或按 Cmd + L / Ctrl + L，消息框里加入如 `bookmarks.js #L13-16` 的引用。

#### 暂存、丢弃与提交

悬停文件可用 **Stage change**（+）、**Unstage change**（−）、**Discard unstaged changes**。

> **Discard unstaged changes 立即生效、不会确认，并会从磁盘删除未跟踪文件**；已暂存的更改保留。

提交流程：

1. 点 **Commit**，对话框显示文件数和目标分支；
2. 有暂存内容时只提交暂存的，要一起提交未暂存的勾选 **Include unstaged changes**（没暂存内容时已默认勾选）；
3. 写提交信息，留空则由智能体根据对话和文件生成；
4. 点 **Commit** 或按 Enter（换行 Shift + Enter）。

箭头菜单里：**Amend** 并入最近一次提交；若已推送要自己强推，如 `git push --force-with-lease`，面板里的 **Push** 不会强推。**Push** 推送到上游，没有上游则发布到 `origin`。无可提交内容但有待推送提交时按钮变成 **Push**。

#### CLI：`/diff`

输入 `/diff` 回车打开全屏查看器（默认 **All Changes**），选文件 Enter 打开，Esc 返回，←/→ 切换视图：**All Changes**（未提交变更）、**Per Turn**（每轮变更）、**Commit Tree**（提交历史，Enter 列出该提交改的文件）。行评论：打开文件 diff，光标移到某行按 C，输入后回车。退出时有未发送评论会出现 **Unsent Comments**：按 Y 作为下一条提示发给智能体，按 N 不发送退出。

---

### Jupyter notebooks

适用：App、CLI、IDE。App 里 `.ipynb` 在侧边栏原生查看器中打开：点对话里的 `.ipynb` 链接或 `@` 提及 chip；点侧边栏头部 **+** 选 **Open File**；或在 **Overview** 面板的文件列表里选。支持标准 `nbformat` v4 和旧的 v3（`worksheets`）。

| 元素 | 渲染 |
|---|---|
| Markdown 单元 | 标题、列表、链接、行内代码、表格，图片相对 notebook 目录解析 |
| 代码单元 | 执行计数（`[1]`，未执行为 `[ ]`）+ 按 notebook 语言元数据（默认 Python）高亮 |
| 输出 | `image/png`、`image/jpeg`、`image/gif`，`stdout`、`stderr`、`text/plain`，错误以突出样式显示 |

- **Preview / Raw**：侧边栏头部切换渲染视图与 JSON；JSON 损坏或结构无效时自动回退 **Raw** 并禁用 **Preview**。
- 单元评论：悬停单元或选中行，点 **Add inline comment**（**+**），写完点 **Add Comment**；评论锚定到 `.ipynb` 对应行，随下一条消息发送。
- diff：在 **Review** 面板或对改过的 `.ipynb` 开 **Show Diff** 时，先去掉单元输出再算会话轮次 diff。
- 让智能体操作：用 `@analysis.ipynb` 引用（CLI 写路径如 `@notebooks/analysis.ipynb`）；让它增改单元或生成新 notebook；让它用 `jupyter nbconvert --to notebook --execute --inplace <notebook.ipynb>` 或 `papermill` 运行并汇总输出或错误。
- CLI：同样可引用、创建、编辑、运行，用 `/diff` 看变更（同样去掉输出）。
- IDE：在编辑器的交互式 notebook 界面打开，可编辑单元、选本地或远程 Jupyter kernel、交互运行，并在侧边栏智能体面板里引用当前 notebook。

---

### Customization discovery：自定义项的发现与优先级

适用：App、CLI、IDE 扩展、IDE。Antigravity 从项目和主目录的标准文件夹加载 rules、skills、custom agents、plugins、hooks、MCP servers。

| 范围 | 位置 |
|---|---|
| Workspace | 项目的 `.agents/`，加 `AGENTS.md`、`GEMINI.md`；提交后可共享 |
| Personal | `~/.gemini/config/`，加 `~/.gemini/AGENTS.md`、`~/.gemini/GEMINI.md`；对所有项目生效，App 和 CLI 都适用 |

两个范围布局相同，下列路径相对 `.agents/` 或 `~/.gemini/config/`：

| 自定义项 | 路径 |
|---|---|
| Rules | `rules/*.md`、`AGENTS.md`、`GEMINI.md` |
| Skills | `skills/<name>/SKILL.md` |
| Custom agents | `agents/<name>.md` 或 `agents/<name>/agent.md` |
| Plugins | `plugins/<plugin-folder>/` |
| Hooks | `hooks.json` |
| MCP servers | `mcp_config.json` |

插件启用时可自带 skills、rules、agents、hooks、MCP servers。CLI 还加载 `~/.gemini/antigravity-cli/skills/` 和 `~/.gemini/skills/`，App 不读。也识别 `.agent/`、`_agents/`、`_agent/`，新项目用 `.agents/`。

#### 向上找，不向下找；只扫一层

Antigravity 检查工作区文件夹及其各级父文件夹，直到仓库根（含 `.git`），每层读 `AGENTS.md`、`GEMINI.md`、`.agents/`。CLI 的工作区是启动 `agy` 的文件夹加 `--add-dir` / `/add-dir` 添加的；App 是项目里添加的每个文件夹。到仓库根为止（不在仓库里则只查工作区文件夹）；**不会**搜索工作区下面的子文件夹（打开仓库根时不会加载 `src/routes/AGENTS.md`）。每个自定义项文件夹只扫一层：

```text
bookmarks-api/
├── AGENTS.md
├── .agents/
│   ├── rules/
│   │   ├── api-conventions.md        ✓ 加载
│   │   └── security/api-keys.md      ✗ 嵌套太深
│   ├── skills/
│   │   ├── add-endpoint/SKILL.md     ✓ 加载
│   │   └── database/migrate-schema/  ✗ 嵌套太深
│   └── skills.json
└── tools/agent-skills/               （标准文件夹之外，需 manifest）
```

要加载嵌套项：上移一层，或在 discovery manifest 里点名。

#### Discovery manifest

manifest 是 JSON，列出标准文件夹之外或嵌套过深的自定义项，是**补充**不是替换。四种：`skills.json`、`rules.json`、`agents.json`、`plugins.json`，放在项目 `.agents/` 或 `~/.gemini/config/`；工作区到仓库根之间每个 `.agents/` 里的都会读。

```json
{
  "entries": [
    { "path": "tools/agent-skills", "exclude": ["load-test"] },
    { "path": ".agents/skills", "include_only": ["database/migrate-schema"] }
  ]
}
```

第一项加载该文件夹里除 `load-test` 外的 skill；第二项补上被一层扫描跳过的嵌套 skill。`rules.json` 用法相同，其中列出的 rule 保留 frontmatter，`trigger` 仍生效。

| 字段 | 位置 | 说明 |
|---|---|---|
| `entries` | 顶层 | 要扫描的文件夹，或要加载的单个自定义项 |
| `inherits` | 顶层 | 要包含的同类型其他 manifest |
| `exclude` | 顶层 | 隐藏的项，所有位置（含个人文件夹）都跳过 |
| `path` | `entries` / `inherits` 内，必填 | `entries` 里是扫一层的文件夹或单个项；`inherits` 里是另一份 manifest 路径 |
| `include_only` / `exclude` | `entries` / `inherits` 内 | 只加载 / 跳过匹配项，可写嵌套项如 `database/migrate-schema` |

- 项目 manifest 的相对路径从包含 `.agents/` 的文件夹算起，不存在则再从仓库根试；个人 manifest 的 `entries` 相对路径从 `~/.gemini/config/` 算起，`inherits` 请用绝对路径。
- **不展开 `~`**，主目录路径要写成绝对路径。**不支持通配符**，`load-*` 匹配不到 `load-test`。匹配方式：文件夹名、文件名（可带或不带 `.md`）、嵌套相对路径。
- 跨项目共享：在项目的 `.agents/skills.json` 里用 `inherits` 指向共享检出里的 manifest（可配 `include_only` / `exclude`）；或在个人 `~/.gemini/config/skills.json` 里用 `entries` 的绝对路径列出共享 skills 目录。

共享仓库里每人一个文件夹时，在仓库根加 `.gemini/config.json`：

```json
{
  "personal_customization_dir": "people/{user}"
}
```

`{user}` 取环境变量 `USER`（Windows 为 `USERNAME`），未设置则用系统账户名；读取其中的 `.agents/`。

#### 同名时谁优先

skill 或 custom agent 同名时只保留一个，优先级从高到低：

1. 仓库内你的个人文件夹（仓库设置了 `personal_customization_dir` 时）；
2. 工作区（含其 manifest）；
3. 个人 `~/.gemini/config/`（含个人 manifest）；
4. 内置 skills 和 agents。

CLI 的 `~/.gemini/antigravity-cli/skills/` 与 `~/.gemini/skills/` 优先于以上全部，manifest 的 `exclude` 也隐藏不了它们。**Rules 不互相替换，找到的全部加载。** 插件里的 skill 命名为 `<plugin>:<skill>`，不冲突。

#### 检查与生效

- App：**Settings** > **Projects** 选项目（没列出点 **Show All**）> 滚到 **Customizations** 区，看 token 用量和 **Skills**、**Rules**、**Plugins**、**Custom Agents** 列表；左侧栏 **Customizations** > **Installed** 里悬停 **Global** 徽标可看来源（**Installed in** / **Included by**）。
- CLI：`/skills`（按来源分组，Enter 看描述和路径）、`/agents`、`/hooks`、`/mcp`、`/plugin`；增删改 skill 后用 `/skills reload` 或开新对话刷新列表。
- rules 和 skills 在每轮前重新读取，新文件在下一条消息就生效，无需重启。

没加载时依次查：位置与深度；文件名（skill 用 `SKILL.md`，rules 和 agents 以 `.md` 结尾，MCP 用 `mcp_config.json`，插件文件夹或其 `.antigravity-plugin/`、`.gemini-plugin/` 里有 `plugin.json`）；`SKILL.md` 和每个 rule 的 YAML frontmatter（rule 要有有效 `trigger`）；manifest 的 JSON、路径和 `exclude`；是否被同名高优先级项覆盖。

---

### Remote Agents：Google 托管的远程环境

适用：App、CLI、网页。Remote Agents 把智能体会话和工具放在隔离的、Google 托管的远程环境里运行，合上浏览器、笔记本睡眠、桌面与手机切换，长任务仍继续。

> **套餐限制**：仅面向用个人 Google 账号登录的 **Google AI Pro** 和 **Google AI Ultra** 订阅用户，**正在逐步推出**；免费版、Google AI Plus、商业账号（Workspace、Google Cloud）不可用。

#### 启动

App：

1. 左侧栏 **Projects** 旁点 **+**（或在消息框打开项目选择器），选 **New Remote Project**；
2. （可选）**Add Resource** 附加 GitHub 仓库、Drive 文件或上传文件，点 **Next**（或 **Skip**）；
3. 在 **Set Project Name** 填名称，点 **Create**；
4. 选中远程项目，发送提示开始远程对话。

也可在消息框项目选择器选 **Remote** > **GitHub Repositories**（未连接则点 **Connect GitHub**），选仓库后自动选用或新建该仓库的远程项目。网页：用符合套餐的个人 Google 账号登录官方网页入口，直接发提示，或点 **Projects** 旁 **Create Project** 配置资源。

CLI：

1. 需 CLI 1.4.12 或更高，更新用 `agy update`；
2. 用符合套餐的个人 Google 账号登录（启动时和 `/help` 显示账号与套餐）；
3. `agy --remote-agent` 启动，输入提示。

状态栏显示 `remote` 和远程项目名。

> 远程智能体在**它自己的环境**里工作，不是你启动 CLI 的目录，读不到也改不了本机文件；要给代码和文件，请作为资源挂到远程项目。

#### 远程项目与资源

远程项目在 **Resources**（而非本地的 **Folders**）下挂载外部上下文，创建时或之后在 **Project Settings** > **Resources** > **Add Resource** 增删：

| 资源 | 说明 |
|---|---|
| GitHub | OAuth 授权后选仓库，对话自动 clone 并挂载；同一项目的所有私有仓库必须属于同一个 GitHub 用户或组织，多个所有者就各建一个项目 |
| Google Drive | 选 Docs、Sheets、Slides、PDF 或文件夹，智能体按需读取 |
| 上传的文件 | 每文件最多 8 MiB，每项目最多 50 个文件和 100 MiB，账号总计 2 GiB；含源码、文本和数据、图片、PDF、`.zip`；位于 `/home/antigravity/upload` |

CLI 里用 `/projects`：**\+ New project…**、输入名称、右方向键选资源，Ctrl + S 创建；改已有项目选中后 Enter > **Edit**。

#### CLI：项目、恢复与设置

首次在一台机器上启动会创建 `CLI Project (<hostname>)` 并复用。

```bash
agy --remote-agent --project="My remote project"      # 按名称或 ID 选项目
agy --remote-agent --new-project                      # 按当前目录名新建
agy --remote-agent -c                                 # 继续最近一次远程对话
agy --remote-agent --conversation=<conversation_id>   # 恢复指定对话
```

`--project` 找不到会警告并改用默认项目；会话中 `/projects` 切换项目，`/resume` 选远程对话。带 `--remote-agent` 只开远程对话，不带只开本地对话。

远程会话沿用 `settings.json` 的 CLI 设置，**权限除外**。`/config` 分 **TUI** 与 **Remote Agent** 两个标签（Tab 切换）：前者隐藏被取代的 **Agent Mode**、**Tool Permission**、**Artifact Review**、**Sandbox Mode**、**Non-Workspace Access**；后者有 **Permission Preset**（`default`、`request review`、`always-proceed (turbo)`）和 **Plan Review Policy**（`always ask` / `always proceed`）。`/permissions` 显示远程智能体的规则。

> **默认很宽松**：除非你改，预设是 `always-proceed (turbo)`，远程智能体运行命令**不会问你**。远程会话始终用默认执行模式，没有 plan、accept-edits、auto。

#### 插件、凭据与暂不支持

**Customizations** > **Marketplace** 可连接 **GitHub**、**Google Drive**、**Docs**、**Sheets**、**Slides**、**Calendar** 等插件（**Connect** 走 OAuth）；CLI 里 `/plugin`。OAuth 凭据静态加密，令牌在访问已授权服务时注入。

CLI 远程会话里暂不可用：`--print`、`/codesearch`、`/diff`、`/fork`、`/rewind`、`/btw`、`/voice`、用 `!` 自己运行命令、用 `@` 给子智能体发消息，以及本机上安装的 MCP servers、plugins、agent skills。不可用的命令会提示 `Not available in remote agent sessions.` 发送第一条消息后，`@` 才会建议远程环境里的文件。

排查：启动不了先确认账号与套餐（`/help`）；`Project Not Found` 表示 `--project` 没匹配项目，用 `/projects` 看名称；`is a local conversation` 表示该 ID 是本地对话，去掉 `--remote-agent` 恢复。

---

### Remote Control：用浏览器遥控你自己的机器

适用：App、CLI。从任意浏览器连接并驱动运行在你各台机器上的桌面会话和 CLI 实例，保留完整本地环境。**宿主机必须保持开机和联网。** 文档没写套餐限制；CLI 报 `Remote control is not enabled for your account.` 表示账号或组织策略没启用该功能。

#### App：启用与连接

1. Cmd + ,（Linux、Windows 用 Ctrl + ,）或点左侧栏 **Settings**；
2. 进 **Application**，打开 **Enable Remote Control**；
3. （可选）填 **Device Name**，如 `workstation-primary`；
4. 在任意设备浏览器打开 Antigravity 的 Remote Control Dashboard（官方网页入口），用与桌面相同的 Google 账号登录，在实例切换器选机器，即可查看对话、开新任务、审阅 implementation plan、检查 artifacts。手机上可把网页应用装到主屏幕以收推送通知。

机器没出现时：检查 **Enable Remote Control** 已开、宿主机联网且未睡眠、两端同一 Google 账号。网络短暂中断网页会自动重连，宿主机联网期间任务继续运行。

#### CLI：两种方式

| 对比 | 交互模式 | 无头守护进程 |
|---|---|---|
| 启动 | `/remote-control` 或 `agy --remote-control` | `agy remote-control start` |
| 范围 | 单个终端会话 | 整台机器（OS 服务） |
| 生命周期 | 终端开着且开启时有效；退出即断开 | 跨登出和重启持续 |
| 命名 | 每会话新名字（如 `host-swift-falcon`） | 固定机器名（`--name` 或 `cliRemoteControlHostname`） |
| 并发 | 同机可多个会话 | 每机一个守护进程 |
| 适合 | 离开正在运行的任务，转手机或网页继续 | 随时远程访问工作站或服务器 |

交互模式：在 `agy` 里运行 `/remote-control`（或 `/remote-control on`）建立反向隧道，会打印主机名和直达当前对话的链接：

```text
Remote control on for this session.
Hostname: devbox-swift-falcon
Open <官方远程控制网址>/<instance-id>?p=c%2F<conversation-id> on another device to continue this conversation.
```

在同一 Google 账号的浏览器打开即可；`/remote-control off` 只断开隧道、保留本地会话。退出 CLI（`/exit`、Ctrl + C、Ctrl + D）或关终端会自动终止隧道。两端实时同步：远程提交的提示流入本地 TUI，终端里的思考、工具执行和 diff 也出现在远程界面；工具确认（y/n）和澄清提问任一端都能答。**设置只能在 CLI 里改**，不能在远程网页界面改。

无头守护进程：

```text
agy remote-control start     注册并启动守护进程
agy remote-control status    显示状态和实例名
agy remote-control stop      停止并注销
```

守护进程注册为 OS 服务（Linux 的 systemd 用户服务、macOS 的 LaunchAgent、Windows 的计划任务），无需打开终端。用 `agy remote-control status` 查实例名，再到 Dashboard 里选它。它使用你登录 CLI 的凭据，与同机编辑器的登录相互独立。

| 行为 | Linux | macOS | Windows |
|---|---|---|---|
| 启动 | 开机，无需登录 | 登录时 | 管理员安装则开机，否则登录时 |
| 登出后继续 | 是 | 否，下次登录恢复 | 开机安装时是 |
| 崩溃后恢复 | 是 | 是 | 是，最多重试三次，之后下次开机或再运行 `start` |

- Windows 要开机启动需管理员提示符（PowerShell 或 Command Prompt）；普通提示符下 `start` 可用，但登出或重启后要重新登录才恢复；`status`、`stop` 普通提示符即可。
- 改名：`agy remote-control start --name "my-workstation"`；不设则首次启动自动生成。也可改 `~/.gemini/config/config.json`（Windows 为 `%USERPROFILE%\.gemini\config\config.json`）里的 `cliRemoteControlHostname`，再运行 `agy remote-control start` 重启后生效，服务运行时编辑无效。**注意**同一文件里 `remoteControlHostname` 是同机编辑器的名字（App 里在 **Settings** > **App** 改），别改错。
- 日志：Linux `journalctl --user -u antigravity-cli-daemon -n 50`；macOS `~/Library/Logs/antigravity-cli-daemon.log`；Windows 为 `%USERPROFILE%\.gemini\antigravity-cli\log` 下最新的 `cli-*.log`。
- 排查：没出现先 `agy remote-control status`，提示登录问题就 `agy` 登录后再 `start`；连接反复断开可能是旧脚本安装的守护进程在争用，再运行 `start` 会自动移除旧服务；Dashboard 里出现两个相似条目是编辑器和守护进程，有意分开。

---

### 连接 SSH：把 App 接到远程 Linux 机器

适用：App。在独立窗口里把 App 连到远程 Linux 机器：Antigravity 通过 OpenSSH 自动安装自己的远程服务器，项目、智能体、Git worktree、终端和自定义项直接在宿主机上运行；关窗口或笔记本睡眠，运行中的智能体继续工作。想在自己的 `ssh` 会话里使用，请在宿主机安装 Antigravity CLI。

| 项 | 要求 |
|---|---|
| 本地 | Antigravity App `v2.33.0+`（macOS、Windows、Linux），`PATH` 里有 `ssh`（或设 `ANTIGRAVITY_SSH_BIN`） |
| 远程系统 | 64 位 Linux（`x86_64` / `arm64`），`glibc` 2.26+（如 Ubuntu 18.04+、Debian 10+、RHEL 8+）；**不支持 macOS、Windows、`musl`（如 Alpine）** |
| 端口转发 | 宿主机和任何 `ProxyJump` 主机的 `sshd_config` 要有 `AllowTcpForwarding yes`（或 `local`） |
| 存储与网络 | 出站 HTTPS，`$HOME` 可写可执行（服务器在 `~/.antigravity-hub-server/`，数据在 `~/.gemini/`） |

#### 连接步骤

1. 打开 SSH 对话框：侧边栏（**Software Engineering** 布局）**Projects** 旁 **Create New Project** > **SSH Host…**；或命令面板（Cmd + Shift + P / Ctrl + Shift + P）运行 **Connect to SSH Host…**；或用 `--ssh-host=<destination>` 启动 App。
2. 从 **Recent** 或 **SSH Config**（`~/.ssh/config`）选主机，或输入 `Host` 别名、`[user@]host[:port]`、`ssh://[user@]host[:port]`；**Choose File…** 可用备用配置（`ssh -F`）。
3. 点 **Connect** 并认证，自动使用 OpenSSH 密钥、`ssh-agent`、`ProxyJump`。macOS、Linux 上口令、2FA、安全密钥触摸等在 App 内提示；Windows 上若没有交互提示，把密钥加载进 `ssh-agent` 并在终端运行一次 `ssh <host>` 记录 `known_hosts`。
4. 在 **`(SSH: <host>)`** 窗口里登录，本地浏览器完成 OAuth，凭据保存在 `<host>` 上（支持个人 Google 账号、Workspace、Pay As You Go、Gemini Enterprise）。

#### 在 SSH 窗口里工作

- 智能体工具、shell 命令、Linux 终端沙箱、权限、侧边栏终端（Cmd + J / Ctrl + J）都在宿主机运行；skills、rules、hooks、MCP servers、plugins 从宿主机的 `~/.gemini/` 和项目文件夹加载。
- 项目：**Create New Project** 浏览远程文件系统，可选 **Local mode** 或 **New worktree mode**。
- 启动时服务器用 `$SHELL -l -i`（10 秒超时）加载环境；改了 `~/.bashrc` / `~/.zshrc` 后运行 **Stop SSH Server…** 再重连；`exec zsh` 要用 `[ -t 0 ]` 保护，慢提示可用 `ANTIGRAVITY_RESOLVING_ENVIRONMENT=1` 跳过。
- 与本地窗口的差异：侧边栏浏览器不显示，链接在本地默认浏览器打开；`/browser` 连的是**宿主机上**开启远程调试的 Chrome；语音会询问 **Allow `<host>` to use your microphone?**，摄像头和屏幕保持禁用；**Open in IDE**、**Reveal in Finder** 等本地文件操作被隐藏。

| 操作 | 窗口与隧道 | 远程服务器与智能体 |
|---|---|---|
| 关闭窗口 | 断开 | 继续运行，可随时重连 |
| 睡眠或网络中断 | 自动重试（**Reconnecting to `<host>`…**） | 继续运行，恢复后重新接上 |
| **Stop SSH Server…** | 断开 | 停止宿主机上的智能体、sidecar 和服务器 |

点侧边栏 **`SSH: <host>`** 选 **This Computer** 可开本地窗口而不断开。

#### OpenSSH 配置与排查

`~/.ssh/config` 的标准设置直接生效，如 `Host devbox` 下的 `HostName`、`User`、`Port`、`IdentityFile`、`ProxyJump`。具体 `Host` 别名（不含 `*`、`?`、`!`）出现在 **SSH Config**。Antigravity 在命令行上覆盖 `ForwardAgent=no`、`ForwardX11=no`、`ControlMaster=no`、`ControlPath=none`、`RemoteCommand=none`、`ServerAliveInterval=15`、`ServerAliveCountMax=3`。自定义配置文件不能被他人写入（`chmod 600`）。

连接失败点 **Show Details**；服务器日志在宿主机 `~/.antigravity-hub-server/data/ls-*.log`。卸载：先 **Stop SSH Server…**，再删宿主机的 `~/.antigravity-hub-server`（连同 `~/.gemini` 会删掉已保存的对话和设置）。

| 错误码 | 原因与处理 |
|---|---|
| `INVALID_DESTINATION` | 主机格式无效或 IPv6 端口没加括号 |
| `SSH_CONFIG_INVALID` | `ssh` 拒绝配置文件（无效选项、权限不安全、UTF-8 BOM / UTF-16、非配置文件） |
| `AUTH_FAILED` | 认证失败；终端里试 `ssh <host>`，查 `IdentityFile` 或 `ssh-add -l` |
| `HOSTKEY_REJECTED` | `known_hosts` 缺失或已变；确认主机确实变更后再 `ssh-keygen -R <host>` |
| `CONNECTION_FAILED` / `CONNECTION_CLOSED` | 缺 `ssh`、主机不可达或隧道断；查 `ssh -V`、`ssh <host>`、`ANTIGRAVITY_SSH_BIN`，点 **Reconnect** |
| `FORWARDING_PROHIBITED` | 端口转发被禁，在宿主机和跳板机设 `AllowTcpForwarding yes`（或 `local`） |
| `UNSUPPORTED_HOST` | 不是 64 位 Linux + `glibc` 2.26+，或用了 `musl` |
| `SHARED_HOME_UNSUPPORTED` | 共享网络 `$HOME` 的另一台主机在运行服务器；连到那台、点 **Take Over**（会停止那里的智能体），或各机用独立 `$HOME` |
| `INSTALL_FAILED` | 检查磁盘（`df -h ~`）、`$HOME` 是否 `noexec`，没有 `curl` / `wget` 时装 `gzip`，让 `~/.bashrc` 在非交互 shell 下保持静默 |
| `LS_START_FAILED` / `LS_STOPPED` | 看 `ls-*.log` 后重连 |

---

### 远程方案选型：SSH、Remote Control、Remote Agents

三者都解决“人不在工位旁，智能体仍要继续干活”，区别是代码和工具在哪。

```text
代码在哪？
  ├─ 一台我自己的远程 Linux 机器，想在 App 窗口里操作 ─────► 连接 SSH
  ├─ 我自己的工作站 / 服务器，想用浏览器或手机遥控 ───────► Remote Control
  └─ GitHub / Drive / 上传文件，不想让任何本地机器常开 ──► Remote Agents（Pro / Ultra）
```

| 对比 | 连接 SSH | Remote Control | Remote Agents |
|---|---|---|---|
| 智能体运行在 | 你的远程 Linux 机器 | 你自己的桌面或服务器 | Google 托管环境 |
| 客户端 | App | 浏览器 | App、CLI 或浏览器 |
| 宿主机设置 | 无（OpenSSH 自动安装） | 安装并登录 App 或 CLI | 无（挂载仓库或文件） |
| 宿主机要开机 | 是（关窗口后智能体继续） | 是，必须开机联网 | 否 |
| 工作区 | 宿主机文件系统和 worktree | 本地文件夹、工具链、Git worktree | 远程 Resources |
| 入口与限制 | 仅 App，`v2.33.0+`；宿主机仅 64 位 Linux | App、CLI | 个人账号 AI Pro / Ultra，逐步推出；CLI 需 1.4.12+ |
| 适合 | 在远程开发机上完整开发 | 本地硬件、本地 dev server、未提交的检出 | 仓库任务、手机场景、离开后后台执行 |

Remote Control 与 Remote Agents 共用网页入口，用实例切换器在 **Remote Conversations** 和各台已连接主机间切换。

#### 安全提醒汇总

- Remote Agents 默认 `always-proceed (turbo)`：进入远程会话前先在 `/config` 的 **Remote Agent** 标签改成 `default` 或 `request review`，并用 `/permissions` 配规则。
- Remote Agents 读不到本机文件，本机的 MCP、插件和 skills 也不会带过去。
- Remote Control 守护进程会让机器在登出或重启后仍可从网页访问，不用时 `agy remote-control stop`。
- SSH 的 **Stop SSH Server…** 会停止宿主机上所有智能体；删除 `~/.gemini` 会删掉已保存的对话和设置。

---

## Part III：扩展、多智能体与自动化

### 扩展机制选型：Rules、Skills、MCP、Plugins、Hooks、Sidecars

#### 各自何时用

| 机制 | 一句话 | 何时用 | 载体 |
|---|---|---|---|
| Rules | 持久的约束与不变量 | “总是用 `zod` 校验”“不要跨包 import `internal/`” | `AGENTS.md` / `GEMINI.md` / `.agents/rules/*.md` |
| Skills | 多步骤工作流（按需加载） | “如何跑数据库迁移” | 含 `SKILL.md` 的文件夹 |
| Slash commands | 手动触发的快捷入口 | 显式调用内置能力或某个 Skill | 输入 `/`；每个 Skill 也是 `/<skill-name>` |
| MCP | 连接外部工具、数据与 API | 查数据库 schema、读构建日志、建 issue | `mcp_config.json` |
| Plugins | 把以上内容打成带版本的包 | 跨项目、跨团队分发 | 含 `plugin.json` 的目录 |
| Hooks | 在循环固定时点运行脚本 | 拦截命令、编辑后格式化、收尾检查 | `hooks.json` |
| Sidecars | 由 Antigravity 托管的后台进程 | 常驻脚本、定时任务、响应事件 | `sidecar.json`（仅 App） |

官方的经验法则：规则管“约束和不变量”，Skill 管“多步骤流程”。

```text
你想让 Antigravity……
│
├─ 永远遵守某条约束？
│    ├─ 每轮都生效 ───────────► Rule（AGENTS.md，always_on）
│    ├─ 只在特定文件类型时 ───► Rule（trigger: glob）
│    └─ 任务相关时才看 ───────► Rule（trigger: model_decision）
├─ 按步骤完成某类任务（可带脚本）？ ► Skill
├─ 访问外部系统（数据库、日志、SaaS）？ ► MCP server
├─ 在工具调用前后强制执行或拦截？ ► Hook
├─ 常驻后台或定时唤醒？ ────► Sidecar（/automation 可生成定时任务）
└─ 让别人一键装上这些？ ────► Plugin（再考虑 Marketplace）
```

#### 推荐落地顺序

1. 先写一份 `AGENTS.md`：项目背景、常用命令、硬性约束。
2. 某个流程被反复讲解时，抽成 Skill。
3. 需要外部数据时再接 MCP，只启用用得到的服务器和工具。
4. 必须“每次都成立”的检查才上 Hook。
5. 要在团队或多个仓库共享时，打包成 Plugin。
6. 有常驻或定时需求时再看 Sidecar。

拿不准选哪种时，可以直接问：`/agy-customizations should this be a rule or a skill?`

### Rules：持久规则

#### 位置与作用域

Rules 由 Antigravity 自动从文件系统发现并注入 agent 上下文，在 App、CLI、VS Code / Visual Studio / Eclipse 扩展和 Antigravity IDE 可用。规则是**累加**的：找到的所有规则合并进提示词，不是后者覆盖前者。

| 作用域 | 位置 |
|---|---|
| 目录级（工作区及父目录，直到仓库根） | `<dir>/AGENTS.md`、`<dir>/GEMINI.md`、`<dir>/.agents/AGENTS.md`、`<dir>/.agents/GEMINI.md`、`<dir>/.agents/rules/*.md`（旧版 `.agent/rules/*.md` 也支持） |
| 全局（所有项目） | `~/.gemini/AGENTS.md`、`~/.gemini/GEMINI.md`、`~/.gemini/config/AGENTS.md`、`~/.gemini/config/GEMINI.md` |
| 全局模块化（必须有 frontmatter） | `~/.gemini/config/rules/*.md` |
| 插件规则（CLI） | `~/.gemini/config/plugins/<plugin-folder>/rules/` |

- 不会加载工作区文件夹**以下**子目录里的规则，即使 agent 读写了那里的文件。
- `.agents/rules/` 只扫描直接子级 `.md`；嵌套子目录会被忽略，除非在 `.agents/rules.json` 中登记。

#### 触发方式

`AGENTS.md` / `GEMINI.md` 不用 frontmatter，整个文件常驻（`always_on`）。`rules/` 下每个 `.md` 必须以 YAML frontmatter 开头并声明合法的 `trigger`。

| 字段 | 必填 | 说明 |
|---|---|---|
| `trigger` | 是 | `always_on` / `model_decision` / `glob` / `manual` |
| `description` | `model_decision` 必填 | 何时适用的简述，会进入提示词索引 |
| `globs`（也接受 `glob`） | `glob` 必填 | 逗号分隔的通配符字符串，如 `"*.py, *_test.py"`；以 `*` 开头要加引号 |

| `trigger` | 行为 |
|---|---|
| `always_on` | 每轮注入完整内容 |
| `model_decision` | 先只注入路径与 `description`，任务匹配时 agent 按需读全文 |
| `glob` | agent 接触匹配文件时自动激活 |
| `manual` | 从不自动加载，只在你 `@` 提及时加入；**CLI 不支持规则提及** |

缺少 frontmatter，或 `trigger` 写成无法识别的值（如驼峰 `alwaysOn`、`modelDecision`），规则会被**静默丢弃**。

#### 示例

`.agents/rules/db-migrations.md`：

```markdown
---
trigger: model_decision
description: "Apply these guidelines whenever writing or reviewing database migrations or SQL schema changes."
---

1. Never drop or rename a column in a single deployment; use an expand-and-contract migration.
2. Always add `CONCURRENTLY` when creating indexes on existing tables in PostgreSQL.
```

`.agents/rules/proto.md`（`glob` 模式）：

```markdown
---
trigger: glob
globs: "*.proto, **/*.pb.go"
description: "Protobuf schema design and backward-compatibility rules."
---

* Never reuse a deleted field number; always mark both the field number and name as `reserved`.
```

#### 引用与限制

- `@[label](path)`：把目标文件内容**内联**进规则（相对路径从规则文件所在目录解析，`~/` 展开；被引用文件的 frontmatter 会被去掉）。
- `@filename`：**不**内联，只把路径改写为规范的工作区绝对路径。
- 单个规则文件上限 24,000 字节（展开内联之后），超出被截断。
- 全局规则与 `always_on` 规则共享 20,000 token 预算；超出时最大的文件被降级为指针 `- <path>: <description>`，agent 需要时再读。
- 共享规则或登记嵌套目录：写 `.agents/rules.json`，支持 `inherits` 与 `entries`（`path`、`exclude`、`include_only`）。

管理入口：App 在 **Customizations** → **Installed** → **Skills & Rules** 筛选 **Rules**；IDE 与扩展在 agent 侧边栏 **…** → **Customizations** → **Rules**，用 **+ Global** / **+ Workspace** 新建；CLI 直接在启动 `agy` 的目录及其父目录放文件。

### Skills：可复用的工作流

#### 目录结构与 SKILL.md

Skill 是遵循开放标准的文件夹。agent 每轮都能看到可用 Skill 的 `name` 和 `description`，觉得相关再读全文（渐进披露）；你也可以在消息里点名。

```text
my-skill/
├── SKILL.md       # 必需：说明与元数据
├── scripts/       # 可选：可执行脚本
├── examples/      # 可选：参考实现
└── resources/     # 可选：模板、schema、数据文件
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | 否 | 唯一标识（小写、连字符）；省略时默认为文件夹名 |
| `description` | 是 | 做什么、何时用；agent 靠它决定是否启用 |
| `disable-model-invocation` | 否 | 仅见于“构建插件”页：`true` 时禁止模型自动调用，只能 `/<name>` 触发 |

插件内的 Skill 另有约束：`name` 为 1–64 字符 kebab-case 且必须与父目录同名，`description` 为 1–1,024 字符。

#### 位置与优先级

| 位置 | 作用域 | 入口 |
|---|---|---|
| `<workspace-root>/.agents/skills/<skill-folder>/` | 工作区（也读各级父目录的 `.agents/skills/`） | App、CLI、IDE |
| `~/.gemini/config/skills/<skill-folder>/` | 全局 | App、CLI、IDE（IDE 还兼容旧的 `~/.gemini/antigravity/skills/`） |
| `~/.gemini/config/plugins/<plugin-folder>/skills/` | 插件提供 | CLI |
| `~/.gemini/antigravity-cli/skills/`、`~/.gemini/skills/` | CLI 专属 | 仅 CLI，App 不读 |

默认目录是 `.agents/skills`，兼容旧的 `.agent/skills`。同名冲突谁生效，本批文档没有给出顺序（指向另一页“Customization discovery”），需要时用 `/agy-customizations` 询问。

#### 调用与示例

- 自动：agent 根据提示判断。手动：输入 `/<skill-name>`。
- CLI 会把每个 Skill 自动转成 TUI 斜杠命令（`deploy-staging` → `/deploy-staging`）；运行中新增后执行 `/skills reload` 或开新对话。

`.agents/skills/code-review/SKILL.md`：

```markdown
---
name: code-review
description: Reviews code changes for bugs, style issues, and best practices. Use when reviewing pull requests or checking code quality.
---

# Code Review Skill

1. **Correctness**: Verify that the code satisfies specifications.
2. **Edge cases**: Ensure error conditions and boundaries are handled.
3. **Style**: Follow project naming and architectural patterns.
```

#### 验证与写作建议

1. App / IDE 在 **Customizations** 里看是否列出；CLI 输入 `/` 看有没有 `/code-review`。
2. 提一个应触发它的任务，或直接输入 `/code-review`。
3. 没出现：检查文件夹位置、`description` 是否存在；CLI 试 `/skills reload`。

官方建议：一个 Skill 只做一件事；`description` 用第三人称并带关键词；脚本当黑盒，让 agent 先跑 `--help`；复杂 Skill 加“决策树”小节。

### Slash commands 总览

在 App 或 CLI 输入框输入 `/` 打开菜单，继续输入可过滤。下表只挑原文列出的常用命令；CLI 全部命令（含 `/clear`、`/help`、`/model`）见 CLI 参考页。

| 目的 | 命令 | 作用 | 可用 |
|---|---|---|---|
| 先规划 | `/plan` | 探索代码库、澄清需求，先出可评审计划再动手 | App、CLI、IDE 扩展 |
| | `/grill-me` | 一次一个问题地访谈你并给推荐答案 | App、CLI、IDE 扩展 |
| 委派 | `/goal` | 持续工作到目标被验证完成 | App、CLI、IDE 扩展 |
| | `/boost` | 多 agent 深度推理；仅付费计划 | App、CLI、IDE 扩展 |
| | `/swarm` | 命名 agent 加验证者在共享消息板协作 | App、CLI |
| | `/teamwork` | 面向多日项目的 agent 团队；仅付费计划 | App、CLI、IDE 扩展 |
| | `/run-workflow` | 写并运行编排多 agent 的脚本 | App、CLI |
| | `/browser` | 浏览器子 agent 做 UI 测试、截图 | App、CLI、IDE 扩展 |
| | `/automation` | 创建定期后台任务 | App |
| | `/schedule` | 稍后在同一对话里唤醒 agent | App、CLI、IDE 扩展 |
| 求助 | `/btw` | 在只读分叉里回答旁支问题 | App、CLI、IDE 扩展 |
| | `/generative_ui` | 生成交互式小组件 | App |
| | `/antigravity-guide` | 依据官方文档回答使用问题 | App、CLI |
| 会话 | `/compact` | 把对话摘要成检查点，腾出上下文 | App、CLI |
| | `/diff` `/context` `/resume` `/agents` | 查看变更 / 上下文用量 / 恢复会话 / 切换自定义 agent | CLI |
| 定制 | `/learn` | 把会话中的纠正整理成规则或 Skill（需你批准） | App、CLI、IDE 扩展 |
| | `/agy-customizations` | 加载定制参考 | App、CLI、IDE 扩展 |
| | `/mcp-app-development` | 加载 MCP Apps 开发指南 | App |
| | `/plugin` | 管理插件、浏览 Marketplace | App、CLI、IDE 扩展 |
| CLI 配置 | `/permissions` `/effort` `/statusline` `/title` | 权限规则 / 推理强度 / 状态栏 / 窗口标题 | CLI |
| 账户 | `/usage` `/credits` `/changelog` | 额度 / AI credit 余额 / 更新日志 | CLI 为主 |

组合：`/plan /grill-me <task>` 把访谈结果变成计划。

#### /agy-customizations 与 /antigravity-guide

| 命令 | 用来做什么 | 可用 |
|---|---|---|
| `/agy-customizations` | 创建、修复、解释 rules / skills / plugins / hooks / MCP 配置 | App、CLI、IDE 扩展、IDE |
| `/antigravity-guide` | 问“Antigravity 怎么用”（CLI、IDE、App、SDK） | App、CLI |

两者都是内置 Skill，agent 遇到相关问题也会自动加载，命令只是确保加载。`/agy-customizations` 加载《Antigravity Customization System Guide》：定制类型与文件位置、发现与优先级、渐进披露与规则去重、用 `skills.json` / `plugins.json` 从其他目录加载；可单独运行，只加载不提问。`/antigravity-guide` 提供各入口速查和官方文档地图，速查没覆盖时 agent 会去读官方文档页，按你的权限预设可能先问是否允许联网读取。

```text
/agy-customizations create a PreToolUse hook that blocks git push --force
/agy-customizations my deploy skill never shows up as /deploy, figure out why
/antigravity-guide where do I change the permission preset for this project?
```

要让 agent 创建或调试定制项用前者，问产品怎么用用后者。

### MCP：连接外部工具与数据

#### 入口与配置位置

MCP 让 agent 获取工作区之外的实时上下文和动作（查实时 schema、拉部署日志、建 Linear issue）。在 App、CLI、各 IDE 扩展、IDE 可用。

| 入口 | 怎么管理 |
|---|---|
| App | **Customizations** → **Installed** → **MCP Servers**；**Add MCP Server (+)** 后 **Add**；**⋮** 里 **Restart / Disable / Delete**；**Copy Config File Path** 取配置路径 |
| CLI | `/mcp` 打开交互式 MCP Manager（状态环、重载配置、连接日志），或直接改配置文件 |
| IDE | agent 侧边栏 **…** → **MCP Servers**，MCP Store 里 **Install**；自定义走 **Manage MCP Servers** → **View raw config** |

```text
~/.gemini/config/mcp_config.json        全局
<workspace>/.agents/mcp_config.json     工作区
<plugin-root>/mcp_config.json           插件（工具名带 <plugin-name>_ 前缀）
```

Python SDK 也能程序化连接 `stdio`、`SSE`、`HTTP` 服务器，并自动发现 `.agents/mcp_config.json`。

根对象是 `mcpServers`，每个条目指定**一种**传输属性：`command`（本地 stdio）或 `serverUrl`（远程）。

```json
{
  "mcpServers": {
    "sqlite-explorer": {
      "command": "node",
      "args": ["/usr/local/bin/sqlite-mcp-server.js"],
      "env": { "SQLITE_DB_PATH": "/var/data/app.db" }
    },
    "my-remote-server": {
      "serverUrl": "https://api.example.com/mcp/",
      "headers": { "Authorization": "Bearer YOUR_API_TOKEN" },
      "timeoutSeconds": 600
    }
  }
}
```

| 属性 | 默认 | 说明 |
|---|---|---|
| `command` / `args` / `env` / `cwd` | — | 仅 stdio：启动命令、参数、环境变量、工作目录 |
| `serverUrl`（或 `url`） | — | 远程 Streamable HTTP 端点 |
| `headers` | `{}` | 远程请求头 |
| `authProviderType` | — | `"google_credentials"` 使用 Google ADC |
| `oauth` | — | 手动 OAuth：`clientId`，可选 `clientSecret` |
| `disabled` | `false` | `true` 暂停连接但保留配置 |
| `enabledTools` / `disabledTools` | — / `[]` | 白名单 / 黑名单；设了白名单则其余全部隐藏 |
| `tools` | — | 按工具名设置：`eager`（布尔）、`background`（`"off"` / `"always"`） |
| `timeoutSeconds` | 调用 180，连接 300 | 连接与每次调用的超时；负值（如 `-1`）关闭；`0` 或不设保持默认 |

#### 传输类型与超时

| 类型 | 配置 | 说明 |
|---|---|---|
| `stdio` | `command` + `args` | 本地子进程 |
| Streamable HTTP | `serverUrl` / `url` | 远程服务器**必须**用它；旧 HTTP+SSE 传输与 `httpUrl` 字段**不支持** |

超时报错形如 `MCP tool call to server "sqlite-explorer" timed out after 3m0s`。注意 `timeoutSeconds` 同时限制连接时间，设太小会让启动慢的服务器连不上。

#### 认证

| 方式 | 配置 | 备注 |
|---|---|---|
| 自定义请求头 | `headers` | 最简单；密钥用占位符，别提交进仓库 |
| Google 凭据 | `"authProviderType": "google_credentials"` | 先 `gcloud auth application-default login`，必要时 `gcloud auth application-default set-quota-project {QUOTA_PROJECT}` |
| OAuth（自动） | 只写 `serverUrl` | OAuth 2.0 + PKCE；服务器需支持 CIMD 或 DCR |
| OAuth（手动） | `oauth.clientId`（可选 `clientSecret`） | 两者都不支持时使用；须在 OAuth 提供方登记文档给出的 `oauth-callback` 重定向 URI |

凭据解析顺序：手动 `oauth` → CIMD → DCR。Antigravity IDE 的 CIMD 支持需 2.6.0 或更高版本。App 里认证：**Customizations** → **Installed** → **MCP Servers** 点 **Authenticate**，浏览器登录并复制授权码，粘贴到 **Enter authorization code** 后 **Submit**。令牌存于 `~/.gemini/antigravity/mcp_oauth_tokens.json`，过期自动刷新。

#### 工具加载与权限

默认所有已启用工具 **lazy** 加载：连接时把每个工具的 schema 写到磁盘（App / IDE：`~/.gemini/antigravity/mcp/<serverName>/`；CLI：`~/.gemini/antigravity-cli/mcp/<serverName>/`），系统提示词只列工具名，agent 需要时读 schema，经内置 `call_mcp_tool` 调用。给高频工具设 `eager` 才每轮发送完整声明（名为 `mcp_<serverName>_<toolName>`），省一步读文件但多占 token：

```json
{ "mcpServers": { "sqlite-explorer": {
    "command": "node",
    "args": ["/usr/local/bin/sqlite-mcp-server.js"],
    "tools": { "read_query": { "eager": true }, "list_tables": { "eager": true } }
} } }
```

被 `disabledTools` 禁用、或不在非空 `enabledTools` 里的工具，即使标了 `eager` 也仍隐藏。

未配置的 MCP 工具默认 **Ask**，每次执行前要你批准。权限策略里可写 `mcp(server/tool)`（单个工具）、`mcp(server/*)`（某服务器全部工具）、`mcp(*)`（全部）。

#### 排错

| 现象 | 先查 |
|---|---|
| 不确定是否连上 | CLI `/mcp` 看状态环与连接日志，必要时手动重载 |
| 服务器 `Unauthorized`、红点 | 点 **Authenticate**；检查 `headers` 令牌 |
| 远程连不上 | 是否 Streamable HTTP；有没有误用 `httpUrl` 或旧 SSE |
| 调用超时 / 慢启动连不上 | 调整 `timeoutSeconds`（默认调用 180 秒、连接 300 秒） |
| 某工具看不见 | 检查 `enabledTools` / `disabledTools` |
| 改了本地 stdio 代码 | **⋮** → **Restart** |

MCP Store 内置数据库、开发者工具、设计、安全、分析等类别的现成服务器。例如 **Google Developer Knowledge** 提供 `search_documents`、`get_documents`、`answer_query`，用 Google 凭据连接，无需 API key。

### MCP Apps 与 Generative UI

#### MCP Apps 是什么

MCP App 让 MCP 工具在纯文本输出之外附带交互式 HTML 界面（`ui://` 资源）。Antigravity 实现开放的 MCP Apps 规范（`io.modelcontextprotocol/ui`），并增加入口扩展 `com.google.antigravity/ui`。

**版本与可用性**：交互式视图只在 Antigravity App（`v2.34.0+`）渲染，要求客户端在 MCP `initialize` 时声明 `capabilities.extensions["io.modelcontextprotocol/ui"]`。CLI、IDE 等非 UI 客户端只把它当普通文本 MCP 工具，所以务必在 `structuredContent` 之外返回清晰的纯文本 `content`。

| 位置 | 声明方式 | 行为 |
|---|---|---|
| 对话内联 | `_meta.ui.resourceUri` | 替换折叠的工具步骤卡片，高度最高 800 px（回退 150 px） |
| 侧边面板 | `aux_pane` 入口 | agent 调用后在对话旁打开；每个 `(server, tool)` 一个标签页 |
| 整页 | `global` 入口 | 从左侧栏 **Plugins** 全宽打开 |
| 文件查看器 | `file` 入口（`extensions` 必填） | 替换匹配扩展名文件的内置查看器；对模型始终隐藏 |
| CLI | `mcp_config.json` | 经 `/mcp` 连接，无 iframe，读纯文本 `content` |

```json
{
  "_meta": {
    "ui": { "resourceUri": "ui://acme-cloud/deployments.html" },
    "com.google.antigravity/ui": {
      "entrypoints": [
        { "type": "aux_pane", "title": "Acme Deployments" },
        { "type": "global", "title": "Acme Deployments" }
      ]
    }
  }
}
```

`file` 入口要放在**单独**的工具上（它始终对模型隐藏，接收 `{"file": {"name", "resourceUri"}}` 而非 `{}`）。带 `aux_pane` 或 `global` 的工具不内联渲染。

#### 如何开发

方式一：让内置命令搭脚手架（仅 App）。它会加载 MCP Apps 开发指南，覆盖单文件 `ui://` 资源与 CSP、入口配置、主题 CSS 变量与宿主动作、`mcp_config.json` 的 `${PLUGIN_ROOT}` / `${PLUGIN_DATA}`、`plugin.json`，以及空白帧与 CSP 拦截的排错。

```prompt
/mcp-app-development Build an MCP App plugin for Antigravity that tracks release checklist items inline in chat and in a side panel board using @modelcontextprotocol/ext-apps.
```

方式二：手动。

1. 按规范与 `@modelcontextprotocol/ext-apps` 打成单文件 `ui://` HTML 资源（`text/html;profile=mcp-app`），用 `_meta.ui.resourceUri` 关联。
2. 把服务器打包进插件，根目录放 `mcp_config.json`。
3. `agy plugin validate <path> --strict`，再 `agy plugin install <path>`。

宿主限制：HTML 与文件 10 MiB；持久化的工具结果 1 MiB；单视图并发 `tools/call` 4 个；隐藏后台帧 5 个（LRU）；关闭时**不**发送 `ui/resource-teardown`，编辑要及时持久化；iframe 默认拒绝、无持久浏览器存储，CSP 只在资源 `_meta.ui.csp` 里放行裸 origin。

迭代：`ui://` 资源每次挂载都重新获取，改 HTML 后关闭再打开视图即可；改了 stdio 服务器代码要在 **Customizations** → **Installed** → 插件 → **MCP** → **⋮** → **Restart**。

#### Generative UI（`/generative_ui`）

仅 App 可用，CLI 无此命令也无法显示小组件。agent 生成单文件 HTML 的交互小组件，存为对话 artifact，默认在侧边面板打开；紧凑的可内联在回复里。

```prompt
/generative_ui Visualize the users and bookmarks tables and how they relate
```

| 位置 | 出现在哪 | 适合 |
|---|---|---|
| 独立 artifact（默认） | 侧边面板，最高面板全高 | 仪表盘、模拟、原型 |
| 内联小组件 | 回复里，最高 500 px | 小图表、示意图、讲解 |

想指定位置就在请求里说（如 `as an inline widget`）。内联组件也存为 artifact，点卡片可在侧边面板放大；**Overview** 的 **Artifacts** 下可找到，**More actions** → **Export Artifact** 导出（不含 App 主题色）。

限制：控件在组件内运行，不给 agent 发消息；沙箱隔离，不能访问你的文件；不能加载外部脚本（Tailwind CSS 脚本除外）；是“时间点快照”，代码或数据变了要让 agent 更新。

**选哪个**：一次性的图、原型、讲解用 `/generative_ui`；可复用、背靠实时 MCP 服务器的界面、侧边面板视图、自定义文件查看器用 MCP Apps。

### Plugins 与 Marketplace

#### 插件是什么

插件把 skills、MCP servers、custom agents / subagents、rules、lifecycle hooks 打成带版本的单元；可从 Marketplace、GitHub 仓库或本地目录安装。在 App、CLI、VS Code / Visual Studio / Eclipse 扩展、IDE 可用。

| 组件 | 默认路径 | 使用方式 |
|---|---|---|
| Skills | `skills/<skill-name>/SKILL.md` | 自动匹配或 `/<skill-name>` |
| MCP servers | `mcp_config.json` | 工具名 `<plugin-name>_<server-name>` |
| Subagents | `agents/<agent-name>.md` | 主 agent 后台委派，或选作主会话 agent |
| Rules | `rules/AGENTS.md` 或 `rules/<rule-name>.md` | 启用时自动生效 |
| Hooks | `hooks.json` | 启用时自动运行 |

#### 安装与管理

- **App**：左侧栏 **Customizations**（**Automations** 下方）→ 切换 **Marketplace** / **Installed**。Marketplace 点卡片 **+** 或详情页 **Add**；Installed 用 **⋮** 启用、停用、卸载。详情页有起始提示词芯片与 **Overview / Skills / MCP / Agents / Rules** 标签。
- **CLI 交互式**：`/plugin`（别名 `/plugins`），Tab 切换 **Discover** / **Installed**；Enter 展开详情，Ctrl + S 安装或卸载，Space（Installed）启用或停用；Discover 里选 **Install from local directory** 装本地目录。
- **CLI 行内命令**：`/plugin install <plugin-name>@antigravity-plugins-official`、`/plugin install <local-path>`、`/plugin enable|disable|uninstall <plugin-name>`、`/plugin list`。

```bash
agy plugin install <plugin-name>@antigravity-plugins-official   # 官方 Marketplace（需 agy 已登录）；裸名默认同此
agy plugin install https://github.com/<owner>/<repo>            # GitHub 仓库
agy plugin install </path/to/local/plugin>                      # 本地目录
agy plugin import [gemini|claude]    # 从 ~/.gemini/extensions/ 或 ~/.claude/plugins/ 迁移
agy plugin list [--json]
agy plugin enable|disable|uninstall <plugin_name>
agy plugin validate [--strict] [--json] ./my-plugin
agy agents                           # 列出内置、工作区与插件 agent
agy --agent <agent-name>             # 以指定 agent 启动会话
```

- `<marketplace-name>` 目前只支持 `antigravity-plugins-official`。App 里装的插件会自动同步到 CLI 的 **Installed**。
- 手动安装：放进 `.agents/plugins/`（仅该项目）或 `~/.gemini/config/plugins/`（全局）。
- 插件声明了 MCP `variables`（API key 等）时，用 CLI 安装后默认**停用**，需在 `/plugin` 里打开并填写；只用 `oauth` 的插件装完即启用，MCP 连接时在浏览器登录。

#### 目录结构与 plugin.json

```text
plugins/<plugin-name>/
├── plugin.json               # 必需（或 .antigravity-plugin/plugin.json，后者优先）
├── README.md                 # 可选：安装前在 Marketplace 详情里展示
├── assets/logo.svg           # 可选
├── scripts/                  # 可选：辅助脚本（Marketplace 包内保持 0755）
├── mcp_config.json           # 可选
├── hooks.json                # 可选
├── skills/<skill-name>/      # SKILL.md、scripts/、references/、assets/
├── agents/<agent-name>.md    # 可选
└── rules/                    # AGENTS.md 与 <rule-name>.md
```

| 字段 | 必填 | 说明 |
|---|---|---|
| `name` | CLI 与 Marketplace 必填 | 小写 kebab-case（`^[a-z0-9]+(-[a-z0-9]+)*$`）；同时是 MCP 命名空间 |
| `displayName` | Marketplace 必填 | 展示名；不要带“Plugin”“MCP”后缀，不要以 `Antigravity`、`Gemini`、`Claude`、`Cursor` 开头 |
| `description` | Marketplace 必填 | 1–2 句摘要（建议 120–160 字符） |
| `version` | Marketplace 必填 | 建议 `MAJOR.MINOR.PATCH`；每次发布递增，客户端据此发现更新 |
| `logo` | Marketplace 必填 | 包内 1:1 方形图标相对路径（`.svg/.png/.jpg/.jpeg/.webp`，不小于 128x128px），亮暗主题都要清晰 |
| `suggestedPrompts` | 推荐 | 最多 3 条起始提示词 |
| `skills` | 否 | 除默认 `skills/` 外额外的 Skill 目录 |
| `disabled` | 否 | 首次安装是否停用，默认 `false` |
| `author` | Marketplace 必填 | 字符串或 `{ "name", "url" }`（`url` 须为 `https://`） |
| `homepage` `repository` `license` `keywords` | 推荐 / 否 / 推荐 / 推荐 | 文档页、源码仓库、SPDX 许可证、小写关键词 |

清单可带 `$schema` 键（值见官方文档）以获得编辑器补全；官方 schema 的 `additionalProperties` 为 `false`，只有 `name` 必填。

#### 构建与发布流程（Build plugins）

1. 创建目录与清单：`mkdir -p my-first-plugin/skills/hello`，再写 `my-first-plugin/plugin.json`：

   ```json
   {
     "name": "my-first-plugin",
     "displayName": "My First Plugin",
     "version": "1.0.0",
     "description": "Starter plugin that greets the user in Antigravity.",
     "suggestedPrompts": ["Greet me and explain how this plugin is structured"]
   }
   ```

2. 写 `my-first-plugin/skills/hello/SKILL.md`（`name` 必须与目录名 `hello` 一致）：

   ```markdown
   ---
   name: hello
   description: >-
     Greet the user warmly and explain how this plugin works. Use when
     the user asks for a greeting or wants to test my-first-plugin.
   ---

   Greet the user by name if known and confirm that `my-first-plugin` is active.
   ```

3. 校验、安装、确认：

   ```bash
   agy plugin validate --strict ./my-first-plugin
   agy plugin install ./my-first-plugin
   agy plugin list
   ```

4. 对话里输入 `/hello`，或在 **Customizations** → **Installed** 点起始提示词芯片。
5. 发布：校验零错误，清单含 `name`、`displayName`、`description`、`version`、`logo`、`suggestedPrompts`、`author`，附 `README.md` 与 logo，再通过官方的 Marketplace 合作方申请表提交（文档提供表单入口）。

**路径变量**：安装后的插件在只读目录 `~/.gemini/config/plugins/<plugin-name>`，更新时会被替换。MCP 定义的 `args`、`cwd`、`env` 中可用 `${PLUGIN_ROOT}`（只读根目录）、`${PLUGIN_DATA}`（插件专属可写数据目录，在工作区之外），二者也会自动导出为同名环境变量；`${VARIABLE_NAME}` 引用 `mcpServers.<server>.variables` 声明的用户值。

```json
{
  "mcpServers": {
    "local-helper": {
      "command": "node",
      "args": ["${PLUGIN_ROOT}/scripts/mcp-server.mjs", "--data-dir", "${PLUGIN_DATA}"]
    },
    "cloud-token": {
      "url": "https://api.acme.example.com/mcp",
      "headers": { "Authorization": "Bearer ${ACME_API_TOKEN}" },
      "variables": [
        { "name": "ACME_API_TOKEN", "title": "Acme Cloud API Token",
          "description": "Personal access token with read:services and deploy scopes." }
      ]
    }
  }
}
```

`variables` 每项：`name`（必填）、`title`、`description`、`link`（可选）。`hooks.json` 的命令以 `<plugin-root>` 为工作目录，用 `./scripts/...` 相对路径，不用 `${PLUGIN_ROOT}`。

自定义 agent（`agents/<agent-name>.md`）frontmatter：`name`、`description`（必填）；`mainAgent`（默认 `true`，可选作主会话 agent）；`subagent`（默认 `true`，可被后台委派）；`model`（`inherit` 默认、`flash_lite`、`flash`、`pro`）；`tools`（内置工具白名单，`send_message` 始终包含）；`commandExecutionPolicy`（`off`/`auto`/`eager`/`sandbox`，默认 `sandbox`）；`skills`；`inheritMcp` / `inheritCustomizations`（默认 `true`）。

**校验分级**：`agy plugin validate` 用与运行时相同的加载器检查：Error 始终非零退出；Warning 默认退出 0，`--strict` 时也失败；Note 仅提示。`--json` 输出供 CI 使用。

**发布前清单**：

| 检查 | 要求 |
|---|---|
| 校验 | `agy plugin validate ./my-plugin` 零错误 |
| 命名 | `name` 为 kebab-case；品牌不以 `Antigravity`/`Gemini`/`Claude`/`Cursor` 开头 |
| Skill 目录名 | `SKILL.md` 的 `name` 与父目录同名 |
| 规则 `globs` | 逗号分隔字符串，不是 YAML 数组 |
| MCP `variables` | 写在 `mcpServers.<server>.variables` 内，不在文件根；不要在 `env` 里重复 `PLUGIN_ROOT` / `PLUGIN_DATA` |
| Hooks | 具名 hook 对象包裹事件数组；`Stop`、`SessionStart`、`PreInvocation`、`PostInvocation` 用扁平 handler 数组 |
| logo 与可执行文件 | `logo` 指向包内方形文件；可执行文件放 `scripts/` 以保持 0755 |

**多工具仓库的预期警告**：仓库里给其他工具用的文件会触发“被忽略/被遮蔽”警告，属正常：根部 `.mcp.json` / `mcp.json`（MCP 只从 `mcp_config.json` 加载）、根部 `plugin.json`（`.antigravity-plugin/plugin.json` 存在时被遮蔽）、根部 `AGENTS.md`（插件规则只从 `rules/` 加载）。CI 里可不加 `--strict`，或用 `--json` 配合 `jq` 过滤这些已知文件。

### Hooks：在 agent 循环中插入脚本

#### 配置位置

Hooks 在执行循环的固定时点运行你的脚本，用于强制规则、跑 linter、收集诊断。在 App、CLI、各 IDE 扩展、IDE 可用。

| 作用域 | 位置 |
|---|---|
| 工作区 | `.agents/hooks.json` |
| 全局 | `~/.gemini/config/hooks.json`（CLI 不加载 `~/.gemini/antigravity-cli/settings.json` 里的 hooks） |
| 插件 | 已安装插件的 `hooks.json`（App 与 CLI 页面明确列出） |

查看已加载的 hooks：CLI 输入 `/hooks`；IDE 用 **…** → **Customizations** → **Hooks**。

#### 事件表

| 事件 | 触发时机 | matcher |
|---|---|---|
| `SessionStart` | 新会话开始，首次调用模型前（一次） | 忽略 |
| `PreToolUse` | 工具执行前 | 工具名 |
| `PostToolUse` | 工具完成后 | 工具名 |
| `PreInvocation` | 调用模型前 | 忽略 |
| `PostInvocation` | 每次模型调用完成后 | 忽略 |
| `Stop` | 执行循环终止时 | 忽略 |

`PreToolUse` / `PostToolUse` 的 `matcher` 是正则：`""` 或 `"*"` 匹配所有工具；`"run_command"` 精确；`"run_command|view_file"` 二选一；`"browser_.*"` 前缀。文档列出的工具名有 `view_file`、`write_to_file`、`replace_file_content`、`multi_replace_file_content`、`list_dir`、`find_by_name`、`grep_search`、`search_web`、`read_url_content`、`run_command`、`manage_task`、`schedule`、`invoke_subagent`、`send_message`、`ask_question`、`generate_image` 等。

#### 配置结构

`hooks.json` 是“hook 名 → 事件配置”的映射。`PreToolUse` / `PostToolUse` 每项包一层 `{ "matcher", "hooks": [...] }`；其余四个事件直接是 handler 数组。

```json
{
  "my-linter-hook": {
    "PostToolUse": [
      { "matcher": "run_command",
        "hooks": [ { "type": "command", "command": "./scripts/lint.sh", "timeout": 10 } ] }
    ]
  },
  "session-init": {
    "SessionStart": [ { "type": "command", "command": "./scripts/init-env.sh" } ]
  }
}
```

| 字段 | 说明 |
|---|---|
| `enabled` | 可选；`false` 停用但不删除，默认 `true` |
| `type` | 可选；目前只支持 `"command"`（默认） |
| `command` | 必填；要执行的 shell 命令 |
| `timeout` | 可选；秒，默认 `30` |

#### 输入、输出与决策

Hook 从 **stdin** 收到 JSON，向 **stdout** 返回 JSON，字段为 camelCase。所有 hook 的公共输入：`conversationId`、`workspacePaths`、`transcriptPath`、`artifactDirectoryPath`、`modelName`、`lastUserInput`。`transcriptPath` 位于 `<app_data_dir>/brain/<conversationId>/.system_generated/logs/transcript.jsonl`，`<app_data_dir>` 在 App 是 `~/.gemini/antigravity`，CLI 是 `~/.gemini/antigravity-cli`，IDE 是 `~/.gemini/antigravity-ide`。

| 事件 | 额外输入 | stdout 输出 |
|---|---|---|
| `SessionStart` | 仅公共字段 | `injectSteps`（可选） |
| `PreToolUse` | `toolCall.name`、`toolCall.args`、`stepIdx` | `decision`（必填）、`reason`、`overwrite`、`permissionOverrides` |
| `PostToolUse` | `toolCall`、`stepIdx`、`error`（失败时） | `{}` |
| `PreInvocation` | `invocationNum`、`initialNumSteps` | `injectSteps`、`terminationBehavior`（`"terminate"` 或空） |
| `PostInvocation` | 同上 | `injectSteps`、`terminationBehavior`（`"force_continue"` / `"terminate"` / 空） |
| `Stop` | `executionNum`、`terminationReason`、`error`、`fullyIdle` | `decision`（必填）、`reason` |

`PreToolUse` 的 `decision`：

| 值 | 效果 |
|---|---|
| `allow` | 放行，交给下一个 hook 或默认策略 |
| `auto_approve` | 默认策略本会询问时自动批准，仍遵守你显式配置的规则 |
| `deny` | 立即硬性拦截 |
| `ask` | 询问你，尊重“Always Allow” |
| `force_ask` | 始终询问，忽略缓存权限 |
| `deny_unless_prior_grant` | 除非此前批准过该资源，否则拒绝 |

- `overwrite`：浅层合并进 `toolCall.args` 后再执行，并在工具结果前加一条被改动键的通知。
- `Stop` 返回 `"decision": "continue"` 会阻止停止、回到循环，`reason` 作为系统消息注入；其他值允许停止。
- `injectSteps` 每项可为 `systemMessage`、`userMessage`、`ephemeralMessage`、`errorMessage`；`errorMessage` 只显示错误，要中止需配 `"terminationBehavior": "terminate"`。
- **退出码**：命令 hook 须以 `0` 退出并向 stdout 写 JSON；非零退出码只记为 hook 执行错误并被忽略（**fail open**，不会拦截）。要拦截请返回 `{"decision": "deny"}`，别靠非零退出码。

#### 最小示例

让所有 `run_command` 都弹出确认。`.agents/hooks.json`：

```json
{
  "confirm-commands": {
    "PreToolUse": [
      { "matcher": "run_command",
        "hooks": [ { "type": "command", "command": "./scripts/confirm.sh", "timeout": 10 } ] }
    ]
  }
}
```

`scripts/confirm.sh`（先 `chmod +x`）：

```bash
#!/bin/bash
cat > /dev/null   # 丢弃 stdin 里的事件 JSON
echo '{"decision": "force_ask", "reason": "Shell commands need manual confirmation."}'
```

验证：CLI `/hooks` 确认已加载，再触发一次匹配的工具调用。没效果时依次查：文件位置、是否有具名对象包裹、`matcher` 是否匹配真实工具名、脚本是否可执行、stdout 是否合法 JSON、是否超时（默认 30 秒）。`Stop` 的 `continue` 要在检查通过时改返回 `{}`，否则会一直不让 agent 结束。

### Sidecars：托管的后台进程

仅 App 可用。Sidecar 是随 Antigravity 运行的后台进程，生命周期由 Antigravity 管理：自动启动，崩溃或出错时重启。适合常驻脚本、定时任务和事件响应。

#### 配置

Antigravity 在两处查找 `sidecar.json`；每个 sidecar 一个目录，目录名即 ID，目录也是命令的工作目录。

```text
~/.gemini/config/sidecars/
├── sidecar1/
│   ├── sidecar.json
│   └── script.py
└── sidecar2/sidecar.json

~/.gemini/config/plugins/<pluginName>/sidecars/<sidecarName>/sidecar.json   # ID 为 <pluginName>/<sidecarName>
```

| 字段 | 说明 |
|---|---|
| `command` | 命令或可执行文件，与 `builtin` 互斥 |
| `builtin` | 内置命令，目前支持 `schedule`，与 `command` 互斥 |
| `args` | 可选；传给命令或内置函数的参数 |
| `restart_policy` | 可选；`always`（默认）、`on-failure`、`never` |
| `description` / `display_name` / `env` | 可选；描述 / UI 显示名 / 环境变量 |

`command` 与 `builtin` 必须二选一。**Sidecar 默认停用**，要在 `~/.gemini/config/config.json` 里启用：

```json
{
  "sidecars": {
    "sidecar1": { "enabled": true },
    "my-plugin/plugin-sidecar": { "enabled": true, "projectId": "<projectId>" }
  }
}
```

`projectId` 可选，是 `agentapi` 创建对话所在的项目；用 `agentapi` 创建对话的 sidecar 必须设置。

#### 常驻脚本与定时任务

```json
{
  "description": "Background worker",
  "command": "python3",
  "args": ["worker.py"],
  "restart_policy": "on-failure"
}
```

`schedule` 内置调度器：第一个参数是标准五段 cron 表达式，其余是要运行的命令与参数；指定时区在表达式前加 `CRON_TZ=<IANA zone>`。`schedule` 应保持 `restart_policy` 为 `always`（`on-failure`、`never` 只适用于一次性 `command` sidecar）。

```json
{
  "description": "Hourly agent to triage review requests.",
  "builtin": "schedule",
  "args": ["0 * * * *", "agentapi", "new-conversation", "Give me a summary of incoming review requests."]
}
```

#### agentapi、Automations 与运行时数据

- `agentapi new-conversation <prompt>` 新建对话（需 `projectId`）；`agentapi send-message <conversation_id> <prompt>` 向已有对话发消息。
- **Automations** 是每次运行都新开 agent 对话的定时 sidecar：`/automation` 在 `~/.gemini/config/sidecars/<automation-id>/sidecar.json` 创建，并列在 **Automations** 标签页。多两个可选字段：`arguments`（启用时让你填的输入，每项有 `key`、`description` 与 `string_arg` 或 `workspace_arg`，用 `${key}` 引用）和 `agent_permissions`（无人值守时免询问的范围：`workspace_uris`、`access_grants`，如 `command(gh pr list)`、`read_file(/abs/path)`、`mcp(<server>/<tool>)`、`read_url(github.com)`）。
- 运行时数据在 `~/.gemini/antigravity/sidecar_data/<sidecarId>/`：`data/`（持久数据，路径由 `ANTIGRAVITY_EXECUTABLE_DATA_DIR` 提供）、`logs/`（带时间戳的 stdout/stderr）、`events/`（`agentapi` 调用的 JSON 记录）。

排错：sidecar 没跑，先确认 `config.json` 里 `enabled` 为 `true`、ID 与目录名一致、`command` 与 `builtin` 只设了一个，再看 `logs/`。

### 多智能体：先选对方式

让多个 agent 一起干活的方式不止一种，区别在三点：**谁协调**、**工作者之间能否对话**、**是否会改同一批文件**。日常编码和快速提问，直接对默认 agent 说话即可。

| 方式 | 解决什么问题 | 谁协调 | 上下文 | 成本 / 时长 | 计划与状态 |
|---|---|---|---|---|---|
| 自定义 Subagents | 把自包含的支线任务（代码研究、浏览器检查、跑测试）丢到后台 | 父 agent 调用 `invoke_subagent`，子 agent 向父 agent 汇报 | 从空白上下文开始，不继承父对话 | 文档未量化；秒到分钟 | 所有计划；未标注预览 |
| `/boost` | 难而不广：并发 bug、竞态、算法、紧耦合重构 | Orchestrator 驱动“实现、挑战、修复、再验证”流水线 | worker 在隔离的 subagent 上下文中运行 | 文档未量化；秒到小时 | 付费计划；未标注预览 |
| `/swarm` | 广而可切分：多个互不冲突的文件、包、调查方向并行 | 当前对话变成 coordinator，worker 与 verifier 在共享消息板上交流 | 每个 worker 独立对话、干净上下文 | 每个 worker 都是独立 agent，配额消耗明显快于单个对话；分钟到小时 | 所有计划；**预览** |
| `/teamwork` | 跨小时到数天的多里程碑项目：仓库级迁移、仿真、形式化证明 | Sentinel 与 Project Orchestrator 按里程碑计划加审核关卡推进 | 各角色通过 artifact 交接 | 文档未量化；小时到天 | 付费计划；未标注预览 |
| Dynamic Workflows（`/run-workflow`） | 同一件事对一批条目重复做，或分阶段的脚本化流程 | agent 写的 Python 脚本，而不是 agent 之间对话 | 每个 agent 独立对话，只向脚本返回符合 JSON Schema 的结果 | 文档未量化 | 所有计划；未标注预览 |

```text
要让多个 agent 一起干活
      │
      ├─ 只想把一个支线任务丢到后台自己继续干？ ──────────► 自定义 Subagents（或自动委派）
      ├─ 问题很难很深，但不宽（竞态、算法、根因）？ ───────► /boost
      ├─ 任务很宽，能按文件/包/调查方向干净切开？ ─────────► /swarm
      ├─ 要跑数小时到数天，有里程碑和验收关卡？ ───────────► /teamwork（先过一轮范围访谈）
      ├─ 同一动作对一批条目重复，或固定的“先测试再修复”？ ─► /run-workflow
      └─ agent 之间必须边干边协商接口、互审 diff？ ────────► /swarm，而不是 workflow
```

查看运行中的工作：CLI 用 `/agents`（Agent Manager 面板：标识、角色、状态、当前步骤）看 agent，用 `/tasks` 看后台 shell 命令、定时器、`/btw` 后台查询和 workflow 任务；App 在子 agent 面板或 **Overview** 里点进对话。

> 每个 worker、每个 agent 都是独立的模型调用。先确认任务真能拆成互不依赖的部分，再选方式。

### Subagents：把支线任务交给后台 agent

#### 是什么与触发方式

子 agent 是并发运行的后台会话，有自己的角色和初始提示词，**不继承**父对话历史，调用后立即执行，父 agent 可同时调用多个。触发：让父 agent 自动委派（调用 `invoke_subagent`）；CLI 中 `@名字` 给它发消息；`/browser` 把网页任务交给内置 `browser` 子 agent。

`invoke_subagent` 的工作区有三种：`inherit`（与父 agent 共用）、`branch`（隔离的 Git worktree）、`share`（共享目录存储）。内置子 agent：`research`（代码库研究、导航）、`browser`（网页操作、截图、查 console 与网络）、`self`（调用者自身的克隆）。

#### 自定义 subagent：位置与字段

用带 YAML frontmatter 的 `.md` 定义可复用子 agent；也可在会话中用 `define_subagent` 工具创建临时的。

| 位置 | 路径 | 作用域 |
|---|---|---|
| 工作区 | `.agents/agents/<name>.md` 或 `.agents/agents/<name>/agent.md` | 仓库根目录 |
| 全局 | `~/.gemini/config/agents/<name>.md` 或 `.../agents/<name>/agent.md` | 本机所有项目 |
| 插件 | `plugins/<plugin_name>/agents/` | 打包的插件 |

| 字段 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `name` | `string` | 必填 | 唯一标识 |
| `description` | `string` | 必填 | planner 据此判断何时委派 |
| `tools` | `string[]` | `[]` | 允许的工具，如 `view_file`、`replace_file_content`、`grep_search`、`run_command` |
| `mainAgent` | `boolean` | `true` | 为 `true` 时可在聊天界面被选为主 agent（CLI 在 `/agents` 面板选） |
| `subagent` | `boolean` | `true` | 为 `true` 时可通过 `invoke_subagent` 调用 |
| `model` | `string` | `inherit` | `inherit`、`flash` 或 `pro` |
| `commandExecutionPolicy` | `string` | `sandbox` | shell 自动执行策略：`off`、`auto`、`eager`、`sandbox` |
| `mcpServers` | `object[]` | `[]` | 该子 agent 的 MCP 服务器 |
| `skills` / `plugins` | `string[]` | `[]` | skill 路径（如 `skills/my-helper-skill`）或插件依赖 |

`---` 之后的正文就是系统提示词，可用 H1 标题组织。

> 已知问题：`tools` 里写了不存在或拼错的工具名，可能导致子 agent 进程卡住。务必核对确切工具名。文档称增强校验与修复已列入后续更新计划。

#### 可照抄示例

`.agents/agents/code-auditor.md`：

```markdown
---
name: code-auditor
description: Specialized subagent for security audits, static analysis, and code quality reviews.
tools:
  - view_file
  - grep_search
  - run_command
subagent: true
mainAgent: false
model: pro
commandExecutionPolicy: sandbox
skills:
  - skills/security-checklist
---

# System Prompt
You are an expert security auditor and code reviewer. Your primary objective is to inspect source code for security vulnerabilities, memory leaks, and anti-patterns.

# Review Guidelines
1. Perform thorough static analysis without altering files unless explicitly asked.
2. Flag potential injection flaws, unvalidated inputs, or hardcoded secrets.
3. Provide concise, actionable remediation steps for each finding.
```

#### 生命周期、权限与嵌套

| 状态 | 含义 | 要点 |
|---|---|---|
| Running | 正在执行 | App 点 **Stop Subagent**（CLI 按 k）取消；父 agent 也能发消息打断或终止 |
| Idle | 已完成并向父 agent 发了结果 | 收到其他 agent 的消息自动唤醒，保留此前全部上下文 |
| Killed | 被永久终止 | 不能唤醒；其临时 worktree 自动清理；历史记录仍可读（JSONL） |

- agent 之间用唯一对话 ID 互发消息，可联系父、子或已知 ID 的同级 agent，也能读彼此的对话记录。
- 嵌套深度上限 **10 层**。
- 子 agent 继承父 agent 允许的终端命令前缀、文件读写范围和沙箱设置；需要授权的调用会上浮到主界面或子 agent 面板。

#### 操作流程：在 CLI 中监控和干预

1. 输入 `/agents` 打开面板，用 ↑/↓ 选中目标，Enter 进入 Subagent Detail View 看完整推理日志和工具调用，Esc 返回。
2. 想追加指令而不经过父 agent，提示以 `@` 加名字开头：

```text
@research:API-Key-Header-Researcher Also check the tests.
```

3. 子 agent 等批准时，Alt+J 跳到下一个待批准的子 agent，或 Ctrl+K 在提示框上方的内联通知处直接批准。

#### 限制与坑

- 子 agent 从空白开始，委派时把需要的信息写进初始提示。
- shell 命令、测试、`/btw` 查询不在 `/agents` 里，要用 `/tasks`。

### /boost：难而不广的问题

#### 是什么与机制

`/boost <任务>` 启动按需的多 agent 推理流水线（Orchestrator，到 `DeepCoder` 或 `DeepInvestigator` 协调者，再到初始与改进 worker），无需复杂设置或范围访谈，立即开始。**付费计划**，入口为 App 与 CLI。

1. Orchestrator 判断任务是需要实现（`DeepCoder`）还是根因分析（`DeepInvestigator`），把你的原话和相关上下文交给协调者。
2. 初始 worker 探索代码、实现候选修改或追踪根因并跑测试；改进 worker 独立回看原始需求，尝试找出并修复上一位的缺陷，再测试。
3. Orchestrator 对照每条要求抽查代码改动和测试记录；仍有失败就再开一轮；全部通过后给出简明总结。

```prompt
/boost Investigate the race condition in the session cache and implement a thread-safe fix with tests.
```

```prompt
/boost Trace why HTTP request timeouts spike when batch payload size exceeds 2MB, without modifying code.
```

#### 限制与坑

- worker 继承工作区的文件访问规则和命令权限，受保护命令仍会弹出授权；它们在隔离上下文里运行，调试日志不进主聊天。
- 任务“广”而不“难”用 `/swarm`；要跨数天用 `/teamwork`。

### Swarm（/swarm）：命名 worker 在共享消息板上协作

#### 是什么

`/swarm <任务>` 让当前对话变成 **coordinator**：把任务切成互不冲突的片段，为每段加一个命名 worker（如 `@parser`、`@api-tests`；相似的可编号成组，如 `@fix-1` 到 `@fix-4`），通常每 2 到 4 个 worker 配一个 `@verifier` 读 diff、跑测试。入口为 App 与 CLI，**所有计划**可用（商业与企业账号需管理员设置允许）。

> **预览**：Swarm 处于预览阶段。每个 worker 都是独立 agent，swarm 消耗配额比单个对话快得多。

与 subagent 的区别：子 agent 私下向启动它的 agent 汇报；swarm 成员在**同一块消息板**上用 `@name`、`@group`、`@all` 沟通，不发私信。

#### 机制

- 每个 worker 从干净上下文开始，它的公开 brief（负责什么、不能碰什么、什么算完成）贴到板上；verifier 把问题报给写代码的 worker 去修。
- 空闲的 worker 或在等汇报的 coordinator 在板上休眠，**不耗 token**，被 `@` 才唤醒。`@monitor` 在有进展时一直睡，只有所有成员都停下而工作未完成时才发言。
- coordinator 不做技术工作，只分派、评判、向你汇报，所以始终能响应你。
- 成员状态：Active（working）、Waiting（休眠）、Killed（已移出，不能复活，只能新增 worker）。成员配额耗尽会暂停而非重试，直到板上有新帖、切换模型或你给它发消息才继续，不会在配额刷新后自行重启。
- 模型：worker 默认与你的对话同模型；要指定别的，在提示里直接说（如 “run the verifier on a different model than the workers”），必须是你在模型选择器里能选的。
- 工作区：默认共用你的工作区；多个 worker 同时编辑时，coordinator 可给每个 worker 一份独立的仓库工作副本。删除 worker 会删掉它的副本，所以会让它先把改动落地。

#### 操作流程

1. 输入 `/swarm`，按 Tab 插入命令，再描述任务和验证方式：

```prompt
/swarm Add unit tests for the date utilities in src/utils/date.ts and the currency formatter in src/utils/currency.ts, and run npm test to verify.
```

2. App：点回复末尾的 **Open the Swarm UI** 打开侧栏 **Swarm** 窗格，有 **Board**（帖子流）、**Members**（**Active** / **Standing by** / **Retired**）、**Graph**（`@` 关系图，可回放）三个标签。重新打开：从对话头部 **Overview** 面板选 **Plugins** > **Swarm**。
3. CLI：回复会提示运行 `/swarm board`；活动期间提示框下方有状态行（Alt+X 可收起）。Alt+S 或 `/swarm board` 全屏打开消息板（**Roster**、**Posts**、**Message** 三个窗格）：j / k 移动，Enter 显示详情，f 开关 follow，p 按成员过滤，Tab 切窗格，o 打开成员对话，q 关闭。macOS 上 Alt+S 仅在终端把 Option 当 Alt（Meta）发送时有效，`/swarm board` 和状态行任何终端都可用。
4. 运行期间继续和 coordinator 对话，改范围、优先级它会转发给对应 worker。要直接和某 worker 说话：App 从 **Members** 或 **Overview** 的 **Subagents** 打开其对话输入；CLI 以 `@worker名` 开头，如 `@verifier also run the integration tests`。这类消息只发给该 worker，不上板。
5. CLI 中 `/agents` 列出 worker，Alt+J 跳到待批准的 worker，Ctrl+K 直接批准。完成后可让 coordinator 移除没事做的 worker，但**移除是永久的**。

#### 可照抄示例

```prompt
/swarm Add input validation to the three API handlers in src/routes/users.ts, src/routes/orders.ts, and src/routes/products.ts. Give each handler its own worker, add tests next to each handler, and have a verifier run npm test when they finish.
```

```prompt
/swarm Find out why our CLI takes four seconds to start. Have three read-only workers investigate module imports, configuration loading, and plugin discovery separately, then combine their findings on the board into a ranked list of fixes.
```

写法：说清想要什么、怎么验证，拆分交给 coordinator；各块越独立效果越好。

### Teamwork（/teamwork）：长周期、多里程碑项目

#### 是什么

`/teamwork <目标>` 运行一支面向大型软件项目、复杂仿真和深度研究的团队，适合持续数天、跨数十个文件重构、或开放式数学与系统问题。**付费计划**，入口为 App 与 CLI。

| 角色 | 职责 |
|---|---|
| **Sentinel** | 你批准提示后接手的协调者：记录请求、分发任务、汇报进度，收尾前启动 Success Auditor |
| **Project Orchestrator** | 拆里程碑、协调并行轨道；里程碑之间交接给全新的继任者以避免上下文劣化 |
| **Explorers** / **Workers** | Explorers 只读研究；Workers 有终端和文件工具，在互不重叠的轨道上实现与写测试 |
| **Critic** / **Challenger** | Critic 独立代码评审；Challenger 构造对抗性测试、边界与压力输入 |
| **Auditor** | 按完整性模式核对测试证据与真实命令输出，确认测试确实通过而非被 mock 或跳过 |
| **Success Auditor** | 最终端到端验证，通过后 Sentinel 才向你交付 |

执行路径会根据你的提示自动调整：默认 General（分布式编码）；说 “keep it small”、“keep it focused” 走 Iterative coding（单个自包含改动，**不能拆解任务**）；“review this paper” 走 Document Review；“prove theorem X” 走 Math / Proof；明确要求 “Use a very large team of agents” 走 Math / Proof（Large Team）。

#### 两阶段流程

**阶段 1：范围访谈**。主 agent 按“**specify what, not how**”访谈你：范围与目标（演示、生产、评测、探索；受众）、需求、为每条需求约定独立验证（测试套件、指标脚本，或独立 agent 按评分表判定）、验收标准、项目工作目录（默认 `~/teamwork_projects/{PROJECT_NAME}`）。结束时生成可评审的 **prompt artifact**，你确认一次即可开始。

**阶段 2：自主执行**。Sentinel 交给 Project Orchestrator，团队无需逐步提示，通过工作区里的 Request、Project plan、Progress 三类 artifact 交接。

#### 完整性模式

访谈时 agent 会问哪些捷径不允许，映射为一种模式并记入 prompt artifact：

| 模式 | 目的 | 验证行为 |
|---|---|---|
| `development` | 快速迭代 | 宽松，只标记伪造输出和空壳实现；允许复用代码、库、现成框架（默认） |
| `demo` | 可复现的演示 | 适中，另禁止：复制开源核心逻辑、把核心工作委托给外部工具、读测试源码反推期望 |
| `benchmark` | 严格评测 | 最严格：完全独立从零实现，只能用语言标准库 |

#### 示例与限制

```prompt
/teamwork Migrate our REST API service from Express to Fastify, including full test coverage, TypeScript typing, and benchmark validation.
```

- 所有操作在专用项目目录内进行，主仓库保持干净；多个 worker 绝不同时编辑同一个文件；每个子 agent 有自己的 scratch 目录。
- 一定先有范围访谈，不适合要立刻开干的小任务。CLI 中 Alt+J 可跳到待批准的子 agent。
- 文档页首列出的入口包含 IDE 扩展和 IDE，但套餐说明只提 App 与 CLI，以你的界面为准。

### Dynamic Workflows（/run-workflow）：脚本编排的多 agent

#### 是什么

agent 为任务写并运行一段短 Python 脚本来协调许多 agent，适合任务能拆成许多独立条目（每个改动文件一次评审、每个包一次审计）或分阶段（先写失败测试、再修代码），并想要可预测的脚本化控制流。入口为 App 与 CLI，**所有计划**可用。

流程：agent 读内置的 `run-workflow` skill 写脚本；Antigravity 展示脚本请你批准（除非权限预设已允许，如 **Turbo**；要改就拒绝并说明）；后台运行，你可继续在同一对话工作；结束后主 agent 收到日志和输出并总结。workflow 启动的 agent 是完整的 agent，**看不到脚本，也看不到彼此**，所以脚本要把所需信息全写进每个 agent 的提示，并要求返回简短、定义明确的结果。

#### 操作流程

1. 输入 `/run-workflow` 加任务（`/run-work` 加 Tab 可补全），或用自己的话要求“Run a workflow that audits each package in parallel.”。
2. 批准脚本：App 的 **Allow running this workflow script?** 选 **Yes, allow this time**（另有本对话始终允许、始终允许、**No**）；CLI 在 **Requesting permission for:** 下问 **Run this command?**，选 **Yes, run command**。
3. 运行中：App 对话里出现 `workflow · Review · 3/4 agents` 形式的步骤，侧栏 **Overview** 的 **Background Tasks** 也列出，点 agent 卡片在侧栏打开其对话（批准请求、发消息）；CLI 状态栏显示 **1 task(s) · /tasks**，Ctrl+O 展开阶段树，`/tasks`（或 Ctrl+T）选中后按 Enter 进入 **Task Details**，Alt+J / Ctrl+K 跳转或批准待处理请求。
4. 提前停止：停止对话，同时会停掉所有正在运行的 workflow agent。

#### 脚本积木

不必会写 Python，但读懂积木有助于审查脚本、在提示里提具体要求。

| 积木 | 作用 |
|---|---|
| `phase(name)` / `log(*args)` | 开始命名阶段 / 向当前阶段和主 agent 收到的结果添加一行 |
| `agent(prompt, schema, role, type_name, workspace, model)` | 定义 agent。`await agent(prompt, schema)` 运行一次返回结果；`a = agent(...)` 后 `await a(prompt, schema)` 在跨调用持续的对话里运行一个回合 |
| `parallel(items, fn)` | 对每个条目同时运行，按条目顺序返回；**一个失败，其余全部取消** |
| `pipeline(items, fn)` | 一次一个地运行，按条目顺序返回 |
| `await trajectory(...)` | 返回当前对话中 workflow 之前的步骤，可按类型、工具名、索引范围或最后 N 步过滤 |
| `args`、`CONVERSATION_ID`、`STEP_INDEX`、`WORKSPACES` | 传给脚本的参数字典、对话 ID、workflow 步骤索引、工作区文件夹（主工作区在前，脚本在其中运行） |

`schema` 是结果的 JSON Schema，agent 通过 `finish` 工具返回并按 schema 校验；自由文本用 `{"type": "string"}`。可在提示里定制：

| 参数 | 取值 |
|---|---|
| `model` | 默认 `inherit`；`flash_lite` 或 `flash` 适合大量快速检查，`pro` 适合更深推理；档位不可用时报错并列出有效选项 |
| `type_name` | `self`（默认）、内置 `research`（只读研究），或你的自定义 subagent 的名字 |
| `workspace` | `inherit`（默认）；`branch` 给每个 agent 独立 Git worktree；`share` 给每个 agent 新工作副本，共享底层仓库存储 |

#### 可照抄示例

```prompt
/run-workflow List the Python files changed since main, then review each file in parallel for unhandled exceptions using the flash model, and log a one-line verdict per file.
```

agent 通常会写出这样的脚本：

```python
phase("Discover")
files = await agent(
    "List the Python files changed since main. Paths only.",
    schema={"type": "array", "items": {"type": "string"}},
)

phase("Review")
# parallel() runs one agent per file at the same time.
# role titles each agent's card.
reviews = await parallel(
    files,
    lambda f: agent(
        f"Review {f} for unhandled exceptions. Reply with a one-line verdict.",
        schema={"type": "string"},
        role=f"Review {f}",
        model="flash",
    ),
)
for r in reviews:
    log(r)
```

```prompt
/run-workflow Have a persistent Reviewer agent list the bugs in src/billing, fan out one Fixer agent per bug in parallel, and then ask the same Reviewer to confirm that every bug it found is fixed.
```

重复运行：让 agent 把脚本存成工作区里的 `.py` 文件，之后 `/run-workflow Run scripts/review-changes.py`；每次仍请你批准，除非选过“始终允许”。

#### 环境、权限与坑

- 脚本在单独的 Python 进程运行，Antigravity 自带 Python（首次运行会解压）。想用自己的，设环境变量 `ANTIGRAVITY_PYTHON` 为 `PATH` 上的解释器名或完整路径，需 Python 3.8 或更高；macOS 上从 Dock 打开的 App 看不到 shell 里的环境变量。
- 批准针对**那个确切的脚本**，命令前缀规则无法覆盖 Python 脚本；每个 workflow agent 在自己的对话里按你的权限预设请求批准，并一直等你回答。
- 会改同一批文件或同一测试套件的条目用 `pipeline()` 串行；并行改代码用 `workspace` 为 `branch`。agent 之间不能对话，需要协商就改用 `/swarm`。

### /plan 与 /grill-me：先想清楚再动手

#### /plan：先探索、出计划

`/plan` 在改文件前引入规划阶段：只读探索，必要时提问澄清，然后生成可评审的 **Implementation Plan** artifact。所有计划；入口覆盖 App、CLI、IDE 扩展、IDE。

```prompt
/plan Refactor our authentication middleware to support OIDC token verification with JWKS caching
```

不带参数则启动开放式探索，agent 会看当前活动文件和近期对话后提出澄清问题（`/plan`）。对 plan artifact 你可以：行内评论、让 agent 拆细或去掉可选项、用 artifact review policy 配置是否停下等你批准，点 **Proceed** 进入执行（agent 按计划逐步执行并用测试验证每个里程碑）。适合复杂重构、需求含糊、高风险改动（数据库迁移、认证改造、安全补丁）。

#### /grill-me：先被“拷问”

`/grill-me <任务>` 让 agent 开工前采访你：一次问一个问题并给出推荐答案；代码能回答的它自己读代码不问你；一直问到理解一致。所有计划。访谈只改变收集需求的方式，工具权限仍适用。

```prompt
/grill-me Add per-user rate limiting to the bookmarks API
```

- App：问题以卡片出现在消息框上方，推荐答案排第一；选答案或在 **Other (write your answer)** 里自己写，点 **Submit**；**Skip** 跳过；右上角 **X** 取消问卷并停止 agent。2.26.0 之前的版本问题会替换消息框，**Cancel** 在卡片底部。
- CLI：编号列表，↑/↓ 加 Enter 或按编号选择，**Write-in…** 写自己的答案，Esc 跳过。
- 不带任务运行时，它通常先探索工作区再问你想做什么。问完后 agent 带着答案继续做任务。

叠用，得到先访谈后出计划，并且**无论 artifact review policy 如何，都会等你批准后才改代码**（App 点 **Proceed**）：

```prompt
/plan /grill-me Add per-user rate limiting to the bookmarks API
```

### /goal：长任务一直做到验证完成

默认 agent 觉得答完就结束回合。`/goal` 让当前请求进入 **goal mode**：它想提前结束时，**goal stop hook** 会拦截并把已用时间、继续次数和原始请求提醒给它，要求它列出每个交付物并对照真实输出（文件内容、测试结果、构建日志）自查。**没有迭代上限**，只有 agent 宣布完成或你让它停才结束；不会强迫它在依赖的子 agent 或后台任务仍在运行时继续。它只改变停止行为，不改变工具权限和 artifact review policy。

```prompt
/goal Migrate the auth package from session cookies to OAuth2 and make sure all unit and integration tests pass
```

- App：输入框上方有 **Active Goal** 指示，侧栏列出先前目标的 **Active**、**Completed**、**Cancelled** 状态。CLI：用 `/tasks` 监控它启动的后台命令、定时器和子 agent。
- 一个对话同一时间只有一个活跃目标，新的 `/goal` 会取消旧的。取消：在后续消息里让它停，例如 `Stop the current goal. I'll take it from here.`。
- 写好目标：写明证据（“all tests in `packages/api` pass”、“`npm run build` exits with zero errors”、“p95 latency in the benchmark is under 50 ms”）、限定范围；大改动先 `/plan` 或 `/grill-me` 再交给 goal mode；想定期收到进度，配合 `/schedule`。

### /schedule 与 /automation：定时与自动化任务

| | `/schedule` | `/automation` |
|---|---|---|
| 适合 | 计时器、CI 轮询、任务过程中的回访 | 每日报告、夜间维护、分诊等周期任务 |
| 入口 | App、CLI、IDE 扩展、IDE | 仅 App |
| 计划形式 | 一次性计时器或 cron 循环 | 以 `CRON_TZ=<IANA 时区>` 固定的 cron 表达式 |
| 生命周期 | 当前对话 | 跨会话、跨重启，直到你关掉 |
| 上下文 | 唤醒已有 agent，带完整历史 | 每次运行开新对话，不记得之前的运行 |
| 存放位置 | agent 发起的一次工具调用 | `~/.gemini/config/sidecars/<automation-id>/sidecar.json`，在 **Automations** 标签管理 |
| 权限 | 当前对话的权限 | 测试运行后记录的、有作用域的 `agent_permissions` |

没打开任何对话也要持续运行，用 `/automation`；想让当前对话里的 agent 计时后醒来或边干活边轮询，用 `/schedule`。

#### /schedule

agent 用内置 `schedule` 工具创建计时器或计划，作为当前对话里的后台任务：创建后结束回合而不是 shell `sleep`，触发时把你的指令当高优先级通知处理。可用自然语言、时长或五字段 cron 表达式描述。

```prompt
/schedule in 10 minutes, check whether the deploy to staging finished and summarize any errors
```

```prompt
/schedule every 2 minutes for the next 30 minutes, poll the CI status for PR #482 and tell me when it goes green
```

两种互斥模式，可直接点名参数：

| 模式 | 参数 |
|---|---|
| 一次性计时器 | `DurationSeconds`（等待秒数）、`Prompt`（触发时的指令）、`TimerCondition`（可选提前取消：`never` 默认、`any` 任何收到的消息、或某个子 agent / 后台任务 ID） |
| 循环 cron | `CronExpression`（五字段，如 `*/5 * * * *`）、`Prompt`（每次触发的指令，末尾带迭代序号）、`MaxIterations`（可选，默认无限）、`IsDaemon`（可选，`true` 时当前任务结束后仍继续触发） |

带 `TimerCondition` 时，匹配的消息（如子 agent 完成）会提前取消计时器并唤醒 agent，例如 `/schedule wait up to 5 minutes for the test subagent to report back, then check its results either way`。只有独立的常驻任务才设 `IsDaemon`。管理：让 agent 列出或取消；CLI 在 `/tasks` 面板里 k 终止、x 清除已结束项、Enter 看详情；App 中显示为 **Timed** 或 **Scheduled** 步骤（**Pending** 到 **Fired**，或 **Active** 到 **Completed**）。

#### /automation

后台 automation（以前叫 Scheduled Tasks）按周期运行一段 agent 提示。`/automation` 让 agent 带你创建，不必手写 `sidecar.json`；也可在 **Automations** 标签选 **Create with Prompt**。

1. 输入 `/automation` 加描述，或单独输入走引导：

```prompt
/automation every Monday at 9am Pacific time, check my open pull requests and list any with failing checks
```

2. agent 问节奏（**Hourly**、**Daily**、**Weekly**、**Custom cron schedule**）和日期时间，确认时区并转成带前缀的五字段 cron，如 `CRON_TZ=America/Los_Angeles 0 9 * * 1`。
3. agent 把任务整理成不依赖前文的自包含提示词，创建 automation，并问是否现在测试运行：选是就在当前对话里跑一次，把实际需要的权限记入 `agent_permissions`。
4. 新 automation 默认在 **Paused** 下；到 **Automations** 标签核对权限并打开，之后可用 **Run Now** 立即运行。

默认用 Flash，创建或编辑时可选 Flash Lite、Flash 或 Pro。`arguments`、`agent_permissions` 等字段见官方 Sidecars 文档。

> 无人值守运行会自动拒绝任何通常需要你批准的操作，所以那次测试运行很重要。agent 被要求只写有作用域的授权，如 `command(gh issue list)`、`read_file(/abs/path)`、`write_file(/abs/path)`、`mcp(<server>/<tool>)`、`read_url(github.com)`，而非裸授权或通配授权。开启前请逐条核对。

### /learn：把这次会话学到的东西存成 rule 或 skill

agent 不记得上一次对话。`/learn` 让它回顾近期交互（你的纠正、屡次忽略的约束、终于跑通的多步设置），分析明确的纠正信号（“no”、“instead”、“that failed”），分类为 **rule**（护栏、约束）或 **skill**（可复用的多步骤流程、速查表），优先更新已有项而非新建，**先提议再写入**。没有可复用内容时它会直说并退出。

```text
/learn
```

```prompt
/learn always run vitest with --pool=forks in this repo
```

带指令时跳过发现步骤，仍会分类并确定作用域。强制的提议流程：

1. 检查你已有的 rules 和 skills（或内置 customizations 指南），让新文件符合现有 frontmatter 与目录约定。
2. 生成 `learning_proposal.md` artifact：分类、理由、打算做的精确文字增删或 diff。
3. 你行内评论调整范围、措辞或位置。
4. 只有你明确批准后才创建或更新文件。

| 类型 | 工作区作用域 | 全局作用域 |
|---|---|---|
| Rule | 工作区文件夹或 `.agents/` 中的 `AGENTS.md` 或 `GEMINI.md`，或 `.agents/rules/<name>.md` | `~/.gemini/AGENTS.md`、`~/.gemini/GEMINI.md`、`~/.gemini/config/AGENTS.md`、`~/.gemini/config/GEMINI.md`，或 `~/.gemini/config/rules/<name>.md` |
| Skill | `.agents/skills/<name>/SKILL.md` | `~/.gemini/config/skills/<name>/SKILL.md` |

`rules/` 目录里的文件需要带有效 `trigger` 的 YAML frontmatter；`AGENTS.md` 和 `GEMINI.md` 不用 frontmatter，始终生效。skill 可作为 `/<skill-name>` 命令运行。

### /btw：边干边旁问

`/btw <问题>` 在 agent 工作时问一个快问题。旁问能读文件、搜工作区核实答案，它做的一切都不写进主对话，也不会打断 agent。机制：

1. 分叉对话：新对话以你的历史副本开头，主 agent 继续运行，分叉内容不写回。
2. 要求模型只读：除非你明确要求，不编辑文件、不改工作区或外部状态、不启动子 agent 或 workflow、不安排后台任务。
3. 沿用你的模型、权限、终端沙箱、MCP 服务器和管理员策略。“只读”是对模型的指示，**不是单独的沙箱**，`/btw fix that typo` 这样的请求可以在你的权限内做到。
4. 一个 agent 回合，调用工具直到答完或你停止，无时间限制。

| 行为 | App（`v2.32.0+`） | CLI（`v1.5.0+`） |
|---|---|---|
| 答案位置 | 侧栏 **Side Question** 标签 | 提示框上方的 `/btw` 卡片 |
| 需批准的工具调用 | 批准请求出现在标签里 | 自动拒绝 |
| 追问 | 在标签里继续发消息 | 不支持，每个 `/btw` 都是新分叉 |
| 保留或丢弃 | **Promote to Conversation** 或删除 | 关闭卡片；答案在本会话卡片历史里保留 |

```text
/btw Where's the retry logic for API calls, and what's the max retry count?
```

- App：需已发过至少一条消息。也可从侧栏 **New Tab**（**+**）选 **Side Question**，或选中对话文字点 **Ask /btw**。关闭标签不等于删除，可从 **Overview** 的 **Side Questions** 重开；**Delete Side Question** 不可撤销。远程对话和早于 v2.32.0 的 App 里旁问不能调用工具，答案在浮动面板。
- CLI：每张卡片一个问题；一次只能一个；仅交互会话，无头模式（`-p`、`--print`）和远程 agent 会话不可用。旁问常用 `grep`、`find`，若它们需批准会被拒，可添加 `command(grep)`、`command(find)` 允许规则。不带参数的 `/btw` 重开最近一张卡片，`/copy btw` 复制最新答案。

### /browser：把网页任务交给 browser 子 agent

`/browser <任务>` 让主 agent 把请求委托给独立上下文里的 `browser` 子 agent：打开页面、操作运行中的应用、检查 console 与网络、截图、验证视觉修复。请写清结束条件，如 “report whether X renders”。管理员可禁用浏览器访问，此时命令菜单里没有 `/browser`。

| | 内置浏览器 | Chrome DevTools MCP |
|---|---|---|
| 适用 | App，个人 Google 账号，`v2.30.0+` | CLI、IDE（`v2.6.0+`）、App 的商业/企业账号 |
| 运行环境 | App 侧栏内置浏览器，无需外部设置；并行标签、实时光标、一键 **Stop** 接管、登录交接 | 控制本机已运行的 Google Chrome（`v144+`），自动启动 Chrome DevTools MCP 服务器 |
| 额外能力 | 这类账号上主 agent 也会从自然语言自动启动 `browser` 子 agent | 性能 trace、Lighthouse、堆快照、节流、`evaluate_script`、可选录制（CLI `v1.5.9+` 为 MP4，其余为 WebM） |

```prompt
/browser Open http://localhost:3000, resize to a 390px-wide mobile viewport, verify the navigation drawer opens and closes, and capture a screenshot of each state
```

使用 Chrome 的场景，先设置：

1. CLI 或 IDE 需安装 Node.js，确保 `npx` 在 `PATH` 上（App 自带运行时）。
2. 打开 Google Chrome（`v144+`），必须与工作区在**同一台本机**（远程 SSH、Dev Container、云工作站无法连你本机的 Chrome）。它不会替你启动 Chrome，Chrome 也不必登录。
3. 访问 `chrome://inspect/#remote-debugging`，打开 **Allow remote debugging for this browser instance**。
4. 运行命令。子 agent 连接时 Chrome 会请求授权，需在 60 秒内点 **Allow**；CLI 中 Alt+J 或 `/agents` 切到子 agent 对话，Ctrl+K 批准站点请求。

权限：App（`v2.30.0+`）在 **Settings** > **Browser** > **Browser Subagent**，有 **Browser Actuation Rules**（**Execute URLs**、**Download files**）和仅 Chrome DevTools MCP 适用的 **Browser Javascript Execution Policy**（**Request Review** 默认、**Always Proceed**、**Disabled**）。IDE 在状态栏点 **Antigravity** 后选 **Open Settings** > **Browser**。

### /compact：按需压缩上下文

`/compact` 把对话总结成结构化检查点，让 agent 在更小的上下文里继续，不必新开对话或等自动压缩。入口为 App（`v2.21.0+`）与 CLI（`v1.2.15+`）。压缩不调用工具、不改磁盘文件；保留关键决策、文件路径、错误与修复、附件、活跃后台任务、存活子 agent 和会话 artifacts；rules、skills、MCP 照常生效；较早消息仍在滚动历史里，只是之后发给模型的内容变短。

```text
/compact
```

压缩后停止，你的下一条消息是在压缩后上下文上运行的第一条。要压缩后继续，在后面加消息，它既引导总结又由 agent 在压缩后回答：

```prompt
/compact Keep the OAuth token refresh findings, and now add unit tests for the new middleware.
```

- 只有**开头**的 `/compact` 才触发；出现在其他文字或附件之后（如 `"explain /compact"`）会被当作普通提示文字。
- App 状态行显示 **Compacting**，点 **Stop** 取消，取消不写检查点。CLI 显示 `Compacting conversation… (<elapsed> · esc to cancel)`，Esc / Ctrl + C 取消；完成后有 `Conversation compacted` 分隔线与摘要行（如 `context 182.4k → ~31.2k`），估算值与 `/context` 一致。
- 还没有完成过回复时，裸 `/compact` 输出 `Nothing to compact yet.`。无头模式恢复对话时也可用，如 `agy -c -p "/compact Now run the integration test suite"`，除非设置了 `--disable-slash-commands`（或 `"disableSlashCommands": true`）。

### 命令对比表：我该用哪个

| 命令 | 一句话 | 参数形式 | 对上下文的影响 | 入口与限制 |
|---|---|---|---|---|
| `/plan` | 先探索、出可评审的计划再动代码 | `/plan [目标]` | 主对话内产出 Implementation Plan artifact | 所有计划；App、CLI、IDE 扩展、IDE |
| `/grill-me` | 一次一问并附推荐答案，把需求聊透 | `/grill-me [任务]`，可写成 `/plan /grill-me ...` | 主对话内多轮问答 | 所有计划；App、CLI、IDE 扩展、IDE |
| `/boost` | 多 agent 迭代推理 | `/boost <任务>` | worker 在隔离上下文里 | 付费计划；App、CLI |
| `/goal` | 长任务不提前收工，直到证据证明完成 | `/goal <带可验证标准的目标>` | 仍在当前对话，只改停止行为 | 文档未标注套餐限制；App、CLI、IDE 扩展、IDE |
| `/schedule` | 当前对话里的定时器与 cron 轮询 | `/schedule <何时、做什么>` | 唤醒已有 agent，带完整历史 | 文档未标注套餐限制；App、CLI、IDE 扩展、IDE |
| `/automation` | 跨会话的后台周期任务 | `/automation [描述]` | 每次运行是全新对话 | 仅 App |
| `/learn` | 把教训存为 rule 或 skill | `/learn [要记的内容]` | 先出 `learning_proposal.md`，批准后才写文件 | 文档未标注套餐限制；App、CLI、IDE 扩展、IDE |
| `/btw` | 只读旁问，不污染主对话 | `/btw <问题>` | 在对话副本里进行，不写回 | App `v2.32.0+`、CLI `v1.5.0+` |
| `/browser` | 网页任务交给 `browser` 子 agent | `/browser <任务与结束条件>` | 子 agent 在独立上下文里 | 取决于账号与入口，见上文 |
| `/compact` | 把历史压成检查点，继续同一任务 | `/compact [引导消息]` | 较早回合替换为检查点 | App `v2.21.0+`、CLI `v1.2.15+` |

容易混的几组：

| 相近的命令 | 区别 |
|---|---|
| `/plan` 与 `/grill-me` | `/plan` 产出写成文的计划，批准后才动代码；`/grill-me` 是访谈，聊完 agent 带着答案继续做任务；叠用则先访谈后出计划，且一定等你批准 |
| `/goal` 与 `/boost` | `/goal` 只改变“何时停”，不增加 agent；`/boost` 引入多 agent 流水线，侧重推理深度 |
| `/schedule` 与 `/automation` | 前者属于当前对话，后者独立于对话、跨会话持续 |
| `/btw` 与 `/compact` | `/btw` 问一个不进历史的问题；`/compact` 缩小之后每个回合要带的历史 |
| `/btw` 与 subagent | 旁问是你主动发起、在对话副本里的只读回合；子 agent 是父 agent 委派、从空白上下文开始的后台会话 |

---

## Part IV：Antigravity CLI（agy）

### Antigravity CLI 选型与概览

Antigravity CLI 是跑在终端里的 TUI（文本界面）入口，命令名是 `agy`。它与 App、IDE 扩展共用同一套智能体后端和登录体系。入门只需记住：`agy` 进入对话，`agy -p "..."` 跑完就退出，`agy -c` 接着上次聊，`Shift + Tab` 切换执行模式，`Esc` 随时打断。

| 我想…… | 用这个 |
|---|---|
| 在终端里和智能体对话、审阅它的改动 | `agy`（交互式 TUI） |
| 在脚本、CI、Git hook 里拿到智能体的输出 | `agy -p "..."`（headless / print 模式） |
| 在 Zed、JetBrains、Android Studio、Xcode 里用同一个智能体 | `agy acp`（由编辑器拉起，不用手动运行） |
| 少被打断，让安全的编辑和命令自动通过 | `auto` 模式（见“选择执行模式”） |

```text
agy
├─ 交互式 TUI：提示框 → 智能体回合 → 审阅 diff / 产物 → 继续
│   ├─ 模式：default / accept-edits / auto / plan（Shift + Tab 循环）
│   ├─ 面板：/agents /codesearch /context /diff /resume /permissions /usage ...
│   └─ 定制：/config、keybindings.json、statusLine、title、Vim 模式
├─ headless：agy -p → text / json / stream-json
└─ ACP：agy acp → 编辑器里的聊天面板
```

### 安装、登录与启动

用官方安装脚本安装（macOS / Linux、Windows PowerShell、Windows cmd 各有一条，见快速入门页）。默认位置：macOS / Linux 为 `~/.local/bin/agy`，Windows 为 `C:\Users\<username>\AppData\Local\agy\bin`；装完用 `agy --version` 验证。脚本可选标志：`--skip-aliases`（不清理旧的 `agy` / `antigravity` 别名）、`--skip-path`（不改 shell 配置里的 `PATH`）。

首次登录与认证方式：

1. 本机启动 `agy`，CLI 先尝试读操作系统密钥环（Apple Keychain、Linux Secret Service/dbus、Windows Credential Manager）；有有效令牌就静默登录。
2. 没有已保存会话时，CLI 自动打开默认浏览器；个人账号选 **Google OAuth**，企业或按量计费选 **Use a Google Cloud project**。
3. 通过 SSH 登录时，CLI 打印一个授权 URL，你在本机浏览器打开并登录，把页面显示的授权码贴回远程终端。
4. 想用自己的 Gemini API Key：在 `~/.gemini/antigravity-cli/settings.json` 里设 `"modelProvider": "gemini"`，并 `export GEMINI_API_KEY="..."`。只设环境变量不生效；`modelProvider` 为 `gemini` 却没设该变量时 CLI 无法启动。这种方式适合无浏览器的 headless / CI 环境。
5. 企业 ADC：先 `gcloud auth application-default login --project {GCP_PROJECT}`，再 `export AGY_ADC_AUTH=true`（可选 `GOOGLE_CLOUD_QUOTA_PROJECT`、`GOOGLE_CLOUD_LOCATION`）。
6. 退出登录：在提示框输入 `/logout`，清除密钥环中保存的令牌（API Key 方式下 `/logout` 无效）。

```bash
agy                             # 在项目目录启动交互式会话
agy -i "explain this project"   # 带初始提示词启动并继续交互
```

- 会话按**当前工作目录**划分：从哪个目录启动，就只看到和恢复该目录相关的会话。
- 关闭 CLI 时，它会自动打印恢复该会话所需的完整命令。
- 项目内的规则文件：在工作区根目录放 `GEMINI.md` 或 `AGENTS.md`，智能体启动时自动读取（全局规则在 `~/.gemini/GEMINI.md`）。

### 设置与使用技巧

设置保存在纯 JSON 文件 `~/.gemini/antigravity-cli/settings.json`。输入 `/config`（别名 `/settings`）打开全屏设置面板：选中一项后改选项或输入文本，选择会立即写盘。

- 部分设置可被启动标志覆盖（如 `--sandbox`、`--dangerously-skip-permissions`）；设置面板会标出覆盖来源，例如 _Sandbox Mode on overridden by `--sandbox`_。磁盘上的值仍可改，但本次会话在重启前以命令行为准。
- 键省略时使用默认值。想在 VS Code 等编辑器里获得补全和校验，可在文件顶部加 `"$schema"`（具体地址见官方设置页）。

常用小技巧：`@` 补全路径；连按两次 Esc 清空提示框（无流式输出时）；提示以 `!` 开头直接执行终端命令；`?` 或 `/help` 查看帮助；在 `/config` 把 verbosity 设为 low 可减少工具调用刷屏。

### 快捷键与自定义键位

键位文件是 `~/.gemini/antigravity-cli/keybindings.json`，可输入 `/keybindings` 打开交互式编辑器，也可直接改文件；删除该文件即恢复默认。一个动作可绑多个键，设为空数组 `[]` 即禁用；文件格式有误时，CLI 对有效部分生效，损坏的动作回退默认。`cli.exit`、`cli.escape`、`cli.enter` 不能被禁用。这些快捷键只对 CLI 生效，在 Antigravity App 里无效。

| 动作 | 按键 |
|---|---|
| 插入换行 | Shift + Enter / Ctrl + J / Alt + Enter |
| 停止流式输出、关闭菜单、清空提示 | Esc |
| 打断当前回合；空闲时连按两次退出 | Ctrl + C |
| 循环执行模式 | Shift + Tab |
| 在外部编辑器里编辑提示 | Ctrl + G |
| 打开产物审阅面板 | Ctrl + R |
| 跳到等待确认的子智能体 | Alt + J |
| 语音听写 | F5 |
| 粘贴文本或媒体 | Ctrl + V / Alt + V |

### 提示与交互

提示框固定在终端底部，处理普通输入、多行文本和媒体粘贴。

1. 输入指令，按 Enter 提交；智能体分析当前目录并开始制定执行计划。
2. 智能体跑偏或循环时，按 Esc 立即中断；Esc 是全局的“逃生键”，同时关闭浮层并回到干净的提示框。
3. 多行输入有三种方式：Shift + Enter 或 Ctrl + J；Apple Terminal 不转发 Shift + Enter，改用 Option + Enter（需在终端偏好设置里勾选 **Use Option as Meta key**）；或在行尾输入反斜杠 `\` 再按 Enter（反斜杠会被自动去掉并换行）。
4. 长提示可按 Ctrl + G，在 `$EDITOR`（或 `editor` 设置指定的编辑器）里编写，保存退出后内容自动回到提示框。
5. 排队消息：智能体工作时发出的消息，默认排队到当前回合结束（`queuedMessages` 为 `"queue"`）；设为 `"send-immediately"` 则打断并立即发送。

#### @ 引用与媒体

| 操作 | 结果 |
|---|---|
| 输入 `@` 并继续键入 | 弹出路径建议，模糊匹配（`@bkjs` 能找到 `src/routes/bookmarks.js`），每项标明 File 或 Directory 及完整路径 |
| ↑ / ↓、Enter、Tab、Esc | 移动、插入路径并加空格、插入路径（目录则展开其内容）、关闭建议但保留已输入内容 |
| 以 `~/` 或 `/` 开头，如 `@~/Develop/` | 引用工作区外的路径；引用不等于授权，权限设置仍决定能否读取 |
| 提示以 `@` 加子智能体名开头 | 直接给该子智能体发消息（名称形如 `<type>:<role>`，需与列表完全一致，不分大小写；否则发给父智能体） |

- 引用文件或目录不会把内容贴进提示：智能体收到的是完整路径，需要时再自行读取。CLI 里只能引用文件和目录。
- 从列表选出的路径以文本留在提示里（如 `@src/routes/bookmarks.js`）；自己手打或改过的路径按普通文本发送。
- 粘贴媒体：Ctrl + V 粘贴截图或录屏。支持的图片格式：PNG、JPEG、GIF、WebP、BMP、TIFF、SVG；视频格式：MP4、MOV、WebM、AVI。

### 选择执行模式

执行模式决定智能体修改文件、运行命令前是否停下来问你。

| 模式 | 行为 | 适合 |
|---|---|---|
| `default` | 改动或新建文件前给出交互式 diff 审阅，运行终端命令前先问 | 常规开发、敏感代码改动、谨慎重构 |
| `auto` | 自动批准工作区内的文件编辑；终端命令默认在沙箱里运行；沙箱外的命令自动评估，安全且可逆的直接运行，不可逆或高风险的仍弹确认 | 想少被打断、又保留对不可逆操作的把关 |
| `accept-edits` | 自动批准工作区内的文件创建和编辑，运行终端命令仍要问 | 在可信代码上反复迭代，但每条 shell 命令仍想亲自批准 |
| `plan` | 自动给提示加上 `/plan` 前缀：先只读调研，输出结构化方案，批准后再写代码 | 陌生架构探索、复杂多步骤特性设计 |

#### auto 模式的限制

- 版本：macOS / Linux 需 `v1.3.5+`，Windows 需 `v1.4.4+`；需要终端沙箱可用；**不适用于商业版或企业版账号**（含 Antigravity in Gemini Enterprise）。
- Windows 上沙箱需要 Windows 11 24H2 及以上并安装 2026 年 8 月更新或更晚。Windows 10、更早的 Windows 11、以及阻止非特权用户命名空间的 Linux 主机或容器里，`auto` 不提供。
- 不可用时：`auto` 不出现在 Shift + Tab 循环和 `/config` 里；`"agentMode": "auto"` 或 `--mode=auto` 会退回 `default`；headless 下 `--mode=auto` 会打印警告并继续用 `default`。
- 自 `v1.5.7` 起，`agentMode` 未设置时，`agy` 在 `auto` 可用的账号和平台上以 `auto` 启动，否则以 `default` 启动。

#### 切换与持久化

```text
Shift + Tab 循环：default → accept-edits → auto → plan → default
（auto 不可用时：accept-edits 直接到 plan）
```

1. 在提示框按 Shift + Tab，看右下角状态栏的模式标记和提示框占位文字确认当前模式。
2. 在 `default` 模式等待文件编辑确认时，按 Shift + Tab 切到 `accept-edits` 或 `auto`，可自动批准待处理的编辑（含排队中的子智能体写入）。
3. 要回到 `default`：连按 Shift + Tab 直到状态栏标记消失，或在 `/config` 把 **Agent Mode** 设为 `default`。
4. 持久化：在 `settings.json` 设 `agentMode`（`""` 为 default，另有 `"accept-edits"`、`"auto"`、`"plan"`）；单次覆盖用 `agy --mode=accept-edits`、`agy --mode=auto` 或 `agy --mode=plan`。

#### default 模式的审阅界面

- 文件编辑确认 “Accept this file edit?”：按 1 接受；按 2 拒绝；高亮拒绝项后按 Tab 输入新指示再回车，可在拒绝的同时告诉智能体怎么改；按 f 打开全屏、可滚动的 diff。待确认的编辑不会因 Esc 消失。
- 新建文件（Create file）的确认显示路径和纯新增的 diff；文件在工作区外时多出 **Yes, and always allow non-workspace access** 选项；这里按 Esc 表示拒绝。

> 常见误区：Shift + Tab 里不会出现 `sandbox`（它是隔离权限设置，不是模式，用 `--sandbox` 或 `/config` 的 **Sandbox Mode**）；`/planning`、`/fast` 已在 `1.1.0` 移除；`agy` 的标志是 `--mode`，不是 `--permission-mode`。
>
> 优先级：`/permissions` 里显式配置的 allow / ask / deny 规则，以及 `--dangerously-skip-permissions`（`always-proceed`）或 `strict` 等全局设置，在所有模式下仍然优先。

### 会话管理

- 会话与启动目录绑定，避免跨项目的上下文污染。
- `agy -c`（`--continue`）恢复当前工作区最近的会话；`agy --conversation <conversation-id>` 按 ID 恢复。`-c` 通过本地缓存 `~/.gemini/antigravity-cli/cache/last_conversations.json`（工作区绝对路径 → 最近会话 ID 的 JSON 映射）查找；会话已被删除或没有记录时，开启新会话。
- `/fork`（别名 `/branch`）复制当前会话到新的独立会话，分配新 ID、保留当前工作区，并立即切到新分支；实验失败后用 `/resume` 回到原会话。

> 注意：fork 复制的是**对话线程**，不是本地 Git 检出。要隔离文件，请配合 Git 分支或先 stash。

- `/compact [message]` 把此前的对话压缩成检查点；`/compact Now add unit tests for the new handler` 可压缩后接着继续。随后用 `/context` 看新的 token 构成。
- `/rewind`（别名 `/undo`）回退到之前的某条消息；`/rename <name>` 重命名当前会话。

#### 会话选择器

`/resume`（别名 `/switch`、`/conversation`）按最近时间列出会话：直接输入可按标题、首条提示首行或会话 ID 过滤；↑/↓ 浏览，←/→ 翻页，Enter 载入；F2 重命名（Enter 保存、Esc 取消）；F4 删除（Enter 或 Y 确认，Esc 或 N 取消）；Tab 切换 **CLI** / **Other** 标签；Esc 先清除搜索，搜索为空时关闭。

从 Antigravity App 导入：在选择器按 Tab 切到 **Other** 标签，高亮在 App 中开始的会话，按 Enter，再确认 `[Import this conversation? (y/n)]`，CLI 会复制其历史、上下文和工具轨迹。`pickerGrouping` 设为 `"grouped"` 可让选择器按工作区分组（默认 `"none"`，按最近时间一个列表）。

### 产物审阅

产物（artifact）是智能体为完成任务产出的结构化交付物，包括实施方案等 Markdown、代码 diff、架构图、图片和视频。审阅产物是一种异步“共同驾驶”：你在关键里程碑处检查方案或改动、加行内评论、纠偏，而不必盯着每一次工具调用。

智能体产出或修改文件后，状态栏会提示 `/artifact to review`。按 Ctrl + R（或输入 `/artifact`）打开产物选择器。选择器内：↑/↓ 移动；h / l（或 ← / →）在行内按钮 open / approve / reject 间切换；p 内联预览约 12 行；y / n 批准或拒绝高亮项；Shift + A / Shift + R 批量批准或拒绝全部待处理项；Enter 执行当前按钮（`open` 打开详情查看器）；Esc 保存审阅状态、把结果回传给智能体并回到提示框。

- 可操作的代码、配置和方案文件需要你批准；图片和视频（PNG、JPG、WebP、SVG、MP4、WebM 等）归到可折叠的 **Media** 抽屉，选中后 Enter 用系统媒体查看器打开。
- 详情查看器：j / k 或方向键滚动，g 到顶部，Shift + G 到底部，l 开关行号。按 c 在光标所在行写多行评论，Esc 保存提交（行上出现 `💬`），d 删除该行评论。
- Mermaid 图：按 m 在三种渲染间循环：Kitty 图形、ASCII 框图（默认）、原始代码；Kitty 模式下用 Ctrl + = / Ctrl + - 缩放。
- `artifactReviewPolicy` 设置控制智能体是否等你审阅：`"asks-for-review"`（默认，总是等）、`"agent-decides"`（目前等同前者）、`"always-proceed"`（从不等）。它不影响文件编辑。

### 行内审阅：/diff 与 /codesearch

`/diff` 打开全屏交互式 diff 查看器，自动支持 Git、Mercurial（Hg）和 Jujutsu（JJ）。用 ← / →（或 Tab）在三个视图间切换：**All Changes**（所有已修改和未跟踪文件）、**Per Turn**（智能体每个回合引入的改动，适合逐步复盘）、**Commit Tree**（提交图，高亮提交按 Enter 列出其改动文件）。

在文件详情视图里：N / Shift + N 跳到下/上一处改动，`/` 搜索，F 切换“仅改动”与“完整文件”，C 在当前行写评论，D 删除评论，Esc 返回。退出时若有未发送的评论，会出现 **Unsent Comments** 屏：按 Y 把评论作为你的下一条提示发给智能体并退出，按 N 丢弃。

`/codesearch <query>`（别名 `/cs`、`/search`）打开全屏 Code Search 面板，在工作区内搜索并按文件分组显示命中及上下文：

| 查询写法 | 含义 |
|---|---|
| `/codesearch UserSession` | 默认是正则；全小写时不区分大小写，含大写字母则区分（smart case） |
| `/codesearch f:store.go Session` | `f:`（别名 `file:`、`path:`）后接 glob，限定文件路径 |

面板内：↑/↓ 在命中间移动，←/→ 跳到上/下一个文件组，Enter 在内置文件查看器中打开命中行，Ctrl + G 用外部编辑器打开。在文件查看器按 C 给某行写评论，按 Esc 离开时选 Y 发送，评论会以 `<file>:<line>: <comment>` 的格式作为下一条消息交给智能体。

### Vim 模式

Vim 模式默认关闭，开启后替换所有多行输入框的编辑模型：提示框、`/diff` 的评论编辑器、产物审阅面板的评论编辑器。

在 `settings.json` 设 `"editorMode": "vim"`（`"vimInsertFirst"` 默认 `false`），或在 `/settings` 里把 **Editor Mode** 选为 `vim` 并设置 **Insert First**。

> 注意：`editorMode` 与 `editor` 无关。`editor` 只决定 Ctrl + G 启动哪个外部程序；把 `editor` 设为 `"vim"` 只会在另一个窗口打开 Vim，不会让提示框变成模态编辑。

| 项目 | 说明 |
|---|---|
| 起始模式 | 默认从 NORMAL 开始；`i` 进入插入，Esc 或 Ctrl + C 回 NORMAL |
| 状态栏徽标 | NORMAL 无徽标；INSERT 为 `-- INSERT --`；VISUAL 为 `-- VISUAL --`；VISUAL LINE 为 `-- V-LINE --` |
| `vimInsertFirst: true` | 每个新提示从 INSERT 开始，且裸 Enter 直接提交 |
| 编辑 | `x`、`r{char}`、`D`、`C`、`p`、`P`；算子 `d` `c` `y`；文本对象 `iw aw`、引号、括号等 |

限制：不支持计数前缀（`3dd` 只删一行，多行请用 `V` 选择后再操作）；没有 `.` 重复；只有一个未命名寄存器；NORMAL 下 `u` / `U` 是撤销 / 重做；Ctrl + R 不是重做，而是打开产物审阅。提交键可在 `keybindings.json` 里通过 `vim.insert.submit`、`vim.insert.insert_newline`、`vim.normal.submit` 三个动作调整（NORMAL 下的 Enter 永远提交）；动作、算子、文本对象不能重映射。

自定义状态栏会取代内置的模式徽标：要么设 `stack_with_default: true` 叠在内置状态栏下方，要么读取 JSON 里的 `vim.mode`（`NORMAL`、`INSERT`、`VISUAL`、`VISUAL LINE`；Vim 模式关闭时该字段不存在）。

### 状态栏与终端标题定制

状态栏位于提示框下方，显示智能体周期、工作区、上下文窗口占用和后台任务。`/statusline` 管理它：无参数为开/关切换；`on`（`enable`）、`off`（`disable`）显式设置；`/statusline <路径>` 配置自定义脚本并立即生效；`delete`（或 `reset`）删除自定义命令、恢复内置状态栏；`help` 查看用法。

#### 状态栏脚本

自定义脚本写在 `~/.gemini/antigravity-cli/settings.json` 的 `statusLine` 块：

```json
{
    "statusLine": {
        "type": "command",
        "command": "~/.gemini/antigravity-cli/statusline.sh",
        "padding": 0,
        "stack_with_default": false
    }
}
```

- 智能体状态变化时，CLI 用 `sh -c`（Windows 用 `cmd /c`）运行你的命令，把会话状态 JSON 通过 `stdin` 传入，把 `stdout` 显示成状态栏；支持多行输出和 ANSI 颜色，`command` 开头的 `~` 会展开为主目录。
- 属性：`command`；`type`（默认 `"command"`，也是唯一支持的值）；`padding`（状态栏上方空行数，默认 0）；`enabled`（默认 `true`，设 `false` 则显示内置状态栏，命令保留）；`stack_with_default`（默认 `false`，设 `true` 则叠在内置状态栏下方）。
- 必须把 `statusLine` 设为对象，不能是 `true` 或 `false`；布尔值会被当成没有 `command` 的对象，脚本不会运行。
- 频率与容错：脚本最多每 300 ms 运行一次，单次超过 5 秒会被终止；连续失败 30 次后，本会话内不再运行，并显示 `⚠ Statusline disabled after 30 consecutive failures.`；用 `/statusline enable` 重试，或 `/statusline delete` 删除。

传给脚本的 JSON 常用字段（完整列表见官方状态栏页）：`model.display_name`、`model.effort`、`context_window.used_percentage`、`agent_state`（`idle`、`working`、`tool_use`、`reviewing`、`authenticating`、`initializing`、`error`）、`cycle_mode`、`quota`、`cost.total_usd`（按标价估算，不是实际账单）、`vim.mode`。

#### 示例脚本

```bash
#!/bin/bash
input=$(cat)
model=$(echo "$input" | jq -r '.model.display_name // "no model"')
state=$(echo "$input" | jq -r '.agent_state')
used=$(echo "$input" | jq -r '.context_window.used_percentage // 0')
echo "$state | $model | ctx ${used}%"
```

保存为 `~/.gemini/antigravity-cli/statusline.sh` 后 `chmod +x`。

#### 终端标题

配置 `title` 命令后，CLI 用命令输出设置终端窗口标题，窗口最小化或失焦时也能看到进度；未配置则不改标题。

- 在 `settings.json` 加 `"title": {"type": "command", "command": "~/.gemini/antigravity-cli/title.sh"}`。机制与状态栏相同：`sh -c`（Windows `cmd /c`）运行，同样的 JSON 经 `stdin` 传入，以 `stdout` 作为标题；多行会用空格拼接；控制字符会被去掉，不解释 ANSI 颜色，所以请输出单行纯文本。
- 属性：`command`、`type`（默认 `"command"`）、`enabled`（默认 `true`）。同样必须是对象。
- `/title` 切换开关，`/title on`、`/title off` 显式设置。

### 语音

`/voice` 用麦克风提供两种方式：

| 方式 | 入口 | 说明 |
|---|---|---|
| 语音对话（Voice mode） | `/voice live` 或 F6 | 双向口语对话，智能体语音回答，工作过程显示在会话里 |
| 语音听写（dictation） | `/voice dictate`、`/record` 或 F5 | 把实时转写流式写进提示框，**不会自动提交** |

- 单独输入 `/voice` 启动哪一种，在 `/settings` > **Voice Mode** 里选。
- 听写流程：按 F5（或输入 `/voice dictate`）→ 状态行从 **Connecting…** 变为 **Recording** → 说话，临时文字会实时更新 → Enter 或 F5 停止并保留转写；Esc 丢弃并恢复原有提示内容；之后按 Ctrl + \_ 可把整段听写作为一次撤销回退。已有文字时，转写追加在后面。
- 限制：需要交互式 TUI，`--print` 下不可用；商业版和企业版账号暂不支持听写；若登录早于听写功能上线，转写会失败，需 `/logout` 后重新登录一次。
- F5 可重映射：在 `/keybindings` 中绑定 `voice.start_dictation`。

通过 SSH 在远程机器上用语音：远程机器没有本地音频硬件，需要用 `agy mic-serve` 把本地麦克风经反向隧道转发过去（两端都要装 Antigravity CLI）。

```bash
# 1) 本机：启动麦克风服务（默认 127.0.0.1:4713，端口被占用时用 --addr 指定）
agy mic-serve

# 2) 本机：开反向隧道
ssh -R 24713:127.0.0.1:4713 <remote-machine>

# 3) 远程会话：指向转发端口再启动 CLI
ANTIGRAVITY_MIC=127.0.0.1:24713 agy
```

`mic-serve` 请保持绑定在 `127.0.0.1`，避免局域网内其他设备访问音频流。转写为空多半是终端进程没有系统麦克风权限；SSH 下连接立即失败，说明转发端口没人监听。

### CLI 命令与标志速查

直接运行的子命令（文档中出现的）：`agy`（交互式）、`agy models`（列出模型）、`agy agents`（列出智能体）、`agy plugin import gemini`（导入 Gemini CLI 扩展）、`agy acp`（ACP 服务）、`agy mic-serve`（本地麦克风服务）。

| 标志 | 默认 | 作用 |
|---|---|---|
| `--mode` | `""` | 本次会话的执行模式：`accept-edits`、`auto`、`plan` |
| `--model` | — | 选模型（见 `agy models`） |
| `--effort` | — | 本次会话的推理强度（`low` 到 `max`，视模型而定） |
| `--agent` | — | 选智能体（见 `agy agents`） |
| `-c`, `--continue` | `false` | 继续最近的会话 |
| `--conversation` | — | 按 ID 恢复会话 |
| `-p`, `--print`, `--prompt` | — | 非交互地跑一条提示并打印回复（headless） |
| `--output-format` | `text` | print 模式输出格式：`text`、`json`、`stream-json` |
| `--input-format` | `text` | print 模式输入格式：`text`、`stream-json` |
| `--json-schema` | — | JSON Schema 字符串或文件路径，强制结构化输出 |
| `--print-timeout` | `0s` | print 模式时限，`0` 表示等到回合结束 |
| `--sandbox` | `false` | 启用终端沙箱（`v1.4.4+` 若沙箱不可用则无效） |
| `--dangerously-skip-permissions` | `false` | 自动批准所有工具权限请求 |

### 斜杠命令速查

在提示框输入 `/` 打开命令菜单。下表按用途归类，列出文档中出现的命令（含别名）。

| 命令 | 别名 | 作用 |
|---|---|---|
| `/config` | `/settings` | 设置编辑器 |
| `/model` | — | 选模型（跨会话保存）；`/model <model> <prompt>` 只用另一个模型跑单条提示 |
| `/effort [level]` | — | 设置或查看推理强度 |
| `/permissions` | — | 工具权限管理器 |
| `/plan [task]` | — | 调研代码库、澄清需求，起草可审阅的实施方案 |
| `/resume` | `/switch`、`/conversation` | 会话选择器 |
| `/fork` | `/branch` | 克隆当前会话为新的并行会话 |
| `/rewind` | `/undo` | 回退到之前的某条消息 |
| `/clear` | `/new` | 清屏并重置当前会话上下文 |
| `/compact [message]` | — | 压缩为检查点，可附带继续的指令 |
| `/context` | — | 上下文用量面板 |
| `/diff` | — | 交互式 diff 查看器 |
| `/codesearch <query>` | `/cs`、`/search` | 工作区代码搜索面板 |
| `/agents` | — | Agent Manager：切换自定义智能体，监控后台子智能体 |
| `/boost <task>`、`/swarm <task>`、`/teamwork <task>` | — | 多智能体深度推理、协作团队（`/teamwork` 标注为付费计划）；`/swarm board`（Alt + S）打开实时看板 |
| `/statusline`、`/title [on/off]` | — | 状态栏、终端标题（见上） |
| `/usage` | `/quota` | 模型配额面板 |
| `/credits` | — | AI 积分面板 |
| `/voice`、`/record` | — | 语音对话 / 听写 |

#### 常用面板细节

- **`/agents`**：在 **Available Agents** 里用 ↑/↓ + Enter 选 `Default agent` 或自定义智能体，Esc 关闭并应用；已在会话中切换智能体会自动 fork 当前会话。自定义智能体放在工作区的 `.agents/agents/<name>/agent.md` 或全局的 `~/.gemini/config/agents/<name>/agent.md`，文件带 YAML frontmatter（`name`、`description`），正文是系统指令。**Subagents** 区显示后台子智能体，状态有 `running`、`done`、`error`、`killed`；Enter 打开详情，K 终止运行中的子智能体，A / D 内联批准或拒绝其权限请求。
- **`/context`**：显示上下文窗口的比例网格和分类 token 明细（用户消息、智能体回复、工具调用、系统提示、规则、技能、MCP 工具、检查点缓冲等）；`◉` 是已用，`□` 是空闲。上下文变长时 Antigravity 会自动把旧回合压缩为检查点。MCP 工具占比大时，用 `/mcp` 关掉不需要的服务器。
- **`/effort`**：级别 `low`、`medium`、`high`、`xhigh`、`max`，可用级别取决于模型（如固定推理预算的 Orchestrator 不可调）；无参数时打开滑块，←/→ 选择，Enter 保存为默认；`/effort low` 直接设置。选择会保存为以后会话的默认值，而 `--effort` 标志只对本次启动生效。headless 下 `agy -p "/effort"` 只读，不接受参数。
- **`/permissions`**：先选范围（**Project**、**Shared**、**Global**），再在 allow / deny / ask 三个标签里查看规则；A 添加、E（或 Ctrl + G）编辑、D（或 Backspace）删除。规则格式是 `action(target)`，如 `command(git)`。

### Headless 模式

headless（也叫 print 模式）发一条提示、返回结果、然后退出。回复写到 `stdout`，错误、认证提示、进度和权限通知写到 `stderr`，所以捕获输出很干净。

```bash
agy -p "In one sentence, what is a git rebase?"
answer=$(agy -p "Name three popular version control systems, comma-separated.")
```

headless 使用缓存的凭据：先在交互式 `agy` 里登录一次。在没有终端的环境（如 CI）里，未登录的运行会以 `authentication required` 错误退出，而不是卡住。

| 目标 | 做法 |
|---|---|
| 输出格式 | `--output-format`：`text`（默认）、`json`（完成时输出一个 JSON 对象）、`stream-json`（NDJSON 事件流） |
| 结构化输出 | `--json-schema`，接受 schema 字符串、`.json` 文件路径，或 `string` / `number` / `integer` / `boolean`；解析结果在 `structured_output` |
| 继续会话 | `-c` 继续最近的；`--conversation <id>` 用上次运行返回的 `conversation_id` |
| 多轮单进程 | `--input-format stream-json` 搭配 `--output-format stream-json`，从 `stdin` 一行一个 JSON 喂提示 |

`json` 输出的主要字段：`conversation_id`、`status`（`SUCCESS`、`ERROR`、`CANCELED`、`INTERRUPTED`、`INVALID`、`WAITING`、`RUNNING`）、`response`、`error`（仅失败时）、`duration_seconds`、`num_turns`、`usage`（`input_tokens`、`output_tokens`、`thinking_tokens`、`cache_read_tokens`、`total_tokens`）；用了 `--json-schema` 时还有 `structured_output` 和 `json_schema`。

```bash
# 结构化输出
agy -p "Parse the semantic version string v2.14.3 into an object with integer fields major, minor, and patch." \
  --output-format json \
  --json-schema '{"type":"object","properties":{"major":{"type":"integer"},"minor":{"type":"integer"},"patch":{"type":"integer"}},"required":["major","minor","patch"]}' \
  | jq '.structured_output'

# 流式：拼接文本增量（用 jq -j 避免插入换行）
agy -p "Explain what a merge conflict is in two sentences." --output-format stream-json \
  | jq -j 'select(.event=="step_update") | .step_update.text_delta // empty'

# 只取最终 token 用量
agy -p "..." --output-format stream-json | jq 'select(.event=="result") | .result.usage'
```

`stream-json` 以一个 `init` 事件开始，若干 `step_update` 事件（`step_type` 有 `user_input`、`agent_response`、`tool`、`checkpoint`；`state` 为 `ACTIVE` 或 `DONE`；工具步骤的 `tool_info` 含 `name`、`parameters`、`output`），以一个 `result` 事件结束（结构同 `json`）。

#### 用 stdin 持续对话

```bash
printf '%s\n' \
  '{"event":"user","message":{"content":"Reply with exactly the word: apple. Nothing else."}}' \
  '{"event":"user","message":{"content":"What word did I ask you to reply with in my previous message? Answer with just that word."}}' \
  | agy --input-format stream-json --output-format stream-json
```

- 每条提示是一行 `{"event":"user","message":{"content":...}}`；`content` 可以是字符串，或文本块列表（只支持 `text` 块）。
- 每轮产生一个 `result`；`response` 只代表当前回合，而 `num_turns`、`usage`、`duration_seconds` 是整个会话的累计值。
- 关闭 `stdin` 才会优雅结束会话；脚本不能等进程退出再读输出，应逐行读取，并在收到当前回合的 `result` 后再写下一条提示。
- 流式会话里不能使用由 CLI 自己应答的斜杠命令（如 `/model`、`/usage`），会得到 `ERROR` 结果并以退出码 `2` 结束；请单独运行 `agy -p /model`。无效 JSON、缺少 `event` 字段、`text` 以外的内容块则以退出码 `1` 结束；未知的 `event` 名只会跳过并在 `stderr` 打印警告。
- 不要在这种模式里再用 `-p` 传提示（会被丢弃），也不要搭配 `--output-format json` 或 `text`（只在退出时输出一份，会丢失前面的回合）。

#### CI 中的权限设置

headless 没有交互提示，需要批准的操作由策略决定。

```text
CI 里没有人能回答提示
├─ 钉住模式                → --mode 或 settings 里的 agentMode
├─ 只允许明确列出的操作    → ~/.gemini/antigravity-cli/settings.json 的 permissions.allow
└─ 信任提示和环境          → --dangerously-skip-permissions（批准所有工具调用）
```

- 自 `v1.5.7` 起，`agentMode` 未设置时 `agy -p` 在 `auto` 可用的地方以 `auto` 运行，否则 `default`；CI 里建议用 `--mode` 或 `agentMode` 固定模式。
- 无法获得批准的操作被**软拒绝**：运行继续、退出码为 `0`，并在 `stderr` 打印通知，指明工具以及如何放行。因此 CI 不能只看退出码，要检查输出。
- 提前放行：在 `permissions.allow` 里添加 `action(target)` 规则。

```json
{
  "permissions": {
    "allow": [
      "command(git)",
      "command(regex:npm run (build|lint|test))",
      "write_file(src/)"
    ]
  }
}
```

- headless 对未知的 `--model` 不会静默回退，而是以非零退出码和 `ERROR` 状态失败，避免流水线悄悄用错模型。成功退出码为 `0`，失败写 `stderr`，`json` / `stream-json` 里也会有 `status` 和 `error`。

#### 可照抄的 CI 脚本

失败即退出，并保存回复：

```bash
#!/usr/bin/env bash
set -euo pipefail

result=$(agy -p "Name three popular version control systems, comma-separated." \
  --output-format json \
  --print-timeout 10m)

status=$(echo "$result" | jq -r '.status')
if [[ "$status" != "SUCCESS" ]]; then
  echo "Agent run failed: $(echo "$result" | jq -r '.error')" >&2
  exit 1
fi

echo "$result" | jq -r '.response' > result.txt
```

### ACP：在其他编辑器里使用 Antigravity CLI

`agy acp` 启动一个 Agent Client Protocol（ACP）服务器，让支持 ACP 的编辑器（文档列出 Zed、JetBrains IDE、Android Studio、Xcode）在自己的聊天面板里运行同一个智能体。编辑器把 `agy acp` 作为子进程拉起，通过 `stdin` / `stdout` 上的 JSON-RPC 通信，**你不用自己运行这条命令**；智能体、工具、登录和设置都来自 Antigravity CLI，所以在编辑器里的行为与终端一致。

#### 接入步骤

1. 安装 CLI，用 `agy acp --help` 确认你的版本包含 ACP。
2. 先用交互式 `agy` 登录一次，或运行 `agy acp --login`；`agy acp` 复用缓存的凭据。
3. 在编辑器里把 `agy` 加参数 `acp` 注册为自定义智能体。以 Zed 的 `settings.json` 为例：

```json
{
  "agent_servers": {
    "Antigravity": {
      "type": "custom",
      "command": "agy",
      "args": ["acp"]
    }
  }
}
```

4. 在编辑器的智能体面板里用 **Antigravity** 开一个新线程。

各编辑器的具体步骤、权限和其他标志，见官方“其他编辑器（ACP）”页。

### 积分与配额

两个相关概念：**配额**（按模型分组的周期限额）与 **AI 积分**（AI Premium credits，配额用尽后的付费回退）。

| 命令 / 设置 | 作用 |
|---|---|
| `/usage`（别名 `/quota`） | 打开 **Models & Quota** 面板：每个模型组（如 Gemini Flash / Pro、Claude 与 GPT）的剩余配额；同组模型共享每周限额和 5 小时限额。打开时自动刷新 |
| `/credits` | 打开积分面板：当前余额、本计费周期的用量，以及购买积分或升级计划的链接（会打开网页） |
| `useG1Credits` | 默认 `false`。设为 `true` 则在计划配额用尽后使用你的积分；商业版账号不可用。也可在 `/config` 里的 **Use AI Credits** 开关 |

`/usage` 面板里：↑/↓（或 J/K）滚动，Esc 或 Q 关闭。状态栏右侧会显示剩余积分（如 `AI Credits: 42`），低于告警阈值时高亮。基础配额、积分消耗规则和计划资格，以官方 Plans 页面为准。

### 从 Gemini CLI 迁移

Antigravity CLI 保留了 Gemini CLI 的核心开发体验，提供首次启动的自动迁移和显式迁移命令。

1. **首次启动 `agy`**：在有旧配置的环境里，CLI 会检测现有配置并弹出清单，让你选择迁移哪些内容：转换扩展和全局配置；把会话令牌安全迁入操作系统密钥环；把视觉参数和渲染缓冲映射到新的设置。
2. **手动转换扩展**：运行 `agy plugin import gemini`，它会搜索旧的本地目录、解析扩展清单并转成原生插件。旧的 commands 会被转换成 skills，MCP 服务器定义迁到 `mcp_config.json`。
3. **规则文件无需改动**：工作区的 `GEMINI.md`、`AGENTS.md` 和全局的 `~/.gemini/GEMINI.md` 照常生效。
4. **迁移技能目录**：把项目里的 `.gemini/skills/` 手动改名或移到 `.agents/skills/`，否则不会被识别。
5. **拆分 MCP 配置**：MCP 服务器不再内嵌在主设置里，改放独立的 `mcp_config.json`；远程 WebSocket / SSE 服务器的 `url` 或 `httpUrl` 键要改成 `serverUrl`。

| 配置 | Gemini CLI | Antigravity CLI |
|---|---|---|
| 全局共享技能 | `~/.gemini/skills/` | `~/.gemini/skills/` 或 `~/.gemini/antigravity-cli/skills/` |
| 工作区技能 | `.gemini/skills/` | `.agents/skills/` |
| MCP（全局） | `~/.gemini/settings.json` 内联 | `~/.gemini/config/mcp_config.json` |
| 远程服务器地址键 | `url` / `httpUrl` | `serverUrl` |

不是完全对等：某些自定义终端主题或实验性视觉浮层可能不被支持。

### 最佳实践

1. **先建立验证回路**：让智能体有本地方式验证自己的工作（测试、构建、格式化脚本）。没有测试就先让它写；改完让它运行测试命令并根据输出迭代。例如：`Implement feature X in main.py. Run npm test afterward to verify the build.`
2. **先探索，再计划，最后执行**：先让它解释代码库如何解决某问题，再要一份实施方案（产物），批准后再让它改。例如：`Explore how our router resolves /docs/:page. Write down an implementation plan to add /docs/best-practices.`
3. **丰富上下文**：用 `@` 精确指向文件或目录；UI 问题就复制截图或录屏，按 Ctrl + V 贴进提示框。
4. **写规则文件**：在工作区根目录放 `GEMINI.md` 或 `AGENTS.md`，写目录规范、风格、测试命令和弃用提醒，智能体启动时自动解析。
5. **按风险选权限**：想少被打断就用 `auto` 模式；否则在 `settings.json` 调 `toolPermission`（常见搭配 `"proceed-in-sandbox"` 加 `"enableTerminalSandbox": true`）：

| `toolPermission` | 行为 |
|---|---|
| `request-review`（默认） | 写入、bash、web、MCP 工具先询问，除非有 allow 规则覆盖 |
| `proceed-in-sandbox` | 沙箱开启且命令在沙箱内运行时不询问；沙箱不可用时（`v1.4.4+`）与 `request-review` 相同 |
| `always-proceed` | 命令、web 请求、MCP 工具不询问，除非命中 ask 或 deny 规则 |
| `strict` | 每个工具动作（含读文件）都询问，即使 allow 规则匹配；会覆盖 `auto` 模式 |

6. **主动管理会话**：发现跑偏立刻 Esc；改坏了用 `/rewind`；阶段结束用 `/compact`；拿不准路线用 `/fork` 试验，失败再 `/resume` 回主线。
7. **自动化**：`agy -p "Review this git diff and draft a conventional commit message"` 可放进 Git hook；大范围任务让主智能体派生并行后台子智能体，用 `/agents` 监控。

### 排错速查

| 症状 | 可能原因 | 解决 |
|---|---|---|
| `agy: command not found` | 二进制目录不在 `PATH`（装在 `~/.local/bin` 或 `%LOCALAPPDATA%\agy\bin`） | 在 `~/.zshrc` 或 `~/.bashrc` 加入该目录到 `PATH` 后 `source`；Windows 在用户 `Path` 里加入 `%LOCALAPPDATA%\agy\bin` 并重启终端 |
| `keyring: secure lock out` / `secret keyring is locked`，启动卡住或出现 D-Bus 警告 | 密钥环被锁定，或后台守护进程不可用 | macOS：在“钥匙串访问”里确认 `Antigravity CLI` 条目允许 `agy`；无头 SSH 下用 `security unlock-keychain` 解锁。Linux：确保 GNOME Keyring / KWallet 已解锁，无头环境确保有 D-Bus 会话（如 `export $(dbus-launch)`） |
| `update.lock` 警告、升级卡住 | 后台自更新进程挂起，或安装目录无写权限 | 删除 `~/.gemini/antigravity-cli/updater/update.lock`；或设环境变量 `AGY_CLI_DISABLE_AUTO_UPDATE=true` 关闭自动更新；确认安装目录可写 |
| headless 报 `authentication required` | 没有缓存的凭据 | 先交互式运行一次 `agy` 登录，或用 Gemini API Key / ADC |
| `--mode=auto` 不起作用，Shift + Tab 里没有 `auto` | 账号是商业版/企业版、版本过旧，或沙箱不可用 | 按“选择执行模式”里的限制核对；此时会运行在 `default` |
| CI 里操作没执行但退出码是 0 | 需要批准的工具被软拒绝 | 查看 `stderr` 通知，补 `permissions.allow` 规则或固定 `--mode` |

---

## Part V：SDK 与 IDE

### Antigravity SDK：先选型，再动手

#### 它是什么

Antigravity SDK 是一个 **Python SDK**，用来构建由 Antigravity 与 Gemini 驱动的自主 Agent。它提供一个安全、有状态的运行时（harness），负责工具执行、上下文管理、安全策略和子代理委派。

```text
你想怎么用 Antigravity 的 Agent？
│
├─ 在自己的 Python 程序里嵌入本地运行的 Agent（自定义工具、策略、hooks）
│  └─ Antigravity SDK（google-antigravity）
├─ 想要托管的云端 REST / gRPC 接口，而不是本地 Python 运行时
│  └─ Gemini API 的 Antigravity Agent（SDK 文档只给出这一指引）
├─ 在脚本 / CI 里非交互地跑 Agent
│  └─ Antigravity CLI 的 headless 模式（ACP 页“下一步”里提到，本节不展开）
└─ 日常交互开发
   └─ Antigravity App / IDE / IDE 扩展
```

| 方案 | 运行位置 | 适合 | 备注 |
|---|---|---|---|
| **Antigravity SDK（Python）** | 你的 Python 进程 | 把 Agent 嵌进自己的应用 | `Agent` 类用异步上下文管理器管理二进制发现、工具执行和会话生命周期 |
| **Gemini API 的 Antigravity Agent** | 托管云端 | 直接调 REST / gRPC | 与本地 SDK 是两个东西 |
| **CLI headless 模式** | CLI 进程 | 脚本与 CI | 文档仅称“非交互运行 Agent” |
| **App / IDE / 扩展** | 图形界面 | 人在回路的日常开发 | 登录与权益各入口统一 |

> 文档没有给出 SDK 与 CLI headless、App 的逐项对比，以上只保留有依据的定位差异。

#### 安装、认证与最小示例

| 项目 | 内容 |
|---|---|
| 安装 | `pip install google-antigravity` |
| Gemini API 密钥 | 环境变量 `GEMINI_API_KEY` |
| Gemini Enterprise | `LocalAgentConfig(vertex=True, project=..., location=...)`；或设 `GOOGLE_GENAI_USE_VERTEXAI=True`、`GOOGLE_CLOUD_PROJECT`、`GOOGLE_CLOUD_LOCATION`，并执行 `gcloud auth application-default login` |
| Python 版本要求 | 文档未写明 |

```bash
pip install google-antigravity
export GEMINI_API_KEY="your_api_key_here"
```

```python
import asyncio
from google.antigravity import Agent, LocalAgentConfig

async def main():
    config = LocalAgentConfig()
    async with Agent(config) as agent:
        response = await agent.chat("What files are in the current directory?")
        print(await response.text())

if __name__ == "__main__":
    asyncio.run(main())
```

> `vertex=True`（或 `GOOGLE_GENAI_USE_VERTEXAI=True`）会让请求用 Google Cloud 凭证走 Gemini Enterprise API（`aiplatform.googleapis.com`）。

#### 核心概念结构

```text
LocalAgentConfig（本地模型另有 LiteRTAgentConfig / LocalOpenAIAgentConfig）
│  system_instructions ── 人设
│  tools / skills_paths ─ 自定义函数 / 含 SKILL.md 的目录
│  capabilities ──────── CapabilitiesConfig（enabled_tools / disabled_tools / enable_subagents）
│  mcp_servers ───────── McpStdioServer / McpStreamableHttpServer
│  policies ──────────── deny / allow / ask_user
│  subagents ─────────── SubagentConfig
│  hooks / triggers ──── @hooks.pre_turn / every(...)
│  response_schema ───── Pydantic 模型
│  save_dir / conversation_id / app_data_dir ── 会话持久化
▼
async with Agent(config) as agent:
    response = await agent.chat(prompt)
    await response.text() | async for token in response | response.thoughts
    await response.structured_output() | response.usage_metadata
```

上图只列文档出现过的字段，不是完整配置清单。

### SDK：Personas（系统指令）

#### 用途与 API 要点

定制 Agent 的身份和行为约束，传给 `LocalAgentConfig(system_instructions=...)`。

| 形式 | 写法 | 效果 |
|---|---|---|
| 字符串 | `system_instructions="You are ..."` | 最简单 |
| 模板化（官方推荐） | `types.TemplatedSystemInstructions(identity=..., sections=[...])` | 保留 SDK 默认脚手架和环境上下文，只覆盖身份并追加段落 |
| 自定义 | `types.CustomSystemInstructions(text=...)` | 绕过所有默认脚手架和环境上下文，直接传原始系统提示 |

段落用 `types.SystemInstructionSection(title=..., content=...)`。

#### 最小示例

```python
from google.antigravity import Agent, LocalAgentConfig, types

identity = "You are an expert Code Quality Reviewer."
review_criteria = types.SystemInstructionSection(
    title="review_criteria",
    content="- Focus on readability and maintainability.",
)

templated_si = types.TemplatedSystemInstructions(
    identity=identity,
    sections=[review_criteria],
)

config = LocalAgentConfig(system_instructions=templated_si)

async with Agent(config) as agent:
    response = await agent.chat("Review the latest pull request.")
    print(await response.text())
```

#### 坑

用 `CustomSystemInstructions` 后，SDK 默认的环境上下文也一并没了，需要自己补全。

### SDK：Tools and skills

#### 用途与 API 要点

用自定义 Python 函数、内置工具和 Agent skills 扩展能力。

| 要点 | 说明 |
|---|---|
| 内置工具枚举 | `google.antigravity.types.BuiltinTools` |
| 工具标识 | `list_directory`、`search_directory`、`find_file`、`view_file`、`create_file`、`edit_file`、`run_command`、`ask_question`、`start_subagent`、`generate_image`、`search_web`、`read_url_content`、`finish` |
| 枚举常量 | 如 `BuiltinTools.RUN_COMMAND`、`BuiltinTools.VIEW_FILE`、`BuiltinTools.SEARCH_WEB` |
| 过滤 | `CapabilitiesConfig(enabled_tools=...)` 或 `disabled_tools` |
| 只读工具组 | `BuiltinTools.read_only()` |
| 自定义函数 | `LocalAgentConfig(tools=[func])` |
| Skills | `LocalAgentConfig(skills_paths=[...])`：可指向含 `SKILL.md` 的单个目录，或包含多个 skill 子目录的父目录 |

#### 最小示例

```python
from google.antigravity import Agent, LocalAgentConfig, CapabilitiesConfig
from google.antigravity.types import BuiltinTools

def get_weather(city: str) -> str:
    """Returns the current weather for a city."""
    return f"It's sunny in {city}."

config = LocalAgentConfig(
    tools=[get_weather],
    # 只开放只读内置工具
    capabilities=CapabilitiesConfig(enabled_tools=BuiltinTools.read_only()),
)

async with Agent(config) as agent:
    response = await agent.chat("What's the weather in Tokyo?")
    print(await response.text())
```

> 该示例由原文两段（`tools=[get_weather]` 与 `enabled_tools=BuiltinTools.read_only()`）合并而成。

#### 坑

网页搜索（`SEARCH_WEB`）和 URL 读取（`READ_URL_CONTENT`）**默认启用**；不想让 Agent 联网，要自己用 `disabled_tools` 或 `enabled_tools` 收窄。

### SDK：MCP

#### 用途与 API 要点

把外部 MCP 服务器接入 Agent，与内置工具、自定义函数在同一执行管线工作。支持 `stdio` 与 `Streamable HTTP` 两类服务器。

| 类（`google.antigravity.types`） | 示例中出现的参数 |
|---|---|
| `McpStdioServer` | `name`、`command`、`args`、`env` |
| `McpStreamableHttpServer` | 原文未给示例，参数请查官方 API |

#### 最小示例

```python
from google.antigravity import Agent, LocalAgentConfig
from google.antigravity.types import McpStdioServer

config = LocalAgentConfig(
    mcp_servers=[
        McpStdioServer(
            name="sqlite-explorer",
            command="node",
            args=["/usr/local/bin/sqlite-mcp-server.js"],
            env={"SQLITE_DB_PATH": "/var/data/app.db"},
        )
    ]
)

async with Agent(config) as agent:
    response = await agent.chat("Query database metrics.")
    print(await response.text())
```

#### 坑

这是 SDK 里的“编程式”MCP 配置；IDE 里的 `mcp_config.json` 是另一套（见 Visual Studio 一节），不要混用。

### SDK：Policies

#### 用途与 API 要点

声明式策略引擎，控制工具能否执行或要求人工批准。默认：自定义 Python 函数和标准只读工具被允许；高风险系统操作（如用 `run_command` 执行 shell）需要显式权限或策略规则。

| API | 作用 |
|---|---|
| `deny("*")` / `allow("view_file")` / `ask_user("run_command", handler=...)`（`google.antigravity.hooks.policy`） | 拒绝 / 允许 / 询问 |
| `policy.safe_defaults(handler=...)` | 允许只读工具，写操作（创建、编辑文件、shell）询问用户 |
| `policy.allow_all()` | 全部允许（本地模型示例使用） |
| `interactive.ask_user_handler`、`run_interactive_loop(config)`（`google.antigravity.utils.interactive`） | 控制台交互式批准与会话 |

#### 最小示例

```python
from google.antigravity import LocalAgentConfig
from google.antigravity.hooks.policy import deny, allow, ask_user

async def my_handler(tool_call):
    print(f"Tool approval requested for: {tool_call.name}")
    return True

policies = [
    deny("*"),                                     # Deny all tools by default
    allow("view_file"),                            # Allow file viewing
    ask_user("run_command", handler=my_handler),   # Ask approval for shell
]

config = LocalAgentConfig(policies=policies)
```

交互式用法：`policies = policy.safe_defaults(handler=interactive.ask_user_handler)`，再 `asyncio.run(run_interactive_loop(config))`；控制台会提示 `Allow execution? (y/n) [n]:`。

#### 坑

- 示例里的 `my_handler` 直接 `return True`，等于自动批准，只能当演示。
- 多条规则的匹配顺序文档没有说明；示例按“先 `deny("*")` 再逐项放行”排列。

### SDK：Subagents

#### 用途与 API 要点

主 Agent 拆解任务，把子任务委派给各有独立上下文窗口的子代理。

| 形式 | 配置 | 说明 |
|---|---|---|
| 动态自克隆 | `capabilities=types.CapabilitiesConfig(enable_subagents=True)` | 子代理继承主 Agent 的权限和工具集，按需生成 |
| 静态自定义 | `subagents=[types.SubagentConfig(...)]` | 有独立工具或系统指令 |

`SubagentConfig` 字段：`name`、`description`、`system_instructions`、`tools`。

#### 最小示例

```python
from google.antigravity import Agent, LocalAgentConfig, types

def my_custom_reviewer_tool(file_path: str) -> str:
    """Audits docstrings in a python file."""
    # Add your custom tool logic here
    return f"Verified {file_path}"

reviewer = types.SubagentConfig(
    name="code_reviewer",
    description="Audits source code for style compliance.",
    system_instructions="Check function docstrings in Python files.",
    tools=[my_custom_reviewer_tool],
)

config = LocalAgentConfig(
    tools=[my_custom_reviewer_tool],
    subagents=[reviewer],
)

async with Agent(config) as agent:
    response = await agent.chat(
        "Check function docstrings in Python files under src/."
    )
    print(await response.text())
```

#### 坑

分配给子代理的自定义工具，**必须同时注册在父 Agent 的 `tools` 列表里**。

### SDK：Structured output（含多模态与流式）

#### 用途与 API 要点

| 能力 | API |
|---|---|
| 流式思考 / 回复 | `async for thought in response.thoughts` / `async for token in response` |
| 完整文本 | `await response.text()` |
| 附件 | `from google.antigravity.types import Image, from_file`；`Image.from_file("chart.png")`、`from_file("spec.pdf")` |
| 多模态 prompt | `agent.chat([文本, 图片, PDF])`（传列表） |
| 结构化输出 | `LocalAgentConfig(response_schema=Model)` + `await response.structured_output()` |

#### 最小示例

```python
import pydantic
from google.antigravity import Agent, LocalAgentConfig

class TaskSummary(pydantic.BaseModel):
    summary: str
    action_items: list[str]

config = LocalAgentConfig(response_schema=TaskSummary)

async with Agent(config) as agent:
    response = await agent.chat("Summarize the meeting notes.")
    data = await response.structured_output()
    print(data["action_items"])
```

流式输出的结构：先 `async for thought in response.thoughts:` 逐段写出思考，再 `async for token in response:` 逐段写出回复。

#### 坑

原文示例按“先迭代 `response.thoughts`，再迭代 `response`”的顺序；反过来或并发消费的行为文档没说明。

### SDK：Lifecycle and hooks

#### 用途与 API 要点

拦截回合、处理工具错误、审计 token、定时触发、跨重启恢复会话。

| 能力 | API |
|---|---|
| 回合前 hook | `@hooks.pre_turn`，接收 `prompt: str`，返回 `types.HookResult(allow=True)` |
| 工具出错 hook | `@hooks.on_tool_error`，接收 `err: Exception` |
| 注册 | `LocalAgentConfig(hooks=[...])` |
| token 审计 | `response.usage_metadata.total_token_count` |
| 定时触发 | `LocalAgentConfig(triggers=[every(60, check_status)])`，`every` 来自 `google.antigravity.triggers` |
| 持久化 | `save_dir`、`conversation_id`、`app_data_dir` |

#### 最小示例

```python
from google.antigravity import Agent, LocalAgentConfig, hooks, types

@hooks.pre_turn
async def log_turn(prompt: str) -> types.HookResult:
    print(f"User prompt: {prompt}")
    return types.HookResult(allow=True)

@hooks.on_tool_error
async def handle_error(err: Exception) -> None:
    print(f"Tool execution failed: {err}")

config = LocalAgentConfig(hooks=[log_turn, handle_error])

async with Agent(config) as agent:
    response = await agent.chat("Analyze repository status.")
    print(await response.text())

    # Token usage auditing
    if response.usage_metadata:
        print(f"Turn tokens: {response.usage_metadata.total_token_count}")
```

会话持久化：`LocalAgentConfig(save_dir="./sessions", conversation_id="session-12345678901234567890123456789012", app_data_dir="/path/to/custom/storage")`（`conversation_id` 用于保存或恢复已有会话）。

#### 坑

- `conversation_id` 必须**至少 32 个字符**，只能含字母、数字和连字符，**不允许下划线**。
- `response.usage_metadata` 可能为空，示例先判断再读取。
- 文档没有说明 `HookResult(allow=False)` 的具体语义，不要凭名字推断。

### SDK：Local models（本地模型）

#### 用途与 API 要点

不需要 API 密钥和网络，在本机用本地模型跑 Agent。

| 类 | 连接什么 | 备注 |
|---|---|---|
| `LiteRTAgentConfig` | 用 Google AI Edge 的 LiteRT 运行时直接跑 `.litertlm` 检查点（如 Gemma 4 26B A4B） | SDK 管理本地回环服务器 |
| `LocalOpenAIAgentConfig` | 外部兼容 OpenAI API 的本地服务（Ollama、LM Studio、vLLM） | 参数 `model`、`base_url`；可调 `.lightweight()` |

LiteRT 准备（原文）：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install google-antigravity litert-lm
litert-lm import \
  --from-huggingface-repo=litert-community/gemma-4-26B-A4B-it-litert-lm \
  gemma-4-26B-A4B-it-gpu.litertlm \
  gemma4-26b
```

导入下载约 16.8 GB，注册到 `~/.litert-lm/models/gemma4-26b/model.litertlm`；建议至少 24 GB 显存或统一内存。

`LiteRTAgentConfig` 选项：`model_path`（必填，`.litertlm` 绝对路径）；`backend`（默认 `LiteRTBackend.GPU`，可取 `'gpu'`、`'npu'`、`'cpu'`）；`vision_backend`（默认 `None`，自动检测 `'gpu'` 或 `'cpu'`）；`audio_backend`；`enable_speculative_decoding`（默认 `False`）；`cache_dir`（须已存在且已展开 `~`）；`port`（默认 `0`，自动选择）；`compaction_config`（默认 `token_threshold=40960`，按 64k KV-cache 推导）。

`LiteRTAgentConfig` 初始化时自动应用 `.lightweight()` 预设（核心编码工具、裁剪系统指令）；前缀缓存自动生效。

#### 最小示例

```python
import os

from google.antigravity import Agent, LiteRTAgentConfig

MODEL_PATH = os.path.expanduser(
    "~/.litert-lm/models/gemma4-26b/model.litertlm"
)

config = LiteRTAgentConfig(model_path=MODEL_PATH)

async with Agent(config) as agent:
    response = await agent.chat("What files are in the current directory?")
    async for token in response:
        print(token, end="", flush=True)
    print()
```

连接 Ollama：`LocalOpenAIAgentConfig(model="gemma4:26b", base_url="http://localhost:11434/v1").lightweight()`，其余用法与上例相同（LM Studio 的 `base_url` 例如 `http://localhost:1234/v1`）。

要让本地 Agent 在项目目录改文件、跑命令：设 `workspaces=[workspace]` 并挂策略，如 `policies=[policy.allow_all()]`（`policy` 来自 `google.antigravity.hooks`）。

#### 坑

- `~` **不会自动展开**：`model_path`、`cache_dir` 用完整路径或 `os.path.expanduser()`；`cache_dir` 还要先 `os.makedirs(..., exist_ok=True)`。
- **不要**用 `LocalOpenAIAgentConfig` 连接 `litert-lm serve`，改用 `LiteRTAgentConfig`。
- 端侧视觉需要 `google-antigravity >= 0.1.24`、`litert-lm >= 0.19.1` 和支持视觉的检查点；更早导入过 `gemma4-26b` 的要重跑 `litert-lm import`。图片用 `from_file()` 或 `Image` 传入 `agent.chat([...])`。
- macOS 上 `litert-lm import` 遇到 SSL 证书校验错误：`pip install certifi`，再 `export SSL_CERT_FILE=$(python3 -c "import certifi; print(certifi.where())")`。

### SDK 上线前检查表

- [ ] 密钥来自环境变量（`GEMINI_API_KEY`），或用 `vertex=True` 加 `project`、`location` 并完成 `gcloud auth application-default login`，代码里不写死。
- [ ] 用 `CapabilitiesConfig(enabled_tools=...)`（如 `BuiltinTools.read_only()`）收窄工具；确认是否保留默认启用的 `SEARCH_WEB` / `READ_URL_CONTENT`。
- [ ] 策略以 `deny("*")` 起手，再逐项 `allow(...)`；`run_command` 用 `ask_user`，handler 要真正判断，不要照抄 `return True`。
- [ ] 静态子代理用到的自定义工具，同时注册在父 Agent 的 `tools` 里。
- [ ] 用 `conversation_id` 持久化时：至少 32 位、仅字母数字连字符，并配置 `save_dir`。
- [ ] 加 `@hooks.on_tool_error`，读 `response.usage_metadata` 做 token 审计（先判空）。
- [ ] 需要机器可读结果时用 `response_schema` 与 `structured_output()`。
- [ ] 本地模型：`~` 全部展开，`cache_dir` 已创建，磁盘与显存满足检查点需求（约 16.8 GB 下载，建议 24 GB 显存或统一内存）。

### Antigravity IDE 与 IDE 扩展：先分清

#### 区别

| 项目 | Antigravity IDE | IDE 扩展 |
|---|---|---|
| 是什么 | Antigravity 自己的编辑器 | 把 Antigravity 带进你现有的编辑器 |
| Tab 补全（Supercomplete、Tab-to-Jump） | 有 | 文档标注仅 Antigravity IDE |
| Workflows | 有（已弃用） | 文档标注仅 Antigravity IDE |
| Agent side panel、编辑器内 Review changes | 有 | VS Code、Visual Studio、Eclipse |
| `/browser` | `v2.6.0+` | VS Code 扩展 `v1.9.0+` |
| 内置浏览器 | 无，始终用本机 Chrome | 同样依赖本机 Chrome |

> 文档没有一页专门对比两者，上表由各页的 “Available on” 标注整理。

#### 登录与权益

登录在 IDE 扩展、Antigravity CLI、Antigravity App 之间统一，身份决定模型权益、配额和数据隐私策略。个人开发者用任意个人 Google 账号登录（含免费档）；企业用户选 **Sign in with Gemini Enterprise**，用公司身份认证（含 BYOID 和 Workforce Identity Federation），并选择 **Google Cloud Project** 与 **Region**。文档称企业场景下代码、prompt 和 Agent 转录不会用于训练 Google 基础模型。

### IDE 扩展：支持矩阵与安装要点

#### 支持矩阵

| 编辑器 | 版本要求 | 平台 | 安装入口 | 备注 |
|---|---|---|---|---|
| VS Code | 1.90 或更高 | macOS、Linux、Windows | 扩展市场搜 **Google Antigravity**（发布者 Google） | 首次启动自动安装本地 `agy` 后端服务 |
| Visual Studio | 2022 17.14+ 或 2026 | Windows 10 / 11 | **Extensions > Manage Extensions** | VSIX；安装需关闭所有实例 |
| Eclipse | 2026-06 或更高 | macOS、Linux、Windows | **Help > Eclipse Marketplace…** | 发布者 **Antigravity Eclipse Publisher** |
| IntelliJ 系 | 2026.2.1+ | macOS、Linux、Windows | **JetBrains AI > Settings > Agents** 搜 **Antigravity** | 含 IDEA、PyCharm、WebStorm、GoLand、CLion、Rider 等 |
| Android Studio | 支持 Bring Your Own Agent（ACP）的版本 | 文档未写 | **Agent** 工具窗口选 **Google Antigravity** | 经 ACP Registry；企业认证为 **Preview** |
| Zed | 0.140.0+ | macOS、Linux | `agent: open settings` > **External Agents** > **Add** > **Install from Registry** | 见文末“文档矛盾” |
| Xcode | Xcode 27 beta 6 或更高 | macOS | **Xcode > Settings > Intelligence** 点 **Get** | beta 版本要求 |
| 其他编辑器 | 支持 ACP 即可 | — | `agy acp` | 见下一节 |

IntelliJ 系、Android Studio、Zed、Xcode 的页面都以 JSON 片段给出认证方式（`{"auth": {"type": ...}}`）：`oauth-personal`（个人 Google 账号）、`oauth-business`（Gemini Enterprise，另需 `gcp.project`、`gcp.location`）、`gemini-api-key`、`agent-platform`（API / ADC，**Preview**，另需 `gcp.project`、`gcp.location`）。

#### VS Code

- 点击 Activity Bar 的 Antigravity 图标登录；聊天用 Cmd + L（Windows、Linux 为 Ctrl + L），选中代码后再按可把选区加进提示。
- `@terminal`：把打开的终端挂进提示，含工作目录、最后一条命令与退出码、约最后 8,000 个字符输出；依赖 VS Code 的 terminal shell integration。
- 审查：每处改动有 **Accept** / **Reject** CodeLens；Option + Enter（Windows、Linux 为 Alt + Enter）接受，加 Shift 拒绝；Esc 中断运行中的 Agent。

| 设置键 | 默认 | 作用 |
|---|---|---|
| `antigravity.enableInlineDiff` | `true` | `false` 时改为并排 diff 标签页 |
| `antigravity.autoAcceptOnChat` | `true` | 发新消息时自动接受后台文件的待审改动，保留当前文件的行内 diff；`false` 则跨回合保持待审 |
| `antigravity.serverPort` | `0` | 后台服务器（`agy --hub`）端口，`0` 为自动；远程与端口转发场景可固定 |
| `antigravity.enableTelemetry` | `true` | **已弃用**，改用 Antigravity **Settings** > **Account** 里的开关 |

其余还有 `antigravity.autoOpenFiles`（默认 `false`）与 `antigravity.serverStartupTimeoutMs`（默认 `30000`）。

企业网络：扩展与后台服务器使用 `HTTP_PROXY`、`HTTPS_PROXY`、`NO_PROXY`（环境变量优先于 VS Code 的 `http.proxy`、`http.noProxy`）。TLS 检查代理（如 Zscaler）场景下，根证书要同时装进操作系统信任库，并让 `NODE_EXTRA_CA_CERTS` 指向 PEM 文件（启动 VS Code 之前设置）。

#### Visual Studio

- 首次启动 **Antigravity** 工具窗口自动打开（停靠在 Solution Explorer 旁）；之后用 **View > Other Windows > Antigravity** 或 Ctrl + \, Ctrl + Alt + L。
- 快捷键是以 Ctrl + \ 开头的和弦：先按 Ctrl + \ 并松开，再按第二个组合键（如新对话 Ctrl + \, Ctrl + Shift + L；中断 Ctrl + \, Esc）。
- 以打开的解决方案目录（或 **Open Folder** 工作区）作为 Agent 工作区，切换时若有对话在进行会先询问。
- 评审使用 Visual Studio 原生 diff，**Accept All** / **Reject All**，待审文件保持只读；Output 窗口选 **Google Antigravity** 看日志。
- MCP：内置 MCP Store；手动配置在 `~/.gemini/config/mcp_config.json`（所有项目）或工作区内 `.agents/mcp_config.json`（仅该工作区）。

#### Eclipse

- 安装后重启；首次启动 **Antigravity** 视图在编辑区右侧打开，需确认信任项目文件夹。开关聊天：**Antigravity > Toggle Chat**（Cmd + Option + E / Ctrl + Alt + K）；账号、模型与 MCP 偏好在 **Antigravity > Settings…**。

#### JetBrains、Android Studio、Zed、Xcode

- IntelliJ 系：**JetBrains AI > Settings > Agents** 搜索 **Antigravity**，点 **Install**。
- Android Studio：**Agent** 工具窗口顶部选择器选 **Google Antigravity**（未安装会自动安装）并登录；模型与 thinking level 选择器出现在提示框下方。也可在 **Settings > Tools > AI > Agents** 的 **Marketplace** 标签页安装。
- Zed：执行 `agent: open settings`，**External Agents** > **Add** > **Install from Registry**，搜 **Antigravity**。
- Xcode：**Xcode > Settings**（Cmd + ,）> **Intelligence**，点 **Get**；装好后选中 **Antigravity**，点 **…** 开始认证。

### IDE 扩展：其他编辑器（ACP）

#### 原理

Antigravity CLI 内置 ACP（Agent Client Protocol）服务器 `agy acp`，通过 `stdin` / `stdout` 以 JSON-RPC 通信。编辑器把它当子进程启动，在自己的面板里渲染回复、工具调用、diff 和权限请求；Agent、工具和设置来自 Antigravity CLI，所以行为与终端一致。可用功能取决于编辑器。

准备：安装 Antigravity CLI，用 `agy --version` 确认；先用交互式 `agy` 登录一次，或执行 `agy acp --login`；用 `which agy` 取绝对路径（有些编辑器启动 Agent 时没有你 shell 的 `PATH`）。

#### 各客户端配置

| 客户端 | 配置 | 要点 |
|---|---|---|
| VS Code | 社区扩展 ACP Client + 用户 `settings.json` 的 `acp.agents` | 该扩展由社区维护，不是 Google；官方扩展见上节 |
| Zed | `settings.json` 的 `agent_servers`，`"type": "custom"` | Agent Panel 点 **+**，在 **External Agents** 选 **Antigravity** |
| JetBrains | `~/.jetbrains/acp.json` 的 `agent_servers` | 在 **AI Chat** 工具窗口（需 **AI Assistant** 插件）；`command` 用绝对路径 |
| Android Studio | ACP Registry | 见上节 |
| Xcode | **Settings > Intelligence > Agents > Add an Agent** | **Command** 填绝对路径，**Arguments** 填 `acp`；Xcode 不展开 `~`，也不读 shell `PATH` |
| 其他 | 运行 `agy acp`，工作目录设为项目目录 | 服务器自称 `Antigravity ACP Server` |

```json
{
  "agent_servers": {
    "Antigravity": {
      "type": "custom",
      "command": "agy",
      "args": ["acp"]
    }
  }
}
```

#### 认证与 flags

无缓存凭证时请求以 `Authentication required`（错误码 `-32000`）失败，客户端可发起这些方式：`oauth-personal`、`oauth-business`、`gemini-api-key`（读 Agent 环境里的 `GEMINI_API_KEY`）、`agent-platform`（原 Vertex AI，用 ADC 加 `GOOGLE_CLOUD_PROJECT`、`GOOGLE_CLOUD_LOCATION`，或 `settings.json` 里的 `gcp.project`、`gcp.location`）、`terminal-login`（可用于 SSH，仅提供给支持 ACP 终端认证的客户端）。

flags 写在 `acp` 之后，如 `["acp", "--sandbox"]`：`--add-dir`（加工作区目录，可重复）、`--agent`（指定 Agent，见 `agy agents`）、`--sandbox`（终端沙箱，默认 `false`）、`--dangerously-skip-permissions`（自动批准所有权限请求，默认 `false`）、`--process-per-session`（每会话独立 worker；编辑器配置 HTTP MCP 服务器时需要）、`--login`、`--log-file`、`--ide-mcp-port`（需 `nc` 在 `PATH`）、`--disable-telemetry`。

#### 坑

- 编辑器设置文件是明文且常被同步；密钥可用 `"env": { "GEMINI_API_KEY": "<your-api-key>" }` 传入，但不要提交含密钥的设置文件，优先用 `agy` 登录。
- `--dangerously-skip-permissions` 只应在沙箱或一次性环境使用。
- 第三方模型只提供给 JetBrains IDE（含 JetBrains Air）、Android Studio、Android Performance Analyzer、Zed 和 Xcode；Gemini 模型在所有客户端可用。thinking level 取值 `Low`、`Medium`、`High`、`Extra`、`Max`。
- 排错：先 `agy acp --help` 确认版本支持 ACP；终端能用而编辑器不行时把 `agy` 换成绝对路径；日志在 `~/.gemini/antigravity-cli/log/acp-*.log`。

### IDE：Tab 补全（Supercomplete 与 Tab-to-Jump）

#### 功能

仅 Antigravity IDE。

| 功能 | 行为 |
|---|---|
| Supercomplete | 在光标附近给建议，可修改文件内多处（如同时改变量名和另一处函数定义），也可建议补缺失的 import。新增以灰字预览，删除以红色删除线。Tab 接受，Esc 放弃 |
| Tab-to-Jump | 在预测的下一处编辑所在行末显示 **Tab to Jump** 标签，按 Tab 直接跳过去 |

#### 设置

点击状态栏的 **Tab Settings**：**Suggestions in Editor** 开关编辑器内建议；**Tab Speed** 取 `Fast`（默认）或 `Slow`（等更多上下文再建议）；**Tab to Jump** 开关跳转。

### IDE：Agent side panel 与编辑器内 Review changes

#### Agent side panel

适用：IDE 扩展（VS Code、Visual Studio、Eclipse）与 Antigravity IDE。

- 头部：新对话、打开侧栏切换对话、跨对话搜索。
- 输入框：`@` 提及文件与上下文，`/` 运行动作，**+** 添加文件、文件夹，并可选模型。
- 每个回合展示工作过程（展开 **Worked for**）、回复，以及改动文件摘要和 **Review** 按钮。
- 有待审改动时，输入框上方出现形如 **1 File With Changes** 的条，带 **Reject all** / **Accept all**。

#### Review changes

流程：行内高亮，每处改动带 Accept / Reject（`Alt+Enter` 接受、`Alt+Shift+Enter` 拒绝，macOS 用 `Option`）；侧面板 Accept all / Reject all，或点回合摘要里的 Review 跳到改动。

- 给 Agent 留评论：选中行，按 `Cmd+Alt+M`（Windows/Linux 为 `Ctrl+Alt+M`）或点行号槽的 **+**，输入后点 **Add Comment to Chat**，评论排队到下一条提示。
- 边界：该页只覆盖 Antigravity IDE；App 或 CLI 里的 diff 审查与提交见另一页“Review changes”。

### IDE：浏览器（Browser、recordings、allowlist/denylist、Chrome profile）

#### /browser 与前置条件

在 Antigravity IDE（`v2.6.0+`）和 VS Code 扩展（`v1.9.0+`）里，Agent 可通过 `/browser` 打开、读取并操作**本机 Google Chrome**，用于测试本地站点、读文档、验证视觉改动。

链路：`/browser <任务>` → browser subagent → Chrome DevTools MCP 服务器（Antigravity IDE 用 `npx` 启动）→ 本机已运行、开启远程调试的 Chrome（共享你当前的 profile 与登录态）。

1. 安装 Node.js，使 `npx` 在 `PATH` 中。
2. 在**与工作区同一台机器**上打开 Chrome 144 或更高，访问 `chrome://inspect/#remote-debugging`，开启 **Allow remote debugging for this browser instance**。
3. 运行 `/browser <task>`，Chrome 询问时 60 秒内点 **Allow**。

设置入口：状态栏点 **Antigravity** > **Open Settings** > **Browser**。

- **Enable Browser Tools**：开关 Agent 浏览器工具。
- **Browser Javascript Execution Policy**：**Request Review**（默认）/ **Always Proceed** / **Disabled**（阻止页面内自定义 JavaScript `evaluate_script`）。
- **Browser Actuation Rules**：点 **Edit** 配置允许与拒绝的站点。

subagent 能导航、点击和填表、看控制台消息与网络请求、跑性能 trace 与 Lighthouse 审计、截图，并可选地把交互录成 WebM。

坑：
- **仅限本地工作区**：Remote-SSH、Dev Containers、云工作站里，远端 Agent 宿主连不到你本机的 Chrome。
- IDE 里自然语言提示不会自动拉起 browser subagent，必须输入 `/browser`。
- 共享一个 live Chrome 连接和当前 profile，一次只跑一个浏览器会话。
- 内置侧栏浏览器、自动调用、并行标签页等仅在 **Antigravity App**（`v2.30.0+`，个人 Google 账号）；Antigravity IDE 没有内置浏览器。

#### 录制（Browser recordings）

subagent 点击、输入、填表时，Antigravity IDE 可把操作录成 WebM。**不保证产出**：只打开页面或截图的会话没有录像，有交互的会话也不一定有。产出后保存为 browser subagent 产物中的 `recording.webm`，在 subagent 结束后由 Agent 告知。

#### Allowlist 与 denylist

适用：Antigravity App、IDE 扩展、Antigravity IDE。subagent 要打开未允许的站点时会提示 **Allow executing actions on this URL?**：

选项：**Yes, allow this time**（本次任务内放行）、**Yes, allow** _domain_ **in this conversation**（对话结束前放行）、**Yes, always allow** _domain_（在规则里加 allow）、**No**（拒绝，可告诉 Agent 改做什么）。

规则列表有两个：**Execute URLs**（`execute_url`）与 **Download files**（`download_url`）。点 **Add**，选 **Allow**、**Ask** 或 **Deny**，填入如 `https://example.com` 或 `localhost:3000`。

| 匹配规则 | 说明 |
|---|---|
| 域名与子域名 | `example.com` 同时覆盖 `docs.example.com`，不区分路径；每次跨源重定向都会检查 |
| Deny 优先 | 同一站点 Deny 压过 Allow，包括 **Turbo** 权限预设下 |
| `localhost` 按端口隔离 | `localhost:3000` 不覆盖 `localhost:8080`；不带端口则匹配所有端口 |
| 敏感 Google 子域 | `accounts.google.com`、`mail.google.com`、`docs.google.com`、`drive.google.com`、`console.cloud.google.com` 等需单独规则，`google.com` 的规则不覆盖；登录与支付主机始终禁止 Agent 操作 |
| 直接读 URL | 主 Agent 不开浏览器直接读网页，由 **Network Access Rules**（`read_url`）控制，在 Permissions 配置 |

组织管理员若限定了域名，subagent 不能打开其他站点；这些域名显示在 **Organization Allowed Domains (Read-Only)**，且仍会先询问，除非你加了 allow 规则。管理员禁用 browser subagent 时，设置里显示提示而不是规则。

#### 单独 Chrome profile

| 场景 | profile 行为 |
|---|---|
| App 内置浏览器（个人 Google 账号，`v2.30.0+`） | 在 App 侧栏浏览器里运行，每个 Antigravity 账号有隔离的持久 cookie profile，与 Chrome 分离 |
| 经 `/browser` 的 Chrome（Antigravity IDE、CLI，以及 App 中的商业 / 企业账号） | **不创建单独的 Chrome profile**，直接连接已运行的 Chrome，共享你日常浏览的 cookie 和登录态 |

所以在 IDE 里实际**没有**单独 profile：它能在你已登录的站点上操作，只放行你愿意让它操作的站点。内置浏览器的历史清理（仅 App）：**Settings > Browser > Built-in Browser** 里 **Browsing History** 旁点 **Clear**。

### IDE：Workflows（已弃用）与 skills

#### 现状与迁移

仅 Antigravity IDE。文档提示：**Workflows 已弃用，新的 workflows 系统在路上**，现有 workflows 暂时继续可用。建议迁移到 Agent skills：按迁移指南手动迁移；自动迁移命令 `/migrate-workflows` 仅在 Antigravity App 2.12.0 及更高版本可用。

#### 用法

| 项目 | 说明 |
|---|---|
| 是什么 | 用 Markdown 文件保存的一串步骤，用于重复任务（如部署服务、回应 PR 评论） |
| 创建 | Agent 面板顶部 **…** 菜单打开 **Customizations** 面板，进入 **Workflows**，点 **\+ Global**（所有工作区）或 **\+ Workspace**（当前工作区） |
| 调用 | 在 Agent 里用 `/workflow-name`；workflow 里可写“Call `/workflow-2`”来调用其他 workflow |
| 限制 | 每个 workflow 文件最多 12,000 个字符 |
| 与 rules 的区别 | rules 是提示层面的持久上下文；workflows 是轨迹层面的步骤序列 |
| Agent 生成 | 可让 Agent 生成，尤其适合先手动带它走完一串步骤之后，它能利用对话历史 |

#### 坑

新工作别再从 workflows 起步，直接用 skills；SDK 里对应的是 `skills_paths` 指向的含 `SKILL.md` 的目录。

---

## 常见场景速查

| 需求 | 推荐做法 |
|---|---|
| 项目约定每次都要遵守 | Rule（`trigger` 用 `always_on`） |
| 步骤固定的流程（发版、迁移） | Skill |
| 读取外部系统（工单、数据库、设计稿） | MCP，或打包了连接器的 Plugin |
| 事前预览智能体的做法 | `/plan`（含需求澄清可叠加 `/grill-me`） |
| 一个难题想要更深的推理 | `/boost` |
| 又宽又浅的任务（大范围搜索、批量调研） | `/swarm` |
| 对一批条目重复做同一件事 | `/run-workflow` |
| 隔离一个支线任务 | subagent |
| 定时、周期性任务 | `/schedule`（对话内）/ `/automation`（跨会话，仅 App） |
| CI 或脚本里运行 | `agy -p`，用 `--mode` 固定执行模式，并配好 `permissions.allow` |
| 在自己的产品里嵌入智能体 | Antigravity SDK |
| 在别的编辑器里用 | IDE 扩展，或经 ACP 接入 |

## 安全与维护清单

- [ ] 先用默认权限预设熟悉行为，再考虑 Auto / Turbo；Turbo 下智能体不会逐项询问。
- [ ] 权限冲突时 Deny 优先于 Ask 优先于 Allow；写 `ask` 的 `command(*)` 会压过所有 `command(...)` 的 allow。
- [ ] 沙箱不可用时会回退到审批流程，不要默认它一定生效，用前确认平台支持。
- [ ] 密钥只放环境变量或 SDK 变量，不写进提交到仓库的 `mcp_config.json`、Rules、Skills。
- [ ] 无人值守（headless / `/automation`）时需要批准的操作会被拒绝，先做一次测试运行并检查权限规则。
- [ ] 无人值守时用 `--mode` 或 `agentMode` 固定模式，不依赖版本相关的默认值。
- [ ] Hook 的非零退出码被忽略（fail open）；需要拦截时必须在 stdout 返回 `{"decision":"deny"}`。
- [ ] Review 面板里的 **Discard unstaged changes** 无二次确认，还会删除未跟踪文件。
- [ ] 不要把 Antigravity 登录用到第三方软件里，原文说明这违反服务条款，可能导致封号。
- [ ] 改完配置后重启，并用命令或界面实际验证加载结果。

## 常见问题

### Rules 和 Skills 有什么区别？

Rule 是按 `trigger` 注入的长期约定（始终、按模型判断、按文件匹配或手动 `@rules:` 引用）；Skill 是按任务加载的完整工作流，可带脚本和资料。

### Rule 写了却不生效？

检查 `.agents/rules/` 下的文件是否有 frontmatter，`trigger` 是否写成合法值（`always_on`、`model_decision`、`glob`、`manual`）；写成 `alwaysOn` 之类会被静默丢弃。该目录只扫描直接子级文件，更深的子目录不加载。

### 为什么 Hooks 写进 `settings.json` 没用？

Hooks 要写在独立的 `hooks.json`（全局在 `~/.gemini/config/`，工作区在 `.agents/`）。

### 用 Gemini API key 登录 CLI 为什么没生效？

需要同时在 CLI 的 `settings.json` 里设置 `"modelProvider": "gemini"` 并导出 `GEMINI_API_KEY`，只设其一不生效。

### 自己域名的 Workspace 账号为什么额度很少？

Workspace 账号本身不含 Antigravity 套餐，用 **Continue with Google** 默认只有有限的 starter 额度。要更多用量：用个人账号订阅，或选 **Pay As You Go** 连接自己的 Google Cloud 项目，或由管理员配置 Gemini Enterprise。

### headless 运行结果是成功，但任务其实没做完？

需要批准却拿不到批准的工具调用会被“软拒绝”：运行继续、退出码为 0，只在 stderr 打印通知。CI 要靠 `permissions.allow` 规则放行所需命令，并检查 `--output-format json` 里的 `status`，不要只看退出码。

### `/browser` 打不开？

它只能在本地工作区使用，需要 Chrome 144 或更高并开启远程调试；Remote-SSH、Dev Containers、云工作站里不可用。

### subagent / swarm 是越多越好吗？

不是。它们都从空白上下文开始，消耗配额更快（`/swarm` 是预览功能且明显更快用完配额）。只有互相独立的任务才适合并行。

> 文档与功能会随 Antigravity 更新。复制配置前，请以当前客户端和官方页面为准。
