# Claude Code 全景指南：配置、自定义与开发者实践

> 🧭 **其他指南**：[Codex 全景指南](README.md) · [Antigravity 全景指南](Antigravity%20%E5%85%A8%E6%99%AF%E6%8C%87%E5%8D%97.md)

> 面向 Claude Code 新手与开发者：从安装、会话、记忆与权限，到 Skills、MCP、Hooks、Subagents、插件、企业部署和 Agent SDK，提供属性说明、操作流程与完整示例。
>
> 内容依据 Claude Code 官方中文文档整理，章节结构对应官方文档的八个标签：快速入门、使用 Claude Code 构建、插件、管理、配置、参考、Agent SDK、资源。
>
> 最后核对：2026-11。产品更新很快，命令、设置键和版本号请以官方页面为准。

## 阅读导航

| 我想…… | 去这里 |
|---|---|
| 第一次安装和使用 | Part I →“安装、登录与第一次使用” |
| 知道该用 CLAUDE.md、Skill、Hook、MCP 还是插件 | Part I →“扩展机制选型” |
| 让 Claude 记住项目规则 | Part I →“记忆：CLAUDE.md、rules 与自动记忆” |
| 恢复、命名、分叉会话 | Part I →“会话管理” |
| 选择终端 / IDE / 桌面 / 云端 / 移动端入口 | Part I →“先看选型：我该用哪个入口？” |
| 代码审查、GitHub Actions、GitLab CI | Part I →“代码审查与 CI” |
| 并行跑多个代理、worktree | Part II →“代理与并行工作” |
| 连接外部系统、写 Skill、自动化 | Part II → MCP / Skills / Hooks / `claude -p` |
| 安装或开发插件、搭建 marketplace | Part III |
| 团队部署、云厂商、网关、监控与成本 | Part IV |
| 写 settings.json、配置权限与沙箱 | Part V |
| 查命令、环境变量、快捷键、Hook 事件 | Part VI |
| 在自己的程序里嵌入 Claude Code | Part VII：Agent SDK |
| 从 Codex 迁移过来 | 下一节“Codex ↔ Claude Code 对照表” |
| 配置不生效 | 文末“常见问题” |

## Claude Code 全景图

```text
Claude Code
│
├─ 快速入门（Part I）
│  ├─ 入口：终端 CLI / VS Code / JetBrains / 桌面 / 云端 Web / 移动端 / Slack / Chrome
│  ├─ 核心概念：agentic loop、内置工具、上下文窗口、prompt caching
│  ├─ 日常使用：CLAUDE.md 记忆、会话、常见工作流、最佳实践
│  └─ 代码审查：/code-review、ultrareview、托管 Code Review、GitHub Actions、GitLab
│
├─ 使用 Claude Code 构建（Part II）
│  ├─ 代理：subagents / agent view / agent teams / workflows / worktrees
│  ├─ MCP、Skills、Artifacts
│  └─ 自动化：hooks、channels、/loop、/goal、claude -p、深链接
│
├─ 插件（Part III）：使用 / 创建 / Mods / Marketplace / 组织管理
├─ 管理（Part IV）：安装认证、托管设置、云厂商、网关、监控、安全与数据
├─ 配置（Part V）：settings.json、权限、权限模式、沙箱、环境、模型、界面
├─ 参考（Part VI）：CLI、命令、环境变量、工具、交互模式、检查点、Hooks
├─ Agent SDK（Part VII）：TypeScript / Python 程序化构建代理
└─ 资源：法律与合规
```

## 小白先走这条路线

```text
安装 Claude Code 并登录
      │
      ▼
在项目目录运行 claude，先问“这个项目是做什么的？”（只读）
      │
      ▼
运行 /init 生成 CLAUDE.md，补上构建、测试命令和禁区
      │
      ▼
让它做一个小改动，观察权限提示；用 /permissions 记住常用命令
      │
      ▼
学会 /resume、/rewind、/compact、/context
      │
      ▼
按需增加 Skill / MCP / Hook / Subagent / 插件
```

完成后你应该看到：Claude 会自动读取 CLAUDE.md 中的规则；有风险的操作会请求批准；改坏了可以用 `/rewind` 回退文件编辑；关掉终端后能用 `claude --continue` 继续。

## Codex ↔ Claude Code 对照表

如果你读过本仓库的 Codex 指南（`README.md`），可以用这张表快速对应概念。两者设计不同，“对应”只表示解决的问题相近，字段不能互相照抄。

| 需求 | Codex | Claude Code |
|---|---|---|
| 用户级配置 | `~/.codex/config.toml`（TOML） | `~/.claude/settings.json`（JSON） |
| 项目级配置 | `.codex/config.toml` | `.claude/settings.json`（提交）+ `.claude/settings.local.json`（个人） |
| 团队强制策略 | `requirements.toml` | 托管设置 `managed-settings.json` / 服务器托管设置 |
| 项目长期指令 | `AGENTS.md` | `CLAUDE.md`（可用 `@AGENTS.md` 导入，见记忆一节） |
| 跨会话记忆 | Memories | 自动记忆（auto memory） |
| 何时请求批准 | `approval_policy` | 权限模式（`default`、`acceptEdits`、`plan`、`auto` 等）+ allow/ask/deny 规则 |
| 命令隔离 | `sandbox_mode` | 沙箱化 Bash 工具（`sandbox.*` 设置） |
| 命令前缀规则 | Rules（`prefix_rule`） | 权限规则 `Bash(...)` |
| 可复用工作流 | Skills（`.agents/skills`） | Skills（`.claude/skills/<name>/SKILL.md`） |
| 外部工具 | `[mcp_servers.*]` | `claude mcp add`、`.mcp.json` |
| 子代理 | `[agents]`、`.codex/agents/*.toml` | `.claude/agents/*.md`（YAML frontmatter） |
| 生命周期钩子 | Hooks | Hooks（`settings.json` 的 `hooks`） |
| 分发单元 | Plugins | 插件 + marketplace |
| 非交互运行 | `codex exec` | `claude -p` |
| 程序化 | Codex SDK | Claude Agent SDK |
| CI 审查 | `openai/codex-action` | Claude Code GitHub Actions / 托管 Code Review |
| 本地审查 | `/review` | `/code-review` |

---

## Part I：快速入门

### Claude Code 是什么与可用入口

Claude Code 是一个代理式（agentic）编码工具：它读取你的代码库、编辑文件、运行命令，并与开发工具集成。所有入口连接同一个底层引擎，因此 `CLAUDE.md`、设置和 MCP 服务器在各入口通用。

| 入口 | 目的 | 备注 |
|---|---|---|
| 终端 CLI | 功能完整的命令行入口 | 支持第三方提供商 |
| VS Code / Cursor 扩展 | 内联差异、@-提及、计划审查、对话历史 | 支持第三方提供商 |
| JetBrains 插件 | 交互式差异、选择上下文共享 | 需要单独安装 CLI |
| 桌面应用 | 可视化差异、并行会话、定期任务、云会话 | 需要付费订阅；Ubuntu/Debian 上为测试阶段 |
| Web（claude.ai/code）与移动应用 | 无需本地环境，适合长任务和本地没有的仓库 | 用 `claude --teleport` 拉回终端需要 claude.ai 订阅 |
| 集成 | GitHub Actions、GitLab CI/CD、Slack、Chrome、远程控制、Agent SDK | 按场景选用 |

界面之外还有三种执行环境：**本地**（默认，在你的机器上）、**云**（Anthropic 管理的虚拟机或组织自托管环境）、**远程控制**（代码仍在你的机器上运行，从浏览器控制）。

> 边界提醒：默认配置下需要能访问 claude.ai 和 Anthropic API 等端点才能安装、登录和使用；在中国大陆网络环境中这些端点可能无法直接访问，开始前先确认连通性。

### 安装、登录与第一次使用

你需要一个终端、一个代码项目，以及以下账户之一：Claude 订阅（Pro、Max、Ultra、Team 或 Enterprise）、Claude Console 账户，或受支持云提供商的访问权限。

#### 安装方式

| 方式 | 命令 | 自动更新 |
|---|---|---|
| 原生安装（推荐），macOS / Linux / WSL | `curl -fsSL https://claude.ai/install.sh \| bash` | 是，后台自动更新 |
| 原生安装，Windows PowerShell | `irm https://claude.ai/install.ps1 \| iex` | 是 |
| 原生安装，Windows CMD | `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd` | 是 |
| Homebrew | `brew install --cask claude-code` | 否，需 `brew upgrade claude-code` |
| WinGet | `winget install Anthropic.ClaudeCode` | 否，需 `winget upgrade Anthropic.ClaudeCode` |

Homebrew 的 `claude-code` 跟踪稳定渠道（通常比最新版晚约一周），`claude-code@latest` 跟踪最新渠道。原生 Windows 建议安装 Git for Windows，否则 Claude Code 用 PowerShell 作为 shell 工具；WSL 不需要。

#### 操作流程

1. 安装后**打开新终端窗口**，运行 `claude --version`，应打印版本号并跟着 `(Claude Code)`。
2. 进入项目并启动：

```bash
cd /path/to/your/project
claude
```

3. 首次使用按提示在浏览器中登录。如果已设置 `ANTHROPIC_API_KEY` 并在询问时批准使用，会跳过登录提示。
4. 用只读问题测试：`what does this project do?`
5. 做一次小改动：`add a hello world function to the main file`。若它征求确认，选择 **Yes**。
6. 试 Git：`what files have I changed?`、`commit my changes with a descriptive message`。

#### 最常用的命令

| 位置 | 命令 | 目的 |
|---|---|---|
| Shell | `claude` / `claude "task"` | 启动交互模式 / 带初始提示启动 |
| Shell | `claude -p "query"` | 运行一次性查询后退出 |
| Shell | `claude -c` / `claude -r` | 继续当前目录最近的对话 / 恢复之前的对话 |
| 会话内 | `/help` | 显示可用命令 |
| 会话内 | `/clear` | 开始新对话；旧对话可用 `/resume` 找回 |
| 会话内 | `/login` | 切换账户或重新认证 |
| 会话内 | `/exit` 或 Ctrl+D 两次 | 退出 |
| 会话内 | `Shift+Tab` | 循环切换权限模式 |

#### 失败排查

| 现象 | 原因与处理 |
|---|---|
| 找不到 `claude` 命令 | 安装目录不在 PATH 中，见官方"排查安装问题" |
| `The token '&&' is not a valid statement separator` | 你在 PowerShell 中，却用了 CMD 的命令 |
| `'irm' is not recognized as an internal or external command` | 你在 CMD 中，却用了 PowerShell 的命令 |
| 其他安装或配置问题 | 会话内运行 `/doctor` 诊断 |

### Agentic loop 与内置工具如何工作

给 Claude 一个任务时，它在三个阶段之间循环，并根据上一步学到的内容决定下一步。你随时可以打断。

```text
你的提示
   │
   ▼
┌──────────┐    ┌──────────┐    ┌──────────┐
│ 收集上下文 │ ─▶ │ 采取行动  │ ─▶ │ 验证结果  │ ──▶ 完成
└──────────┘    └──────────┘    └──────────┘
      ▲                               │
      └────────── 未完成则继续 ─────────┘

任意时刻：Esc 立即停止；或直接输入更正并 Enter（排队，不打断）
```

循环由两部分驱动：**模型**负责推理，**工具**负责行动。Claude Code 是包在模型外面、提供工具并管理上下文的那一层。用 `/model` 在会话中切换模型，或用 `claude --model <name>` 启动。

| 工具类别 | Claude 可以做什么 |
|---|---|
| 文件操作 | 读取文件、编辑代码、创建新文件、重命名和重新组织 |
| 搜索 | 按模式查找文件、用正则搜索内容、探索代码库 |
| 执行 | 运行 shell 命令、启动服务器、运行测试、使用 git |
| 网络 | 搜索网络、获取文档、查找错误消息 |
| 代码智能 | 编辑后查看类型错误和警告、跳转到定义、查找引用（需要代码智能插件） |

#### 两个安全机制

**Checkpoints**：Claude 编辑文件前先做快照，用 `/rewind` 或按两次 `Esc` 回退。它只覆盖 Claude 用自身文件工具做的编辑；仅被 Bash 命令改动的文件不会恢复，数据库、API、部署等远程操作也无法回退。

**权限模式**：决定 Claude 不询问就能做什么，按 `Shift+Tab` 循环切换。

常见模式有四种：**Auto**（分类器在后台审查操作，阻止有风险的操作而不是询问你；v2.1.283 或更高版本中是交互式终端和 VS Code 会话的内置起始模式）、**Manual**（文件编辑和 shell 命令之前都询问）、**Accept edits**（编辑文件和 `mkdir`、`mv` 等常见文件系统命令不询问）、**Plan**（只探索并提出计划，不编辑源文件）。

> 边界提醒：Auto 模式是模型判断，不是硬性规则。"绝不"类规则应放在权限 `deny` 规则或 hook 中。

### 扩展机制选型：什么时候用哪个

内置工具覆盖大多数编码任务。扩展层用来补充 Claude 的知识、连接外部服务和自动化工作流。不需要一次配齐，从 CLAUDE.md 开始，出现下面的信号时再添加。

| 机制 | 作用 | 何时用 | 上下文成本 |
|---|---|---|---|
| CLAUDE.md | 每次对话加载的持久上下文 | 项目约定、"始终执行 X" 规则 | 每个请求都占用 |
| `.claude/rules/` | 按主题拆分、可限定路径的指令 | 只对某种语言或目录有意义的规则 | 每个会话，或打开匹配文件时 |
| Skill | 可重用的说明、知识和工作流 | 参考文档、可重复任务、用 `/<name>` 触发的流程 | 低：启动时只加载描述，使用时加载全文 |
| Subagent | 返回摘要的隔离执行上下文 | 上下文隔离、并行任务、专门的工作者 | 与主会话隔离 |
| Hook | 由生命周期事件触发的脚本、HTTP 请求、MCP 工具调用、提示或 subagent | 必须在每个匹配事件上运行的自动化 | 零，除非 hook 返回输出 |
| MCP | 连接外部服务 | 外部数据或操作 | 低：启动时只有工具名称，完整 schema 按需加载 |
| Plugin | 打包层：把 skills、hooks、subagents、MCP servers 捆成一个可安装单元 | 多个仓库复用同一套设置，或通过 marketplace 分发 | 取决于包含的组件 |

官方选型表还包括 Output style、Code intelligence、Dynamic workflow 等，按需查阅。

#### 决策树

```text
你想加的东西是什么？
├─ Claude 每个会话都该知道的事实或约定 ─────────▶ CLAUDE.md
│    └─ 只对部分路径有意义 ───────────────────▶ .claude/rules/ + paths
├─ 回复本身的语气、长度、格式 ─────────────────▶ Output style
├─ 偶尔才需要的参考资料，或用 /<name> 触发的流程 ──▶ Skill
│    └─ 有副作用（如发布）───────────────────▶ 加 disable-model-invocation: true
├─ 必须每次都发生 / 绝不能发生，零例外 ──────────▶ Hook 或权限 deny 规则
├─ 数据或操作在外部系统里 ────────────────────▶ 有 CLI 先用 CLI，否则 MCP
├─ 要读很多文件、输出嘈杂、可并行、需要独立审查 ──▶ Subagent
│    └─ 规模超出少数几个 subagent ─────────────▶ Dynamic workflow
└─ 第二个仓库也需要同一套设置 ─────────────────▶ Plugin
```

#### 同名时如何分层

- **CLAUDE.md** 累加：所有级别同时进入上下文，冲突时由 Claude 判断。
- **Skills / Subagents / MCP 服务器** 按名称覆盖：skills 为托管 > 用户 > 项目；subagents 为托管 > CLI 标志 > 项目 > 用户 > plugin；MCP 服务器为本地 > 项目 > 用户。Plugin skills 带命名空间（如 `/my-plugin:review`）。
- **Hooks** 合并：所有注册的 hook 都为其匹配的事件触发。

> 边界提醒：CLAUDE.md 或 skill 里的"永远不要编辑 `.env`"是请求，不是保证；阻止编辑的 `PreToolUse` hook 才是强制执行。

### `.claude` 目录与 `~/.claude` 目录

大多数用户只编辑 `CLAUDE.md` 和 `settings.json`，其余都是可选的。项目文件提交到 git 与团队共享；`~/.claude` 是个人配置，适用于所有项目。

```text
your-project/
├── CLAUDE.md                   # 项目指令，会话开始时加载（也可放在 .claude/CLAUDE.md）
├── CLAUDE.local.md             # 个人私有偏好；手动创建并加入 .gitignore
├── .mcp.json                   # 团队共享的 MCP 服务器
├── .worktreeinclude            # 新 worktree 需要复制的 gitignored 文件
└── .claude/
    ├── settings.json           # 权限、hooks、环境变量、模型默认值（提交）
    ├── settings.local.json     # 个人覆盖；Claude Code 写入时自动 gitignore
    ├── rules/*.md              # 按主题拆分的指令，可用 paths 限定路径
    ├── skills/<name>/SKILL.md  # 用 /name 调用或由 Claude 自动调用
    ├── commands/*.md           # 单文件命令；新命令建议用 skill
    ├── agents/*.md             # subagent 定义
    ├── workflows/*.js          # 动态工作流脚本
    ├── output-styles/*.md      # 自定义输出样式
    └── agent-memory/<agent-name>/MEMORY.md   # subagent 的持久记忆
```

```text
~/.claude.json                  # 账户详情、各项目信任决定、个人 MCP 服务器、UI 开关
~/.claude/
├── CLAUDE.md                   # 所有项目的个人指令
├── settings.json               # 所有项目的个人设置
├── keybindings.json            # 自定义快捷键（/keybindings 创建或打开）
├── themes/*.json               # 自定义颜色主题
├── rules/  skills/  commands/  agents/  workflows/  output-styles/  agent-memory/
├── plugins/                    # 已安装的插件，由 claude plugin 命令管理
├── projects/<project>/
│   ├── <session-id>.jsonl      # 会话记录
│   └── memory/                 # 自动记忆：MEMORY.md + 主题文件
├── file-history/<session>/     # checkpoint 用的编辑前快照
└── history.jsonl               # 你输入过的提示词历史
```

在 Windows 上 `~/.claude` 解析为 `%USERPROFILE%\.claude`。设置 `CLAUDE_CONFIG_DIR` 后，上述 `~/.claude` 路径都位于该目录下。

设置文件优先级从高到低：托管设置 ▶ `--settings` 标志 ▶ `.claude/settings.local.json` ▶ `.claude/settings.json` ▶ `~/.claude/settings.json`。

#### 应用数据与隐私边界

- 会话记录和提示历史是**纯文本、未加密**的，只靠操作系统文件权限保护。工具读到的 `.env` 内容或命令打印的凭证会写进 `projects/<project>/<session>.jsonl`。
- 会话记录等数据超过 `cleanupPeriodDays`（默认 30 天，最小 1，设为 `0` 会验证报错）后自动清理；自动记忆文件不在清理范围内。
- 用 `claude purge` 删除某个项目的本地状态（v2.1.288 之前为 `claude project purge`）。先预览：`claude purge ~/work/my-repo --dry-run`。
- 不要删除 `~/.claude.json`、`~/.claude/settings.json` 或 `~/.claude/plugins/`。

### 上下文窗口：里面有什么、何时压缩

```text
输入之前      系统提示词 + CLAUDE.md + 自动记忆（MEMORY.md）+ MCP 工具名称 + skill 描述
     │
     ▼
Claude 工作时  每次文件读取、搜索结果、命令输出、hook 输出；
     │        路径限定规则和子目录 CLAUDE.md 随匹配文件按需加载
     ▼
委派时        subagent 在自己的窗口里读文件，只把摘要带回主对话
     │
     ▼
接近上限      自动压缩：用结构化摘要替换对话历史（/compact 可手动触发）
```

#### 压缩后保留什么

| 内容 | 压缩后 |
|---|---|
| 系统提示词和输出样式 | 仍然适用 |
| 项目根目录 CLAUDE.md、无范围规则、自动记忆 | 从磁盘重新注入 |
| 带 `paths:` frontmatter 的规则、子目录中的嵌套 CLAUDE.md | 按需重新加载（重新触发前不在上下文中） |
| Claude 读取或编辑过的文件 | 重新读取最多五个，最近修改的优先 |
| 已调用的 skill 正文 | 重新注入，每个 skill 上限 5,000 token，总计 25,000 token |
| 仅在对话中给出的指令 | 被摘要，细节可能丢失 |

#### 主动管理

| 动作 | 命令 | 什么时候用 |
|---|---|---|
| 查看占用 | `/context` | 想知道什么在占空间；`/context all` 展开全部 |
| 带重点压缩 | `/compact focus on the auth bug fix` | 开始较长的新任务之前 |
| 压缩部分对话 | `/rewind` 后选 **Summarize from here** 或 **Summarize up to here** | 只想丢掉某一段 |
| 更早压缩 | `/autocompact 500k` | 想自定义自动压缩的触发点 |
| 清空 | `/clear` | 切换到不相关的工作 |

> 边界提醒：持久规则写进 CLAUDE.md，不要依赖对话历史。若规则必须在压缩中保持不变，去掉 `paths:` frontmatter 或移到项目根目录 CLAUDE.md。是否有 100 万 token 窗口取决于模型和计划。

### Prompt caching 要点

Claude Code 每次发消息都会重新发送完整上下文。缓存让 API 复用已处理过的**前缀**，只完整处理变化的部分。Claude Code 自动处理缓存，你只需要知道哪些操作会让它失效。

请求按"很少变化的在前"排序：系统提示词（核心指令、工具定义）▶ 项目上下文（CLAUDE.md、自动记忆、无作用域的规则）▶ 对话。匹配是精确的：前缀中任何一处变化，其后所有内容都要重新计算。每个模型有自己的缓存。

| 使缓存失效（下一轮更慢更贵） | 保持缓存 |
|---|---|
| 用 `/model` 切换模型 | 编辑仓库中的文件 |
| 更改 effort 级别（大多数模型） | 会话中编辑 CLAUDE.md（要重新加载需 `/clear`、`/compact` 或重启） |
| 启用快速模式 | 更改权限模式 |
| 连接或移除 MCP 服务器 | 更改输出样式 |
| 启用或禁用插件 | 调用 skill 和命令 |
| 压缩对话 | 运行 `/recap` |
| 升级 Claude Code | 用 `/rewind` 回溯对话 |

缓存在一段不活动后过期（TTL）：主对话在 Claude 订阅且处于计划用量内时为一小时，使用额度、API 密钥或云提供商时为五分钟；subagent 等其他请求默认五分钟。可用 `promptCacheTtl`（主对话）和 `subagentPromptCacheTtl`（其他）自己选择，值只能是 `5m` 或 `1h`，需要 v2.1.242 或更高版本。

#### 验证

- 会话内运行 `/usage`，查看 `Prompt cache (main)` 行的命中率与未命中次数（需要 v2.1.251 或更高版本）。
- 运行 `claude -p "hello" --output-format json`，读取结果中的 `usage.cache_creation`：`ephemeral_1h_input_tokens` 是一小时写入，`ephemeral_5m_input_tokens` 是五分钟写入。

> 实用规则：会话开始时就选好模型和 effort 级别；把 `/compact` 留到任务之间的自然间歇。

### 记忆：CLAUDE.md、rules 与自动记忆

每个会话都从全新的上下文窗口开始。两套机制跨会话传递知识：**CLAUDE.md** 是你写的指令，每个会话加载；**自动记忆**是 Claude 根据你的更正和偏好自己写的笔记，按存储库保存。

> 两者都是**上下文**，不是强制执行的配置。要无条件阻止某个操作，用 `PreToolUse` hook。

#### CLAUDE.md 放在哪里

下表按加载顺序排列，从最广到最具体：

| 作用域 | 位置 | 目的 | 共享对象 |
|---|---|---|---|
| 托管策略 | macOS：`/Library/Application Support/ClaudeCode/CLAUDE.md`；Linux 和 WSL：`/etc/claude-code/CLAUDE.md`；Windows：`C:\Program Files\ClaudeCode\CLAUDE.md` | 组织范围指令 | 组织中的所有用户 |
| 用户指令 | `~/.claude/CLAUDE.md` | 所有项目的个人偏好 | 仅你 |
| 项目指令 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | 团队共享的项目指令 | 通过源代码控制的团队成员 |
| 本地指令 | `./CLAUDE.local.md` | 个人的项目特定偏好；加入 `.gitignore` | 仅你（当前项目） |

#### 加载规则

```text
启动时加载
  工作目录及其上方每个目录的 CLAUDE.md 和 CLAUDE.local.md
  ├─ 全部连接进上下文，互不覆盖
  ├─ 顺序：从文件系统根目录向下到工作目录（越近越晚读取）
  └─ 同一目录内：CLAUDE.local.md 追加在 CLAUDE.md 之后

按需加载
  工作目录下子目录中的 CLAUDE.md / CLAUDE.local.md
  └─ 当 Claude 读取、写入或编辑该子目录中的文件时才加载
```

| 属性 | 类型/默认值 | 作用 |
|---|---|---|
| `@path/to/import` | 导入语法；相对路径相对于**包含导入的文件**解析；递归最大深度四跳 | 把其他文件展开进上下文（不减少上下文成本） |
| `paths` | 规则文件 frontmatter；YAML 列表或逗号分隔字符串 | 让规则只在 Claude 处理匹配文件时加载 |
| `claudeMdExcludes` | 设置键；glob 数组，匹配绝对路径 | 跳过不相关的 CLAUDE.md（无法排除托管 CLAUDE.md） |
| `autoMemoryEnabled` | 设置键；本地会话默认开启 | 开关自动记忆 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 环境变量；设为 `1` | 禁用自动记忆 |

- 每个 CLAUDE.md 目标在 200 行以下；超过 4 MiB 的文件会被跳过。
- 用反引号包起来的 `@README` 不会被导入；块级 HTML 注释在注入上下文前被剥离。
- 项目级文件导入工作目录外的路径时，首次会弹出批准对话框。

#### AGENTS.md

默认情况下，只有当工作目录或其上方没有 `CLAUDE.md`、`.claude/CLAUDE.md` 或 `CLAUDE.local.md` 时，Claude 才读取 `AGENTS.md`（需要 v2.1.277 或更高版本）。想两者一起加载，在 `/config` 中把**项目说明**设为 `claude-md-and-agents-md`；或在 `CLAUDE.md` 中写 `@AGENTS.md` 导入。不读取 `AGENTS.local.md`、`AGENTS.override.md` 或 `.agents/` 目录。

#### 最小可用示例

`CLAUDE.md`：

```markdown
# Commands
- Install: pnpm install
- Run one test file: pnpm vitest run path/to/file.test.ts
- Typecheck: pnpm tsc --noEmit

# Conventions
- Use the error helpers in src/lib/errors.ts, never throw raw strings

# Additional Instructions
- git workflow @docs/git-instructions.md
```

`.claude/rules/api.md`（路径限定规则）：

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules

- All API endpoints must include input validation
- Use the standard error response format
```

#### 自动记忆

```text
~/.claude/projects/<project>/memory/
├── MEMORY.md            # 索引；会话开始时加载前 200 行或 25KB（以先到者为准）
├── user_role.md         # 主题文件，按需读取
└── feedback_testing.md
```

- Claude 保存四种笔记，用 frontmatter 的 `type` 字段标记：`user`、`feedback`、`project`、`reference`。
- 自动记忆是机器本地的，不在机器或云环境之间共享；同一 git 存储库的所有 worktrees 共享一个目录。
- 你说"记住……"时，Claude 存入自动记忆；想写进 CLAUDE.md 要明确说"将其添加到 CLAUDE.md"。

#### 操作流程

1. 会话内运行 `/init` 生成起始 CLAUDE.md（已存在时只建议改进，不覆盖）。
2. 删减到只剩 Claude 无法从代码推断的内容。
3. 只对部分路径有效的规则，拆到 `.claude/rules/` 并加 `paths`。
4. 验证：运行 `/context all`，检查 **Memory files** 下的列表。
5. 用 `/memory` 打开和编辑记忆文件、开关自动记忆。

#### 失败排查

| 现象 | 处理 |
|---|---|
| Claude 不遵循 CLAUDE.md | 先用 `/context all` 确认已加载；把指令写得更具体；查找冲突指令 |
| 子目录 CLAUDE.md 不在 **Memory files** 列表里 | 正常：它按需加载，加载时终端出现带路径的 `Loaded` 行 |
| `AGENTS.md` 没加载 | 检查路径上是否有 `CLAUDE.md`、`.claude/CLAUDE.md` 或 `CLAUDE.local.md`；确认版本 |
| `/compact` 后指令丢失 | 该指令只在对话中给出，或来自尚未重新触发的嵌套文件/路径规则；写进项目根 CLAUDE.md |
| 文件太大或有过时内容 | `/doctor` 给出修剪建议；`/doctor prompt-audit` 查过时与冲突指令（需要 v2.1.283 或更高版本） |

### 会话管理

会话是与项目目录关联的已保存对话，在你工作时持续保存到本地。本节只涵盖 CLI；桌面应用、claude.ai/code 和 VS Code 扩展各自维护会话列表。

| 命令 | 功能 |
|---|---|
| `claude --continue` | 重新打开当前目录中最近的会话 |
| `claude --resume` | 打开会话选择器 |
| `claude --resume <name>` | 直接恢复命名的会话 |
| `claude --resume <session-id>` | 恢复具有该 ID 的会话（可从任何目录运行） |
| `claude --resume <transcript-path>` | 恢复该绝对路径的 `.jsonl` 会话记录 |
| `claude --from-pr <number>` | 打开会话选择器，筛选链接到该 Pull Request 的会话 |
| `/resume` | 从活跃会话内切换到另一个会话 |

```text
恢复（--continue / --resume）    同一个会话 ID，新消息追加到原对话
分支（/branch / --fork-session） 复制历史到新的会话 ID，原会话保持不变
```

#### 命名、选择器与分支

- **命名**：启动时 `claude -n auth-refactor`；会话中 `/rename auth-refactor`；选择器中选中后按 `Ctrl+R`。
- **选择器**：`Enter` 恢复，`Space` 预览，`/` 搜索（可粘贴 PR 或 MR 的 URL）；`Ctrl+A` 显示所有项目，`Ctrl+W` 显示当前存储库所有 worktrees，`Ctrl+B` 过滤到当前 git 分支。
- **分支**：会话内运行 `/branch try-streaming-approach`；命令行运行 `claude --continue --fork-session`。

> 未命名会话的默认显示名称（如 `my-app-3f`）**不是**恢复句柄，传给 `--resume` 找不到会话。自动生成的会话标题可以用来恢复。

#### 导出与存储

| 目的 | 做法 |
|---|---|
| 导出当前对话 | `/export`（复制到剪贴板或保存为纯文本文件） |
| 脚本读取结果 | `claude -p --resume <session-id> --output-format json "summarize what we changed"` |
| 会话记录位置 | `~/.claude/projects/<project>/<session-id>.jsonl`，`<project>` 是工作目录路径，非字母数字字符替换为 `-` |
| 更改 30 天保留期 | `settings.json` 中的 `cleanupPeriodDays` |
| 禁止一次非交互式运行的写入 | `claude -p` 加 `--no-session-persistence` |

#### 边界提醒

- 恢复时**不会**还原这些启动标志，需要再次传递：`--mcp-config`、`--plugin-dir`、`--add-dir`、`--settings`、`--fallback-model`、`--tools`、`--allowedTools`、`--disallowedTools`。
- 同一会话同时在两个终端恢复，两边写同一份记录，消息可能缺失。要保留两条对话，给每个命令加 `--fork-session`。
- 用 `claude -p` 或 Agent SDK 创建的会话不出现在选择器和 `claude --continue` 中，但可用 `claude --resume <session-id>` 恢复。
- JSONL 条目是内部格式，版本间会变。脚本请用 `/export` 或上面的脚本接口，不要直接解析。

### 常见工作流

每个配方都是"先理解，再动手，再验证"的几步对话：

| 任务 | 依次输入 |
|---|---|
| 理解新代码库 | `give me an overview of this codebase` → `explain the main architecture patterns used here` → `trace the login process from front-end to database` |
| 修复错误 | `I'm seeing an error when I run npm test` → `suggest a few ways to fix the @ts-ignore in user.ts` → `update user.ts to add the null check you suggested` |
| 创建 PR | `summarize the changes I've made to the authentication module` → `create a pr` |

#### 给 Claude 递材料

| 方式 | 用法 | 备注 |
|---|---|---|
| `@` 引用文件 | `Explain the logic in @src/utils/auth.js` | 直接包含文件内容；大于 256KB 的文本文件不会被包含 |
| `@` 引用目录 | `What's the structure of @src/components?` | 只显示文件列表，不含内容 |
| 图像 | 拖放、`Ctrl+V` 粘贴，或给出图像路径 | Windows 和 WSL 上粘贴用 `Alt+V` |
| 管道 | `git log --oneline -20 \| claude -p "summarize these recent commits"` | 非交互运行 |

> 边界提醒：用 `@` 包含文件不经过工具调用，`PreToolUse` hook 不会运行。要管控读取，请为这些路径添加 `Read` 拒绝规则。

#### 四个值得养成的模式

| 模式 | 命令 | 目的 |
|---|---|---|
| 编辑前规划 | `claude --permission-mode plan`，或会话中按 `Shift+Tab` 直到显示 `⏸ plan mode on` | 在改动接触磁盘前审查 |
| 并行会话 | `claude --worktree feature-auth` | 每个任务独立检出，编辑互不冲突（仓库需至少一个提交） |
| 委派研究 | `use a subagent to investigate how our auth system handles token refresh` | 文件读取留在 subagent 的窗口里，只带回结论 |
| 跨会话继续 | `claude --continue` | 从离开的地方继续，而不是重新解释上下文 |

### 最佳实践精华

默认设置在大多数情况下已经够用。会话不如预期时，几乎总是下面四种情况之一：

| 现象 | 根因 | 对策 |
|---|---|---|
| 做出的东西与设想不同，或每次都重新发现同一事实 | 缺少仓库里读不到的信息 | 具体的提示词；精简的 CLAUDE.md；rules；skill；MCP |
| Claude 说完成了，但还差一点 | 没有办法检查自己的工作 | 给它一个返回通过或失败的检查 |
| 做了绝不该做的事，或跳过必须执行的步骤 | 提示词无法保证 | hook、权限 `deny` 规则、沙箱、托管设置 |
| 做对了，但耗时、token 或权限提示过多 | 默认设置不适合这个任务 | 调整权限模式、模型和 effort；`/usage` 看消耗 |

核心区分：提示词、`CLAUDE.md` 和 skill 是**非确定性**的，由 Claude 决定是否遵循；hook、拒绝规则和托管设置是**确定性**的，由 Claude Code 每次强制执行。如果你不断给 `CLAUDE.md` 的某一行加强调而 Claude 仍然遗漏，这一行应该放进 hook；反过来，只是偏好的事情不要做成 hook。

#### 可照抄：把规则变成强制执行

类型检查必须通过才能结束（`Stop` hook，退出码 2 阻止停止），并禁止读取密钥文件（`deny` 规则在每种权限模式下都有效）。放在 `.claude/settings.json`：

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "type": "command", "command": "pnpm tsc --noEmit 1>&2 || exit 2" }
        ]
      }
    ]
  },
  "permissions": {
    "deny": ["Read(./.env)", "Read(./.env.local)", "Read(./secrets/**)"]
  }
}
```

验证：让 Claude 做一个会破坏类型的改动，它应收到编译器错误并继续修复；再问它 `.env` 里有什么，无论用 `Read` 还是 `cat .env` 都应失败。

> 边界提醒：`Read` 规则能拦截 `cat .env` 这类常见读取命令，但拦不住一个自己打开该文件的脚本。安全相关事项要同时启用沙箱（会话内 `/sandbox`；限制 macOS、Linux 和 WSL2 上的 shell 命令）。

#### 引导会话

- `Esc`：中途停止 Claude，保留到那时为止的所有内容。
- `/rewind`，或空提示词上按两次 `Esc`：打开 rewind 菜单，把代码、对话或两者回滚到更早的提示词。
- `/clear`：同一件事更正两次仍然不对时，清空并写一个更好的提示词，而不是第三次更正。

#### 无人值守前的五个问题

1. 它知道完成是什么样子吗？（目标条件、规范文件、必须通过的失败测试）
2. 它能检查自己的工作吗？（测试套件、`Stop` hook、审查步骤）
3. 它被允许触及什么？（权限规则、`dontAsk`、沙箱、worktree）
4. 什么会让它停下来？（`--max-turns`、`--max-budget-usd`、支出限额）
5. 结果去哪里，谁来签核？（PR、评论或总结，由人批准合并）

### 先看选型：我该用哪个入口？

Claude Code 在所有入口运行的是同一个底层引擎，区别只在三件事：**代码在哪里运行、你从哪里操作、本地配置是否可用**。

```text
你想怎么用 Claude Code？
│
├─ 在终端里写代码、写脚本、连远程服务器
│  └─ CLI（功能最全；脚本与 Agent SDK 仅限 CLI）
├─ 不想离开编辑器
│  ├─ VS Code / Cursor ──> VS Code 扩展
│  └─ IntelliJ / PyCharm / WebStorm 等 ──> JetBrains 插件 + CLI
├─ 想要图形界面、并行会话、可视化 diff
│  └─ 桌面应用的 Code 选项卡
├─ 任务很长，希望合上电脑也继续跑
│  ├─ 单个任务 ──> 云端会话（claude.ai/code 或 claude --cloud）
│  ├─ 一组持续产生任务的工作 ──> Projects
│  └─ 定时 / API / GitHub 事件触发 ──> 云端定时任务（Routines）
├─ 人不在电脑前，但工作必须用本机文件和工具
│  └─ Remote Control（手机或浏览器遥控本地会话）
└─ 让 Claude 操作代码库以外的东西
   ├─ 网页 ──> Chrome 集成
   ├─ 原生应用 / 模拟器 ──> Computer use
   └─ 团队在 Slack 里派活 ──> Slack 中的 Claude Code / Claude Tag
```

| 入口 | 代码在哪运行 | 最适合 | 前置条件 |
|---|---|---|---|
| CLI | 你的机器 | 终端工作流、脚本、远程服务器 | 安装 CLI；支持第三方提供商 |
| VS Code 扩展 | 你的机器 | 内联 diff、`@` 提及 | VS Code 1.94.0+；付费订阅或 Console 账户 |
| JetBrains 插件 | 你的机器（IDE 终端里的 CLI） | IDE diff 查看器、选区共享 | 先装 CLI，再装插件 |
| 桌面应用 | 你的机器 / 云 / SSH / WSL | 视觉审查、并行会话 | Pro、Max、Ultra、Team 或 Enterprise |
| 云端会话（Web） | 云端 VM，默认由 Anthropic 托管 | 长任务、并行任务、本地没有的仓库 | claude.ai 账户 + GitHub 连接 |
| 移动端 | 不运行代码，只是客户端 | 离开电脑时启动和监控 | Claude iOS / Android 应用 |
| Remote Control | 你的机器 | 从另一台设备接着干 | claude.ai 登录；机器保持开机 |
| Slack | Anthropic 云 | 把频道里的 Bug 报告变成 PR | Pro、Max、Ultra + 云端会话 |

一句话记忆：**要本机环境就用本地入口（必要时加 Remote Control），要"关机也继续"就用云端；手机永远只是遥控器。**

### Remote Control：用手机或浏览器遥控本地会话

Remote Control 把 claude.ai/code 或 Claude 移动应用连接到**你机器上正在运行**的会话。Claude 始终在本地执行，文件系统、MCP 服务器和项目配置都可用；网页和手机只是这个本地会话的一个窗口。

#### 前置条件

- 套餐：Pro、Max、Ultra、Team、Enterprise，用 `/login` 通过 claude.ai 登录；不支持 API 密钥。Team 和 Enterprise 需要 Owner 先在管理员设置中打开 Remote Control 开关。
- 不可用的配置：Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry；`ANTHROPIC_BASE_URL` 指向 `api.anthropic.com` 以外的主机；启用零数据保留或 HIPAA 配置的组织。

#### 怎么开启

| 方式 | 命令 | 特点 |
|---|---|---|
| 服务器模式 | `claude remote-control`（别名 `claude rc`） | 进程常驻，可同时服务多个会话（`--capacity` 默认 32） |
| 交互式会话 | `claude --remote-control`（或 `--rc`） | 本地能输入，远程也能控制 |
| 已有会话中开启 | `/remote-control`（或 `/rc`） | 沿用当前对话历史；VS Code 和桌面应用也用它 |

```bash
# 在项目目录启动服务器，每个远程会话使用独立 worktree
claude remote-control --name "My Project" --spawn worktree
```

连接另一台设备：打开终端显示的 URL；或在服务器模式下按空格键显示二维码扫码；或在 claude.ai/code 与 Claude 应用的 **Code** 会话列表里按名称找（在线会话带绿色状态点）。

想让每个交互式会话自动连接：运行 `/config` 设置 **Enable Remote Control for all sessions**，或在 `~/.claude/settings.json` 中把 `remoteControlAtStartup` 设为 `true`。要完全关闭用 `disableRemoteControl`。

#### 容易踩的坑

- **本地进程必须活着。** 关终端、退出桌面应用或 VS Code，会话就离线。在远程机器上请放进 `tmux` 或 `screen`。
- 项目或本地设置文件里的 `remoteControlAtStartup: true` 会被忽略（`false` 生效），仓库不能替每个人打开 Remote Control。
- `/plugin`、`/resume` 等终端界面命令不能从手机或网页使用；`/model`、`/effort` 要带参数，例如 `/model sonnet`。

### Projects：让 Claude 协调一组云端会话

> 公开测试版，仅 Pro、Max、Ultra，并在逐步推出；尚未在 Team 或 Enterprise 上提供。

项目是一个长期对话，Claude 在其中充当协调员：你把任务丢进对话，它为每项工作启动一个**线程**。线程通常是一个云端会话，在自己的分支上工作并按需打开 PR。

适合跨多个仓库的一个目标、比单个会话更大的迁移。一个会话能完成的任务直接开云端会话；每个任务都依赖本机就用本地会话；多人在 Slack 频道里派活用 Claude Tag。

#### 操作流程

1. 在 claude.ai/code 或桌面应用 Code 选项卡的侧边栏选择 **Projects**（可能在 **More** 下），点击 **New project**。
2. 填写 **Name**（唯一必填项）、目标和 **Context**（GitHub 仓库、文件、文件夹或 Google Drive 文件夹）。
3. 点击 **Create project**，在对话里描述工作。
4. 已有云端会话想升级为项目：在会话侧边栏菜单选择 **Continue in project** > **Continue as new project**。

#### 容易踩的坑

- 代码必须在 github.com，且仓库上要安装 **Claude GitHub App**。只用 `/web-setup` 连接的令牌对项目线程不够用。
- 云线程读取各仓库的 `CLAUDE.md`，但**不**加载 `.mcp.json` 服务器，也不加载 `.claude/settings.json` 里的 hook、权限规则和 `env`，更不继承你本机的设置。
- 项目消耗套餐限额更快，尤其在 Pro 上。强制上限是每天跨项目 500 个新线程。
- 云线程沙箱若无法恢复会从新克隆继续，未提交的更改可能丢失。长任务要让 Claude 提交并推送。
- 项目属于单个用户，不能共享；终端 CLI、VS Code 扩展、JetBrains 插件里没有 Projects。

### 移动端：手机是客户端，不是运行环境

Claude Code 没有单独的移动应用：云端会话和 Remote Control 都在 Claude iOS / Android 应用的 **Code** 选项卡里。

| 功能 | 你连接到什么 | 何时使用 |
|---|---|---|
| 云端会话 / 项目 | 云基础设施上的会话 | 仓库在 GitHub 上，放下手机后任务要继续 |
| Remote Control | 你电脑上运行的会话 | 需要本地文件、工具或 MCP 服务器 |

1. 在 Claude Code 会话中运行 `/mobile`（`/ios`、`/android` 同效）显示二维码，安装应用。
2. 用与 Claude Code 相同的 claude.ai 账户和组织登录。API 密钥和第三方提供商不行。
3. 点击 **Code**，选仓库和分支提交任务，或从列表里选一个 Remote Control 会话。
4. 想要推送通知：在终端运行 `/config`，启用 **Push when Claude decides** 和/或 **Push when actions required**（需 Remote Control 处于活动状态）。

边界提醒：云端会话提供接受编辑、Plan、Auto；Remote Control 会话多一个 Manual；两者都不能从应用里选 Bypass permissions。

### Chrome 集成：让 Claude 操作浏览器

Claude Code 通过 Claude in Chrome 扩展程序做浏览器自动化：读控制台、填表单、提取数据、录 GIF。它共享你浏览器的登录状态，遇到登录页或 CAPTCHA 会暂停让你手动处理。

#### 前置条件

- Google Chrome、Microsoft Edge，或 Brave、Arc、Vivaldi、Opera 等 Chromium 浏览器。**WSL 不支持。**
- Claude in Chrome 扩展程序 1.0.36 或更高版本。
- 直接 Anthropic 套餐（Pro、Max、Ultra、Team、Enterprise），并用 `/login` 登录。API 密钥、`claude setup-token` 长期令牌、第三方提供商都不行；组织启用零数据保留时也不可用。

#### 怎么开启

```bash
claude --chrome
```

| 操作 | 方法 |
|---|---|
| 查看连接状态、管理权限、重连、切换浏览器 | `/chrome` |
| 以后不带标志也启用 | `/chrome` 中选择默认启用（CLI 与 VS Code 共享该设置） |
| VS Code 中按需使用 | 输入框里输入 `@browser` 加指令 |
| 查看全部浏览器工具 | `/mcp` > `claude-in-chrome` > **View tools** |

#### 容易踩的坑

- 默认启用会让浏览器工具一直加载，**增加上下文消耗**。嫌贵就关掉，按需用 `--chrome` 或 `@browser`。
- 项目的 `.claude/settings.json` 不能替你打开 Chrome：`env` 里的 `CLAUDE_CODE_ENABLE_CFC` 会被忽略并给出警告。
- 扩展程序必须登录到与 Claude Code **相同的 claude.ai 账户和组织**，否则报 "Browser extension is not connected"。用 `/status` 核对 `Organization` 行。
- 长时间不活动后扩展的 service worker 会休眠：运行 `/chrome` 选择重新连接扩展程序。

### Computer use：让 Claude 操作你的屏幕

> CLI 中为研究预览版，仅 macOS；需要 Pro、Max 或 Ultra，Team 和 Enterprise 不可用；`-p` 非交互模式下不可用。

Computer use 让 Claude 打开应用、点击、输入并截图，适合原生应用、模拟器和没有 API 的 GUI 工具。它适用面最广也最慢，所以 Claude 的选择顺序是：**MCP server → Bash → Claude in Chrome → Computer use**。

#### 在 CLI 中开启

1. 在交互式会话中运行 `/mcp`，找到显示为已禁用的 `computer-use`，选择启用。该设置按项目持久化。
2. 首次使用时授予两项 macOS 权限：**Accessibility** 和 **Screen Recording**。授予后可能需要重启 Claude Code。
3. 每个会话里，Claude 首次使用某个应用时会提示，选择 **Allow for this session** 或 **Deny**。

桌面应用里的 Computer use 支持 macOS 和 Windows，在设置中用 **Enable computer use** 开关启用，并可配置拒绝应用列表（CLI 尚不可用）。

#### 容易踩的坑

- 它运行在你的**真实桌面**上，不在沙箱里。只批准任务确实需要的应用，留意 Terminal、Finder、System Settings 的额外警告。
- 控制级别按应用类别固定：浏览器和交易平台**仅查看**，终端和 IDE **仅点击**，其他大多数应用完全访问。
- 同一时间只有一个会话能用，锁在会话**退出**时才释放，不是任务完成时。
- Claude 工作时其他可见应用会被隐藏，轮次结束自动恢复。随时按 `Esc` 中止。
- `/mcp` 里看不到 `computer-use`：检查是否 macOS、是否 Pro/Max/Ultra（`/status`）、是否通过 claude.ai 登录、是否交互式会话。

### VS Code 扩展

扩展提供原生图形面板，是在 VS Code 中使用 Claude Code 的推荐方式；Cursor 以及 Devin Desktop、Kiro 等分支版本也能安装。

#### 操作流程

1. 前置：VS Code 1.94.0 或更高；任何付费 Claude 订阅或 Claude Console 账户。
2. 按 `Cmd+Shift+X`（Mac）或 `Ctrl+Shift+X`（Windows/Linux）打开扩展视图，搜索 "Claude Code"，点击 **Install**。
3. 点击编辑器右上角的 Spark 图标（需要先打开文件），或右下角状态栏的 **✻ Claude Code**。
4. 点击 **Sign in** 在浏览器完成授权，然后发送提示词。

常用快捷键：`Cmd+Esc` / `Ctrl+Esc` 在编辑器和 Claude 之间切换焦点；`Option+K` / `Alt+K` 插入当前文件和选区的 `@` 引用（前者 Mac，后者 Windows、Linux）。

扩展设置在 VS Code 设置的 **Extensions > Claude Code** 下；`~/.claude/settings.json` 由扩展和 CLI 共享。

| 设置 | 默认值 | 作用 |
|---|---|---|
| `useTerminal` | `false` | 以终端模式而非图形面板启动 |
| `initialPermissionMode` | - | `default`、`plan`、`acceptEdits` 或 `bypassPermissions`；只读用户设置，忽略工作区值 |
| `disableLoginPrompt` | `false` | 跳过登录提示（用于第三方提供商配置） |

#### 容易踩的坑

- **扩展不会把 `claude` 加进 PATH。** 它自带的私有 CLI 副本只供聊天面板用；想在集成终端里运行 `claude` 需要另外安装独立 CLI。
- 扩展只提供部分命令（输入 `/` 查看），没有 `!` Bash 快捷方式和 Tab 补全。
- 扩展和 CLI 共享对话历史：在终端用 `claude --resume` 接着扩展里的对话。
- 选中的文本和打开的文件会自动作为上下文。要排除 `.env` 等敏感文件，为其路径添加 `Read` 拒绝规则。
- 设置了 `ANTHROPIC_API_KEY` 仍提示登录，多半是 VS Code 没继承 shell 环境：从终端用 `code .` 启动。

### JetBrains 插件

插件适用于 IntelliJ IDEA、PyCharm、Android Studio、WebStorm、PhpStorm、GoLand 等大多数 JetBrains IDE。与 VS Code 扩展不同，它**不自带 CLI**：插件是在 IDE 集成终端里运行 `claude` 并与之连接。

1. 先安装 Claude Code CLI，确认 `claude --version` 可用。
2. 从 JetBrains Marketplace 安装 Claude Code 插件，重启 IDE。
3. 在 IDE 集成终端里运行 `claude`，集成功能自动生效。
4. 从外部终端使用时，在 Claude Code 中运行 `/ide` 连接；成功会显示类似 `Connected to IntelliJ IDEA.`。

快捷键：`Cmd+Esc`（Mac）/ `Ctrl+Esc`（Windows、Linux）快速启动。diff 显示位置由 `/config` 中的 **Diff tool** 控制（`auto` 在 IDE 显示，`terminal` 留在终端）；插件设置在 **Settings → Tools → Claude Code [Beta]**。

#### 容易踩的坑

- `claude` 不在 PATH 时插件提示"无法启动 Claude Code"：在插件设置的 Claude 命令里填完整路径。
- 远程开发时，插件必须通过 **Settings → Plugin (Host)** 装在**远程主机**上。
- WSL2 报"未检测到可用的 IDE"：通常是 NAT 网络或 Windows 防火墙所致。添加防火墙规则，或在 Windows 11 22H2 及以上把 `.wslconfig` 设为 `networkingMode=mirrored`。
- 在 JetBrains 中用 `acceptEdits` 模式时，Claude 可能改到会被 IDE 自动执行的配置文件，官方建议对编辑使用手动模式。

### Slack 与 Claude Tag

Slack 里有两套 `@Claude`，先分清你用的是哪一套：

| | Slack 中的 Claude Code（早期版本） | Claude Tag |
|---|---|---|
| 套餐 | Pro、Max、Ultra | Team、Enterprise（公开测试版） |
| 身份 | 每个会话以**你自己的**账户运行 | 组织的共享身份，管理员配置访问 |
| 计费 | 计入个人套餐限额 | 频道工作按用量从组织余额扣，受 Owner 设置的支出上限约束 |
| 工作区要求 | 未被任何组织连接到 Claude Tag | Owner 把 Slack 工作区与 Claude 组织配对 |

#### Slack 中的 Claude Code：操作流程

1. 工作区管理员从 Slack 应用程序市场安装 Claude 应用。
2. 在 Slack 中打开 Claude 应用主页，点击 **Connect Claude account**。
3. 用同一账户登录 claude.ai/code，连接 GitHub 并认证至少一个仓库。
4. 在应用主页选择**路由模式**：仅代码，或代码 + 聊天。
5. 在频道里输入 `/invite @Claude`，然后 `@Claude` 加编码请求。完成后点击 **View session** 看完整记录，或 **Create PR**。

Claude Tag 的设置由组织 Owner 完成：在 Slack 发送 `@Claude connect` 获取配对码，在 Claude Tag 管理设置页配对并设置月度支出上限后上线。之后频道里的任何人都能直接使用，无需逐个用户设置。

#### 容易踩的坑

- 早期版本只在频道（公开或私有）里工作，**私信里不起作用**；仓库必须在 GitHub 上；每个会话只能创建一个 PR。
- "This workspace isn't set up for Claude Tag yet"：工作区没连 Claude Tag，且你的账户不是 Pro、Max、Ultra。
- "The legacy Claude in Slack bot is retired"：工作区已连到某个 Claude 组织，但该频道仍选着早期版本，需要组织所有者在 Claude Tag 设置里启用。
- "Claude Code is not enabled for your account"：用同一账户在 claude.ai/code 登录一次，完成 Web 入门以创建默认云环境。

### 云端（Web）会话与云环境

云端会话在云端 VM 上运行 Claude Code，默认由 Anthropic 托管。合上笔记本后继续运行，用量计入套餐限额，云端 VM 不单独收费。

> 套餐：Pro、Max、Ultra、Team，以及拥有高级席位或 Chat + Claude Code 席位的 Enterprise 用户。启用零数据保留、HIPAA 配置或客户管理的加密密钥（CMEK）的组织无法使用。

```text
提交任务（claude.ai/code、移动应用、桌面应用选 Cloud、claude --cloud）
   ↓
克隆仓库到隔离 VM，运行设置脚本（如已配置）
   ↓
按云环境的访问级别配置网络
   ↓
Claude 修改代码、运行测试
   ↓
推送分支 ──> 你审查 diff、留行内注释、Create PR
```

#### 连接 GitHub 与云环境

| 方式 | 如何连接 | 能访问的仓库 |
|---|---|---|
| GitHub App | 在 claude.ai/code 入门时授权 | 公开仓库，以及安装了 Claude GitHub App 的私有仓库 |
| `/web-setup` | 终端先 `gh auth login`，再在 Claude Code 里运行 `/web-setup` | 你的 `gh` 令牌能访问的任何仓库 |

自动修复 PR 和 Projects 都依赖仓库上已安装 Claude GitHub App。Team 和 Enterprise 上，`/web-setup` 只有在 Owner 打开 **Quick setup** 后才可用。

云环境是一份保存的配置，控制**网络访问、环境变量和设置脚本**。入门流程会创建名为 **Default** 的环境，使用 **Limited** 网络访问：只能访问常见包注册表等允许列表中的域。需要其他域名时编辑环境的 **Allowed domains**，或改选 **Full**。

#### 在终端和云之间移动任务

```bash
# 新建云端会话（克隆的是 GitHub 远程，本地提交要先推送）
claude --cloud "Fix the authentication bug in src/auth/login.ts"

# 给已有云端会话发后续消息后立即退出
claude -p "your message" --cloud <session-id>

# 把云端会话拉回终端（会话内可用 /teleport 或 /tp）
claude --teleport <session-id>
```

在 PR 分支上运行 `/autofix-pr`，可以生成云端会话并让 Claude 监视 CI 失败和审查评论。

#### 容易踩的坑

- **`--cloud` 和 `--remote-control` 毫无关系**：前者创建云端会话，后者遥控本地会话。`--remote` 是 `--cloud` 的已弃用别名。
- 从 CLI 看切换是单向的：能把云端拉回终端，不能把已有终端会话推上云。桌面应用可通过 **Open in** > **Cloud** 发送本地会话。
- Teleport 要求：干净的 git 状态、同一仓库的检出（不是 fork）、分支已推送、同一 claude.ai 账户。拉回后终端拿到的是副本，新工作不会回到云端会话。
- 云端会话只有接受编辑、Plan、Auto 三种权限模式；它不使用你的本地配置，只有仓库里的内容。
- 没有 git 远程或未安装 GitHub App 时，`--cloud` 会把本地仓库打包上传：须小于 100 MB，未跟踪文件不包含。**原生 Windows 上未提交更改按原样上传**，不会像 macOS、Linux、WSL 那样排除 `.env` 等敏感文件。

### Routines：云端定时任务

> 研究预览版，行为、限制和 API 可能变化。适用于 Pro、Max、Ultra、Team、Enterprise。功能已改名为"定时任务"，旧名 Routine 仍出现在网址、`/routines` 别名和 API 路径中。

云端定时任务是一份保存的配置：提示词 + 仓库 + 连接器。每次运行都是一个完整的云端会话，电脑关着也能跑。一个任务可以组合多个触发器：

| 触发器 | 何时运行 | 备注 |
|---|---|---|
| Schedule | 每小时、每天、每周，或未来某个时间运行一次 | 最小间隔 1 小时 |
| API | 向任务专属端点发带 Bearer 令牌的 POST | 只能在 Web 端添加，令牌只显示一次 |
| GitHub | Pull request、Release、Issue 事件 | 仓库必须安装 Claude GitHub App |

#### 操作流程

1. 打开 claude.ai/code/routines，点击 **New task**（部分账户显示 **New routine**）。
2. 写自包含的提示词，在 **Trigger** 行选择触发器。
3. 展开 **Advanced settings**，选择仓库和 **Cloud environment**；在 **Connectors** 标签页移除不需要的连接器。
4. 点击 **Create**；想立即试跑就在详情页点击 **Run now**。

CLI 里用 `/schedule` 对话式创建，例如 `/schedule daily PR review at 9am`；用 `/schedule list`、`/schedule update`、`/schedule run` 管理。

API 触发示例（URL 和令牌替换为你在编辑对话框中复制的值）：

```bash
curl -X POST <FIRE_URL> \
  -H "Authorization: Bearer <ROUTINE_TOKEN>" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Alert fired in prod. Stack trace attached."}'
```

#### 容易踩的坑

- 定时任务属于你的**个人账户**，不与队友共享；提交、PR 和连接器操作都显示为你本人执行。
- 五字段 cron 表达式按 **UTC** 填小时（Web 预设频率才是本地时区）；频率高于每小时一次会被拒绝。
- 创建时**默认包含你所有已连接的连接器**，包括写入能力，务必删到最少。`claude mcp add` 添加的本地 MCP 服务器不在其中。
- API 的 `text` 会被包进 `<routine-fire-payload>` 块并标记为不受信任数据。提示词里必须显式引用这个负载，否则它只是惰性上下文。
- 历史里的 **Succeeded** 只表示会话正常结束，不代表活干完了，要打开会话记录看。
- 每小时限制：计划运行每账户 100 次；**Run now** 与 API 触发每个任务合计 30 次。不提供超额。

### 桌面应用

桌面应用里，Claude Code 就是侧边栏顶部的 **Code** 选项卡。它自带 Claude Code，不需要安装 Node.js 或 CLI。需要 Pro、Max、Ultra、Team 或 Enterprise 订阅。

#### 第一次会话

1. 安装并登录：macOS（Intel 和 Apple Silicon 通用版）、Windows（x64 与 ARM64）、Linux（测试版）。
2. 点击 **Code** 选项卡，在输入框下方一行依次选择**环境**、**项目文件夹**、**权限模式**和**模型**。
3. 输入任务按 **Enter**；点击 `+12 -1` 这样的指示器打开 diff 视图审查。

| 环境 | 代码在哪运行 | 适合 |
|---|---|---|
| 本地 | 你的机器 | 日常开发 |
| Cloud | 云端 VM | 长任务，关掉应用也继续 |
| SSH | 远程机器（首次连接自动安装 Claude Code） | 云虚拟机、开发容器 |
| WSL（Windows） | WSL 2 发行版内 | 仓库在发行版文件系统里 |

权限模式有 Manual（`default`）、Accept edits（`acceptEdits`）、Plan（`plan`）、Auto（`auto`）；Bypass permissions（`bypassPermissions`）需在设置中启用，仅在沙箱容器或虚拟机中使用。

从 CLI 过来的用户注意：

- Desktop 和 CLI 读取相同的配置：`CLAUDE.md`、`~/.claude.json` 与 `.mcp.json` 中的 MCP 服务器、hook、skill、`~/.claude/settings.json`。
- 把 CLI 会话搬到桌面：在终端运行 `/desktop`，或 `claude --desktop --resume <session-id>`；也可以在 Desktop 输入框输入 `/resume`。
- 界面和文档对不上时先检查布局：环境选择器在输入框**上方**说明是早期布局，更新应用即可。

#### Linux（测试版）与 WSL

- Linux 要求：Ubuntu 22.04 或更高、Debian 12 或更高；x86_64 或 arm64。Fedora、Arch 等请改用 CLI。
- 添加 Anthropic 的 apt 存储库后运行 `sudo apt update && sudo apt install claude-desktop`。**Linux 上不会自动更新**，靠 `sudo apt update && sudo apt upgrade`；测试版尚不包含 Computer use 和语音听写。
- WSL 要求：Windows 10 或 11 + WSL 2（不支持 WSL 1），至少一个发行版，发行版内装有 `git`。在环境选择器的 **WSL** 部分选发行版，再用 Linux 路径选项目文件夹。
- WSL 会话限制：插件不可用；文件窗格只能查看不能编辑；信任按"发行版 + 文件夹"授予。

#### 桌面计划任务（本地）

本地定时任务需要 Claude Desktop 1.1.5368 或更高版本。三种调度方式的区别：

| | 云端定时任务 | 桌面任务 | `/loop` |
|---|---|---|---|
| 运行位置 | 云 | 你的机器 | 你的机器 |
| 需要机器开启 | 否 | 是 | 是 |
| 需要打开会话 | 否 | 否 | 是 |
| 访问本地文件 | 否（新克隆） | 是 | 是 |
| 最小间隔 | 1 小时 | 表单中 1 小时，由 Claude 设置时 1 分钟 | 1 分钟 |

1. 在 **Code** 选项卡侧边栏点击 **Scheduled**，再点击 **New task**，填写 **Name** 和指令。
2. 展开 **Advanced settings**，启用 **Only on this computer**，选择 **Local folder**。
3. 在 **Trigger** 卡片选择 Hourly、Daily、Weekdays 或 Weekly；**Manual** 和 **Custom**（cron，本地时间）在创建后的 **Edit** 表单里。
4. 点击 **Create**，随后点 **Run now** 试跑一次，把遇到的权限提示选为 always allow。

边界提醒：

- 只在**应用打开且电脑醒着**时触发。可启用 **Keep computer awake**，但合上笔记本盖仍会睡眠。
- 错过的运行：唤醒后只为过去七天内最近一次补跑**一次**。上午 9 点的任务可能晚上 11 点才跑，时间敏感就在提示词里写护栏。
- 默认直接在工作目录运行，包括未提交的更改；要隔离就在 **Edit** 表单勾选 **Worktree**。

#### iOS 模拟器窗格

> 公开测试版，仅 macOS 上的 Claude Code Desktop。Pro、Max、Ultra、Team、Enterprise 可用；启用 HIPAA 配置的 Enterprise 组织和第三方提供商不可用。

- 要求：Claude Desktop v1.24012.0 或更高、Mac、安装了 iOS 平台的 Xcode。仅本地会话可用。
- 不需要命令或开关：对 Claude 说"构建应用并在模拟器中运行它以检查入门流程"，窗格会自动打开。
- 窗格直接驱动模拟器，**不需要 Computer use**，也不会接管屏幕。在 CLI 中则要通过 Computer use 操作模拟器。
- 设备截图会发送到 Anthropic，不要在 Claude 使用的模拟器上登录真实账户。

### 代码审查与 CI：先选对工具

Claude Code 里叫“审查”的东西有好几种，运行位置、触发人和计费方式都不同。先对号入座：

| 方式 | 在哪运行 | 谁触发 | 计费 | 适合场景 |
|---|---|---|---|---|
| 本地 `/code-review` | 你的 Claude Code 会话 | 你输入命令，或 Claude 按你的自然语言请求启动 | 计入套餐用量或 API 计费；任何套餐可用 | 边写边查、提交前自查 |
| `/code-review ultra`（ultrareview，研究预览） | Anthropic 云沙箱 | 只能由你手动启动 | 3 次一次性免费运行，之后按使用额度，通常 $5–$25/次 | 合并重大变更前的深度审查 |
| 托管 Code Review | Anthropic 基础设施 | PR 打开/推送，或评论 `@claude review` | 按 token 走使用额度，记到组织；仅 Team / Enterprise | 团队所有 PR 自动审查，不想维护工作流 |
| GitHub Actions | 你自己的 CI 运行器 | `@claude` 提及或任意 GitHub 事件 | Actions 分钟 + API token（或订阅用量） | 想自己掌控提示词、模型和触发器 |
| GitLab CI/CD（测试阶段） | 你的 GitLab runner | 流水线事件、手动或 webhook | Runner 时间 + API token | 代码在 GitLab |
| 安全类：security-guidance 插件、`/security-review`、Claude Security 插件 | 你的 Claude Code 会话 | 自动（hooks）或按需 | 模型审查计入用量；逐次编辑的模式匹配不花钱 | 写代码时拦漏洞、对仓库做深度漏洞扫描 |

```text
写代码 ──> security-guidance 插件（会话内自动拦）
   │
   ▼
提交前 ──> /code-review（快） ──> /code-review ultra（深，合并大改动前）
   │
   ▼
开 PR ──> 托管 Code Review（Team/Enterprise）或 自建 GitHub Actions / GitLab CI
   │
   ▼
按需 ──> /security-review（当前分支）/ Claude Security 插件（全仓库深扫）
```

> 所有这些审查都是 AI 审查，可能漏报。没有发现不等于代码没有 bug 或是安全的。

### 本地审查：`/code-review`

`/code-review` 在当前会话里审查分支变更，查找正确性 bug，并在对话中报告；`/review` 是它的别名。终端里它作为后台子代理运行，不占用你的对话上下文。

#### 选择审查目标

| 审查目标 | 输入 |
|---|---|
| 当前分支和未提交的变更 | `/code-review` |
| 最近 15 个提交 | `/code-review HEAD~15..HEAD` |
| 一个分支相对于另一个分支 | `/code-review main...my-feature` |
| 单个提交 | `/code-review abc1234^..abc1234` |
| 按编号指定的 Pull Request | `/code-review 1234` |
| 文件或目录路径 | `/code-review src/auth` |

不指定目标时，审查的是分支领先于上游的提交加未提交变更；没有上游就与 `main` 或上一个提交比较。

#### 标志与 effort

| 写法 | 作用 |
|---|---|
| `--fix` | 审查后把修复应用到工作树，编辑仍受权限模式约束 |
| `--comment` | 把发现以行内评论发到 GitHub PR，或以单条备注发到 GitLab 合并请求（后者走 `glab` CLI，需 v2.1.257+） |
| `--max-findings <n>` / `all` / `default` | 调整发现数量上限，会被记住直到传 `default`（需 v2.1.288+） |
| `low` … `max` | effort 级别：`low` 只报最有把握的问题，往上逐步扩大覆盖 |
| `ultra` | 升级为云端 ultrareview，见下一节 |

顺序是：先 effort 级别和标志，最后才是目标，例如 `/code-review high --fix src/auth`。不写级别时沿用你上次输入的级别。

#### 三个容易踩的点

- 后台审查的 `--fix` 编辑在检查点之外，`/rewind` 恢复不了审查改动的其他文件，请用 git 撤销。
- 本地审查读取 `CLAUDE.md`，**不读取** `REVIEW.md`（后者只给托管 Code Review 用）。
- 想保留手动输入但禁止 Claude 和定时任务自行启动：在设置文件的 `skillOverrides` 里写 `"code-review": "user-invocable-only"`。

### 云端深度审查：ultrareview

> 研究预览功能，定价和可用性可能变化。需要用 claude.ai 账户登录；Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry、启用零数据保留或 HIPAA 配置的组织不可用，此时 `/code-review ultra` 会退回为本地审查。使用 CMEK 的组织无法运行。

ultrareview 在云沙箱里启动一队审查代理，每个发现都经过独立复现和验证，通常耗时 5 到 15 分钟。

启动前会弹出确认对话框（审查范围、剩余免费次数、预估费用）；确认后在后台运行，用 `/tasks` 查看或停止，停止后不会返回部分发现。

```text
/code-review ultra                 # 当前分支 vs 默认分支
/code-review ultra develop         # 换一个基准分支
/code-review ultra 1234            # 审查 github.com 上的 PR（云端克隆，不上传本地内容）
/code-review ultra 1234 --post     # 预选“以我的 GitHub 账户把发现发到 PR”，仍会询问（v2.1.227+）
/code-review ultra --fix           # 发现返回后应用修复
```

分支审查默认上限为 500 个更改文件、8,000 行更改；仓库太大无法打包时，推送分支并开草稿 PR，改用 PR 模式。

计费：Pro、Max、Ultra 各有 3 次免费运行（每账户一次性，不刷新），之后按使用额度计费，通常 $5–$25/次。云会话一启动就算一次，提前停止也会消耗免费次数。付费审查要求先启用使用额度，可用 `/usage-credits` 检查。

#### 在脚本或 CI 中运行

用 `claude ultrareview` 子命令，它会阻塞到审查完成并把发现打印到 stdout（进度写 stderr）。`claude -p '/code-review ultra'` 不等结果就退出，拿不到发现。

```bash
claude ultrareview origin/main
claude ultrareview 1234 --trust-workspace
```

常用标志：`--json`（输出原始 `bugs.json` 负载）、`--timeout <minutes>`（默认 45）、`--post` / `--no-post`（是否把发现发到 PR，默认不发）、`--trust-workspace`（仅本次信任当前文件夹，CI 中避免 `Error: Workspace not trusted`，需 v2.1.309+）。

退出码：`0` 审查完成（无论有无发现），`1` 启动失败、被停止、出错或超时，`130` 你按了 Ctrl-C。子命令提前退出后再运行是一次**新**审查，会再次计费。

### 托管 Code Review：PR 上的自动审查

> 仅限 Team 和 Enterprise 套餐，仓库需托管在 GitHub（含 GitHub Enterprise Server）。启用零数据保留、HIPAA 配置或 CMEK 的组织不可用。

多个 Agent 并行阅读 diff 和周围代码，经过验证、去重、按严重程度排序后，把结果发到 PR：行内评论、审查正文摘要，以及名为 **Claude Code Review** 的检查运行。它只读仓库、不构建不运行代码、不推送提交；**永远不会批准或阻止 PR**，检查运行也不会以失败结束。

#### 设置（由 Claude 组织 Owner 完成）

1. 打开 claude.ai 的 **Admin settings > Claude Code**，在 **Code review** 部分点击 **Allow code review**。
2. 在 **Connect GitHub** 对话框中连接 GitHub 账户。
3. 仓库没列出来就点 **Install GitHub App**，安装 Claude GitHub App 并选择仓库。
4. 在 **Add a repository** 中选择仓库并添加（需要对仓库有写入权限）。
5. 在 **Code review is set up** 对话框点 **Finish**，否则开关保持关闭。
6. 开一个非草稿的测试 PR，几分钟内应出现 **Claude Code Review** 检查运行。

两项按仓库可调的设置：

- **Review level**：Quick / Standard（默认）/ Deep，越深越贵。
- **Review runs**：Once after PR creation / After every push / Manual (@claude review)。新添加的仓库默认 **After every push**，审查次数最多。

无论选哪项，自动审查都会跳过草稿和来自 fork 的 PR。

#### 自定义：`CLAUDE.md` 与 `REVIEW.md`

- `CLAUDE.md`：项目规则，违反即成为一项发现；托管 Code Review 和本地 `/code-review` 都读。
- `REVIEW.md`：放在仓库根目录，只管审查（标记什么、多严重、跳过什么）；仅托管 Code Review 读。

一份可照抄的根目录 `REVIEW.md`（官方示例节选）：

```markdown
# Review instructions

Report at most five Nits per review.

## Do not report

- Anything CI already enforces: lint, formatting, type errors
- Generated files under `src/gen/` and any `*.lock` file

## Always check

- New API routes have an integration test
- Log lines don't include email addresses, user IDs, or request bodies
```

安全边界与限制：

- `REVIEW.md` 只从 PR 的基础分支或默认分支读取，**绝不从 PR 自身的更改读取**。改规则要先合并。
- 请保护默认分支。只有从受保护分支读取时，降低严重程度、排除路径、限制 nit 数量这类指令，以及 `@` 包含文件和子目录 `REVIEW.md` 才会生效。
- 包含其他文件：在根 `REVIEW.md` 里单独一行写 `@review-security.md`，文件须放在仓库根目录；skill 用 `@.claude/skills/<folder>/SKILL.md`。最多八个文件，合计小于 150,000 字节。
- `test`、`fixtures`、`examples`、`generated`、`vendor`、`node_modules` 等目录下的子目录 `REVIEW.md` 不会被应用。

#### 在 PR 上阅读与请求审查

三种严重程度：🔴 Important（应在合并前修复的 bug）、🟡 Nit（小问题，不阻塞合并）、🟣 Pre-existing（代码库中已存在、并非此 PR 引入的 bug）。

处理发现：推送修复（下次审查会自动解决讨论串），或在 GitHub 上自行解决讨论串。**只回复不算忽略**，回复也不会触发审查。

| 评论命令 | 作用 |
|---|---|
| `@claude review` / `@claude review once` | 启动一次审查，不订阅后续推送 |
| `@claude review always` | 启动一次审查，并让该 PR 之后每次推送都审查 |
| `@claude review stop` / `off` / `pause` | 取消该 PR 的推送订阅，不启动也不取消审查 |

命令生效的条件：必须是**新的顶层 PR 评论**，命令在评论开头同一行，评论者有 write、maintain 或 admin 权限，PR 处于打开状态。是否真的启动以检查运行为准，👀 回应不算数。

想在自己的 CI 里按结果拦截合并：用 `gh api` 读检查运行 `output.text` 的最后一行，它是形如 `<!-- bughunter-severity: {"nit":1,"normal":2,"pre_existing":0} -->` 的注释，其中 `normal` 就是 Important 的数量。

#### 成本

没有按次固定价格：按 token 用量走使用额度，与套餐包含的用量分开，**记到组织**而不是开 PR 的人。费用 = 每次审查费用（审查级别、变更大小等）× 审查次数（**Review runs** 设置）。

1. 先在 claude.ai/admin-settings/usage 为 Claude Code Review 服务设置每月支出上限。
2. 挑几个代表性仓库试点，**Review runs** 设为 **Once after PR creation** 或 **Manual (@claude review)**。
3. 在 **Repositories** 表的 **Average cost** 列看平均费用，乘以每月 PR 数；用量在 claude.ai/analytics/code-review 跟踪。

#### 排错：先看检查运行标题

| 检查运行标题 | 含义与处理 |
|---|---|
| **Code review skipped — no new changes** | 最新推送没有新变更，未重新审查；需要就评论 `@claude review` |
| **Code review not repeated — earlier findings still open** | 之前的 Important 发现仍未解决；推送修复或评论 `@claude review` |
| **Code review failed** / **Code review timed out** | 评论 `@claude review`，或在 **Checks** 标签页点 **Re-run**（fork 的 PR 不行） |
| **Code review declined** | 看摘要，多半是预算或支出限额 |
| 没有检查运行 | 评论 `@claude review`；仍没有则检查仓库是否已启用、评论是否满足条件、是否有写入权限 |

PR 上出现以 "⚠️ Code review skipped" 开头的评论，说明预算不足，请管理员到 claude.ai/admin-settings/usage 提高上限或添加使用额度，然后重新评论 `@claude review`。

### GitHub Enterprise Server（GHES）

> GHES 支持仅适用于 Team 和 Enterprise 计划。

Owner 把自托管实例连接一次，开发人员就能使用云端会话和自动 PR 审查，无需按仓库配置。

- 支持：云端会话（`claude --cloud`）、Teleport（`claude --teleport`）、代码审查、插件市场（必须用完整 git URL，`owner/repo` 简写始终解析为 github.com）。
- 支持但要手动：GitHub Actions 需自己写工作流，`/install-github-app` 仅适用于 github.com。
- 不支持：贡献指标、GitHub MCP server（替代做法：`gh auth login --hostname github.example.com` 后让 Claude 用 `gh`）。

#### 管理员设置流程

1. 在 **Organization settings > Git providers** 的 GitHub Enterprise 部分点击 **Connect**（已有实例则 **Add instance**），选 **Set up automatically**，填显示名称和 GHES 主机名。
2. 点击 **Continue to GitHub Enterprise**，在实例上点 **Create GitHub App**，再把应用安装到需要的仓库或组织。
3. 到 claude.ai/admin-settings/claude-code 为 GHES 仓库启用 Code Review。

重定向被网络挡住时改选 **Add manually**。实例在防火墙后面时，要把 Anthropic 的出站 IP 地址加入允许列表，否则审查和云会话会超时。ultrareview 的 PR 模式不支持 GHES，只能审查本地分支。

### 安全指导与漏洞扫描

安全检查是分层的：会话内用 security-guidance 插件，按需单次扫描当前分支用 `/security-review`，深度扫描仓库或 diff 用 Claude Security 插件，PR 上靠 Code Review。另有托管的 Claude Security 产品（Enterprise 计划）负责监控已连接的仓库，与这里的插件不是一回事。

#### security-guidance 插件：边写边查

```text
/plugin install security-guidance@claude-plugins-official
```

安装后自动运行，分三层：

| 时机 | 做什么 | 成本 |
|---|---|---|
| 每次文件编辑 | 对 `eval(`、`pickle`、`.innerHTML =` 等危险模式做字符串匹配 | 无模型调用，不花钱 |
| 每个回合结束 | 把本回合的 git diff 交给独立的 Claude 做安全审查 | 计入用量 |
| Claude 执行 `git commit` / `git push` | 更深的代理审查，会读周围代码 | 计入用量，每滚动小时上限 20 次 |

- 需要 `PATH` 中有 Python 3.7 或更高版本（代理提交审查需要 3.10+），后两层需要 git 仓库。
- 你自己在 shell 里做的提交不会被审查。
- 自定义规则：`.claude/claude-security-guidance.md`（给模型审查的自然语言指导）和 `.claude/security-patterns.yaml`（逐次编辑的匹配规则）。规则只能追加，不能关掉内置检查。
- 团队启用：在 `.claude/settings.json` 的 `enabledPlugins` 中写 `"security-guidance@claude-plugins-official": true`。
- 诊断日志在 `~/.claude/security/log.txt`。

> 这些层都**不会阻止**写入或提交，只是把发现交给 Claude 去修。要硬性拦截，请配合 hooks 或 CI 检查。

#### Claude Security 插件：扫描整个代码库

```text
/plugin install claude-security@claude-plugins-official
```

需要付费计划、Anthropic API 访问权限或第三方提供商，以及 `PATH` 中的 `python3`（3.9 或更高）。

1. 运行 `/claude-security`，选择 **Scan codebase**（也可以直接写 `/claude-security scan my branch`）。
2. 选整个仓库或聚焦区域，确认运行。扫描可能用掉大量 token。
3. 到 `CLAUDE-SECURITY-<timestamp>/` 目录读 `CLAUDE-SECURITY-RESULTS.md`（同时有 `.jsonl` 和 `.sarif`）。
4. 再次运行 `/claude-security`，选择 **Suggest patches**，挑要修的发现。
5. 自己应用补丁，每个补丁单独一个 PR：

```bash
git apply CLAUDE-SECURITY-<timestamp>/patches/F1.patch
```

扫描工作量可选 `low`、`medium`（默认）、`high`，例如 `/claude-security scan my branch at high`。补丁永远不会自动应用；扫描是非确定性的，两次结果可能不同。

### GitHub Actions：`anthropics/claude-code-action@v1`

Action 有两种模式，由工作流自动决定：不给 `prompt` 是**交互模式**（等待 `@claude` 提及并在评论中回复）；给了 `prompt` 是**自动化模式**（直接运行，结果默认在工作流运行日志里）。

#### 两种安装方式（都需要仓库管理员权限）

| 方式 | 做法 |
|---|---|
| 快速设置 | 先 `gh auth login`，再在仓库里启动 `claude` 并运行 `/install-github-app`，按提示安装 Claude GitHub App、选择工作流和凭据，最后创建并合并它准备好的 PR。仅适用于 github.com |
| 手动设置 | 安装 Claude GitHub App → 添加密钥 `ANTHROPIC_API_KEY` 或 `CLAUDE_CODE_OAUTH_TOKEN`（本地运行 `claude setup-token` 生成）→ 把 examples/claude.yml 复制到 `.github/workflows/` |

#### 最小 workflow：响应 `@claude`

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    if: contains(github.event.comment.body, '@claude')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
      issues: write
      id-token: write
      actions: read
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 1
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

用订阅认证时，把最后一行换成 `claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`。

想在 PR 上自动审查，把触发器换成 `pull_request`（`types: [opened, synchronize, ready_for_review, reopened]`），并在 `with:` 中加上：

```yaml
          plugin_marketplaces: "https://github.com/anthropics/claude-plugins-official.git"
          plugins: "code-review@claude-plugins-official"
          prompt: "/code-review:code-review --comment ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
          claude_args: '--allowedTools "mcp__github_inline_comment__create_inline_comment"'
```

没有 `--comment` 就不会发评论，只能在运行日志里看结果；`claude_args` 这一行也不能省。

#### 常用输入

| 输入 | 作用 |
|---|---|
| `prompt` | 纯文本指令或 skill 调用；省略则响应触发短语 |
| `claude_args` | 传给 Claude Code 的 CLI 参数，如 `--max-turns`、`--model`、`--allowedTools` |
| `anthropic_api_key` / `claude_code_oauth_token` | 二选一的认证方式 |
| `github_token` | GitHub 操作令牌；省略时以 Claude GitHub App 身份认证 |
| `plugin_marketplaces` / `plugins` | 执行前安装插件 |
| `settings` | Claude Code 设置，JSON 字符串或文件路径 |
| `trigger_phrase` | 触发短语，默认 `@claude` |
| `use_bedrock` / `use_vertex` / `use_foundry` | 改走云厂商 |

#### 云厂商凭证：用 OIDC，不存静态密钥

| 提供商 | 输入 | 凭证步骤 | 需要的密钥 |
|---|---|---|---|
| Amazon Bedrock | `use_bedrock: "true"` | `aws-actions/configure-aws-credentials@v4` | `AWS_ROLE_TO_ASSUME` |
| Google Cloud 的 Agent Platform | `use_vertex: "true"` | `google-github-actions/auth@v2` | `GCP_WORKLOAD_IDENTITY_PROVIDER`、`GCP_SERVICE_ACCOUNT` |
| Microsoft Foundry | `use_foundry: "true"` | `azure/login@v2` | `AZURE_CLIENT_ID`、`AZURE_TENANT_ID`、`AZURE_SUBSCRIPTION_ID` |

工作流必须带 `id-token: write` 权限，凭证步骤放在 `anthropics/claude-code-action@v1` 之前。官方示例还用 `actions/create-github-app-token@v2` 生成自定义 GitHub App 令牌并传给 `github_token`；用官方 Claude GitHub App 时删掉该步骤和这一行。完整 YAML 见云厂商页面。

#### 安全边界

- 密钥只放 GitHub Secrets，永远不要提交到仓库；只授予工作流所需的权限。
- 默认只有对仓库有写入权限的人能触发，机器人会被拒绝（例外用 `allowed_non_write_users`、`allowed_bots`）。
- 公共仓库中，fork PR 触发的运行拿不到密钥，审查只在同仓库分支的 PR 上运行。
- 同时启用托管 Code Review 时，`@claude review` 也会触发交互模式的 Action，把 `trigger_phrase` 改成别的短语来区分。
- 用 `--max-turns`、工作流超时和并发控制限制成本。

### GitLab CI/CD

> 目前处于测试阶段，由 GitLab 维护，功能可能变化。

#### 操作流程

1. 在 **Settings** → **CI/CD** → **Variables** 添加掩码变量 `ANTHROPIC_API_KEY`。
2. 把下面的作业加到 `.gitlab-ci.yml`。
3. 从 **CI/CD** → **Pipelines** 手动运行测试，或从 MR 触发。

```yaml
stages:
  - ai

claude:
  stage: ai
  image: node:24-alpine3.21
  rules:
    - if: '$CI_PIPELINE_SOURCE == "web"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  variables:
    GIT_STRATEGY: fetch
  before_script:
    - apk update
    - apk add --no-cache git curl bash
    - curl -fsSL https://claude.ai/install.sh | bash
    - export PATH="$HOME/.local/bin:$PATH"
  script:
    # 可选：如果您的设置提供了 GitLab MCP 服务器，请启动它
    - /bin/gitlab-mcp-server || true
    - >
      claude
      -p "${AI_FLOW_INPUT:-'Review this MR and implement the requested changes'}"
      --permission-mode acceptEdits
      --allowedTools "Bash Read Edit Write mcp__gitlab"
      --debug
```

#### 要点

- `@claude` 提及不是开箱即用的：需要你自己加一个“Comments (notes)”项目 webhook 和事件监听器，由它带着 `AI_FLOW_INPUT`、`AI_FLOW_CONTEXT` 等变量调用管道触发 API。
- GitLab API 操作默认用 `CI_JOB_TOKEN`，或用具有 `api` 范围的项目访问令牌，存为掩码变量 `GITLAB_ACCESS_TOKEN`。
- 云厂商：Amazon Bedrock 设置 `CLAUDE_CODE_USE_BEDROCK: "1"`，Google Cloud 的 Agent Platform 设置 `CLAUDE_CODE_USE_VERTEX: "1"`，都通过 `id_tokens:` 生成的 OIDC 令牌换取临时凭证，不存长期密钥。
- 用 `--max-turns` 和作业级 `timeout`（例如 `timeout: 30m`）控制成本；像审查其他贡献者一样审查 Claude 的 MR。

---

## Part II：使用 Claude Code 构建

### 代理与并行工作：先选对方式

Claude Code 让多个 Claude 同时干活的方式不止一种。区别主要在三点：**谁协调**、**工作者之间能否对话**、**是否会改同一批文件**。

| 方式 | 解决什么问题 | 上下文 | 谁协调 | 成本 | 状态 |
|---|---|---|---|---|---|
| Subagents | 把搜索结果、日志等噪声留在会话内的独立工作者里，只取回摘要 | 各自独立的 context window，结果回到调用者 | Claude，在一个对话内逐轮委派 | 较低：结果汇总回主上下文，但每个子代理仍消耗自己的 token | 正式功能 |
| Agent view（后台会话） | 一屏分派并监控多个互不相关的后台会话 | 每行是一个完整、独立的会话 | 你自己分派、按需介入 | 每个会话独立消耗配额，十个并行约为一个的十倍 | 研究预览 |
| Agent teams | 让一组会话共享任务列表、互发消息 | 各自独立，不继承负责人的对话历史 | 负责人（Team lead）会话 | 较高：每个队友都是独立 Claude 实例，随人数线性增加 | 实验性，默认禁用 |
| 跨会话消息 | 在你自己开的多个会话之间传递发现和状态 | 只传一段纯文本，不带历史和文件 | 无；各会话照常由你驱动 | 送达的消息像你输入的提示词一样计入用量 | 满足版本要求即默认启用 |
| Dynamic workflows | 任务大到几个子代理协调不过来，或需要交叉验证 | 中间结果留在脚本变量，Claude 的上下文只拿最终答案 | 脚本，而不是 Claude 的逐轮判断 | 明显高于在对话中完成同一任务 | 所有付费计划可用；Pro 需在 `/config` 启用 |
| Worktrees | 并行会话或子代理改同一仓库时互不覆盖 | 不涉及上下文，只隔离文件 | 不是协调方式，是支撑工具 | 无额外模型成本 | 正式功能 |

```text
要并行处理多件事
      │
      ├─ 只是不想让搜索/日志淹没主对话？ ──────────────► Subagents
      ├─ 几件互不相关的任务，交出去稍后再看？ ─────────► Agent view（claude agents）
      ├─ 需要 Claude 拆分、分配，工作者还要互相讨论？ ──► Agent teams（实验性）
      ├─ 数十到数百个代理、要可重跑的编排或交叉验证？ ──► Dynamic workflows
      ├─ 自己开的几个会话之间要互通消息？ ─────────────► 跨会话消息
      └─ 会改到同一批文件？ ──────────────────────────► 叠加 Worktrees
                              （Agent teams 不给队友隔离 worktree，只能划分文件归属）
```

查看运行中的工作，入口名字相近但不是一回事：

| 命令 | 看什么 |
|---|---|
| `claude agents` | 打开 agent view，列出所有后台会话 |
| `/agents` | 只打印一条提醒，指向子代理文件位置；与 `claude agents` 无关 |
| `/tasks` | 当前会话后台运行的一切，包括已完成的子代理 |
| `/workflows` | 工作流的运行列表、阶段和代理完成数 |

官方对比页另外列出了云端的 **项目**（Pro、Max、Ultra 上的公开测试版）和打包用法 `/batch`（把一个大型更改拆给 5 到 30 个 worktree 隔离的子代理），本节不展开。

> 同时运行多个会话或子代理会成倍增加 token 使用量。先确认任务确实能拆成互不依赖的部分。

### Subagents：会话内的专业分工

#### 目的

子代理是带 YAML frontmatter 的 Markdown 文件。每个子代理在自己的 context window 中运行，有自己的系统提示词、工具范围和权限，完成后只返回摘要。它的请求计入与主对话相同的使用限制。

- 适合：跑测试、取文档、读日志等高输出操作；需要强制只读或限制工具的角色；能用一份摘要交付的独立工作。
- 不适合：频繁来回、多阶段共享大量上下文、很快的小改动。非分叉子代理从零开始，收集上下文需要时间。

#### 内置子代理

| Agent | 工具 | 用途 |
|---|---|---|
| `Explore` | 只读，拒绝 Write 和 Edit | 文件发现、代码搜索；调用时指定 quick / medium / very thorough |
| `Plan` | 只读，拒绝 Write 和 Edit | plan mode 期间为规划做代码库研究 |
| `general-purpose` | 所有可用于子代理的工具 | 复杂研究、多步骤操作、代码修改 |

另有辅助 Agent：`claude`（通用，也是后台会话的默认代理）、`statusline-setup`、`claude-code-guide`、`web-fetch`。`Explore` 和 `Plan` 启动时不加载 CLAUDE.md 和 git 状态快照；只想移除这两个，设置 `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`（需要 v2.1.198 或更高版本）。

#### 文件位置与优先级

同名时，优先级高的位置胜出：

| 位置 | 作用域 | 优先级 |
|---|---|---:|
| 托管设置 | 整个组织 | 1（最高） |
| `--agents` CLI 标志 | 当前会话，JSON 传入，不落盘 | 2 |
| `.claude/agents/` | 当前项目 | 3 |
| `~/.claude/agents/` | 你的所有项目 | 4 |
| 插件的 `agents/` 目录 | 启用插件的位置 | 5（最低） |

- 项目和用户目录都递归扫描；身份只来自 `name`，与子文件夹路径无关。
- 项目子代理从工作目录向上找到仓库根，多处同名时用离工作目录最近的。
- 插件子代理忽略 `hooks`、`mcpServers`、`permissionMode`；其子文件夹会进入名称，例如 `my-plugin:review:security`。
- 这两个目录的改动几秒内生效，无需重启；通过 `--add-dir` 添加的目录除外。

#### Frontmatter 字段

只有 `name` 和 `description` 必需。多词字段用 camelCase，写错的字段会被**静默忽略**，不报错。

| 字段 | 必需 | 目的 |
|---|---:|---|
| `name` | 是 | 唯一标识符，最多 256 个字符；不能包含 `:`，不要以 `-` 开头；文件名不必与之相同 |
| `description` | 是 | 告诉 Claude 何时委派给它 |
| `tools` | 否 | 允许列表，逗号分隔字符串或 YAML 列表；省略则继承全部可用工具 |
| `disallowedTools` | 否 | 拒绝列表；带说明符的条目如 `Bash(git push *)` 仍会移除整个工具 |
| `model` | 否 | `sonnet`、`opus`、`haiku`、`fable`、完整模型 ID 或 `inherit` |
| `permissionMode` | 否 | `default`、`acceptEdits`、`auto`、`dontAsk`、`bypassPermissions`、`plan`，或 `default` 的别名 `manual` |
| `maxTurns` | 否 | 最大轮次数；达到后返回标记为部分结果的输出，可恢复继续 |
| `autoCompactWindow` | 否 | 把自动压缩窗口降到 `100000` 或以上的 token 数；写纯整数，`200k` 会被读作 200 |
| `skills` | 否 | 启动时把完整 skill 内容预加载进上下文 |
| `mcpServers` | 否 | 该子代理可用的 MCP 服务器：已配置服务器的名称，或内联定义 |
| `hooks` | 否 | 只在该子代理活动期间运行的生命周期 hook |
| `memory` | 否 | 持久记忆作用域：`user`、`project` 或 `local` |
| `background` | 否 | `true` 使其始终在后台运行，即使 Claude 被要求等待它 |
| `omitClaudeMd` | 否 | `true` 时初始上下文不含用户、项目和本地 CLAUDE.md |
| `effort` | 否 | `low`、`medium`、`high`、`xhigh`、`max`；可用级别取决于模型 |
| `isolation` | 否 | 设为 `worktree`，在临时 git worktree 中运行 |
| `color` | 否 | `red`、`blue`、`green`、`yellow`、`purple`、`orange`、`pink` 或 `cyan` |
| `initialPrompt` | 否 | 作为主会话 Agent 运行时，自动作为第一个用户轮次提交 |
| `experimental` | 否 | 实验性选项映射；其中 `cacheTtl` 可设为 `5m` 或 `1h` |

`--agents` JSON 用 `prompt` 代替 Markdown 正文，并且不接受 `color` 和 `experimental`。

#### 最小示例

`.claude/agents/code-reviewer.md`：

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

正文就是系统提示词。子代理拿到的是这段提示词加上工作目录等基本环境信息，**不是** Claude Code 的系统提示词。

#### 调用方式

| 方式 | 写法 | 效果 |
|---|---|---|
| 自动委托 | 无 | Claude 依据任务、`description` 和上下文决定；描述里写 "use proactively" 可鼓励主动委托 |
| 自然语言 | `Use the code-reviewer subagent to ...` | 点名，由 Claude 决定是否委托 |
| @-mention | `@"code-reviewer (agent)" look at the auth changes` | 让 Claude 为这个任务运行该子代理；也可手输 `@agent-<name>` |
| 整个会话 | `claude --agent code-reviewer` | 主线程采用该子代理的系统提示词、工具限制和模型 |
| 项目默认 | `.claude/settings.json` 中 `"agent": "code-reviewer"` | 每个会话默认如此；CLI 标志优先 |

`/agents` 现在只打印一条提醒，让你询问 Claude 或直接编辑 `.claude/agents/` 和 `~/.claude/agents/`；在 v2.1.197 及更早版本中它才是交互式向导。

#### 前台、后台与分叉

- **前台**：阻塞主对话直到完成，权限提示直接传给你。
- **后台**：与你并发运行；需要权限时，提示出现在主会话并指明是哪个子代理。按 Esc 只拒绝那一次调用，不停止子代理。
- **默认行为**：未做配置时，Claude 从主对话生成的子代理（`web-fetch` 除外）在后台启动。在交互式终端、VS Code 和桌面应用里想让 Claude 等待子代理，需把 `CLAUDE_CODE_FORK_SUBAGENT` 设为 `0` 关闭 fork 模式后重启。
- **分叉（fork）**：继承到目前为止的整个对话，而不是从零开始。用 `/subtask <任务>` 自己启动一个；分叉无法再生成分叉。

> 命名易混：agent view 开启时，`/subtask` 启动分叉子代理，`/fork` 把整个会话复制成新的后台会话。agent view 关闭时，`/subtask` 不可用，`/fork` 改为启动分叉子代理。

#### 恢复子代理

每次调用都创建新实例。要接着上次的工作，直接说 `Continue that code review and now analyze the authorization logic`，Claude 会用 `SendMessage` 工具，以 Agent 的 ID 或名称作为 `to` 恢复它，并保留完整历史。

- 以后要继续的工作用 `general-purpose` 或自定义子代理；`Explore` 和 `Plan` 完成时可能不提供可恢复的 ID。
- 你亲自停止的子代理（在 `/tasks` 中按 `x`）不会被自动恢复。
- 会话记录在 `~/.claude/projects/{project}/{sessionId}/subagents/` 下，文件名为 `agent-{agentId}.jsonl`，过了 `cleanupPeriodDays`（默认 30 天）后清理。

#### 工具与权限限制

- `tools` 和 `disallowedTools` 同时设置时，先应用 `disallowedTools`，再从剩余工具池解析 `tools`。
- 只给 `Read, Grep, Glob, Bash` 的子代理**仍可通过 Bash 改文件**。想保留 Bash 又禁特定命令，在设置的 `permissions.deny` 里加 Bash 拒绝规则。
- 有些工具即使列在 `tools` 里也拿不到，例如 `AskUserQuestion`、`EnterPlanMode`、`Workflow`；后台子代理的内置工具集更小。
- 不设 `permissionMode` 时继承主对话的权限模式。主对话处于 `bypassPermissions`、`acceptEdits` 或自动模式时，子代理以相同模式运行，你设置的值被忽略。
- 禁用某个子代理：`"permissions": { "deny": ["Agent(Explore)", "Agent(my-custom-agent)"] }`，或 `claude --disallowedTools "Agent(Explore)"`。
- 嵌套：默认最多到主对话之下三层，用 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 调整，设为 `1` 关闭嵌套。
- 并发：默认同时 20 个，超出报 `Concurrent subagent limit reached`，用 `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` 调整（需要 v2.1.217 或更高版本）。

#### 常见模式

```text
隔离高输出：Use a subagent to run the test suite and report only the failing tests with their error messages
并行研究：  Research the authentication, database, and API modules in parallel using separate subagents
串联：      Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

许多子代理各自返回详细结果，同样会吃掉主上下文。让它们只回结论和文件位置。

### Agent view：一屏管理后台会话

> 研究预览阶段，界面和快捷键可能改变。

#### 目的

`claude agents` 打开一个接管整个终端的列表，显示你所有项目里的后台会话：哪些在工作、哪些在等你、哪些已完成。每行是一个完整的 Claude Code 会话，由单独的监督进程托管，关掉终端也继续跑。会话生成的子代理和队友不会单独成行。

#### 操作流程

1. 在 shell 运行 `claude agents`。
2. 在底部输入框写任务，按 `Enter`。每条提示词启动一个**新的**后台会话，不是给上一个会话发后续消息。
3. 选中一行按 `Space` 窥视最新输出或待答问题，直接输入回复。
4. 按 `Enter` 或 `→` 附加进完整对话；在空输入框按 `←` 分离回列表。分离永远不会停止后台会话。
5. 想把正在用的会话移进来：在该会话里运行 `/bg`（即 `/background`），或在空输入框按 `←`。

其他常用键：`Ctrl+S` 切换按状态或按目录分组，`Ctrl+T` 固定，`Ctrl+R` 重命名，`Ctrl+X` 停止（两秒内再按一次即删除），`?` 显示全部快捷键。

#### 从 shell 管理

```text
claude --bg "<prompt>"                         # 直接启动后台会话（长形式 --background）
claude --agent code-reviewer --bg "<prompt>"   # 以某个子代理为主 Agent
claude agents --cwd <path>                     # 只看该目录下启动的会话
claude agents --json                           # 以 JSON 数组打印会话
claude attach <id>   claude logs <id>   claude stop <id>   claude rm <id>
```

`--bg` 不能与 `-p` 或 `--print` 组合。

#### 边界提醒

- 从 agent view 分派或用 `claude --bg` 启动的会话，在编辑文件前会移入 `.claude/worktrees/` 下的隔离 worktree。用 `←` 或 `/background` 转入后台的会话**不会**，它继续在原处编辑。
- 在 agent view 里删除会话会连同 Claude 为它创建的 worktree 一起移除，**包括未提交的更改**；`claude rm` 在有未提交更改时会保留。先提交再删。
- 会话是本地的：机器睡眠时保留，关机时停止。
- 完全关闭：把 `disableAgentView` 设置设为 `true`，或设置 `CLAUDE_CODE_DISABLE_AGENT_VIEW` 环境变量。

### Agent teams：会互相对话的协作团队

> 实验性功能，默认禁用。在会话恢复、任务协调和关闭行为方面存在已知限制。

#### 启用

在 shell 环境或 `settings.json` 中设置：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

启用后会改变普通委派：Claude 自行命名的子代理会作为队友启动，所以即使你没要求，团队也可能形成。想恢复成普通子代理，把该变量设为 `0`。生成队友需要交互式会话，`-p` 和 Agent SDK 下不会生成。

然后用自然语言描述任务和队友，例如 `Spawn three teammates to explore this from different angles: one on UX, one on technical architecture, one playing devil's advocate.`

#### 架构

| 组件 | 角色 |
|---|---|
| Team lead | 生成队友并协调工作的主 Claude Code 会话 |
| Teammates | 各自处理分配任务的独立 Claude Code 实例 |
| Task list | 队友认领和完成的共享工作项列表 |
| Mailbox | 用于 Agent 之间通信的消息系统 |

- 任务有三种状态：待处理、进行中、已完成；依赖未完成时显示为已阻塞。
- 队友加载 CLAUDE.md、MCP 服务器和 skill，但**不继承负责人的对话历史**，任务细节要写进生成提示词。
- 团队配置在 `~/.claude/teams/{team-name}/config.json`，任务列表在 `~/.claude/tasks/{team-name}/`。不要手动编辑团队配置；没有项目级的团队配置文件。
- 可复用角色用子代理定义：`Spawn a teammate using the security-reviewer agent type to audit the auth module.`

#### 显示模式

- **进程内**（默认）：所有队友在主终端内；上下方向键选择，Enter 查看并发消息。任何终端可用。
- **分屏窗格**：每个队友一个窗格。需要 tmux，或安装了 `it2` CLI 的 iTerm2。

通过 `teammateMode` 设置切换：`"auto"`、`"tmux"` 或 `"iterm2"`。单次会话可用 `claude --teammate-mode auto`，该标志是实验性的，不出现在 `claude --help` 中。

#### 限制

- 进程内队友没有会话恢复：`/resume` 和 `/rewind` 不会恢复它们。
- 每个会话一个团队；没有嵌套团队；负责人固定，不能转移。
- 生成时无法为每个队友单独设置权限模式；队友的权限提示出现在负责人会话。
- 队友不在 worktree 中隔离，两个队友编辑同一文件会互相覆盖。
- 任务状态可能滞后，关闭可能很慢。
- 规模建议：从 3-5 个队友开始，每个队友 5-6 个任务。

### 跨会话消息：让你的几个会话互通

#### 目的

一个会话发现了另一个会话需要的东西（破坏性更改、已合并的内容、迁移是否跑完），由 Claude 直接发过去，不用你在终端之间复制粘贴。消息只是一段纯文本，不带发送者的对话历史或文件。

满足要求即默认启用，无需配置：macOS、Linux、WSL 2 需要 v2.1.224 或更高版本，原生 Windows 需要 v2.1.234 或更高版本。

#### 怎么用

Claude 用 `ListAgents` 找目标、用 `SendMessage` 发送，你不需要自己调用。直接说：

```text
Ask the session running in my other terminal whether the migration finished
Let @api-worker know the schema migration finished
Tell me when the migration session finishes what it's working on
```

- 会话名来自 `/rename` 或 `--name`；`@` 后输入名称首字母可从列表选择。
- `/list-agents`（别名 `/peers`）查看 Claude 能联系到哪些会话。
- 本机会话之间走本地套接字，永远不通过 Anthropic 服务器；其他机器和云端会话要经 Remote Control 和 Anthropic 服务器。
- 入站由 `crossSessionInbound` 设置控制：`accept` 全部传递，`hold` 只显示通知不传递，`refuse` 丢弃。未设置时按双方权限模式逐条决定。

#### 边界提醒

- 来自另一个会话的消息**不算你的同意**：不能批准权限提示，不能要求改权限设置、`CLAUDE.md` 或配置，其中的 `/compact` 之类命令只是纯文本。
- 两个会话如果回复得足够慢，可以在没有你参与的情况下互发数千条消息，每条都计入用量。结束其中一个会话即可停止。
- 发往本机以外需要你批准：设置 `"isolatePeerMachines": true`。
- 彻底关闭：`crossSessionInbound` 设为 `refuse` 停止接收；在权限拒绝规则里加 `SendMessage` 和 `ListAgents` 停止发送和列出。拒绝 `SendMessage` 也会切断向子代理和队友发消息。

### Dynamic workflows：用脚本编排大量子代理

> 所有付费计划可用；Pro 上需在 `/config` 的 Dynamic workflows 行启用。

#### 目的

动态工作流是一个 JavaScript 脚本，由 Claude 为你描述的任务编写，运行时在后台执行。循环、分支和中间结果都在脚本里，Claude 的上下文只拿最终答案。典型场景：代码库范围的审计、500 个文件的迁移、需要交叉检查来源的研究。

#### 触发方式

| 方式 | 写法 | 范围 |
|---|---|---|
| 内置工作流 | `/deep-research <question>` | 单次；需要 WebSearch 工具可用 |
| 提示词关键字 | `ultracode: audit every API endpoint under src/routes/ for missing auth checks` | 单个任务；说 "use a workflow" 同样有效 |
| 会话级 | `/effort ultracode`，关闭用 `/effort ultracode off` | 会话内每个实质性任务都规划工作流 |
| 已保存 | `/<name>` | 在 `/workflows` 里选中运行按 `s` 保存后可用 |

关键字只在你亲手输入的提示词里生效；通过 `-p` 传入的提示词、定时任务的提示词不会触发。

#### 脚本长什么样

保存位置是项目的 `.claude/workflows/` 或个人的 `~/.claude/workflows/`，同名时项目优先。文件以 `meta` 块开头，后面是带顶级 `await` 的纯 JavaScript：

```javascript
export const meta = {
  name: 'audit-routes',
  description: 'Audit every route handler for missing auth checks',
}

const found = await agent('List every .ts file under src/routes/.', {
  schema: { type: 'object', required: ['files'], properties: { files: { type: 'array', items: { type: 'string' } } } },
})

const audits = await pipeline(found.files, file =>
  agent(`Audit ${file} for missing authentication checks.`, { label: file }),
)

return audits.filter(Boolean)
```

`agent()` 生成一个子代理，`pipeline()` 为列表每一项运行一个，`parallel()` 同时运行一组并等待全部完成。还可以用 `phase()` 分组、`log()` 显示消息、读取 `args` 全局变量。

运行后用 `/workflows` 看进度：`p` 暂停或恢复，`x` 停止，`r` 重启选定代理，`s` 保存。

#### 规模与成本

| 约束 | 数值 |
|---|---|
| 并发代理 | 最多 16 个，可用 `CLAUDE_CODE_WORKFLOW_MAX_CONCURRENT_AGENTS` 改为 1 到 256 |
| 每次运行代理总数 | 1,000 个 |
| 单个 `parallel()` 或 `pipeline()` 调用 | 最多 4,096 个项目 |
| `Large workflow` 警告 | 超过 25 个代理，或预计 token 总量超过 150 万；仅提示，不暂停 |

规模指导用 `workflowSizeGuideline` 设置：`small`（少于 5 个）、`medium`（少于 10 个）、`large`（少于 50 个）、`unrestricted`。默认 `medium`，Pro 套餐为 `small`。它是建议而不是上限。大任务先在一个目录上试跑。

#### 边界提醒

- 运行中途不能向脚本输入；脚本本身没有文件系统或 shell 访问，也不能 `import()`。
- 脚本内 `Date.now()`、`Math.random()` 和无参数 `new Date()` 会抛异常，时间戳要通过 `args` 传入。
- 把会话转入后台或退出后，运行**不会自行继续**，需要让 Claude 重新启动它。
- 开启 ultracode 后每个请求都更费 token、更慢，并且不再显示 `Large workflow` 警告，也不受并发子代理限制约束。
- 关闭：`/config` 里关 Dynamic workflows，或设置 `"disableWorkflows": true`，或 `CLAUDE_CODE_DISABLE_WORKFLOWS=1`。

### Worktrees：给并行会话各一份文件

#### 目的

git worktree 是一个有自己文件和分支的独立工作目录，与主检出共享同一份仓库历史。每个会话一个 worktree，一个做功能、一个修 bug，互不覆盖。它只隔离文件编辑，本身不是运行代理的方式。

#### 启动与位置

```bash
claude --worktree feature-auth     # 或 -w；省略名称则自动生成
claude --worktree "#1234"          # 从拉取请求分支，创建在 .claude/worktrees/pr-<number>
```

- 目录：仓库根目录下的 `.claude/worktrees/<name>/`；分支：名为 `worktree-<name>` 的新分支。
- 基础：仓库在远程上的默认分支，而不是你当前的 `HEAD`；只检出被跟踪的文件。
- 也可以在会话中说"在 worktree 中工作"，Claude 会用 `EnterWorktree` 工具创建。
- 把 `.claude/worktrees/` 加进 `.gitignore`，避免在主检出里显示为未跟踪文件。

#### 相关设置与 hooks

| 配置 | 目的 |
|---|---|
| `worktree.baseRef` | `"fresh"`（默认）从远程默认分支分支；`"head"` 从当前本地 `HEAD` 分支，带上未推送的提交。不能设为分支名 |
| `worktree.bgIsolation` | 设为 `"none"` 让后台会话直接编辑工作副本，不先移入 worktree |
| `.worktreeinclude` | 放在项目根目录，`.gitignore` 语法；把匹配且被 gitignored 的文件（如 `.env`）复制进新 worktree |
| `isolation: worktree` | 写在子代理 frontmatter 里，让它总在自己的临时 worktree 中运行 |
| `WorktreeCreate` / `WorktreeRemove` hook | 完全替换默认 git 逻辑，用于 SVN、Perforce、Mercurial 等；此时 `.worktreeinclude` 不被处理 |

#### 清理

- worktree 是干净的：未命名的会话退出时自动删除 worktree 及其分支；已命名的会话先提示你。
- worktree 里有工作：提示保留或删除。保留后可用退出时打印的 `claude --worktree <name> --resume` 回来。
- 用 `-p` 的非交互式运行没有退出提示，不会清理，需要自己 `git worktree remove`；被锁定时先 `git worktree unlock`。
- 子代理的 worktree：没有更改时自动删除；有更改的保留到定期扫描，超过 `cleanupPeriodDays` 且不会丢失工作时才删。

#### 边界提醒

- 隔离中的会话不能用 `Edit`、`Write`、`NotebookEdit` 改主检出里的路径，也不能把 git 重定向回主检出。但这些检查不跟踪 shell 命令实际写了哪些文件，`cp` 或 shell 重定向写进主检出不会被拦下，仍由权限和沙箱设置处理。
- worktree 与主检出共享 `.git` 目录、项目作用域的插件，以及"是，不再询问"保存下来的权限规则。
- hook 里的 `${CLAUDE_PROJECT_DIR}` 不跟随 worktree，仍指向会话启动的项目根；需要 worktree 路径时读 hook 输入 JSON 的 `cwd`。
- worktree 是全新检出，依赖要重新安装。

### Artifacts：把会话输出变成可分享的页面

> 在 Pro、Max、Ultra、Team 和 Enterprise 计划中可用，需要使用 `/login` 登录的会话。

#### 目的

Artifact 是 Claude Code 从会话发布到 claude.ai 私密 URL 的实时交互式网页。适合"看比读更容易"的输出：带注释的 diff、仪表板、并排的方案对比、长任务的调查时间线。它不是托管应用：一个自包含的静态页面，没有后端，无法提供多个路由。

#### 操作流程

1. 直接请求（Claude 也可能在合适时自动发布）：`Make an artifact that walks through this PR with the diff annotated inline.`
2. 首次发布后 Claude 打印 URL，浏览器打开新页面。此时**只有你能看到**。
3. 要修改就让 Claude 改，它会重新发布到相同的 URL；每次发布成为一个版本。
4. 要分享，在浏览器中打开该页面，用页面标题中的**分享**控件。

| 操作 | 方式 |
|---|---|
| 重新打开最近的 Artifact | `Ctrl+]` |
| 列出你拥有的和与你共享的 | `/artifacts`（`Enter` 打开或附加，`o` 打开，`c` 复制链接，`d` 删除） |
| 从另一个会话更新 | 给 Claude 该 URL，或用 `/artifacts` 附加；否则会新建一个 |
| 阻止浏览器自动打开 | `CLAUDE_CODE_ARTIFACT_AUTO_OPEN=0` |
| 草拟设计画布 | `/design <简要说明>` |

#### 分享范围

- **组织内**（特定人员、所有人，或共享文件夹）：仅 Team 和 Enterprise 计划。
- **公开链接**，无需登录：Team 和 Enterprise 上默认关闭，直到所有者为组织启用。
- 被分享者默认是查看者；Team 和 Enterprise 上可以把某人设为编辑者。组织内共享时，对方可以在页面上留评论，并让 Claude 读取和回复。

#### 限制

| 约束 | 效果 |
|---|---|
| 外部请求 | 字体只能来自 Google Fonts；脚本只能来自 cdnjs、unpkg、Tailwind 和 jQuery CDN，以及 jsDelivr 上的选定路径。所有外部图像被阻止，需嵌入为数据 URI |
| 无后端 | 静态页面，无法自行对查看者做身份验证 |
| 下载 | 页面不能自行启动下载，需由 Claude 声明下载功能 |
| 源文件类型 | 必须是 `.html`、`.htm` 或 `.md` |
| 呈现大小 | 16 MiB 或更小 |
| 实时数据 | 只能调用 claude.ai 账户上的连接器；`.mcp.json` 里的本地 MCP 服务器只能在构建时提供数据 |

不可用的情况：使用 API 密钥、网关令牌或云提供商凭据的会话；Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry；组织启用了 CMEK、HIPAA 或零数据保留；Agent SDK、GitHub Action 和 `-p` 运行中默认关闭。不满足时 Claude 会写入本地 HTML 文件，或说明无法发布。

为自己关闭：`/config` 里关 **Artifacts** 行，或设置 `"enableArtifact": false`，或 `CLAUDE_CODE_DISABLE_ARTIFACT=1`，或把 `Artifact` 加入 `permissions.deny`。

> 样式化页面比同样内容的终端文本更费输出 token，嵌入为数据 URI 的图像尤其明显。图表优先用 SVG 或 HTML 和 CSS。

### MCP：连接外部工具与数据

#### 目的与结构

MCP（Model Context Protocol）让 Claude Code 使用内置工具之外的能力：查 issue、读监控、查数据库、控制浏览器。当你总在把别的系统里的数据复制进对话时，就该接一个 MCP 服务器。

```text
┌──────────────┐   MCP Client    ┌────────────────────────────────┐
│ Claude Code  │ ──────────────► │ MCP Server（本地进程或远程服务） │
└──────────────┘                 │ Tools     可执行动作            │
 配置来源                         │ Resources 用 @ 引用的数据       │
 ├─ ~/.claude.json   (local/user) │ Prompts   变成 / 命令的提示     │
 ├─ <项目根>/.mcp.json (project)  └────────────────────────────────┘
 └─ 插件 / claude.ai 连接器 / 组织托管配置
```

每个已连接的服务器都会占用一部分上下文（工具名称和服务器说明每个会话都加载），不用的请移除或禁用。

#### 传输类型

| 类型 | `type` 值 | 适合场景 | 说明 |
|---|---|---|---|
| HTTP | `http`（JSON 中 `streamable-http` 是别名） | 远程/云端服务 | 推荐；支持 OAuth |
| SSE | `sse` | 只提供 SSE 端点的旧服务 | **已弃用**；v2.1.265+ 用 `--transport http` 添加时会自动回退到 SSE |
| stdio | `stdio`（默认） | 本机进程、访问本地资源 | 以子进程启动，环境中注入 `CLAUDE_PROJECT_DIR` |
| WebSocket | `ws` | 主动推送事件的远程服务 | 只能用 `.mcp.json` 或 `claude mcp add-json` 配置；不支持 OAuth，`--transport` 不接受 `ws` |

#### 添加与管理

```bash
claude mcp add --transport http <name> <url>                       # 远程 HTTP
claude mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"
claude mcp add --transport sse <name> <url>                        # 已弃用的 SSE
claude mcp add [options] <name> -- <command> [args...]             # 本地 stdio
claude mcp add --env API_KEY=<value> --transport stdio <name> -- npx -y <package>
claude mcp add-json <name> '<json>'                                # 直接传 JSON

claude mcp list                    # 列表 + 健康状态
claude mcp get <name>              # 详情与所在作用域
claude mcp remove <name> [--scope <scope>]
claude mcp add-from-claude-desktop # 仅 macOS 和 WSL
claude mcp reset-project-choices   # 重置 .mcp.json 服务器的批准选择
```

- 这些命令在**终端**里运行；会话内用 `/mcp` 查看状态、重连、认证、禁用。
- stdio 必须写 `--`，其后内容原样作为启动命令，否则服务器自己的标志会被当成 Claude Code 的选项。
- 不要让服务器名称紧跟在 `--env` 之后，中间至少隔一个其他选项（如 `--transport stdio`）。
- 名称只能含字母、数字、连字符和下划线；`workspace`、`claude-in-chrome`、`computer-use`、`Claude Preview`、`Claude Browser` 是保留名称。

#### 作用域与优先级

| 作用域 | 存储位置 | 谁能用 |
|---|---|---|
| `local`（默认） | `~/.claude.json` 中当前项目的条目下 | 仅你，仅当前项目 |
| `project` | 项目根目录的 `.mcp.json`（提交到版本控制） | 克隆仓库的所有人 |
| `user` | `~/.claude.json` 顶层 `mcpServers` | 仅你，所有项目 |

用 `--scope user` 或 `--scope project` 指定。作用域在添加时固定，要更换只能先 `remove` 再重新添加。MCP 的 "local" 存在主目录的 `~/.claude.json`，不是项目里的 `.claude/settings.local.json`。

同名服务器只连接一次，使用优先级最高来源的**整个条目**，字段不合并：

```text
组织 managedMcpServers（v2.1.259+） ▶ Local ▶ Project ▶ User ▶ 插件 ▶ claude.ai 连接器
```

项目作用域服务器在交互式会话中首次使用前需要批准。**`claude -p` 和 Agent SDK 会话无法显示该提示，会直接加载**；要排除可用 `disabledMcpjsonServers`、`--setting-sources` 或 `--strict-mcp-config`（只加载 `--mcp-config` 传入的服务器）。

#### `.mcp.json` 字段与环境变量展开

```json
{
  "mcpServers": {
    "docs": {
      "type": "http",
      "url": "${API_BASE_URL:-<默认地址>}/mcp",
      "headers": { "Authorization": "Bearer ${API_KEY}" }
    },
    "browser": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"],
      "env": { "MY_VAR": "value" }
    }
  }
}
```

| 字段 | 目的 |
|---|---|
| `type` | `stdio` / `http` / `sse` / `ws`。**有 `url` 却没有 `type` 会被当作 stdio 并跳过** |
| `command`、`args`、`env` | stdio：启动命令、参数、传给进程的环境变量 |
| `url`、`headers` | 远程类型：端点地址与静态请求头 |
| `headersHelper` | 连接时运行的命令，输出合并进请求头（短期令牌、内部 SSO） |
| `oauth` | 预配置 OAuth，如 `clientId`、`callbackPort` |
| `timeout` | 单次工具调用的硬性时限（毫秒）；低于 1000 的值被忽略 |
| `alwaysLoad` | `true`：该服务器的工具不延迟、始终加载；`false`：全部留在工具搜索之后（v2.1.287+） |

`${VAR}` 和 `${VAR:-default}` 可用于 `command`、`args`、`env`、`url`、`headers`：

- 未设置且无默认值：配置仍加载，`claude mcp list` 给出警告，并原样使用 `${VAR}` 文本。
- 设置为空字符串也算"已设置"，`:-default` 不会生效。
- **凭据变量读取为空**：在远程服务器的 `url`/`headers` 中，`ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN`、`AWS_BEARER_TOKEN_BEDROCK`、`HTTPS_PROXY`、`NPM_TOKEN` 等不会展开，防止项目配置把你的凭据发往第三方。确需传递时复制到自己命名的变量再引用。

`.mcp.json` 只在会话启动时读取，编辑后需退出并重启。

#### 认证

1. 添加后 `claude mcp list` 显示 `! Needs authentication`，属于预期。
2. 会话内：`/mcp` → 选择服务器 → `Authenticate` → 浏览器登录；或在终端运行 `claude mcp login <name>`（无浏览器环境加 `--no-browser`）。
3. 撤销：`/mcp` 中的 "Clear authentication"，或 `claude mcp logout <name>`。

- OAuth 适用于 HTTP 服务器，令牌自动刷新。浏览器没打开时复制终端里的 URL 手动打开。
- 固定回调端口用 `--callback-port <port>`；服务器不支持动态客户端注册时用 `--client-id <id> --client-secret`（密钥经掩码提示输入）。
- 配置了 `Authorization` 请求头却被拒绝时，直接报连接失败，不会回退到 OAuth。
- `claude -p` 没有 `/mcp` 面板，无法运行 OAuth 流程，需先在交互式会话或用 `claude mcp login` 登录。

#### 超时、输出限制与工具搜索

| 环境变量 | 默认值 | 目的 |
|---|---|---|
| `MCP_TIMEOUT` | 30 秒 | 服务器启动超时（毫秒），如 `MCP_TIMEOUT=60000 claude` |
| `MCP_TOOL_TIMEOUT` | 约 28 小时 | 跨服务器的工具调用时限 |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | 远程 5 分钟，stdio 30 分钟 | 无响应、无进度通知时的空闲超时；`0` 禁用 |
| `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` | 2 分钟 | 主对话中工具调用转入后台任务的阈值；`0` 关闭 |
| `MAX_MCP_OUTPUT_TOKENS` | 25,000 | 单次工具输出的最大 token 数；超限结果保存到文件，对话中只留路径 |
| `ENABLE_TOOL_SEARCH` | 未设置 | 未设置/`true`：工具延迟、按需加载；`auto`/`auto:N`：工具定义低于上下文窗口 10%（或 N%）时预先加载；`false`：全部预先加载 |

工具搜索默认启用：会话开始只加载工具名称和服务器说明，所以多接几个服务器对上下文影响很小。较早的模型等环境会回退为预先加载。

#### 资源、提示与反向使用

- **资源**：输入 `@` 可在自动完成中看到 MCP 资源，格式为 `@server:protocol://resource/path`，如 `@github:issue://123`。`claude -p` 中资源提及保留为纯文本。
- **提示**：MCP 提示在 `/` 菜单中显示为 `/servername:promptname (MCP)`，也可输入 `/mcp__servername__promptname`，参数用空格分隔。
- **把 Claude Code 作为 MCP 服务器**：`claude mcp serve`。启动后无输出是正常的。其他客户端按 `"command": "claude", "args": ["mcp", "serve"]` 配置；找不到可执行文件会报 `spawn claude ENOENT`。它只公开 Claude Code 的工具，**用户确认需由你的客户端自己实现**。

#### 验证与常见失败

| 现象 | 处理 |
|---|---|
| `✔ Connected` | 就绪 |
| `✘ Failed to connect` / `✘ Connection error` | 服务器没启动或 URL 无响应；stdio 请直接在终端运行同一条命令看报错 |
| `⏸ Pending approval` | `.mcp.json` 服务器尚未批准，交互式运行 `claude` 批准 |
| `/mcp` 显示 No MCP servers | 在别的项目里添加的 local 服务器，或改错了文件 |
| 已连接但没有工具 | 多半缺少必需的环境变量（如 API 密钥），用 `--env` 或 `env` 字段传入 |

Claude Code **不读取** `~/.claude/.mcp.json`、`~/.claude/mcp.json`，也不读取 `settings.json` 里的 `mcpServers`；正确位置只有 `~/.claude.json` 和 `<项目根>/.mcp.json`（键是 `mcpServers`，不是 `servers`）。

安全提醒：连接前确认你信任该服务器，会抓取外部内容的服务器可能带来提示注入风险。粘贴令牌时留意首尾空白，Claude Code 只警告、不修剪。

### Skills：可复用的指令与工作流

#### 目的与结构

Skill 是一个带 `SKILL.md` 的目录。当你反复粘贴同一段说明、检查清单或多步流程，或 CLAUDE.md 的某一节已经变成"流程"而不是"事实"时，就把它做成 Skill。与 CLAUDE.md 不同，Skill 的正文只在使用时才加载。

```text
my-skill/
├── SKILL.md        # 必需：YAML frontmatter + Markdown 指令（建议 500 行以内）
├── reference.md    # 可选：详细资料，在 SKILL.md 中链接，需要时才读
└── scripts/
    └── helper.py   # 可选：被执行而不是被加载的脚本
```

#### 放置位置与同名优先级

| 位置 | 路径 | 生效范围 |
|---|---|---|
| Enterprise | 托管设置目录下的 `.claude/skills/<skill-name>/SKILL.md` | 组织部署到的所有用户 |
| Personal | `~/.claude/skills/<skill-name>/SKILL.md` | 本机所有项目 |
| Project | `.claude/skills/<skill-name>/SKILL.md` | 当前仓库，提交后团队共享 |
| Nested | `<subdir>/.claude/skills/<skill-name>/SKILL.md` | 在该子目录内启动的会话；从上层启动时，Claude 处理到该处文件才加载 |
| Additional directory | `--add-dir` 目录中的 `.claude/skills/` | 该会话 |
| Plugin | `<plugin>/skills/<skill-name>/SKILL.md` | 插件启用处，命令为 `/plugin-name:skill-name` |

同名时 **Enterprise 优先于 Personal，Personal 优先于 Project**；Skill 优先于 `.claude/commands/` 的同名文件；插件 Skill 带命名空间不冲突。不要把文件夹命名为 `synced` 或 `anthropic-skills`（保留名称）。

#### frontmatter 字段

所有字段都可选，只建议写 `description`。无法识别的字段会被**静默忽略**；开头的 `---` 必须在文件第一行。

| 字段 | 目的 |
|---|---|
| `name` | `/` 菜单中的命令名，默认为目录名 |
| `description` | 做什么、何时用；Claude 据此决定是否自动加载。省略时取正文第一个非空行 |
| `when_to_use` | 追加的触发说明；与 `description` 合并后在列表中截断为 1,536 个字符 |
| `argument-hint` | 自动补全时显示的参数提示，如 `[issue-number]` |
| `arguments` | 命名位置参数，供 `$name` 替换使用 |
| `disable-model-invocation` | `true` 时 Claude 不能自行调用，只能用 `/name` 触发。默认 `false` |
| `user-invocable` | `false` 时从 `/` 菜单隐藏，只有 Claude 能调用。默认 `true` |
| `allowed-tools` | 调用该 Skill 的那一轮中免确认可用的工具；你发下一条消息即清除 |
| `disallowed-tools` | Skill 活动期间从可用工具池移除的工具 |
| `model` | Skill 活动期间使用的模型，仅当前轮有效；`inherit` 保留当前模型 |
| `effort` | 当前轮的 effort 级别：`low`、`medium`、`high`、`xhigh`、`max` |
| `context` | 设为 `fork` 时在分叉的子代理上下文中运行 |
| `agent` | `context: fork` 时使用的子代理类型 |
| `background` | 仅 `context: fork` 时有效；`false` 表示在本轮等待结果。默认 `true` |
| `hooks` | 调用 Skill 时注册的 hook |
| `paths` | 激活该 Skill 的文件 glob 模式 |
| `shell` | `` !`command` `` 使用的 shell：`bash`（默认）或 `powershell` |
| `metadata` / `license` / `compatibility` | 自定义键值、许可证、环境要求；Claude Code 接受但不处理 |

#### 最小示例与验证

保存为 `~/.claude/skills/summarize-changes/SKILL.md`：

```yaml
---
description: Summarizes uncommitted changes and flags anything risky. Use when the user asks what changed, wants a commit message, or asks to review their diff.
---

## Current changes

!`git diff HEAD`

## Instructions

Summarize the changes above in two or three bullet points, then list any risks you notice.
```

1. 打开一个 git 项目，改动任意文件，运行 `claude`。
2. 自动触发：问 `What did I change?`；手动触发：输入 `/summarize-changes`。
3. 没出现时运行 `/reload-skills`，并用 `/skills` 确认。新增或修改 Skill 一般无需重启。

#### 参数、替换与动态上下文

| 变量 | 含义 |
|---|---|
| `$ARGUMENTS` | 全部参数；正文没有任何占位符时以 `ARGUMENTS: <value>` 附加在末尾 |
| `$ARGUMENTS[N]` / `$N` | 第 N 个参数，从 0 开始 |
| `$name` | `arguments` 中声明的命名参数 |
| `${CLAUDE_SESSION_ID}` / `${CLAUDE_EFFORT}` | 当前会话 ID / effort 级别 |
| `${CLAUDE_SKILL_DIR}` / `${CLAUDE_PROJECT_DIR}` | 该 `SKILL.md` 所在目录 / 项目根目录 |
| `${CLAUDE_PLUGIN_ROOT}` / `${CLAUDE_PLUGIN_DATA}` | 插件安装目录 / 持久数据目录，仅插件 Skill 中替换 |

- 多词参数加引号：`/my-skill "hello world" second`；字面 `$1.00` 写成 `\$1.00`。
- 运行自带脚本而不弹权限提示：在 `allowed-tools` 和正文两处使用同一路径，如 `allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/render.sh *)`。
- `` !`<command>` `` 在内容发给 Claude **之前**执行并替换为输出；多行命令用以 ` ```! ` 开头的围栏代码块。`!` 必须在行首或紧跟空格之后；替换只进行一次。
- 设置 `"disableSkillShellExecution": true` 可禁用注入（捆绑和托管 Skill 不受影响）。

#### 调用控制

| frontmatter | 你能调用 | Claude 能调用 | 描述是否在上下文中 |
|---|---|---|---|
| 默认 | 是 | 是 | 是 |
| `disable-model-invocation: true` | 是 | 否 | 否 |
| `user-invocable: false` | 否 | 是 | 是 |

- 有副作用的流程（提交、部署、发消息）用 `disable-model-invocation: true`。
- `/deploy staging` 放在消息**开头**才直接运行；放在句子中间只是授权 Claude 可以运行它。
- 权限规则：`Skill(name)` 精确匹配，`Skill(name *)` 前缀匹配；deny 规则里写 `Skill` 可禁用全部 Skill。
- 不想改 `SKILL.md` 时用设置项 `skillOverrides`，值为 `"on"`、`"name-only"`、`"user-invocable-only"`、`"off"`（不影响插件 Skill）。

#### 在子代理中运行

在 frontmatter 中加入 `context: fork` 和 `agent: Explore`，正文写成完整任务（如 `Research $ARGUMENTS thoroughly and summarize findings with specific file references.`）。

- 子代理**看不到对话历史**，Skill 内容就是它的全部任务，必须写成可独立执行的任务，而不是"遵循这些约定"式的参考。
- `agent` 可选 `Explore`、`Plan`、`general-purpose` 或 `.claude/agents/` 中的自定义子代理；省略时为 `general-purpose`。内置 Explore 和 Plan 启动时不加载 CLAUDE.md。
- 默认在后台运行；`background: false` 或 `-p` 模式下会等待结果。后台分叉 Skill 的编辑不在检查点内，`/rewind` 无法恢复，请用 git 还原。

#### 生命周期与旧 commands

- Skill 被调用后，内容作为一条消息留在对话中，后续轮次不会重新读取文件。自动压缩后每个 Skill 只保留前 5,000 个 token（共享 25,000 token 预算），所以**最重要的指令放在顶部**；发现 Claude 不再遵循就重新调用一次。
- 自定义命令已合并到 Skill。`.claude/commands/*.md` 继续有效，支持相同的 frontmatter（`name` 和 `paths` 除外）。新工作优先用 Skill；迁移时把 `.claude/commands/deploy.md` 移到 `.claude/skills/deploy/SKILL.md`，命令仍是 `/deploy`。

#### 常见失败

| 症状 | 检查 |
|---|---|
| Skill 不出现 | 写成了 `.claude/skills/name.md`？必须是 `.claude/skills/name/SKILL.md` |
| Claude 不自动触发 | `description` 是否包含用户会说的关键词；是否设了 `disable-model-invocation: true`；用 `--debug` 看解析错误 |
| 描述被截断 | Skill 太多时列表预算（上下文窗口的 1%）不够；运行 `/doctor`，或把低优先级条目设为 `"name-only"` |
| frontmatter 写坏了 | `claude plugin validate .claude/skills`（v2.1.233+） |

### Hooks 入门：确定性的自动化

Hook 是在 Claude Code 生命周期特定时点自动运行的 shell 命令。Skill 靠模型判断是否使用，Hook **一定会执行**：必须每次成立的规则（格式化、禁止改某些文件、审计）交给 Hook 或权限，而不是 CLAUDE.md。

> 本节只覆盖入门与常见用法；完整的事件列表、输入输出字段和退出码规则见"参考"部分的 Hooks 参考。

#### 配置位置

Hook 写在设置文件里：`~/.claude/settings.json`（你的所有项目）、`.claude/settings.json`（单个项目，可提交）、`.claude/settings.local.json`（单个项目，不共享）；此外还可来自托管策略设置、插件的 `hooks/hooks.json` 以及 Skill / 子代理的 frontmatter。

项目和用户配置**没有独立的 hooks 文件**，Hook 必须写在设置文件的 `"hooks"` 键下。

#### 最小示例：编辑后自动格式化

写入项目根的 `.claude/settings.json`（示例依赖 `jq`）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "f=$(jq -r '.tool_input.file_path') && npx prettier --write --ignore-unknown \"$f\""
          }
        ]
      }
    ]
  }
}
```

1. 已有 `hooks` 键时，把新事件作为同级键加入，不要覆盖整个对象。
2. 在会话里输入 `/hooks`，确认它出现在对应事件下。
3. 触发一次真实操作并检查效果（成功的 Hook 在对话里不显示任何内容）。
4. 没有效果时按 `Ctrl+O` 查看会话记录，或用 `claude --debug` 启动观察。

#### 输入、输出与常见用途

Hook 通过 stdin 收到事件 JSON（如 `hook_event_name`、`tool_name`、`tool_input`），通过退出码回应：

| 退出码 | 效果 |
|---|---|
| `0` | 没有异议。对 `PreToolUse` **不等于批准**，仍走正常权限流程 |
| `2` | 阻止该操作，stderr 作为反馈交给 Claude（部分事件不可阻止） |
| 其他 | 多数事件下操作继续，记为非阻止错误 |

| 目标 | 事件 | 要点 |
|---|---|---|
| Claude 等待输入时发桌面通知 | `Notification` | `matcher` 为空字符串 |
| 编辑后自动格式化 | `PostToolUse` | `matcher` 用 `Edit\|Write` |
| 阻止编辑 `.env` 等受保护文件 | `PreToolUse` | 脚本以退出码 2 阻止 |
| 压缩后重新注入上下文 | `SessionStart` | 用 `compact` 匹配器，stdout 加入上下文 |
| 审计配置变更 | `ConfigChange` | 追加到审计日志 |
| 切换目录时重载环境变量 | `SessionStart` + `CwdChanged` | 配合 direnv 等工具 |

除 `"type": "command"` 外还有 `http`、`mcp_tool`、`prompt`（单轮模型评估）和 `agent`（多轮验证，**实验性**）。

#### 边界提醒

- `matcher` 是**单个字符串**且区分大小写：`"Edit|Write"` 可以，数组和 `"bash"` 都不行。
- `PostToolUse` 发生在工具执行之后，无法撤销。
- Hook 返回 `"allow"` 不能绕过设置里的拒绝规则；返回 `"deny"` 在任何权限模式下都生效，包括 `bypassPermissions`。
- `Stop` Hook 连续阻止 8 次且期间没有工具调用时会被覆盖；脚本应检查输入中的 `stop_hook_active`。
- 用 `"$CLAUDE_PROJECT_DIR"` 引用脚本路径并 `chmod +x`；`"disableAllHooks": true` 可整体关闭。

### Channels：把外部事件推送进会话（研究预览）

Channel 是会**主动推送**消息的 MCP 服务器：CI 结果、聊天消息、webhook 到达你已经打开的会话，Claude 可通过同一渠道回复。

> 研究预览：`--channels` 语法和协议可能变化。需要通过 claude.ai 或 Console API 密钥进行 Anthropic 身份验证；在 Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry 上不可用。Team 和 Enterprise 组织须由管理员先启用（托管设置 `channelsEnabled`）。

预览包含 Telegram、Discord、iMessage 插件和本地演示用的 fakechat，均需要先安装 Bun。体验流程：

1. 会话中安装：`/plugin install fakechat@claude-plugins-official`
2. 退出后重启：`claude --channels plugin:fakechat@claude-plugins-official`
3. 在 fakechat 本地页面发消息；终端显示入站行，回复出现在聊天页面。

- 只写在 `.mcp.json` 里不够，服务器必须在 `--channels` 中点名才会推送。
- 事件只在会话打开时到达；会话卡在权限提示上时会暂停。
- 能通过渠道回复的人可能批准或拒绝你会话中的工具调用，只把信任的人加入发送者允许列表。
- 预览期间 `--channels` 只接受 Anthropic 维护的允许列表（或组织 `allowedChannelPlugins`）中的插件，且该标志不出现在 `claude --help` 中。

### 定时任务与 `/loop`

会话范围的定时任务让 Claude 在**当前会话保持打开期间**按间隔重复运行提示词，适合轮询部署、盯 PR、设置提醒。

| 你提供的内容 | 示例 | 行为 |
|---|---|---|
| 间隔 + 提示词 | `/loop 5m check the deploy` | 按固定计划运行 |
| 仅提示词 | `/loop check the deploy` | Claude 每次迭代后自选 1 分钟到 1 小时的间隔 |
| 什么都不给 | `/loop` | 运行内置维护提示词；存在 `loop.md` 时运行它 |

- 间隔单位 `s`、`m`、`h`、`d`；秒向上取整到分钟，`7m` 这类无法整齐映射的会被舍入。
- 可把 Skill 当提示词：`/loop 20m /review-pr 1234`。内置命令、`disable-model-invocation: true` 的 Skill 和 MCP 提示在定时触发时只被当作纯文本。
- 自定义默认提示词：`.claude/loop.md`（项目级，优先）或 `~/.claude/loop.md`。
- 一次性提醒直接说：`remind me at 3pm to push the release branch`。管理任务也用自然语言，底层工具是 `CronCreate`、`CronList`、`CronDelete`。
- 停止：自定节奏的循环在空输入框按 `Esc`；**固定间隔的循环 `Esc` 和 `Ctrl+C` 都停不了**，要让 Claude 取消。

限制：重复任务创建 **7 天后自动过期**；一个会话最多 50 个任务；时间按本地时区解释且有抖动；只在 Claude Code 运行且空闲时触发，错过不补跑；`--resume` / `--continue` 会恢复未过期的任务，但自定节奏的 `/loop` 不会恢复。`CLAUDE_CODE_DISABLE_CRON=1` 可完全禁用。

需要脱离会话可靠运行时，改用云端定时任务（最小间隔 1 小时，不需要机器开着）、Desktop 定时任务或 GitHub Actions。

### `/goal`：朝着完成条件持续工作

`/goal` 设置一个完成条件。每轮结束后由小型快速模型检查条件是否满足：没满足就自动开始下一轮，而不是把控制权还给你。

```text
/goal all tests in test/auth pass and the lint step is clean
/goal                 # 查看状态：条件、时长、已评估轮次、token 支出
/goal clear           # 清除（别名：stop、off、reset、none、cancel）
```

与 `/loop`（按时间间隔触发）和 Stop hook（由你的脚本决定）不同，`/goal` 由评估器在每轮结束后判断是否继续。

写好条件：给出**可测量的最终状态**，**写明如何证明**（如"`npm test` 退出 0"），并写上**约束**（如"不修改其他测试文件"）。评估器不会自己跑命令或读文件，只看对话里已呈现的内容。需要上限时加 `or stop after 20 turns`。

- 每个会话只有一个活跃目标；条件最多 4,000 个字符。
- `/goal` **不改变权限模式**：要无人值守请在自动模式下运行。
- 评估 token 在主轮次之外单独计费。
- 身份验证失败、额度耗尽、模型不可用等错误会清除目标，修复后重新设置。
- 非交互：`claude -p "/goal <condition>"`，建议加 `--output-format stream-json --verbose`。
- `disableAllHooks` 为 `true` 或托管设置启用 `allowManagedHooksOnly` 时不可用。

### 程序化运行：`claude -p`

给 `claude` 加 `-p`（或 `--print`）即以非交互方式运行：成功退出码为 0，失败非零；stdin 可管道输入（上限 10MB）。

```bash
claude -p "What does the auth module do?"
cat build-error.txt | claude -p 'concisely explain the root cause of this build error' > output.txt
```

| 目标 | 标志 | 说明 |
|---|---|---|
| 输出格式 | `--output-format` | `text`（默认）、`json`（文本在 `result`，含 `session_id`、`total_cost_usd`）、`stream-json` |
| 结构化输出 | `--json-schema '<schema>'` | 配合 `--output-format json`，结果在 `structured_output` 字段 |
| 流式输出 | `--output-format stream-json --verbose --include-partial-messages` | 每行一个 JSON 事件 |
| 免确认工具 | `--allowedTools "Read,Edit,Bash(git diff *)"` | 使用权限规则语法 |
| 权限基线 | `--permission-mode` | 如 `auto`、`dontAsk`、`acceptEdits` |
| 无人应答 | `--permission-prompts none` | 会弹提示的调用直接拒绝且不重试（v2.1.259+） |
| 追加系统提示词 | `--append-system-prompt` | 保留默认行为；`--system-prompt` 为完全替换 |
| 继续会话 | `--continue` / `--resume <session-id>` | 最近一次 / 指定会话 |
| 最小化启动 | `--bare` | 跳过 hook、skill、插件、MCP、自动记忆和 CLAUDE.md 的自动发现 |

```bash
claude -p "Extract the main function names from auth.py" \
  --output-format json \
  --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}}},"required":["functions"]}' \
  | jq '.structured_output'

session_id=$(claude -p "Start a review" --output-format json | jq -r '.session_id')
claude -p "Continue that review" --resume "$session_id"
```

使用 `structured_output` 前先检查字段是否存在（Claude 拒绝请求时不会有）。

#### bare 模式与 CI 权限

```bash
claude --bare -p "Summarize README.md" --allowedTools "Read"
```

- `--bare` 是脚本和 SDK 调用的推荐模式，文档说明它将在未来版本成为 `-p` 的默认值。
- bare 模式**不读取 OAuth 凭据和系统密钥链**：需设置 `ANTHROPIC_API_KEY` 或在 `--settings` 中提供 `apiKeyHelper`。
- 需要的上下文显式传入：`--append-system-prompt`、`--settings`、`--mcp-config`、`--agents`、`--plugin-dir`。bare 模式下 hook 一律不运行。

```text
CI 中没有人能回答提示
├─ 只允许明确列出的操作 → --allowedTools + --permission-mode dontAsk
├─ 允许改文件           → --permission-mode acceptEdits（其他 shell 命令仍需规则）
└─ 让分类器把关         → --permission-mode auto --permission-prompts none
```

- 手动模式的 `-p` 运行中，需要批准的调用会被**直接拒绝**；v2.1.302+ 在没有任何设置指定权限模式时，`-p` 以手动模式启动。
- `Bash(git diff *)` 中 `*` 前的空格有意义：`Bash(git diff*)` 还会匹配 `git diff-index`。
- 不带 `--bare` 时，`-p` 会运行项目 `.claude/settings.json` 里的 hook 并连接 `.mcp.json` 里的服务器，**即使该目录从未被信任**，也不弹信任或批准提示。
- 用 `stream-json` 时可检查 `system/init` 事件的 `plugin_errors` 和 `mcp_server_errors`，在插件或 MCP 未加载时让 CI 失败。
- 提示词里可写 `/skill-name` 调用 Skill；`/login` 这类仅终端的内置命令不可用。

### 深链接：从 URL 启动会话

`claude-cli://` 链接在新终端窗口中打开 Claude Code，可携带工作目录和预填充的提示词，适合放进运行手册、告警、CI 失败通知。

```text
claude-cli://open?repo=<owner>/<name>&q=<URL 编码后的提示词>
```

| 参数 | 目的 |
|---|---|
| `q` | 预填充的提示词，需 URL 编码，换行用 `%0A`，最多 5,000 个字符 |
| `cwd` | 工作目录的绝对路径；网络路径、UNC 路径和含 `..` 的路径会被拒绝 |
| `repo` | GitHub `owner/name`；解析为 Claude Code 之前见过的本地克隆，找不到则在主目录打开 |

- 同时传 `cwd` 和 `repo` 时 `cwd` 优先。大家路径一致用 `cwd`，各自克隆位置不同用 `repo`。
- **链接只填充输入框，不会自动发送**；输入框下方会显示 `Prompt from an external link`。
- 处理程序在你向交互式会话发送过任意内容后自动注册。点击没反应时，先在该机器的 `claude` 会话里发一条消息再试。
- 从 shell 打开：macOS 用 `open "<链接>"`，Linux 用 `xdg-open`。
- GitHub 等只允许 `http`/`https` 的平台会去掉该链接，请放进代码块让读者复制。
- 禁止注册：在 `settings.json` 中把 `disableDeepLinkRegistration` 设为 `"disable"`。

### 大型代码库与 monorepo 设置

| 我想要 | 使用 |
|---|---|
| 只加载当前所碰代码的约定 | 按目录分层的 CLAUDE.md（根文件放全局规则，子目录放本区域约定） |
| 排除从不处理的包的 CLAUDE.md | `claudeMdExcludes` |
| 不让 Claude 打开已提交的生成代码、vendor 依赖 | `permissions.deny` 中的 `Read` 拒绝规则 |
| 用语言服务器查定义和引用，而不是扫文件 | 代码智能插件，如 `/plugin install typescript-lsp@claude-plugins-official` |
| worktree 只检出任务需要的目录 | `worktree.sparsePaths` |
| 同一会话读写同级包或另一个仓库 | `--add-dir` 或 `additionalDirectories` |
| 某个区域专用的流程 | 按目录的 Skill |

启动位置决定加载内容：从**仓库根**启动可访问所有文件，但只加载根 CLAUDE.md（子目录的按需加载）；从**子目录**启动只能访问该子树，加载该目录及所有祖先目录的 CLAUDE.md。`.claude/settings.json` 项目设置**不会像 CLAUDE.md 那样从父目录继承**。

```json
{
  "claudeMdExcludes": ["**/packages/web/**"],
  "permissions": {
    "deny": ["Read(./**/dist/**/*)", "Read(./**/vendor/**/*)", "Read(./**/*.generated.*)"],
    "additionalDirectories": ["../shared", "../web"]
  },
  "worktree": { "sparsePaths": [".claude", "packages/api", "packages/shared"] }
}
```

- 内容搜索默认尊重 `.gitignore`，`node_modules/`、`dist/` 等无需额外配置；拒绝规则不覆盖自行打开文件的子进程。
- `sparsePaths` 写目录而非文件；根级文件总会检出，根级目录不会，所以要列出 `.claude`。
- `additionalDirectories` 只授予文件访问，**不加载**那里的 CLAUDE.md 和 Skill；`--add-dir` / `/add-dir` 会加载 Skill，加载 CLAUDE.md 还需 `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`。
- 验证：运行 `/context all`，查看 **Memory files** 下实际加载的文件。

### 排错：安装、登录、配置与常见错误

#### 先选对入口

```text
出问题了
├─ claude 起不来 / 安装失败 / 登录失败            → 终端运行 claude doctor，查"安装与登录"
├─ 配置没生效（CLAUDE.md、设置、hook、MCP、skill） → /context、/doctor，查"配置不生效"
├─ 能用但慢、卡、搜不到文件                        → 查"运行问题"
└─ API Error / 限额 / 网络报错                    → 查"常见错误速查"
```

| 命令 | 用途 |
|---|---|
| `claude doctor` | 终端运行；只读的安装和设置诊断，不启动会话 |
| `/doctor` | 会话内的完整检查，提议修复并在应用前询问 |
| `/context`（全屏模式用 `/context all`） | 上下文窗口里实际加载了什么 |
| `/status` | 当前生效的设置来源，包括托管设置是否生效 |
| `/memory`、`/skills`、`/hooks`、`/mcp`、`/permissions` | 记忆文件、Skill、Hook、MCP 服务器、已解析的权限规则 |
| `/debug [issue]` | 启用调试日志并让 Claude 协助诊断 |
| `claude --safe-mode` | 禁用 CLAUDE.md、Skill、插件、Hook、MCP、自定义命令和 Agent 后启动，用于定位 |

#### 安装与登录

| 你看到的 | 处理 |
|---|---|
| `command not found: claude` | 安装目录不在 PATH：macOS/Linux 为 `~/.local/bin/claude`，Windows 为 `%USERPROFILE%\.local\bin\claude.exe`；`which -a claude` 检查是否有多个安装 |
| `syntax error near unexpected token '<'` | 安装脚本返回了 HTML，检查网络和代理 |
| `irm is not recognized` / `'bash' is not recognized` | 在一种 shell 里用了另一种 shell 的安装命令 |
| `unable to get local issuer certificate`、TLS 错误 | 配置企业 CA 证书 |
| `That doesn't look like the full code` | 重新复制完整登录代码（包含中间的 `#`） |
| WSL2 / SSH / 容器中浏览器打不开 | 按 `c` 复制登录 URL 到本机浏览器，再把代码粘贴回终端；或用 `claude auth login` |
| 反复要求重新登录 | 运行 `/login`，并检查系统时钟 |

原因不明时的通用重置：`/logout` → 关闭 Claude Code → 重新运行 `claude` 完成认证。

#### 运行问题

- **CPU/内存高**：定期 `/compact`；大任务之间重启；用 `claude --safe-mode` 判断是否由插件、MCP 或 Hook 引起。
- **`Autocompact is thrashing`**：让 Claude 分块读大文件；`/compact keep only the plan and the diff`；把大文件工作交给子代理。
- **卡死**：`Ctrl+C`；仍无响应就关终端，回到同一目录运行 `claude --resume`，对话不会丢。
- **搜索或 `@file` 找不到文件**：安装系统的 `ripgrep`，并把 `USE_BUILTIN_RIPGREP` 设为 `0`。WSL 下把项目放在 Linux 文件系统而不是 `/mnt/c/`。

#### 配置不生效

| 症状 | 原因与修复 |
|---|---|
| 全局的权限、hooks、env 被忽略 | 写进了 `~/.claude.json`；它们属于 `~/.claude/settings.json`，这是两个不同的文件 |
| `settings.json` 的值不生效 | 被 `settings.local.json`、托管设置、命令行标志或环境变量覆盖，用 `/status` 查看 |
| Hook 从不触发 | `matcher` 写成数组、工具名小写，或 Hook 放在了独立文件里 |
| 子目录 CLAUDE.md 被忽略 | 子目录文件按需加载，让 Claude 读一下该目录里的文件 |
| MCP 服务器在某些目录启动失败 | `command`/`args` 的相对路径按启动目录解析，改用绝对路径 |
| `Bash(rm *)` 拒绝规则挡不住 `/bin/rm` | Bash 规则匹配命令字符串而非可执行文件；要硬保证用 PreToolUse hook 或沙箱 |

仍找不到原因，用干净配置对照；问题消失说明原因在你的用户或项目配置中：

```bash
mkdir -p ~/claude-clean/empty && cd ~/claude-clean/empty
CLAUDE_CONFIG_DIR=~/claude-clean/config claude --setting-sources user
```

#### 常见错误速查

Claude Code 在显示错误前已对瞬时故障做过指数退避重试（最多 10 次）。

| 消息 | 含义 | 处理 |
|---|---|---|
| `API Error: 500 Internal server error` | 服务端故障，与你的提示词和账户无关 | 稍后重试，查看 status.claude.com |
| `API Error: Repeated 529 Overloaded errors` | API 暂时满负荷；**不计入你的用量** | 几分钟后重试，或 `/model` 换模型 |
| `Request timed out` | 请求超时（默认 10 分钟） | 重试；检查 `API_TIMEOUT_MS` |
| `You've hit your session limit` / `weekly limit` | 订阅额度用完，跨模型共享 | 等待重置；`/usage` 查看 |
| `You've hit your Opus limit` / `Sonnet limit` | 仅该模型系列受限 | `/model` 切到该系列之外的模型 |
| `Request rejected (429)` | 触及 API 密钥或云项目的速率限制 | 稍后重试，检查提供商配额 |
| `Credit balance is too low` | Console 预付额度用完，或本想用订阅却走了 API 密钥 | `/status` 看 `API key` 行；取消设置 `ANTHROPIC_API_KEY` 后 `/login` |
| `Not logged in · Please run /login` | 没有有效凭据 | 运行 `/login` |
| `Invalid API key · Fix external API key` | `ANTHROPIC_API_KEY` 或 `apiKeyHelper` 的密钥被拒绝 | 检查拼写和换行符，确认未被撤销 |
| `Prompt is too long` / `Context limit reached` | 对话加附件超过上下文窗口 | `/compact` 或 `/clear` |
| `Unable to connect to API` | 网络、DNS、代理或防火墙问题 | 按括号里的错误代码排查 |
| `SSL certificate verification failed` | 企业代理用自己的证书拦截 TLS | 把 `NODE_EXTRA_CA_CERTS` 指向 CA 证书包 |

仍未解决：会话内用 `/feedback` 报告；账户、账单和订阅问题联系 Anthropic 支持。

---

## Part III：插件

### 插件是什么：打包、安装与加载的单元

插件（Plugin）是一个目录，里面放着 skills、agents、hooks、MCP 服务器等组件，Claude Code 把它**作为一个单元安装和加载**。它不是新的编写格式：Skill 仍是 Skill，Hook 仍是 Hook，插件只负责"打包 + 版本 + 分发"。市场（marketplace）则是列出插件及其获取位置的目录清单，不是托管商店。

```text
my-plugin/
├── .claude-plugin/plugin.json   → 清单：插件名 my-plugin
├── skills/review/SKILL.md       → 以 /my-plugin:review 运行
├── agents/reviewer.md           → 可委派的子代理 my-plugin:reviewer
├── hooks/hooks.json             → 生命周期 hooks（加 modules 键即成为 mod）
└── .mcp.json                    → MCP 服务器，为 Claude 提供工具
```

| 场景 | 选择 | 命名 |
|---|---|---|
| 只服务一个项目或只给自己用 | 独立配置：`.claude/` 或 `~/.claude/` | `/deploy`、`reviewer` |
| 分享给团队、装到多个项目、发布带版本的发行 | 插件 | `/my-plugin:deploy`、`my-plugin:reviewer` |

启用的插件是**每个会话**的一部分：每个 skill、agent 的名称和描述每回合都在上下文里并计入用量（正文用到才加载），MCP 服务器随会话运行，hooks 在各自事件上触发，且都以你的身份运行。

> 边界提醒：插件市场不是 claude.com/marketplace 上的 Claude Marketplace 网站；VS Code 扩展、JetBrains 插件也不是 Claude Code 插件。claude.ai 与 Cowork 用同一插件格式，但加载的组件集合不同。

### 使用插件：安装、作用域与日常管理

#### 操作流程（终端）

1. 会话中运行 `/plugin install commit-commands@claude-plugins-official`。它不会立刻安装，而是打开详情面板；只输入 `/plugin` 则进入 **Discover** 选项卡浏览。
2. 阅读 **Will install**（将添加的组件）；官方市场插件还显示 **Context cost** 估算。
3. 选择作用域，Claude Code 安装并打印摘要。
4. 读摘要最后一句：`Plugin is now active.` 可用；`Run /reload-plugins to apply.` 需重新加载；出现 `The plugin couldn't` 去 **Errors** 选项卡看原因。
5. 验证：输入 `/` 查找 `/commit-commands:commit`，或在 shell 运行 `claude plugin list`。

| 作用域 | 写入的设置文件 | 谁获得 |
|---|---|---|
| user（shell 安装的默认值） | `~/.claude/settings.json` 的 `enabledPlugins` | 你，在本机每个项目 |
| project | `.claude/settings.json`（需提交） | 该仓库的每个协作者 |
| local | `.claude/settings.local.json` | 你，仅此仓库 |

多处设置时 local 覆盖 project，project 覆盖 user；托管设置最高。

> project 作用域只是"为协作者打开"，不会把插件下载到他们的机器。每人仍需在自己的克隆里运行一次 `claude plugin install <name>@<marketplace> --scope project`。

其他界面：桌面应用点输入框旁 **+** → **Add plugins**；VS Code 输入 `/plugins` 打开 **Manage plugins**；`claude -p` 中 `/plugin` 不可用，改用 shell 的 `claude plugin ...`；云端会话没有插件浏览器，也不加载 user 或 local 作用域的插件。

#### 添加市场

| 来源 | 示例 |
|---|---|
| GitHub `owner/repo`，可加 `#ref` | `/plugin marketplace add your-org/plugins#v1.2.0` |
| 任意主机的 git 克隆 URL | `/plugin marketplace add https://gitlab.example.com/your-group/your-marketplace.git#v1.0.0` |
| 本地目录或文件（以 `./`、`../` 开头） | `/plugin marketplace add ./my-marketplace` |
| 托管的 `marketplace.json` | `/plugin marketplace add https://example.com/marketplace.json` |

- 常见失败：裸写 `gitlab.example.com/group/repo.git` 会被当成 GitHub 简写而拒绝，必须带 `https://` 或用 `git@host:path`。
- 私有市场用本机已有的 git 凭证克隆，**从不提示**，凭证不可用就直接失败。
- 一步到位：`/plugin install deploy-helper --marketplace your-org/plugins`（需 v2.1.275+）；shell 形式需 v2.1.292+，且没有确认步骤。

#### 启用、停用、更新、卸载

```bash
claude plugin disable formatter@your-org
claude plugin enable formatter@your-org
claude plugin update formatter@your-org
claude plugin uninstall formatter@your-org --scope project
```

- 面板：`/plugin` 的 **Installed** 选项卡，**Space** 启停，**Enter** 进详情（**Update now**、**Uninstall**）。
- 自动更新默认只对官方市场（`knowledge-work-plugins`、`first-party-plugins` 除外）和从 claude.ai 添加的市场开启，其余默认关闭，可在 **Marketplaces** 选项卡逐个切换。
- 没有"一次更新全部插件"的命令；不带名称的 `claude plugin marketplace update` 只刷新目录，不动已安装版本。
- **删除市场会卸载从它安装的全部插件**。只想刷新请用 `marketplace update`。
- claude.ai 账户里打开的插件会同步到终端，id 为 `<name>@synced`（需 v2.1.273+）；本机安装的插件不会反向进入账户。

### Anthropic 的市场与代码智能（LSP）插件

| | 官方 | 社区 |
|---|---|---|
| 仓库 | `anthropics/claude-plugins-official` | `anthropics/claude-plugins-community` |
| 市场名称（`@` 后输入） | `claude-plugins-official` | `claude-community` |
| 获取方式 | 首次启动交互式终端会话时自动添加 | 手动 `/plugin marketplace add anthropics/claude-plugins-community` |

- 托管策略或 `CLAUDE_CODE_DISABLE_OFFICIAL_MARKETPLACE_AUTOINSTALL` 可阻止自动添加；没开过交互式会话的机器上，脚本要先 `claude plugin marketplace add anthropics/claude-plugins-official`。
- `anthropics/claude-code` 里的 `claude-code-plugins` 市场已弃用。其余市场都是第三方，Anthropic 不审查。

代码智能插件让 Claude 在每次编辑后拿到语言服务器的诊断，并获得只读的 `LSP` 工具做符号级导航。插件**只负责连接，不包含语言服务器二进制**：

1. 先装二进制并确认在 `PATH` 上，例如 `npm install -g typescript-language-server typescript`。
2. 安装插件：`/plugin install typescript-lsp@claude-plugins-official`。
3. 验证：让 Claude 在该语言文件里引入一个类型错误再修复，编辑下方出现 `Found N new diagnostic issues in M files (ctrl+o to expand)` 即成功。

常用插件与二进制：`typescript-lsp`（`typescript-language-server`）、`pyright-lsp`（`pyright-langserver`）、`gopls-lsp`（`gopls`）、`rust-analyzer-lsp`（`rust-analyzer`）；官方表中还有 `clangd-lsp`、`csharp-lsp`、`jdtls-lsp`、`kotlin-lsp`、`liquid-lsp`、`lua-lsp`、`php-lsp`、`ruby-lsp`、`swift-lsp`。

常见失败：没有诊断行时看 `/plugin` 的 **Errors** 选项卡，`Executable not found in $PATH: "<binary>"` 指出缺哪个二进制。云端会话不启动插件语言服务器。

### 安全与信任

> 你安装的插件可以用你的用户权限在本机执行任意代码。市场名称不能说明其中每个插件做什么。

| 插件内容 | 权限规则与沙箱是否覆盖 |
|---|---|
| Hooks、Monitors、MCP 与 LSP 服务器进程、Mod | 否，以完整用户权限在沙箱外运行 |
| 对插件 MCP 工具的调用、运行 `bin/` 里可执行文件的 Bash 命令 | 是，属于 Claude 的工具调用 |
| Skills、commands、agents | 作为指令进入上下文，影响 Claude 如何用已有工具 |

安装前审查流程：

1. `claude plugin marketplace list`：确认市场从哪里添加。
2. `/plugin` 选中插件读 **Will install**。它只显示 hook 存在，不显示 hook 运行什么。
3. 读源码：`hooks/hooks.json`、`.mcp.json`、`bin/` 下每个文件。
4. 克隆后列清单（不启动会话）：`claude --plugin-dir <plugin directory> plugin details <plugin name>`。

开了自动更新的市场会在后台更新插件，你审查过的文件可能变化。卸载用 `claude plugin uninstall <plugin>`；缓存文件还会在 `~/.claude/plugins/cache/` 下保留 14 天，要立即清除需手动删除。

### 创建插件：目录结构与 manifest

#### 操作流程

```bash
mkdir -p my-first-plugin/.claude-plugin my-first-plugin/skills/hello
```

`my-first-plugin/.claude-plugin/plugin.json`：

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

再写 `my-first-plugin/skills/hello/SKILL.md`：frontmatter 含 `name: hello` 和 `description`，正文是给 Claude 的指令；加 `disable-model-invocation: true` 表示 Claude 不会自行运行它。

```bash
claude plugin validate ./my-first-plugin     # 期望 ✔ Validation passed
claude --plugin-dir ./my-first-plugin        # 仅本次会话加载，不安装
```

会话内运行 `/my-first-plugin:hello` 验证；编辑文件后用 `/reload-plugins` 生效。

#### 标准目录结构

```text
deploy-tools/
├── .claude-plugin/plugin.json   # 清单；这个目录里只放它
├── skills/deploy/SKILL.md       # 每个 skill 一个 <name>/SKILL.md
├── commands/status.md           # 旧格式，新插件用 skills/
├── agents/reviewer.md           # 每个子代理一个 Markdown
├── hooks/hooks.json             # 顶层 "hooks" 键，形状同 settings.json
├── monitors/monitors.json       # 后台 shell 命令，输出作为通知送达 Claude
├── output-styles/ themes/ workflows/
├── bin/deploy-tool              # 启用时加入 Bash 工具的 PATH
├── .mcp.json                    # MCP 服务器
└── .lsp.json                    # LSP 服务器
```

> 最常见的坑：把 `skills/`、`hooks/` 放进 `.claude-plugin/` 里，组件不会加载。插件根目录的 `CLAUDE.md` 也不会作为上下文加载，要带说明就写成 skill。

#### `plugin.json` 关键字段

| 字段 | 目的 |
|---|---|
| `name` | 唯一必需。kebab-case，不含空格、`@`、`:`；所有组件以它为前缀 |
| `displayName` | UI 中显示的名称，不参与命名空间 |
| `version` | 设置后用户停留在该版本直到你改它；省略则跟踪提交 |
| `description` / `author` / `homepage` / `repository` / `license` / `keywords` | 展示与发现用元数据 |
| `defaultEnabled` | 用户未设置时是否默认启用，默认 `true` |
| `dependencies` | 必须同时启用的其他插件 |
| `userConfig` | 启用时向用户询问的配置值 |
| `skills` | 额外的 skill 目录，**添加到**默认 `skills/` 扫描 |
| `commands` / `agents` / `outputStyles` / `workflows` | **替换**对应默认目录的扫描 |
| `hooks` / `mcpServers` / `lspServers` | 与 `hooks/hooks.json`、`.mcp.json`、`.lsp.json` **合并** |
| `experimental.themes` / `experimental.monitors` / `experimental.evals` | 实验性容器，清单形式仍可能变化 |

清单里的组件路径相对插件根目录，必须以 `./` 开头、不含 `..` 且存在。以 `claude-`、`anthropic-`、`anthropics-`、`cc-plugin-` 开头的名称会让 `claude plugin validate` 报错。

#### 组件命名、路径变量与用户配置

| 组件 | 加载后的名称 | 要点 |
|---|---|---|
| Skill | `/<plugin>:<directory>` | frontmatter 的 `name` 只替换最后一段 |
| Agent | `<plugin>:<name>` | 忽略 `permissionMode`、`hooks`、`mcpServers`、`initialPrompt` |
| Hook | 无前缀 | 会话加载插件时即注册，不等你用到某个 skill |
| MCP 服务器 | `plugin:<plugin>:<server>` | 工具名 `mcp__plugin_<plugin>_<server>__<tool>`，权限规则和 hook 匹配器都用全名 |
| 可执行文件 | 裸命令 | 排在用户自己的 `PATH` 之后，不能遮蔽 `git`、`ls` |

| 变量 | 解析为 | 用途 |
|---|---|---|
| `${CLAUDE_PLUGIN_ROOT}` | 插件已安装版本的绝对路径 | 引用捆绑的脚本。**更新后路径会变，不要往里写状态** |
| `${CLAUDE_PLUGIN_DATA}` | `~/.claude/plugins/data/<id>/` | 跨更新保留的 `node_modules`、缓存 |
| `${CLAUDE_PROJECT_DIR}` | 项目根目录 | 项目本地脚本和配置 |

```json
{ "type": "command", "command": "\"${CLAUDE_PLUGIN_ROOT}/scripts/format.sh\"" }
```

- shell 形式的 `command` 要给变量加双引号；exec 形式（带 `args`）不需要。
- `userConfig` 每项必需 `type`、`title`、`description`；`"sensitive": true` 会掩盖输入并存入安全存储。引用用 `${user_config.KEY}`，hook 进程里读 `CLAUDE_PLUGIN_OPTION_<KEY>`。
- shell 形式 hook 命令、monitor 命令、MCP `headersHelper` **拒绝** `${user_config.*}`。
- 配置对话框只在会话内 `/plugin` 安装或启用时出现；`--plugin-dir` 和 shell 安装都不弹，改用 `/plugin configure <plugin>@<marketplace>` 或安装时传 `--config KEY=VALUE`。

### 本地测试、依赖与 evals

| 加载方式 | 用法 | 说明 |
|---|---|---|
| `--plugin-dir` | `claude --plugin-dir ./my-first-plugin --plugin-dir ./other-plugin.zip` | 目录或 `.zip`，可重复 |
| `--plugin-url` | `claude --plugin-url https://example.com/my-first-plugin.zip` | 启动时下载，只指向你信任的存档 |
| `CLAUDE_CODE_PLUGIN_DIRS` | 环境变量，列绝对路径 | 无法加标志时使用（需 v2.1.280+） |
| `claude plugin init my-tool` | 写入 `~/.claude/skills/my-tool/` | 每个会话自动加载，id 为 `my-tool@skills-dir` |

改动没生效时按顺序检查：

1. shell：`claude plugin validate <path>`，加 `--strict` 把警告也当失败。
2. 会话：`/reload-plugins`，看 `Reloaded:` 行的计数。
3. 会话：`/plugin` 的 **Installed** 看发现了哪些组件，**Errors** 看什么没加载。
4. shell：`claude --plugin-dir ./my-plugin plugin list`，看 `Status: ✔ loaded`。

常见失败：

- `--plugin-dir` 指向市场根目录时不读 `marketplace.json`，`plugins/` 下的插件不加载且**没有报错**。
- 从 `.claude/` 迁移后忘了删原件：skill 和 agent 因前缀不冲突，但 **hook 没有前缀，两边都在时每次事件运行两次**。

依赖写在 `plugin.json`，裸名称在同一市场解析，带约束的写成对象：

```json
"dependencies": ["audit-logger", { "name": "secrets-vault", "version": "~2.1.0" }]
```

安装时依赖在同一作用域自动安装并启用；卸载后自动装上的依赖会保留，用 `claude plugin prune` 清理。版本范围按 `{name}--v{version}` 形式的 git 标签解析，被依赖方用 `claude plugin tag` 打标签。

#### 用 evals 检查插件是否真的改变了行为

`claude plugin eval` 对每个用例启动隔离的非交互会话，**加载插件跑一组、不加载再跑一组**，差值 `Δ` 才是插件的贡献。需 v2.1.269+。

```bash
claude plugin eval init        # 交互式：Claude 读插件、提议用例与评分器并写入 evals/
claude plugin eval .           # 运行 evals/ 下每个用例
```

- 摘要表的 `WITH`、`W/OUT` 是加载与不加载插件时的分数，`Δ` 为正表示插件提高了分数。
- 每次运行和每个评判都是**真实模型调用**，计入计划用量或 API 账单。
- 评分器 `regex`、`tool_used`、`tool_order`、`file_exists` 不花钱；`llm`、`baseline` 调用评判模型。
- 运行不会停下来要权限：`Bash`、`Write`、`Edit`、`WebFetch`、`WebSearch` 需用 `--allow-tools` 授予。授予 shell 需要沙箱后端，原生 Windows 没有。
- CI：`--trust-plugin`、`--json results.json`、`--threshold`、固定 `--model` 与 `--judge-model`、`--no-publish`、`--max-cost-usd`。退出码 `0` 通过，`1` 未达阈值或出错，`2` 部分运行。
- `Δ` 接近零且 `tool_used: Skill` 失败，说明 Claude 没选中你的 skill，先改 skill 的 `description`。

### 发布分发、成本度量与 CLI hints

| 路线 | 谁能装 | 用户自动拿到更新吗 |
|---|---|---|
| 不用市场：发目录或 `.zip` | 你发给的人 | 否 |
| 自己的市场：带 `.claude-plugin/marketplace.json` 的仓库 | 能访问仓库的人，可为私有 | 自动更新默认关闭 |
| Anthropic 目录 | 在 claude.ai 或 Cowork 中添加它的人，经账户同步到 Claude Code | 是，新版本发布后 |

发布前检查：

1. 选一个永久的 kebab-case 名称。改名等于换了一个插件，想换标签改 `displayName`。
2. 定版本策略：每次发布都提升 `version`，或省略它改用提交 SHA。**设了 `version` 又不改，用户收不到新提交。**
3. `claude plugin validate --strict ./your-plugin`，再从本地市场真实安装一遍。
4. 补齐 `description`、`author`、`homepage`、`repository` 和 `README.md`；有 eval 套件就跑一遍。

套餐限制：提交到 Anthropic 目录需要付费 claude.ai 计划（Pro、Max、Ultra 从自己账户提交；Team、Enterprise 由 Owner 提交）。`claude-plugins-official` 不通过目录门户接受提交。

度量：

- `claude plugin details formatter` 的 `Always-on` 是插件加到每个会话的 token，压低办法是缩短描述、拆分大插件。
- `/plugin` 的 **Not used recently**：至少 14 天且 10 个会话未使用的已启用市场插件。`/skill-doctor`、`/doctor` 也会标出未使用项。
- `/usage` 按插件归因用量份额，仅 Pro、Max、Ultra、Team、Enterprise 计划。
- 组织侧：OpenTelemetry 事件 `claude_code.plugin_installed`、`claude_code.plugin_loaded`、`claude_code.skill_activated`。非官方插件名默认被编辑为 `third-party` 或省略，设 `OTEL_LOG_TOOL_DETAILS=1` 才给真实名称。Analytics API 仅 Enterprise 计划。

CLI hints（仅适用于列在官方名称市场里的插件）：你的命令行工具检测到 `CLAUDECODE` 时，向 stderr 单独写一行标签，Claude Code 在模型看到之前删掉它，并向用户显示一次性安装提示：

```text
<claude-code-hint v="1" type="plugin" value="example-cli@claude-plugins-official" />
```

### Mods：在 Claude Code 内部运行的插件

Mod 是带有 **hooks 模块**的插件：一个 JavaScript 或 TypeScript 文件，其函数在事件发生时由 Claude Code 在自己的进程内调用。官方文档未把 mods 标为预览功能：终端 v2.1.287+、桌面应用 v2.1.286+ 默认开启，VS Code 扩展中绘制需 2.1.299+。

| 需求 | 选择 |
|---|---|
| 要窗格、输入框上方的条带、自定义命令或重写事件（四者中只有它能在界面中绘制） | Mod |
| 用已有脚本阻止、允许或记录事件 | 设置 hook |
| 反复粘贴同一套指令 | Skill |
| Claude 需要访问外部系统 | MCP 服务器 |

> Mod 不在沙箱中运行，以你的权限读写文件、启动进程、发网络请求，能看到并改写每个提示词和工具调用。只从信任的作者和市场安装。

#### 创建流程

最省事的办法是在交互式会话里描述需求，Claude 用内置的 `plugin-authoring` skill 编写到 `~/.claude/dev-mods/<会话 ID>/` 下，并询问是否启用热重载。手写则是三个文件：

```text
first-mod/
├── .claude-plugin/plugin.json
└── hooks/
    ├── hooks.json      # { "modules": ["./register.js"] }，有 modules 键才是 mod
    └── register.js     # hooks 模块
```

```javascript
let calls = 0

export function register(on) {
  on('tool.call', async ($, e, next) => {
    calls += 1
    $.ui.invalidate('ui.render')
    return next(e)
  })
  on('ui.render', { component: 'Spinner' }, async ($, e, next) => {
    return next({ ...e, props: { ...e.props, suffix: ' · tool calls: ' + calls + '…' } })
  })
}
```

```bash
claude --plugin-dir ./first-mod      # 加载，保存文件时热重载
claude plugin validate ./first-mod   # 不运行代码，列出 hooks: 与 calls: 行
claude plugin test                   # 运行 *.test.ts，无需会话、登录或网络
```

#### 事件、API 与绘制要点

| 概念 | 要点 |
|---|---|
| hook 的三个参数 | `$`（mods API）、`e`（事件输入，深度冻结）、`next`（下一个处理程序） |
| 三种处理方式 | 观察：`return next(e)`；重写：把改过的副本传给 `next`；回答：不调 `next` 直接返回结果 |
| 常用事件 | `session.start`、`tool.call`、`tool.check`、`command.run`、`ui.render`、`prompt.submit` |
| 绘制位置 | `Pane`（窗格，用 `$.ui.open({ id })` 打开）、`AbovePrompt`（输入框上方条带），以及 `Spinner` 等 Claude Code 自己绘制的位置 |
| 常用 API 命名空间 | `$.ui`、`$.command`、`$.tool`、`$.model`、`$.fs`、`$.process`、`$.http`、`$.store`、`$.state` |

- 静态分析规则（违反会让 `validate` 失败）：事件名必须是字符串字面量；每次调用完整写成 `$.命名空间.方法`，不能把 `$.ui` 赋给变量；只能用 `import` 相对导入插件目录内的文件。
- `claude -p` 和 Agent SDK 中 hook 会运行，但绘制内容不出现。单个 hook 自身执行超过 10 秒会被跳过。
- 事件和方法随版本变化，以 Claude Code 写入 mod 的 `.claude-plugin/types/` 下的类型声明为准。

#### 排错与开关

1. `claude plugin validate ./first-mod`：能抓出拼错的事件名（如 `"tool.calls" is not an event`）。通过但没有 `hooks:` 行，说明 `hooks/hooks.json` 缺 `modules` 键。
2. `/plugin` 选项卡下的暗色行应显示 `1 mod active · first-mod`。
3. 没列出时用 `claude --debug` 启动，在调试日志里找 `hooks module <name> not loaded:` 后面的原因。

关闭 mod：单个在 `/plugin` 里禁用其插件；本次会话全部关闭用 `--safe-mode`；长期关闭在 `~/.claude/settings.json` 设 `"disableAllHooks": true`（设置 hook 和自定义状态栏也会停）。这些都不影响内置 mod（如 `cc-plugin-diff`、`cc-plugin-agents-md`）。

#### 组织管理

- 内置守卫 `sec-default@builtin`（`/plugin` 中显示为 `cc-plugin-sec-default`）在机器有托管设置或 `managed-mcp.json`，或会话使用 Team、Enterprise 的 `/login` 凭据时加载，先于用户的每个 mod 运行。加载后用户的 mod 不能重写系统提示词和托管说明，被 `deny` 规则拒绝的调用仍被拒绝。
- 只允许组织自己的 mod，在托管设置里写：

```json
{ "pluginConfigs": { "cc-plugin-sec-default@builtin": { "options": { "allowManagedModsOnly": true } } } }
```

- `prependPlugins`、`appendPlugins` 决定组织 mod 相对用户 mod 的运行顺序。设置 `prependPlugins` 时要把 `sec-default@builtin` 一并列入，否则守卫被移除。

### Marketplace：创建、来源类型与托管

市场就是一个含 `.claude-plugin/marketplace.json` 的目录，加上它列出的插件。把插件复制到 `my-marketplace/plugins/` 下，再写 `my-marketplace/.claude-plugin/marketplace.json`：

```json
{
  "name": "my-marketplace",
  "owner": { "name": "Your Name" },
  "plugins": [
    { "name": "my-first-plugin", "source": "./plugins/my-first-plugin", "description": "A greeting plugin to learn the basics" }
  ]
}
```

```bash
claude plugin validate ./my-marketplace
claude plugin marketplace add ./my-marketplace
claude plugin install my-first-plugin@my-marketplace
claude plugin list                                  # 期望 Status: ✔ enabled
```

| 字段 | 位置 | 目的 |
|---|---|---|
| `name` / `owner` / `plugins` | 顶级，必需 | 市场标识符（用户在 `@` 之后输入）、维护者、插件条目数组 |
| `metadata.pluginRoot` | 顶级 | 裸插件源名称解析到的目录（需 v2.1.239+） |
| `renames` | 顶级 | 旧名到新名的映射，删除的插件映射为 `null`；只追加，不改旧条目 |
| `forceRemoveDeletedPlugins` | 顶级 | 为 `true` 时，从 `plugins` 删除的插件会在用户机器上卸载 |
| `name` / `source` | 条目，必需 | 安装 id 的前半段，以及从哪里获取 |
| `strict` | 条目 | 默认 `true`。`false` 时条目若再声明组件字段，与插件自带的 `plugin.json` 冲突而加载失败 |
| `relevance` | 条目 | 何时向用户建议该插件的信号 |

| 插件来源类型 | 字段 | 何时使用 |
|---|---|---|
| 相对路径 | 字符串本身，以 `./` 开头 | 插件文件在市场目录内 |
| `github` | `repo`、`ref`、`sha` | 插件是独立的 GitHub 仓库 |
| `url` | `url`、`ref`、`sha` | 任意主机的 git 仓库 |
| `git-subdir` | `url`、`path`、`ref`、`sha` | 其他仓库的子目录，如 monorepo |
| `npm` | `package`、`version`、`registry` | npm 包，不运行安装脚本 |
| `archive` | `url`、`sha256` | HTTPS 上的 zip |
| `command` | `command`、`timeout`、`mode` | 用户机器上的命令打印出插件目录；运行前需用户接受 |

常见失败：

- 相对路径从**市场根目录**（含 `.claude-plugin/` 的目录）解析。指向不存在目录的路径验证能过，安装时才报 `Source path does not exist: <path>`。
- 条目 `name` 与 `plugin.json` 的 `name` 不一致时，按清单名安装会报 `Plugin "<manifest-name>" not found in marketplace "<marketplace>"`。
- 用户以裸 `marketplace.json` URL 添加市场时只下载这一个文件，相对路径条目无法解析，改用对象来源或托管成 git 仓库。
- `name` 不能用官方或社区保留名称，也不能用 `inline`、`builtin`、`skills-dir`、`synced` 或以 `claudeai-` 开头。

托管与维护：

- 给一个仓库的所有人注册：`claude plugin marketplace add your-org/your-marketplace --scope project`，提交它写入的 `.claude/settings.json`。
- 不要在 `plugin.json` 和市场条目里同时设 `version`。固定版本用条目上的 `ref`、`sha`，或让用户添加 `your-org/your-marketplace#stable`。
- 为组织推荐插件：条目加 `relevance`（如 `"signals": { "cli": ["terraform"], "filesRead": ["**/*.tf"] }`），信号在本机匹配，命中后只做建议、**永不自动安装**。终端里要生效，管理员还得把市场名加入托管设置的 `pluginSuggestionMarketplaces`。

### 组织管理：用 managed settings 控制插件

常见做法：允许官方市场和自己的市场，注册两者，强制启用若干插件，并禁止旁加载。

```json
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "anthropics/claude-plugins-official" },
    { "source": "github", "repo": "your-org/*" },
    { "source": "skills-dir" }
  ],
  "extraKnownMarketplaces": {
    "claude-plugins-official": { "source": { "source": "github", "repo": "anthropics/claude-plugins-official" } },
    "your-marketplace": { "source": { "source": "github", "repo": "your-org/your-marketplace" } }
  },
  "enabledPlugins": { "code-formatter@your-marketplace": true },
  "disableSideloadFlags": true
}
```

| 键 | 它执行什么 | 它做不到什么 |
|---|---|---|
| `strictKnownMarketplaces` | 市场来源允许列表；`[]` 阻止一切来源，包括官方市场 | 不注册市场，不限制市场内的条目，不拦 `--plugin-dir` |
| `blockedMarketplaces` | 市场来源阻止列表，先于允许列表检查 | 不阻止已从不匹配来源注册的市场 |
| `extraKnownMarketplaces` | 在每台机器注册市场，可带 `autoUpdate` | 自身也必须通过允许列表 |
| `enabledPlugins` | `true` 强制启用；`false` 在每个作用域阻止并隐藏 | 不安装市场未注册或不被允许的插件 |
| `disableSideloadFlags` | 启动时拒绝 `--plugin-dir`、`--plugin-url`、`--agents`、`CLAUDE_CODE_PLUGIN_DIRS` 等 | 不限制 `.mcp.json`、`claude mcp add` |
| `disableCommandPluginSources` | 阻止 `command` 来源的插件安装、更新或加载 | 不影响其他来源类型 |
| `syncClaudeAiPlugins` | 设 `false` 停止加载从 claude.ai 同步的插件 | 不能只关某一个同步插件 |
| `strictPluginOnlyCustomization` | 阻止来自用户和项目来源的 skill、agent、hook、MCP 服务器 | 不限制用户装哪些插件 |
| `pluginTrustMessage` | 在安装前的信任警告后追加组织文本 | 不改变警告原文 |

- **允许列表不会注册任何东西**，注册靠 `extraKnownMarketplaces`，两者要配套写。
- 允许列表里漏掉 `{ "source": "skills-dir" }`，`~/.claude/skills/` 和项目 `.claude/skills/` 下带清单的插件会停止加载。
- 允许列表匹配很严格：没有 `ref` 的条目不覆盖带 `ref: "main"` 的来源；同一仓库的 `owner/repo` 与 git URL 是不同的值，这类主机改用 `hostPattern`。
- CI 和容器：构建时用 `CLAUDE_CODE_PLUGIN_CACHE_DIR` 把市场和插件装进种子目录，运行时用 `CLAUDE_CODE_PLUGIN_SEED_DIR` 指向它；种子里的插件仍需 `enabledPlugins` 才启用。`-p` 运行里插件在后台安装，设 `CLAUDE_CODE_SYNC_PLUGIN_INSTALL=1` 让第一轮之前等它装完。
- 关闭全机队的插件自动更新：托管 `env` 中设 `DISABLE_AUTOUPDATER`（Claude Code 自身更新也会停）。

验证：机器上运行 `/status`，`Setting sources` 行应包含 `Enterprise managed settings`，再用 `/plugin` 看市场和插件是否列出。

### 加载规则与排错

一个插件要能用，必须同时存在于三层：**设置**里已声明（`enabledPlugins`、`extraKnownMarketplaces`）→ **磁盘**上已获取（`~/.claude/plugins/` 下的 `installed_plugins.json` 和 `cache/`）→ **会话**已加载（启动时或 `/reload-plugins` 时）。

| id 结尾 | 怎么来的 |
|---|---|
| `@<marketplace>` | 从你添加的市场安装 |
| `@inline` | `--plugin-dir`、`--plugin-url`、`CLAUDE_CODE_PLUGIN_DIRS` 或 Agent SDK 的 `plugins` 选项，仅本次会话 |
| `@skills-dir` | `~/.claude/skills/` 或项目 `.claude/skills/` 下带 `.claude-plugin/plugin.json` 的目录 |
| `@synced` | 从 claude.ai 账户同步 |

- 同名冲突的优先顺序：托管设置 `enabledPlugins` 命名的插件 → `--plugin-dir` 等会话插件 → 已安装的市场插件 → skills 目录插件 → claude.ai 同步插件。
- 版本计算顺序：`plugin.json` 的 `version` → 市场条目的 `version` → 来源自身（git 来源取提交 SHA 的前 12 位）。版本没变，磁盘上的副本就不会被替换。
- 多数市场插件安装时被**复制**到 `cache/<marketplace>/<plugin>/<version>/`，插件目录之外的文件不会跟过去，脚本里读 `../shared` 会找不到。

排错顺序：先看 `/plugin` 的 **Installed**（是否已列出并启用），再看 **Errors**（大多数问题在这里有消息和指导），本次会话中安装的再运行 `/reload-plugins`（提示会使提示缓存失效时加 `--force`）。

| 现象或消息 | 原因与处理 |
|---|---|
| `/plugin isn't available in this environment` | 不在交互式终端会话中，改用所在界面的入口或 shell 的 `claude plugin install` |
| `Marketplace "claude-plugins-official" not found` | 官方市场未注册，运行 `/plugin marketplace add anthropics/claude-plugins-official` |
| 以 `isn't installed here` 结尾的行 | 项目设置启用了但本机没装，运行 `claude plugin install <name>@<marketplace> --scope project` |
| `Disabled in ~/.claude/settings.json but still loads` | 更高优先级的来源（如托管设置）重新启用了它 |
| `<component> path not found: <path>` | 清单里的组件路径不存在，修路径后 `/reload-plugins` |
| 插件加载了但 skill 缺失 | `skills/` 被放进了 `.claude-plugin/`，或清单 `skills` 条目指向了文件 |
| `is blocked by enterprise policy` | 组织的允许或阻止列表拦截，联系管理员 |
| 更新后显示 `is already at the latest version` | 作者固定了 `version` 而没有提升 |

### 速查：命令一览

`claude plugin ...` 在 shell 中运行（`claude plugins` 是别名）。`<plugin>` 写作 `name` 或 `name@marketplace`；`--scope` 取 `user`、`project`、`local`。

| shell 命令 | 作用 | 会话内对应 |
|---|---|---|
| `claude plugin install <plugin>` | 安装；`--scope`、`--config`、`--marketplace <source>`、`-y` | `/plugin install <plugin>`（打开详情） |
| `claude plugin uninstall <plugin>` | 卸载；`--keep-data` 保留数据目录，`--prune` 连同孤立依赖 | `/plugin uninstall <plugin>` |
| `claude plugin enable` / `disable <plugin>` | 启用、停用 | `/plugin enable`、`/plugin disable` |
| `claude plugin update <plugin>` | 更新到市场提供的最新版本 | 无，面板里用 **Update now** |
| `claude plugin list` | 列出已安装插件；`--json`、`--available` | `/plugin list` |
| `claude plugin details <name>` | 组件清单与预计 token 开销 | 无 |
| `claude plugin validate <path>` | 验证插件或市场；`--strict`、`--json` | `/plugin validate <path>` |
| `claude plugin init <name>` | 在 `~/.claude/skills/<name>/` 搭脚手架；`--with skills hooks` | 无 |
| `claude plugin tag [path]` | 创建 `<name>--v<version>` 发布标签；`--push`、`--dry-run` | `/plugin tag` |
| `claude plugin prune` | 移除不再需要的自动安装依赖；`--dry-run` | 无 |
| `claude plugin eval [target]` / `eval init` | 运行或创建 eval 套件 | 无 |
| `claude plugin test [directory]` | 运行 mod 的 `.test.ts` 测试 | 无 |
| `claude plugin marketplace add <source>` | 添加市场；`--scope`、`--sparse`、`--claudeai` | `/plugin marketplace add [source]` |
| `claude plugin marketplace list` / `update [name]` / `remove <name>` | 列出、刷新、移除市场 | `/plugin marketplace list` / `update` / `remove` |
| `claude --plugin-dir <path>` / `--plugin-url <url>` | 本次会话加载目录、`.zip` 或 URL 上的存档 | `/reload-plugins [--force]` 应用磁盘上的变更 |

---

## Part IV：管理

### 管理篇导读与部署选型

这一篇回答四个问题：Claude Code 从哪里拿模型、策略怎么下发到每台机器、花了多少钱、数据去了哪里。

- **个人开发者**只需看：“安装、更新与卸载”“认证与凭证优先级”“成本管理”“数据使用”。
- **团队管理员**按顺序看：部署选型 → 托管设置 → 托管 MCP → 云厂商或网关 → 监控与成本 → 数据与合规。
- **网络与安全团队**重点看：“企业网络”“企业启动器与开发容器”“网关”“ZDR 与 HIPAA”。

| 方式 | 认证 | 关键环境变量或设置 | 适合谁 |
|---|---|---|---|
| Claude 订阅（Pro、Max、Ultra、Team、Enterprise） | `/login` 浏览器登录 | CI 用 `CLAUDE_CODE_OAUTH_TOKEN` | 个人与大多数团队；Team/Enterprise 是官方默认建议 |
| Claude Console（API） | API 密钥或 Console 无密钥登录 | `ANTHROPIC_API_KEY` | API 优先、按用量付费 |
| Amazon Bedrock | AWS 凭证链或 Bedrock API 密钥 | `CLAUDE_CODE_USE_BEDROCK`、`AWS_REGION` | 继承现有 AWS 合规与计费 |
| Claude Platform on AWS | SigV4 或工作区 API 密钥 | `CLAUDE_CODE_USE_ANTHROPIC_AWS`、`ANTHROPIC_AWS_WORKSPACE_ID` | AWS Marketplace 计费加 Claude API 功能 |
| Google Cloud（Anthropic 管理） | Google 应用默认凭据 | `CLAUDE_CODE_USE_ANTHROPIC_GOOGLE_CLOUD` 等三个变量 | 经 GCP 项目访问 Anthropic 运营的 API |
| Google Cloud's Agent Platform（原 Vertex AI） | GCP 凭证 | `CLAUDE_CODE_USE_VERTEX`、`CLOUD_ML_REGION`、`ANTHROPIC_VERTEX_PROJECT_ID` | GCP 原生部署 |
| Microsoft Foundry | API 密钥、Entra ID 或 Bearer 令牌 | `CLAUDE_CODE_USE_FOUNDRY`、`ANTHROPIC_FOUNDRY_RESOURCE` | Azure 原生部署 |
| Claude apps gateway（自托管） | 企业 IdP（OIDC）经 `/login` | 托管设置 `forceLoginMethod`、`forceLoginGatewayUrl` | 必须走自有云，又要 SSO、按组策略和支出限制 |
| 其他 LLM 网关 | 网关颁发的凭证 | `ANTHROPIC_BASE_URL` 加 `ANTHROPIC_AUTH_TOKEN` | 组织已在运行网关 |

> 边界提醒：云端会话、云端定时任务、Code Review、Remote Control、Chrome 扩展都需要 claude.ai 账户，仅凭 Console API 密钥或云厂商凭据无法使用。SSO、SCIM、座位分配在 Claude 账户层配置。

### 安装、更新与卸载

系统要求：macOS 13.0+、Windows 10 1809+ 或 Windows Server 2019+、Ubuntu 20.04+、Debian 10+、Alpine Linux 3.19+；4 GB+ RAM，x64 或 ARM64；需要联网。账户需为 Pro、Max、Ultra、Team、Enterprise 或 Console，免费 claude.ai 计划不含 Claude Code。

```bash
# 原生安装（推荐，后台自动更新）：macOS、Linux、WSL
curl -fsSL https://claude.ai/install.sh | bash
# 指定 stable 渠道
curl -fsSL https://claude.ai/install.sh | bash -s stable
# Homebrew：claude-code 跟踪 stable，claude-code@latest 跟踪 latest
brew install --cask claude-code
# npm：需要 Node.js 22 或更高版本，不要用 sudo
npm install -g @anthropic-ai/claude-code@latest
```

```powershell
irm https://claude.ai/install.ps1 | iex      # Windows PowerShell，无需管理员
winget install Anthropic.ClaudeCode          # WinGet
```

安装后用 `claude --version` 确认，用 `claude doctor` 看只读诊断。原生 Windows 可选装 Git for Windows，没有时改用 PowerShell 工具；沙箱只在 WSL 2 支持。Alpine 要先 `apk add bash curl`。

| 设置或变量 | 目的 |
|---|---|
| `autoUpdatesChannel` | `"latest"`（默认）或 `"stable"`（约晚一周，跳过有重大回归的版本） |
| `minimumVersion` | 更新下限；切到 stable 时防止被降级 |
| `requiredMinimumVersion`、`requiredMaximumVersion` | 仅托管设置；版本超出范围直接拒绝启动 |
| `DISABLE_AUTOUPDATER` | 只停后台检查，`claude update` 仍可用 |
| `DISABLE_UPDATES` | 阻止所有更新路径，适合自行分发二进制的组织 |

Homebrew、WinGet、apt、dnf、apk 默认不自动更新。手动更新用 `claude update`；npm 升级重跑安装命令，不要用 `npm update -g`。

```bash
# 卸载原生安装
rm -f ~/.local/bin/claude && rm -rf ~/.local/share/claude
# 可选：删除设置、MCP 配置和会话历史（不可恢复）
rm -rf ~/.claude && rm ~/.claude.json
```

其他方式对应 `brew uninstall --cask claude-code`、`winget uninstall Anthropic.ClaudeCode`、`npm uninstall -g @anthropic-ai/claude-code`。VS Code 扩展、JetBrains 插件和桌面应用也会写 `~/.claude/`，要彻底清除需先卸载它们。

### 认证与凭证优先级

首次运行 `claude` 会打开浏览器登录；没弹出时按 `c` 复制登录 URL。切换账户用 `/logout` 再 `/login`，当前凭证用 `/status` 查看。

```text
同时存在多个凭证时，按此顺序取第一个
1. 云提供商凭证            CLAUDE_CODE_USE_BEDROCK / CLAUDE_CODE_USE_VERTEX / CLAUDE_CODE_USE_FOUNDRY
2. ANTHROPIC_AUTH_TOKEN    作为 Authorization: Bearer 发送（网关常用）
3. ANTHROPIC_API_KEY       作为 X-Api-Key 发送；交互模式需批准一次，-p 模式始终使用
4. apiKeyHelper            脚本输出，适合轮换或保管库凭证
5. CLAUDE_CODE_OAUTH_TOKEN claude setup-token 生成的一年期令牌
6. Anthropic 配置文件与联合凭证（ANTHROPIC_PROFILE、WIF）
7. /login 的订阅 OAuth     订阅用户的默认方式
列表之外：已登录的 Claude apps gateway 会话优先于以上全部
```

- 有订阅又设了 `ANTHROPIC_API_KEY`，批准后走 API 计费；运行 `unset ANTHROPIC_API_KEY` 回到订阅。
- `claude setup-token` 只打印令牌、不保存；该令牌只能做模型请求，`--bare` 模式不读取它。
- 凭证位置：macOS 在 Keychain；Linux 在 `~/.claude/.credentials.json`（模式 `0600`）；Windows 在 `%USERPROFILE%\.claude\.credentials.json`。
- 多账户并存：给每个账户一个 `CLAUDE_CONFIG_DIR`。

团队把登录限制在本组织（放进托管设置）：

```json
{
  "forceLoginMethod": "claudeai",
  "forceLoginOrgUUID": ["<your-org-uuid>"],
  "allowedProviders": ["anthropic", "bedrock"]
}
```

| 键 | 目的 | 边界 |
|---|---|---|
| `forceLoginMethod` | 限定登录方式：`claudeai`、`console`、`gateway` | 终端交互式登录界面只是预选，不能靠它阻止其他登录 |
| `forceLoginOrgUUID` | claude.ai 登录必须属于所列组织 | 不检查已保存的 Console 凭据属于哪个组织 |
| `allowedProviders` | 限制机器可用的 API 提供商 | 需要 v2.1.285 或更高版本 |

设置前两个键的任意一个后，用 `ANTHROPIC_API_KEY`、`ANTHROPIC_AUTH_TOKEN` 或 `apiKeyHelper` 认证的会话会在启动时被阻止。这两个键要靠设备管理工具下发：开发者先通过组织认证才会收到服务器托管设置，所以无法用后者重定向首次登录。

### 托管设置：团队如何下发策略

托管设置（managed settings）优先于用户、项目、本地和 `--settings` 的值，只有少数安全敏感的例外允许更低层级的更严格值生效。

| 机制 | 位置 | 读取时机 | 何时使用 |
|---|---|---|---|
| 服务器托管 | claude.ai 管理控制台，或自托管 Claude apps gateway | 启动时获取，每小时轮询 | 想在一处改策略、不碰每台机器 |
| MDM 或操作系统策略 | macOS `com.anthropic.claudecode` 首选项域；Windows `HKLM\SOFTWARE\Policies\ClaudeCode` | 启动时读取，每 30 分钟检查 | 已用 Jamf、Intune、组策略管理设备 |
| 文件 | `managed-settings.json` 与可选的 `managed-settings.d/` | 启动时读取，文件变更时重载 | 没有 MDM 的机器、Linux 主机、自建镜像 |
| HKCU 注册表 | `HKCU\SOFTWARE\Policies\ClaudeCode` | 同 MDM | 用户可写，只是便利默认值，不是强制通道 |

文件路径：macOS `/Library/Application Support/ClaudeCode/managed-settings.json`；Linux 和 WSL `/etc/claude-code/managed-settings.json`；Windows `C:\Program Files\ClaudeCode\managed-settings.json`。

操作流程：

1. 编写 `managed-settings.json`，形状与 `settings.json` 相同。
2. 用现有的设备管理工具放到上面的系统目录。
3. 在一台机器上运行 `/status`，`Setting sources` 行应显示 `Enterprise managed settings (file)`。
4. 确认后再推到全部设备；条目被丢弃时用 `claude doctor` 查原因。

```json
{
  "permissions": {
    "deny": ["Read(./.env)", "Read(./secrets/**)"],
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true
}
```

```text
多个来源的优先级（高 → 低）
1. 远程设置（claude.ai 服务器托管 / 网关）
2. MDM：macOS plist 或 HKLM 注册表
3. 文件：managed-settings.d/*.json 与 managed-settings.json 合并
4. HKCU 注册表（不算管理员源）

managedSourcesBehavior
├─ "first-wins"（默认）：只用提供了策略键的最高排名源
└─ "merge"（v2.1.242+）：每个管理员源都生效，列表合并，锁取最严格值
```

- 数组类设置（如 `permissions.allow`）会合并更低层级的条目，开发者能加不能删；要彻底锁住，配上对应的 `allowManaged*Only` 键。
- 托管的 `model` 只是默认值，不是锁；限制模型用 `availableModels`。
- WSL 默认只读 `/etc/claude-code`；要继承 Windows 策略，在 HKLM 或 `C:\Program Files\ClaudeCode` 的文件里设 `wslInheritsWindowsSettings: true`。
- 常用强制项：`sandbox.enabled`、`sandbox.network.allowedDomains`、`strictKnownMarketplaces`、`allowManagedHooksOnly`、`strictPluginOnlyCustomization`、`maxEffortLevel`，以及 `env` 里的 `DISABLE_TELEMETRY`。
- 权限规则和沙箱是两层：拒绝 WebFetch 后，只要允许 Bash，`curl` 仍能访问任意 URL；这个缺口要靠沙箱的网络域名允许列表补上。

#### 服务器托管设置（server-managed settings）

在 claude.ai 的 **Organization settings > Claude Code > Managed settings** 填 JSON，客户端自动获取。仅 Claude for Teams 和 Enterprise 可用（按组下发需要 Enterprise），由 Primary Owner 或 Owner 管理，需要直连 `api.anthropic.com`。

- 它是客户端控制，不是安全边界；设备已纳入 MDM 时，端点托管设置的保证更强。
- shell 里导出了 `CLAUDE_CODE_USE_*` 提供商变量或非默认 `ANTHROPIC_BASE_URL`，会话会跳过获取；云厂商部署要改用文件、MDM 或 Claude apps gateway。
- Console API 密钥属于 Console 组织，与 claude.ai 组织相互独立，用它的 CI 作业收不到 claude.ai 上的设置。
- 获取失败时默认用缓存继续；设 `forceRemoteSettingsRefresh: true` 改为失败即退出。
- Hook、自定义 `env` 变量、`apiKeyHelper` 等设置会在交互式会话里弹出安全批准对话框。
- 云端会话读不到设备上的 MDM 或文件，策略只能来自这里；Cowork 会话则从不获取它。

### 托管 MCP：allow 与 deny

默认任何人都能连接任意 MCP 服务器，Anthropic 不对 MCP 服务器做安全审计。三种机制可以组合：

| 模式 | 效果 | 配置 |
|---|---|---|
| 禁用或固定部署 | 只加载你列出的服务器（空映射即禁用），用户自己的不加载 | 系统目录下的 `managed-mcp.json` |
| 提供的服务器 | 每个用户多得一组远程服务器，自己的保留 | 托管设置 `managedMcpServers`（v2.1.259+） |
| 批准的目录 | 只有允许列表内的能用 | `allowedMcpServers` 加 `allowManagedMcpServersOnly: true` |
| 软允许列表 | 用户可在自己的设置里扩展 | 只设 `allowedMcpServers` |
| 仅拒绝列表 | 阻止已知的坏服务器 | `deniedMcpServers` |

```json
{
  "allowManagedMcpServersOnly": true,
  "allowedMcpServers": [
    { "serverUrl": "https://*.<your-internal-domain>/*" },
    { "serverCommand": ["npx", "-y", "<approved-package>"] }
  ],
  "deniedMcpServers": [
    { "serverName": "<blocked-server-label>" }
  ]
}
```

评估顺序：先合并各作用域的列表（锁打开时只保留托管允许列表，拒绝列表始终全部合并）→ 查拒绝列表，匹配即阻止且无法覆盖 → 查允许列表，远程服务器匹配 `serverUrl`，stdio 服务器匹配 `serverCommand`。

- `allowedMcpServers` 未设置表示全部允许，设为 `[]` 表示全部不允许。
- `serverName` 只是用户起的标签，**不是安全控制**；要管住实际运行的服务器，用 `serverUrl` 或 `serverCommand`。
- `managed-mcp.json` 机器上任何用户都能读，不要在 `env` 里放密钥；它不能通过服务器托管设置下发。
- `managedMcpServers` 的条目只接受 `http` 或 `sse` 类型和 `https://` URL，不展开 `${VAR}`。
- `allowManagedMcpServersOnly` 与 `allowManagedPermissionRulesOnly` 是两把不同的锁。

### 自动模式配置（autoMode）

自动模式把工具调用交给分类器审查，阻止不可逆、破坏性或指向环境之外的操作。分类器默认只信任工作目录和当前仓库的远程；推送到公司的源码组织、写团队存储桶，都要先加进 `autoMode.environment`。

| 字段 | 目的 |
|---|---|
| `autoMode.environment` | 声明受信的仓库、存储桶、域和服务；多数组织只需要这一项 |
| `autoMode.allow`、`autoMode.soft_deny` | 软阻止规则及其例外；明确的用户意图可以解除软阻止 |
| `autoMode.hard_deny` | 无条件阻止，用户要求也不行 |
| `autoMode.classifyAllShell` | 设为 `true` 时暂停 Bash 和 PowerShell 允许规则，全部交给分类器 |

```json
{
  "autoMode": {
    "environment": ["$defaults", "Source control: <your-git-host>/<your-org> and all repos under it"],
    "hard_deny": ["$defaults", "Never send repository contents to third-party code-review APIs"]
  }
}
```

- 每个你设置的列表都要包含字面字符串 `"$defaults"`，否则会丢掉内置规则。
- 只从 `~/.claude/settings.json`、托管设置、`--settings` 或 Agent SDK 读取，**不读**项目的 `.claude/settings.json` 和 `.claude/settings.local.json`。
- 各范围的条目相加，开发者的 `allow` 能覆盖组织的 `soft_deny`，所以它不是硬边界；硬阻止用托管设置里的 `permissions.deny`，推送前的人工确认用 `permissions.ask`。
- 内置规则用 `claude auto-mode defaults` 查看；被拒操作在 `/permissions` 的 **Recently blocked** 选项卡里。

### 云厂商接入

通用规则：给团队部署前先用 `ANTHROPIC_DEFAULT_OPUS_MODEL`、`ANTHROPIC_DEFAULT_SONNET_MODEL`、`ANTHROPIC_DEFAULT_HAIKU_MODEL` 固定模型版本；`/logout` 在云厂商认证下不可用；用 `/status` 确认实际走的提供商。Bedrock 和 Agent Platform 可在登录提示处选 **3rd-party platform** 走交互式向导。

```bash
# Amazon Bedrock
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1          # 配置文件已带区域时可省略

# Claude Platform on AWS
export CLAUDE_CODE_USE_ANTHROPIC_AWS=1
export ANTHROPIC_AWS_WORKSPACE_ID=<your-workspace-id>
export AWS_REGION=us-east-1

# Google Cloud's Agent Platform（原 Vertex AI）
export CLAUDE_CODE_USE_VERTEX=1
export CLOUD_ML_REGION=global
export ANTHROPIC_VERTEX_PROJECT_ID=<your-project-id>

# Microsoft Foundry
export CLAUDE_CODE_USE_FOUNDRY=1
export ANTHROPIC_FOUNDRY_RESOURCE=<your-resource-name>
export ANTHROPIC_FOUNDRY_API_KEY=<your-azure-api-key>   # 用 Entra ID 时不要设置
```

Google Cloud 上的 Claude 模型（由 Anthropic 管理）官方示例写在 `~/.claude/settings.json`：

```json
{
  "env": {
    "CLAUDE_CODE_USE_ANTHROPIC_GOOGLE_CLOUD": "1",
    "ANTHROPIC_GOOGLE_CLOUD_PROJECT": "<your-project-id>",
    "ANTHROPIC_GOOGLE_CLOUD_WORKSPACE_ID": "<your-workspace-id>"
  }
}
```

| 提供商 | 关键注意点 |
|---|---|
| Amazon Bedrock | 默认 AWS SDK 凭据链，或 `AWS_BEARER_TOKEN_BEDROCK`；SSO 刷新用 `awsAuthRefresh` 设置；首次要在控制台提交 Anthropic 用例表单；WebSearch 不可用；自定义端点用 `ANTHROPIC_BEDROCK_BASE_URL`，Mantle 端点用 `CLAUDE_CODE_USE_MANTLE` |
| Claude Platform on AWS | 请求直达 Anthropic 运营的 API；认证用 SigV4 或 `ANTHROPIC_AWS_API_KEY`；`ANTHROPIC_AWS_WORKSPACE_ID` 必填；Marketplace 订阅会新建独立的 Anthropic 组织；Bedrock 和 Foundry 在路由中优先，需先取消它们的变量 |
| Google Cloud（Anthropic 管理） | 需要 v2.1.307 或更高版本；先 `gcloud auth application-default login`，账号需 `roles/claude.user`；不支持快速模式、Remote Control、云端会话、服务器托管设置 |
| Agent Platform | 需启用 API 并在 Model Garden 申请模型；IAM 角色 `roles/aiplatform.user`；`CLOUD_ML_REGION` 未设时回退 `us-east5`，个别模型用 `VERTEX_REGION_CLAUDE_*` 单独指定；刷新凭据用 `gcpAuthRefresh` |
| Microsoft Foundry | 三种认证：`ANTHROPIC_FOUNDRY_API_KEY`、Entra ID（先 `az login`）、`ANTHROPIC_FOUNDRY_AUTH_TOKEN`（优先级最高）；可用 `ANTHROPIC_FOUNDRY_BASE_URL` 代替资源名；没有设置向导；模型变量填的是**部署名称** |

#### 功能可用性差异

CLI 与所有本地功能（Agent SDK、IDE 扩展、子代理、hooks、skills、插件、MCP、沙箱、OpenTelemetry、托管设置文件）在每个提供商上一致，差异集中在依赖服务器端或 claude.ai 账户的功能。

| 功能 | 可用范围 |
|---|---|
| 云端会话、`/schedule`、Remote Control、Chrome 扩展、Claude Code Desktop、语音听写 | 仅 claude.ai 账户登录 |
| Code Review、分析仪表板、服务器托管设置、SSO | Team 和 Enterprise |
| SCIM、Compliance API、Enterprise Analytics API | 仅 Enterprise |
| Web search | Bedrock 不可用 |
| Advisor | Bedrock、Agent Platform、Foundry 不可用 |
| 自动模式 | Bedrock、Agent Platform、Foundry 上仅部分较新模型支持 |

云厂商上的替代做法：定时用 `/loop` 代替 `/schedule`，云端会话用 GitHub Actions 或 GitLab CI/CD，网络查询用带具体 URL 的 WebFetch。

### 企业网络：代理、CA 与 mTLS

| 环境变量 | 目的 |
|---|---|
| `HTTPS_PROXY`、`HTTP_PROXY`、`NO_PROXY` | HTTP/HTTPS 代理与绕过列表；不支持 SOCKS |
| `NODE_EXTRA_CA_CERTS` | 自定义 CA 证书的 PEM 路径 |
| `CLAUDE_CODE_CERT_STORE` | 证书来源 `bundled`、`system`，默认 `bundled,system` |
| `CLAUDE_CODE_CLIENT_CERT`、`CLAUDE_CODE_CLIENT_KEY`、`CLAUDE_CODE_CLIENT_KEY_PASSPHRASE` | mTLS 客户端证书、私钥与口令 |

```bash
export HTTPS_PROXY=http://<proxy-host>:<port>
export NODE_EXTRA_CA_CERTS=/path/to/ca-cert.pem
export CLAUDE_CODE_CLIENT_CERT=/path/to/client-cert.pem
export CLAUDE_CODE_CLIENT_KEY=/path/to/client-key.pem
```

- **后台代理不继承当前 shell**：要让每个后台会话都生效，把变量写进 `~/.claude/settings.json` 的 `env` 块或托管设置。
- 代理值写完整 URL。NTLM、Kerberos 这类高级认证建议改用 LLM 网关。用 `claude --debug` 确认配置已加载。
- 防火墙至少放行 `api.anthropic.com`、`claude.ai`、`claude.com`、`platform.claude.com`、`downloads.claude.ai`，用 SSO 的组织加 `sso.claude.com`；完整清单见官方页面。
- 走云厂商或网关时，WebFetch 的域名安全检查仍会调用 `api.anthropic.com`，可用 `skipWebFetchPreflight: true` 关闭。

### 企业启动器与开发容器

#### 企业启动器（corporate launcher）

在 `PATH` 上包一层 `claude` 管不到后台服务和 agent view 会话，因为它们从二进制的直接路径启动。`CLAUDE_CODE_PROCESS_WRAPPER`（v2.1.208+）或等效的 `processWrapper` 设置（v2.1.210+）可以覆盖这些进程。

```json
{
  "env": { "CLAUDE_CODE_PROCESS_WRAPPER": "/opt/corp/launcher" }
}
```

1. 在绝对路径创建可执行脚本，做完组织自己的准备后必须以 `exec "$@"` 结尾。
2. 把变量写进 `~/.claude/settings.json` 或托管设置；shell `export` 不够。
3. 运行 `claude daemon stop --any`，再重启已打开的会话。
4. 用 `/status` 的 Self-exec 条目或 `claude daemon status` 验证。

契约要点：不改参数，原样传递继承的环境变量，约三秒内到达 `exec`，之前不写标准输出、不读标准输入。Windows 上该变量被忽略；项目和本地设置无法配置启动器。

#### 开发容器（devcontainer）

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": { "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {} },
  "mounts": ["source=claude-code-config,target=/home/node/.claude,type=volume"],
  "containerEnv": { "CLAUDE_CONFIG_DIR": "/home/node/.claude" }
}
```

- `:1.0` 固定的是 Feature 的安装脚本，不是 Claude Code 版本。命名卷加 `CLAUDE_CONFIG_DIR` 让登录在重建后保留。
- 组织策略：在 Dockerfile 里把文件复制到 `/etc/claude-code/managed-settings.json`；有仓库写权限的人能改掉这一步，绕不过去的策略要靠服务器托管设置或 MDM。
- 云凭证通过 `containerEnv`、Codespaces 密钥或工作负载身份传入，不要挂载主机的 `~/.ssh` 或云凭证文件。
- 非 root 用户可用 `--dangerously-skip-permissions`，但它挡不住恶意项目带走容器内可访问的内容（包括 `~/.claude` 里的凭证），应配合出站流量限制。

### 网关：Claude apps gateway

网关是组织放在 Claude Code 与模型提供商之间的代理：开发者只持有网关凭证，提供商凭证留在服务端。

```text
开发者机器                    你的基础设施                          模型提供商
Claude Code ──开发者凭证──▶  网关（认证/用量/预算/路由） ──组织凭证──▶ Anthropic API / Bedrock / Google Cloud / Foundry
```

Claude apps gateway 是 Anthropic 的自托管网关，内置在 `claude` 二进制里，用 `claude gateway --config gateway.yaml` 启动，随每个版本一起测试，不需要维护 beta 标头允许列表。

| 前提 | 说明 |
|---|---|
| 版本 | 网关服务器和每台开发者机器都要 v2.1.195 或更高版本 |
| 身份提供商 | 任何符合 OIDC 的 IdP；不支持 SAML 和 LDAP |
| 存储 | PostgreSQL 11 或更高版本 |
| 网络 | HTTPS，且主机名只能解析为私有地址 |
| 运行平台 | 服务器仅 Linux；macOS 仅用于本地开发 |

配置：`gateway.yaml` 最小有五个部分 `listen`、`oidc`、`session`、`store`、`upstreams`，机密通过 `${ENV_VAR}` 展开读取；可选 `managed`（按 IdP 组下发模型允许列表与托管设置）、`telemetry`、`admin`。开发者一侧把三个键放进每台机器的托管设置：

```json
{
  "forceLoginMethod": "gateway",
  "forceLoginGatewayUrl": "https://<gateway-host>",
  "parentSettingsBehavior": "merge"
}
```

- 开发者运行 `/login` 按 Enter 即可，首次连接会提示确认 TLS 证书指纹。要防绕过，再加 `"allowedProviders": ["gateway"]`。
- **没有服务令牌流程**：登录始终是浏览器设备流程，CI 作业要直接对接提供商。
- 网关不可达时，已登录的会话约 10 秒后报错退出，而不是无策略启动。
- 网关会话不可用：服务器端网络搜索、Remote Control、`/design-sync`、依赖功能标志获取的功能；也不符合 HIPAA 配置的条件。

支出限制：配置 `admin:` 块后，网关在 `/v1/organizations/spend_limits` 提供 admin API 并实时执行上限。

| 字段 | 取值 |
|---|---|
| `scope.type` | `user`、`rbac_group`、`organization` |
| `amount` | 美分的整数字符串；`null` 为无限制，`"0"` 阻止所有请求 |
| `period` | `daily`、`weekly`、`monthly`，按 UTC 日历边界重置 |

组或组织上限是每个成员继承的按座位默认值，不是共享池；超限返回 `429`。Postgres 不可达时默认失败打开，设 `enforcement.fail_closed_on_error: true` 改为失败关闭。金额是估算值，是断路器而不是发票。

部署：作为无状态容器跑在 Kubernetes、Cloud Run 或自有平台，健康探针是 `GET /healthz` 和 `GET /readyz`。官方的 AWS 与 Google Cloud 两份示例均标注为工作示例，不是受支持的生产部署。

### 网关：其他 LLM 网关

组织已有网关时可以直接用。Anthropic 不认可、维护或审计第三方网关产品，也不支持经任何网关把 Claude Code 路由到非 Claude 模型。

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://<llm-gateway-host>",
    "ANTHROPIC_AUTH_TOKEN": "<gateway-issued-key>"
  }
}
```

| 凭证放在 | 发送的标头 | 何时用 |
|---|---|---|
| `ANTHROPIC_AUTH_TOKEN` | `Authorization: Bearer` | 网关要 bearer token；不确定时先用它 |
| `ANTHROPIC_API_KEY` | `x-api-key` | 网关要 API key |
| `apiKeyHelper` | 两者都发 | 凭证轮换或来自保管库 |

- 放错变量会得到 `401`，换另一个再试。凭证不要写进会被提交的项目 `.claude/settings.json`。
- 只设 `ANTHROPIC_BASE_URL` 而不设网关凭证，生效的仍是 claude.ai 登录，订阅的限额和计费照常适用。
- 网关前置云厂商时改用提供商变量，例如 `ANTHROPIC_BEDROCK_BASE_URL` 配 `CLAUDE_CODE_SKIP_BEDROCK_AUTH=1` 和 `CLAUDE_CODE_USE_BEDROCK=1`。
- 只允许出站到网关的网络，再设 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1`；它同时禁用自动更新。
- 管理员下发时不要与 `forceLoginMethod`、`forceLoginOrgUUID`、`forceLoginGatewayUrl` 同放，否则网关凭证会在启动时被阻止。

| 兼容性要求 | 不满足的后果 |
|---|---|
| 提供一种 API 格式：Anthropic Messages（`/v1/messages`）、Bedrock InvokeModel 或 Agent Platform rawPredict | 无法连接 |
| 流式转发 SSE，含保活 `ping`，不缓冲整个响应 | 会话停滞或中途超时 |
| 原样转发 `anthropic-beta` 和 `anthropic-version`，把标头与请求体字段当开放列表 | 新版本功能失效，报 `400` |
| 原样返回上游错误与 `retry-after` 等响应头 | 自动恢复与重试失效 |
| 让 `/v1/messages` 路径免于 WAF 请求体检查 | 真实会话返回 `403` |

上游拒绝预发布字段时设 `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`。升级 Claude Code 前先对网关做测试，可用 `requiredMaximumVersion` 把开发者固定在已测版本。

### 监控：OpenTelemetry

OpenTelemetry 导出在所有提供商上都可用，也是唯一能把按用户的 token 与成本近实时送进自有可观测平台的办法。它与发往 Anthropic 的操作遥测是两条独立的路径。

```json
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://<collector-host>:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
  }
}
```

| 变量 | 目的 |
|---|---|
| `CLAUDE_CODE_ENABLE_TELEMETRY` | 启用遥测收集（必需） |
| `OTEL_METRICS_EXPORTER`、`OTEL_LOGS_EXPORTER` | 指标可选 `otlp`、`prometheus`、`console`、`none`；日志可选 `otlp`、`console`、`none` |
| `OTEL_RESOURCE_ATTRIBUTES` | 自定义属性，如团队、部门、成本中心 |
| `OTEL_LOG_USER_PROMPTS`、`OTEL_LOG_TOOL_DETAILS` | 记录提示内容；记录工具参数、Bash 命令、MCP 服务器与工具名（默认都关闭） |
| `OTEL_LOG_RAW_API_BODIES` | 记录完整 API 请求与响应体，含整个对话历史 |
| `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA`、`OTEL_TRACES_EXPORTER` | 启用 Traces（测试版） |

| 指标 | 含义 |
|---|---|
| `claude_code.session.count` | 启动的会话数 |
| `claude_code.lines_of_code.count` | 修改的代码行数 |
| `claude_code.pull_request.count`、`claude_code.commit.count` | 创建的 Pull Request 数、git 提交数 |
| `claude_code.cost.usage` | 会话成本（USD） |
| `claude_code.token.usage` | 使用的 token 数 |
| `claude_code.code_edit_tool.decision` | 代码编辑工具权限决策计数 |
| `claude_code.active_time.total` | 总活跃时间（秒） |

| 事件 | 何时发出 |
|---|---|
| `claude_code.user_prompt` | 用户提交提示词 |
| `claude_code.tool_result`、`claude_code.tool_decision` | 工具执行结果与权限决策 |
| `claude_code.api_request`、`claude_code.api_error` | 每次 API 请求与失败 |
| `claude_code.mcp_server_connection` | MCP 服务器连接 |
| `claude_code.permission_mode_changed`、`claude_code.auth` | 权限模式变更、身份验证 |

- 验证：后端出现 `claude_code.session.count` 即指标链路通；没数据时用 `claude --debug-file <path>` 找 `[3P telemetry]` 错误。
- 仓库里的 `.claude/settings.json` 不能打开遥测或改导出目标；写在托管设置里则会锁定目标。
- 通过 OAuth 认证时 `user.email` 会进入遥测属性，只发往你配置的端点。打开 `OTEL_LOG_*` 开关前先评估敏感信息。

### 成本管理与 Analytics

官方给出的企业部署参考：平均每个开发者每个活跃日约 $13，每月 $150-250，90% 的用户每个活跃日低于 $30。先用小规模试点建立自己的基线。

- `/usage`：**Session** 块显示当前会话的 token 用量与估算成本，`/clear` 后重置。订阅用户的用量已含在套餐内，这里的美元数字与计费无关。
- `/context` 查看什么占用了上下文；`/insights` 分析本机最近的会话并生成 HTML 报告。

| 部署方式 | 查看支出 | 限制支出 | 按用户数据 |
|---|---|---|---|
| Teams 或 Enterprise | 组织分析中的支出报告 | 管理设置中的支出限制 | 支出报告 CSV；Enterprise 另有 Enterprise Analytics API |
| Claude Console | Console 成本页面 | 工作区支出限制 | Console 仪表板、Claude Code Analytics API |
| Bedrock、Agent Platform、Foundry | 云计费控制台 | 云预算控制 | OpenTelemetry 或网关 |

合同价与标价不同时，用仅托管的 `modelPricing` 设置（v2.1.242+）让 `/usage`、状态栏和 OpenTelemetry 的数字对齐账单；它只改显示，不改收费。

降低成本的方法：

1. 不相关的任务之间用 `/clear`；需要延续时用 `/compact <保留什么>`。
2. 日常用 Sonnet，复杂架构再用 Opus；`/model` 随时切换，简单子代理指定 `model: haiku`。
3. 用 `/mcp` 禁用不用的 MCP 服务器；有 CLI（`gh`、`aws`、`gcloud`）时优先用 CLI。
4. CLAUDE.md 控制在 200 行以内，专门流程挪到 skill 按需加载。
5. 用 `/effort` 降低 effort 级别；固定思考预算的模型可设 `MAX_THINKING_TOKENS`。
6. 跑测试、读日志这类冗长操作交给子代理；agent team 保持小规模，用完即关。
7. 提示词写具体；复杂任务先用计划模式，走偏了立刻按 Escape。

> 长会话更贵的原因：每个请求都带完整对话历史；超过缓存生命周期后的第一条消息会重新处理全部上下文。

Analytics 仪表板：Teams 和 Enterprise 在 claude.ai/analytics/claude-code（使用指标、贡献指标、排行榜、CSV 导出），Console 在 platform.claude.com/claude-code（使用指标、支出、团队洞察）。贡献指标处于公开测试版，需要 Owner 角色并安装 Claude GitHub 应用，启用 ZDR 的组织不可用。云厂商上的用量不回传 Anthropic，仪表板不涵盖；下发 `DISABLE_TELEMETRY` 后，受影响开发者的数据也不再进入仪表板。

### 安全模型与最佳实践

```text
Claude Code 的安全分层
├─ 权限模式      自动模式（分类器审查）/ Manual mode（先问再做）/ Accept Edits
├─ 权限规则      allow / ask / deny，deny 与 ask 在分类器之前生效
├─ 沙箱          Bash 的文件系统与网络隔离，/sandbox 配置
├─ 信任验证      新文件夹的工作区信任对话框、项目 .mcp.json 的单独批准
└─ 组织策略      托管设置、托管 MCP、OpenTelemetry 审计
```

- 工作目录边界是权限提示，不是隔离：你批准的 Bash 命令仍能写到用户账户有权写的任何位置；要在操作系统层面限制，启用沙箱。
- `-p` 会话不显示工作区信任对话框，也不显示 `.mcp.json` 批准提示，自动化里要自己把关。
- 只看 `.mcp.json` 看不到会话可能加载的全部服务器，其他作用域、claude.ai 连接器和插件也能添加。
- Remote Control 的代码执行留在本机，但连接时会话记录会存到 Anthropic 服务器用于跨设备同步。

最佳实践：

1. 批准前审查建议的命令和对关键文件的改动，不要把不受信任的内容直接用管道传给 Claude。
2. 与外部服务交互的脚本和工具调用放在虚拟机、沙箱运行时或 dev container 里。
3. 用 `/permissions` 定期审计权限；团队用托管设置统一标准，用 `ConfigChange` hooks 审计设置变更。
4. 可疑行为用 `/feedback` 报告；安全漏洞走 HackerOne，不要公开披露。

### 数据使用、保留、ZDR 与 HIPAA

| 账户类型 | 是否用于训练 | 服务端保留期 |
|---|---|---|
| 消费者（Free、Pro、Max、Ultra） | 由用户在隐私设置中选择 | 允许训练 5 年；不允许 30 天 |
| 商业（Team、Enterprise、API、第三方平台） | 不用于训练，除非明确选择加入 | 标准 30 天 |
| 启用 ZDR 的 Enterprise 组织 | 不用于训练 | 响应返回后不存储，法律要求或防滥用除外 |

- 本地缓存：会话记录默认以明文存在 `~/.claude/projects/` 下 30 天，用 `cleanupPeriodDays` 调整。
- `/feedback`、`/bug`、`/share` 会把对话历史（含代码）发给 Anthropic 并保留 5 年；在第三方提供商上改为写入 `~/.claude/feedback-bundles/` 下的本地存档。

| 变量或设置 | 关闭什么 |
|---|---|
| `DISABLE_TELEMETRY` | 使用指标 |
| `DISABLE_ERROR_REPORTING` | 错误报告 |
| `DISABLE_FEEDBACK_COMMAND` | `/feedback` 报告 |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY` | 会话质量调查 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 一次性关闭所有非必要流量 |
| `skipWebFetchPreflight: true` | WebFetch 域名安全检查（只发送主机名） |

在 Bedrock、Agent Platform、Foundry 和 Claude Platform on AWS 上，指标、错误报告和 `/feedback` 默认关闭；会话质量调查和 WebFetch 域名检查不分提供商都默认开启。`DISABLE_TELEMETRY` 和 `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 会连带关闭 Remote Control 依赖的功能标志评估。

#### 零数据保留（ZDR）

- 仅适用于 Claude for Enterprise 的合格账户，不含在标准 Enterprise 计划内，不能在管理员设置里自行开启，要联系 Anthropic 账户团队；按组织逐一启用。
- 只覆盖发往该组织的请求：用 `forceLoginMethod` 和 `forceLoginOrgUUID` 防止开发者用个人账户登录。
- 不覆盖：claude.ai 聊天、Cowork、Claude in Chrome、第三方集成与 MCP 服务器。
- 后端自动禁用：云端会话、Claude Tag、Artifacts、反馈提交、Remote Control。
- 仅适用于 Anthropic 直接平台；Bedrock、Agent Platform、Foundry 以各平台的保留政策为准。因政策违规被标记的会话，相关输入输出可能保留最长 2 年。

#### HIPAA 配置

适用于已签署 BAA 的 Claude Enterprise 组织，由 Primary Owner 应用，范围是“本地模式”：终端里的 Claude Code、Claude Desktop 的 Code 标签页和 Cowork。VS Code 和 JetBrains 扩展仍可使用，但 BAA 不涵盖它们。需要 Claude Code v2.1.285 和 Claude Desktop v2.19675.0 或更高版本。

```json
{
  "forceLoginMethod": "claudeai",
  "forceLoginOrgUUID": "<your-org-uuid>",
  "allowedProviders": ["anthropic"],
  "cleanupPeriodDays": 30
}
```

- 只有“Claude Enterprise 账户登录并直连 Claude API”的会话才适用；云厂商、Claude apps gateway、LLM 网关或自定义 `ANTHROPIC_BASE_URL`、Console API 密钥都不符合条件。
- `forceLoginOrgUUID` 填错会让所有 claude.ai 登录的开发者在启动时退出，部署前务必核对。
- 应用后：WebFetch 不可用（网页搜索仍可用），`--cloud`、`/teleport` 和 Remote Control 被拒绝，`/feedback` 和 `/bug` 不可用，不能发布 Artifact，MCP 工具搜索关闭。
- 本地会话数据的保护与删除由组织自己负责；保留清理只在有人启动 Claude Code 时运行。

### 采用工具包

官方为推广准备了两份可直接改写的材料，都不涉及配置。**Communications kit**（通信工具包）面向管理员和工程主管，包含推出公告及其变体、分主题的技巧系列消息和常见问题的单行回复；官方要求把它当草稿，用本组织的语气重写并替换占位符。**Champion kit**（倡导者手册）面向已在使用 Claude Code 的工程师，围绕三件事展开：分享你的发现、成为大家提问的对象、扩大圈子，并附三十天行动计划和对常见顾虑的回应。

---

## Part V：配置

### 设置文件：作用域、位置与优先级

设置（settings）是改变 Claude Code 行为的 JSON 键：启动用哪个模型、哪些操作不用问就能跑、哪些文件不能读、终端长什么样、组织强制了什么。终端、VS Code / JetBrains 扩展和桌面应用读取同一批设置文件；云端会话只读取其中一部分。给 Claude 的"常设指令"不属于设置，写在 `CLAUDE.md` 里。

#### 四个作用域与文件位置

| 作用域 | 文件 | 影响范围 | 典型用途 |
|---|---|---|---|
| 用户（User） | `~/.claude/settings.json` | 你在这台机器上的每个项目 | 主题、编辑器模式、默认模型、个人权限规则 |
| 共享项目（Project） | `.claude/settings.json` | 在该文件夹工作的所有人；提交到 git 后队友才拿得到 | 团队权限、hook、插件、项目环境变量 |
| 项目本地（Local） | `.claude/settings.local.json` | 仅你、仅这一个项目 | 个人覆盖、共享前的试验 |
| 托管（Managed） | `managed-settings.json`、MDM 策略、claude.ai 控制台的服务器托管设置 | 组织部署到的所有人 | 安全策略与合规要求 |

```text
~/.claude/                      # Windows 上为 %USERPROFILE%\.claude；可用 CLAUDE_CONFIG_DIR 改位置
└── settings.json               # 用户设置
~/.claude.json                  # Claude Code 自己写：账户、MCP 配置、信任决定、全局配置键；不要手改

repo-root/
└── .claude/
    ├── settings.json           # 共享项目设置，提交到仓库
    └── settings.local.json     # 项目本地设置，不进提交

系统目录（管理员部署，你不需要创建）
├── macOS        /Library/Application Support/ClaudeCode/managed-settings.json
├── Linux / WSL  /etc/claude-code/managed-settings.json
└── Windows      C:\Program Files\ClaudeCode\managed-settings.json
```

- 安装 Claude Code 不会创建任何设置文件。你在 `/config` 里保存偏好时它才写用户设置；你在权限提示上选 "Yes, and don't ask again" 时它才写本地设置。
- 本地文件由 Claude Code 首次写入时自动加进全局 git 排除文件；手动创建的要自己加 `.gitignore`。
- 在 git 仓库子目录里启动，本地文件仍读写在仓库根目录。共享的 `.claude/settings.json` 则从会话的主工作目录读取，没有父目录回退：想用仓库根那份，就从仓库根启动。

#### 优先级：同一个键出现在多处时听谁的

```text
高  1. 托管设置        managed-settings.json / MDM / 服务器托管设置；文件和 --settings 都盖不过
    2. 命令行          claude --settings <file-or-json>，仅本次会话
    3. 项目本地设置    .claude/settings.local.json
    4. 共享项目设置    .claude/settings.json
低  5. 用户设置        ~/.claude/settings.json
```

- **环境变量不是这个堆栈里的一层。** 谁优先是逐对决定的：`ANTHROPIC_MODEL` 优先于任何文件里的 `model`，`--model` 在一个会话内同时盖过两者。设置文件里的 `env` 块是普通键，遵循上面的层级。
- **少数安全敏感键"取更严的值"**，哪怕来自低优先级作用域。例如任何作用域里的 `permissions.blockReadsOutsideWorkingDirectories: true` 都生效，即使托管设置写的是 `false`。
- **数组会合并而不是覆盖。** `permissions.allow` 这类列表键在多个文件里都设置时，每个文件都能加条目，谁也删不掉别人的条目。例外是模型相关的 `fallbackModel`、`modelPicker` 和托管的 `availableModels`。

#### 三种修改入口

| 入口 | 做法 | 写到哪里 |
|---|---|---|
| `/config` 菜单 | 会话内运行 `/config` 打开**配置**选项卡，或直接 `/config verbose=true` | 大多数选项写 `~/.claude/settings.json`；输出样式写 `.claude/settings.local.json`；全局配置选项写 `~/.claude.json` |
| 编辑 JSON | 用编辑器打开对应作用域的文件 | 你打开的那个文件 |
| 只改一次会话 | `claude --settings '{"model": "claude-opus-5-5"}'`、专用标志（如 `--model`）或环境变量 | 不写任何文件 |

`/config` 只列一小组个人选项，不是全部设置键；它属于终端界面，VS Code 聊天面板和桌面应用不打开它。

#### 修改后何时生效、如何确认

Claude Code 监视设置文件并在变化时重新加载，**大多数编辑在运行中的会话里立即生效**，包括 `permissions`、`hooks` 和 `apiKeyHelper`。例外：

- `model`、`effortLevel`、`modelSettings` 只在会话启动时读取一次，会话中途用 `/model`、`/effort` 切换。
- 来自 MDM 或控制台的托管设置按计划送达，而不是保存即到。

排查三件套：`/status` 的 `Setting sources` 行列出本会话加载了哪些设置来源（但不说明每个键来自哪个文件）；终端里的 `claude doctor` 列出被拒绝的条目；启动时的"设置错误"对话框说明文件有无效 JSON 或架构拒绝的值。`-p` 运行不显示对话框，直接跳过坏文件或坏值。

### 先认识 settings.json：只需掌握四件事

设置文件是**严格的 JSON**。新手最容易踩的坑不是 Claude Code，而是 JSON 语法和层级。

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "model": "claude-sonnet-5",
  "spinnerTipsEnabled": false,
  "permissions": {
    "defaultMode": "default",
    "allow": ["Bash(npm run lint)", "Bash(npm run test *)"]
  }
}
```

1. **顶层键**：整个文件是一对花括号；键名和字符串用双引号；布尔值是不带引号的 `true` / `false`；键之间用逗号分隔。
2. **嵌套对象**：文档里写的 `permissions.defaultMode`、`sandbox.network.allowedDomains` 是"路径"写法，落到文件里是一层套一层的对象，不是带点号的键名。
3. **数组**：权限规则是字符串数组，每条规则是一个带引号的字符串。
4. **`$schema`**：指向已发布的 JSON 架构，在 VS Code、Cursor 等编辑器里提供补全和内联校验。架构可能滞后于最新 CLI，新键上的校验警告不代表配置无效。

#### 常见错误

- **写注释或尾部逗号。** `//` 注释和最后一个键后面的逗号都是语法错误，下次启动会被报告为设置错误。官方示例里带注释的版本只是给人看的，不能直接保存。
- **把布尔值写成字符串。** `"enabled": "true"` 是字符串；正确写法是 `"enabled": true`。反过来，`env` 块里的值必须是字符串，如 `"DISABLE_AUTO_COMPACT": "1"`。
- **放错层级。** 把 `"allow": [...]` 写在顶层而不是 `permissions` 里，或把 `allowedDomains` 直接写在 `sandbox` 下而不是 `sandbox.network` 下。
- **放错文件。** `"defaultMode": "auto"` 写在 `.claude/settings.json` 里不报错也不生效，会话悄悄以别的模式启动。
- **把 `/Users/alice/file` 当成绝对路径。** 在 Read/Edit 权限规则里，单个前导斜杠是"相对于设置源"，绝对路径要写 `//Users/alice/file`。沙箱路径相反，`/tmp/build` 就是绝对路径。

### 一份安全的起步 settings.json

只复制你理解并需要的键。保存为 `~/.claude/settings.json`：

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "defaultMode": "default",
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(git diff *)"
    ],
    "ask": [
      "Bash(git push *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "autoUpdatesChannel": "stable",
  "cleanupPeriodDays": 30
}
```

| 行 | 作用 |
|---|---|
| `$schema` | 让编辑器提供补全和校验，不影响 Claude Code 行为 |
| `"defaultMode": "default"` | 新的终端会话以 Manual（手动）模式启动：大多数编辑、shell 命令和网络请求先问你。删掉这一行就回到内置默认值（v2.1.283 及以上的交互式终端和 VS Code 会话为 auto） |
| `allow` | lint、测试和 `git diff` 不用问。`Bash(npm run lint)` 是精确匹配，末尾的 ` *` 覆盖带参数的形式 |
| `ask` | 以 `git push` 开头的命令每次都确认，即使之后切到会自动批准的模式 |
| `deny` | 文件工具以及 `cat .env` 这类点名文件的命令读不到 env 文件和 `secrets` 目录 |
| `autoUpdatesChannel` | `"stable"` 渠道：通常约一周前发布、并跳过有重大回归的版本 |
| `cleanupPeriodDays` | 会话记录等本地数据的保留天数；`30` 即默认值，最小 `1`，`0` 无法通过校验 |

验证流程：

1. 保存后在会话里运行 `/status`，确认 `Setting sources` 里有 `User settings`；状态栏应显示 `⏸ manual mode on`。
2. 运行 `/permissions`，确认上面的规则都列出来了，并标着来源文件。
3. 让 Claude 运行 `npm run lint`（不提示）、编辑一个文件（提示）、读取 `.env`（被拒绝）。
4. 有异常就在终端运行 `claude doctor` 看被拒绝的条目。

> `Read(./.env)` 挡不住 `grep -r pattern .` 这种不点名文件的读取，也挡不住脚本自己打开文件。要在操作系统层面强制，需要再启用沙箱。

### 核心 settings.json 键

未注明作用域的键可以写在任意文件（用户 / 项目 / 本地 / 托管）。`sandbox.*` 键见后文沙箱一节。

#### 权限

| 键 | 类型/常见值 | 目的 |
|---|---|---|
| `permissions.allow` | 权限规则字符串数组 | 无需询问即可批准的工具使用；`--allowedTools` 可为单次会话追加 |
| `permissions.ask` | 权限规则字符串数组 | 即使在会自动批准的模式下也要提示的工具使用 |
| `permissions.deny` | 权限规则字符串数组 | 阻止的工具使用；`--disallowedTools` 可为单次会话追加 |
| `permissions.additionalDirectories` | 目录路径数组 | 额外的工作目录，只授予文件访问权限 |
| `permissions.defaultMode` | `"default"` / `"acceptEdits"` / `"plan"` / `"auto"` / `"dontAsk"` / `"bypassPermissions"`；`"manual"` 是 `"default"` 的别名 | 新会话的默认权限模式；`auto` 和 `bypassPermissions` 在项目或本地设置中不生效 |
| `permissions.disableBypassPermissionsMode` | 字符串 `"disable"` | 阻止进入 `bypassPermissions`；优先于 `--dangerously-skip-permissions` |
| `permissions.disableAutoMode` | 字符串 `"disable"` | 从 `Shift+Tab` 循环中移除自动模式；也可写成顶层的 `disableAutoMode` |
| `permissions.blockReadsOutsideWorkingDirectories` | boolean | 文件工具在所有模式下拒绝读取工作目录之外的内容 |
| `permissions.blockWritesOutsideWorkingDirectories` | boolean | `Write`、`Edit`、`NotebookEdit` 在所有模式下拒绝写到工作目录之外；不限制 shell 命令（v2.1.314+） |
| `permissions.denyInsteadOfPrompting` | boolean；仅用户、托管和 `--settings` | 无人应答时拒绝本该提示的调用，让会话继续（v2.1.319+） |
| `useAutoModeDuringPlan` | boolean，默认 `true`；用户、本地或托管 | 计划模式下是否让自动模式分类器审查 shell 命令 |
| `autoMode` | 含 `environment`、`allow`、`soft_deny`、`hard_deny` 等的对象；用户或托管 | 告诉自动模式分类器你信任哪些仓库、存储桶和域名 |
| `allowManagedPermissionRulesOnly` | boolean；仅托管 | 让托管设置成为权限规则的唯一来源 |

#### 模型、环境变量与 Hooks

| 键 | 类型/常见值 | 目的 |
|---|---|---|
| `model` | 字符串，模型别名或完整模型 ID | 每个新会话使用的模型；会话中用 `/model` 切换 |
| `effortLevel` | `"low"` / `"medium"` / `"high"` / `"xhigh"` | 尚未保存级别的模型的默认 effort 级别 |
| `modelSettings` | 模型名到对象的映射，如 `{ "claude-sonnet-5": { "effortLevel": "xhigh" } }` | 按模型保存 effort 级别（v2.1.251+）；`/effort` 写这里 |
| `fallbackModel` | 模型别名或 ID 的数组 | 主模型过载或不可用时按顺序尝试的备用模型 |
| `availableModels` | 模型别名或 ID 的数组 | 限制可选模型；放托管设置里对组织强制 |
| `outputStyle` | 字符串，内置或自定义输出样式名 | 选择输出样式 |
| `language` | 字符串，任意语言名称，如 `"japanese"` | 让 Claude 默认用该语言回复；不做校验 |
| `env` | 变量名到字符串值的对象 | 为每个会话及其子进程设置环境变量；会覆盖 shell 里的同名导出；明文保存 |
| `hooks` | 以 hook 事件为键的对象，值为 `{ "matcher", "hooks" }` 组数组 | 在生命周期节点运行自定义逻辑；各文件之间合并 |
| `disableAllHooks` | boolean | 关闭 hook、自定义状态栏和自定义文件建议命令 |
| `apiKeyHelper` | 字符串，一条 shell 命令 | 运行你自己的命令来生成请求凭据 |

模型名会变化，以当前客户端 `/model` 里能选到的为准。项目和本地设置里的 `env` 要等你信任工作区后才应用。

#### MCP、插件与 Git 署名

| 键 | 类型/常见值 | 目的 |
|---|---|---|
| `enableAllProjectMcpServers` | boolean | 无需询问即批准项目 `.mcp.json` 中的所有服务器 |
| `enabledMcpjsonServers` / `disabledMcpjsonServers` | 字符串数组，`.mcp.json` 中的服务器名 | 批准 / 拒绝指定的项目服务器 |
| `allowedMcpServers` / `deniedMcpServers` | 对象数组，每项恰好一个键：`serverName`、`serverCommand` 或 `serverUrl` | 用户可添加的 MCP 服务器允许列表 / 拒绝列表 |
| `enabledPlugins` | `plugin-name@marketplace-name` 到 boolean 的对象 | 启用或禁用单个插件 |
| `extraKnownMarketplaces` | 市场名到 `{ "source": {...} }` 的对象 | 注册额外的插件市场；仓库里的条目要等信任后才识别 |
| `attribution` | 含 `commit`、`pr` 字符串和 `sessionUrl` boolean 的对象，或 `false`（v2.1.281+） | 自定义或隐藏提交与 Pull Request 上的署名；子键设为空字符串即隐藏 |
| `includeGitInstructions` | boolean，默认 `true` | 是否向 Claude 提供内置的 git 相关指引 |

`includeCoAuthoredBy` 自 v2.0.62 起已弃用，由 `attribution` 取代。

#### 界面、会话与更新

| 键 | 类型/常见值 | 目的 |
|---|---|---|
| `statusLine` | `{ "type": "command", "command": "..." }`，可选 `padding`、`refreshInterval`、`hideVimModeIndicator` | 用你自己的命令渲染输入框下方的状态栏 |
| `theme` | `"auto"` / `"dark"`（默认）/ `"light"` / `"dark-daltonized"` / `"light-daltonized"` / `"dark-ansi"` / `"light-ansi"` / `"custom:<slug>"` | 界面颜色主题 |
| `editorMode` | `"normal"`（默认）/ `"vim"` | 输入框按键绑定模式 |
| `verbose` | boolean，默认 `false` | 是否展开显示每个工具调用的细节 |
| `spinnerTipsEnabled` | boolean，默认 `true` | 是否在加载指示器下显示小提示 |
| `preferredNotifChannel` | `"auto"`（默认）/ `"terminal_bell"` / `"iterm2"` / `"iterm2_with_bell"` / `"kitty"` / `"ghostty"` / `"notifications_disabled"` | 任务完成或等待权限提示时的通知方式 |
| `plansDirectory` | 字符串，相对于项目根的路径 | 计划文件存放位置；未设置时用 `~/.claude/plans` |
| `cleanupPeriodDays` | 整数，最小 `1`，默认 `30` | 会话记录和其他本地数据的保留天数 |
| `autoUpdatesChannel` | `"latest"`（未设置时的行为）/ `"stable"` | 自动更新遵循的发布渠道 |
| `autoCompactEnabled` | boolean，默认 `true` | 上下文接近上限时自动压缩对话 |
| `autoMemoryEnabled` | boolean，默认 `true` | 开关自动记忆 |
| `fileCheckpointingEnabled` | boolean，默认 `true` | 每次编辑前为文件做快照，供 `/rewind` 恢复 |

#### 设置分组索引

其余键按官方 reference 的分组查找，每个条目都列出作用域、类型、默认值和示例：

| 分组 | 代表性键 |
|---|---|
| 模型与回复 | `advisorModel`、`deniedModels`、`modelPicker`、`modelOverrides`、`promptCacheTtl`、`fastMode` |
| 权限设置 | `autoMode.classifyAllShell`、`skipAutoPermissionPrompt`、`skipDangerousModePermissionPrompt` |
| 沙箱设置 | `sandbox.*`、`sandbox.filesystem.*`、`sandbox.network.*`、`sandbox.credentials.*` |
| 记忆与上下文 | `claudeMd`、`claudeMdExcludes`、`autoMemoryDirectory`、`idleCompaction`、`bashOutputMaxChars` |
| 界面与终端 | `defaultShell`、`fileSuggestion`、`respectGitignore`、`spinnerVerbs`、`tui`、`viewMode`、`voice` |
| Git 与署名 | `prUrlTemplate`、`attribution.commit`、`attribution.pr`、`attribution.sessionUrl` |
| Hook 与自动化 | `allowedHttpHookUrls`、`allowManagedHooksOnly`、`httpHookAllowedEnvVars`、`enableWorkflows` |
| 插件与 skill | `strictKnownMarketplaces`、`blockedMarketplaces`、`skillOverrides`、`pluginConfigs` |
| MCP | `allowManagedMcpServersOnly`、`disableClaudeAiConnectors`、`managedMcpServers` |
| Agent、会话与 worktree | `agent`、`teammateMode`、`worktree.*`、`crossSessionInbound` |
| Remote、桌面与通知 | `remoteControlAtStartup`、`disableRemoteControl`、`sshConfigs`、`enableArtifact` |
| 身份验证与提供商 | `forceLoginMethod`、`forceLoginOrgUUID`、`awsAuthRefresh`、`allowedProviders` |
| 更新、隐私与遥测 | `minimumVersion`、`requiredMinimumVersion`、`feedbackSurveyRate`、`skipWebFetchPreflight` |
| 企业与托管设置 | `policyHelper`、`managedSourcesBehavior`、`forceRemoteSettingsRefresh` |
| 全局配置设置（存于 `~/.claude.json`） | `autoConnectIde`、`diffTool`、`copyOnSelect` |

### 权限规则：allow / ask / deny

权限规则由 Claude Code 强制执行，而不是由模型。提示词或 `CLAUDE.md` 里的说明只影响 Claude "想做什么"，改变不了规则。

Manual 模式下的基线：工作目录内的文件读取不用问；Bash 命令要问，内置只读命令集（`ls`、`cat`、`echo`、`pwd`、`head`、`tail`、`grep`、`find`、`wc`、`which`、`diff`、`stat`、`du`、`cd` 和 `git` 的只读形式）除外；文件修改、WebFetch、WebSearch 要问。对 Bash 命令或 WebFetch 域选"是，不再询问"会把 allow 规则存进 `.claude/settings.local.json`；对文件修改选它只持续到会话结束。

#### 求值顺序

```text
Claude 发起工具调用
      │
      ▼
PreToolUse hook ── 以退出码 2 阻止 ──► 调用停止（早于规则求值）
      │
      ▼
deny 规则命中？ ── 是 ──► 拒绝（任何作用域、任何模式，包括 bypassPermissions）
      │ 否
      ▼
ask 规则命中？ ─── 是 ──► 提示你（hook 返回 "allow" 也照样提示）
      │ 否
      ▼
allow 规则命中？ ─ 是 ──► 直接运行
      │ 否
      ▼
由当前权限模式决定：提示 / 交给分类器 / 拒绝 / 放行
```

- 顺序固定为 deny → ask → allow，**第一个命中的决定结果，规则写得多具体都不改变顺序**。宽泛的 deny 里开不出 allow 例外：`Bash(aws *)` 被拒后，`Bash(aws s3 ls)` 的 allow 无效。
- 任何作用域的 deny 都先于 allow：用户级 deny 会挡住项目级 allow，反之亦然。托管 deny 无法被 `--allowedTools` 覆盖。
- 裸工具名的 deny（如 `Bash`）把整个工具从 Claude 的上下文里移除；带范围的 deny（如 `Bash(rm *)`）保留工具，只拦匹配的调用。

#### 规则语法速查

格式是 `Tool` 或 `Tool(specifier)`，工具名要写规范名称，而不是界面上显示的标签。

| 规则 | 匹配 | 不匹配 / 备注 |
|---|---|---|
| `Bash` 或 `Bash(*)` | 所有 Bash 命令 | |
| `Bash(npm run build)` | `npm run build` | `npm run build --watch` |
| `Bash(npm run *)` | `npm run build`、`npm run test --watch`、`npm run` | `npm install` |
| `Bash(ls *)` | `ls -la`、`ls` | `lsof`；写成 `Bash(ls*)` 才会匹配它 |
| `Read(./.env)` | 当前目录的 `.env` | |
| `Read(~/.zshrc)` | 主目录下的 `.zshrc` | |
| `Edit(/src/**/*.ts)` | 相对于设置源的 `src` 下的 ts 文件 | 不是文件系统根下的 `/src` |
| `Read(//Users/alice/secrets/**)` | 绝对路径 `/Users/alice/secrets/**` | |
| `WebFetch(domain:example.com)` | 仅 `example.com` | 子域要另写 `WebFetch(domain:*.example.com)` |
| `mcp__puppeteer` 或 `mcp__puppeteer__*` | `puppeteer` 服务器的所有工具 | |
| `mcp__puppeteer__puppeteer_navigate` | 该服务器的单个工具 | |
| `mcp__*` | 所有 MCP 工具 | 仅 deny 和 ask 可用 |
| `Agent(Explore)` | 名为 Explore 的子代理 | 常放进 `deny` 来禁用 |
| `Agent(model:opus)` | 按输入参数匹配 | 仅 deny 和 ask 可用 |

#### Bash：按命令文本匹配

- `*` 匹配任何文本（含空格），可以出现在开头、中间或结尾。**把 `*` 放在子命令之后**：`Bash(git log *)` 只放行 `git log`，`Bash(git *)` 放行所有 git 命令。
- 尾部 ` *`（前面有空格）也匹配不带参数的裸命令。`Bash(ls:*)` 与 `Bash(ls *)` 等价，但 `:*` 只在末尾有效。
- 复合命令按 `&&`、`||`、`;`、`|`、`|&`、`&` 和换行拆开，**每个子命令都必须被允许**；deny 和 ask 只要任一子命令命中就生效。
- 匹配前会剥掉固定的一组包装器：`timeout`、`time`、`nice`、`nohup`、`stdbuf`、`command`、`builtin`、zsh 的 `noglob`，以及不带标志的 `xargs`。`npx`、`docker exec`、`devbox run` 不在其中，所以 `Bash(devbox run *)` 等于放行 `devbox run rm -rf .`。
- **Bash 规则不是安全边界。** `Bash(curl *)` 的 deny 拦得住 `curl https://example.com`，拦不住 `/usr/bin/curl ...` 或 `sh -c 'curl ...'`；`Bash(git push *)` 拦不住 `git -C . push`。需要硬保证时用沙箱或 PreToolUse hook。

#### Read 和 Edit：gitignore 风格的路径模式

- 四种前缀：`//path` 是文件系统绝对路径；`~/path` 从主目录开始；`/path` 相对于设置源（项目和本地设置里是主工作目录，用户设置里是 `~/.claude`）；`path` 或 `./path` 相对于当前目录。
- `*` 只在单个路径段内匹配，`**` 跨目录。裸文件名在任何深度匹配：`Read(.env)` 等价于 `Read(**/.env)`。
- `src/**` 这种单段目录模式，作为 allow 只匹配 `<cwd>/src`，作为 deny 或 ask 则匹配任何深度的 `src` 目录。
- **路径限制只用 `Read` 和 `Edit` 写。** `Edit` 规则覆盖所有编辑文件的内置工具；`Read` 规则尽力覆盖 Grep、Glob 和 `@file` 提及。`Write(path)`、`Grep(path)`、`Glob(path)` 会被接受但不匹配任何东西。
- `Read` deny 也会阻止同一路径上的 Edit 和 Write。项目里的 `.claudeignore` 文件不起作用，要改写成 `Read` deny 规则。

#### WebFetch 与 MCP 的细节

- `WebFetch(domain:*.example.com)` 匹配任意深度的子域，但不含 `example.com` 本身。裸 `WebFetch` 与 `WebFetch(domain:*)` 都覆盖每个 URL，但只有 `domain:` 形式会把域同步进沙箱的允许或拒绝域名列表。
- 只允许 WebFetch 域名并不能限制联网：只要 Bash 被允许，Claude 仍可用 `curl`、`wget`。
- MCP 的 allow 规则不要带括号里的说明符，带了就不匹配任何调用；`"*"`、`"mcp__*"` 这类未锚定的 allow 会被跳过。服务器名里的 `.` 或空格在规则里写成 `_`。

#### /permissions 与 additionalDirectories

在会话里运行 `/permissions`，可以看到所有规则及其来自的设置文件，并添加或删除规则；改动从同一轮的下一个工具调用开始生效。自动模式可用时还多一个 **Auto mode** 选项卡，被分类器拦下的操作列在 **Recently blocked** 下。

Claude 默认能访问启动目录（主工作目录）里的文件。扩展方式有三种：启动时 `--add-dir <path>`、会话中 `/add-dir`、持久化的 `permissions.additionalDirectories`。附加目录里的文件变为可读而无需提示，编辑仍遵循当前权限模式。附加目录**只给文件访问，不当配置根目录**：`--add-dir` 和 `/add-dir` 添加的目录还会加载其中的 skills、命令和子代理，而 `permissions.additionalDirectories` 列出的目录连这些也不加载。

#### 工作区信任：为什么提交的 allow 没生效

项目 `.claude/settings.json` 里的 `permissions.allow` 和 `permissions.additionalDirectories` 是在"授予能力"，要等你接受该文件夹的工作区信任对话框后才应用；`deny` 和 `ask` 只做限制，立即生效。`claude -p` 和 SDK 会话不显示信任对话框，未信任文件夹里的项目 allow 规则不会被使用，并向 stderr 打印 `this workspace has not been trusted` 警告。

#### 与 hooks、沙箱的关系

- **Hooks**：PreToolUse hook 在权限提示之前运行，可以拒绝、强制提示或跳过提示，但绕不过 deny 和 ask 规则。想"放行所有 Bash，只拦少数几条"，做法是 allow 里加 `"Bash"`，再用 PreToolUse hook 拒绝那几条。
- **沙箱**：权限在工具运行**之前**按命令字符串判断；沙箱在进程运行**期间**由操作系统强制。两边的路径和域名会合并进最终沙箱配置：`Edit` allow 规则等同于加可写路径，`Read` / `Edit` deny 规则会阻止访问对应路径，`WebFetch(domain:...)` 规则进入沙箱域名列表。

### 权限模式

权限规则决定"某一类调用"怎么处理，权限模式决定没有规则命中时的基线。

#### 六种模式对比

| 模式 | 无需询问即可运行 | 状态栏 | 最适合 |
|---|---|---|---|
| `default`（界面上叫 Manual，别名 `manual`） | 工作目录内的文件读取及其他无需批准的操作 | `⏸ manual mode on` | 敏感工作、不熟悉的代码 |
| `acceptEdits` | 读取、文件编辑、常见文件系统命令（`mkdir`、`touch`、`rm`、`rmdir`、`mv`、`cp`、`sed`），限工作目录和附加目录内 | `⏵⏵ accept edits on` | 迭代你正在审查的代码 |
| `plan` | 读取；自动模式可用时还包括分类器批准的命令。不编辑源文件 | `⏸ plan mode on` | 改动前先探索代码库 |
| `auto` | 一切，带后台分类器安全检查 | `⏵⏵ auto mode on` | 长任务、减少提示疲劳 |
| `dontAsk` | 工作目录内的文件读取和预先批准的工具；其余会提示的调用一律拒绝 | `⏵⏵ don't ask on` | 锁定的 CI 和脚本 |
| `bypassPermissions` | 一切 | `⏵⏵ bypass permissions on` | 仅限隔离的容器和虚拟机 |

贯穿所有模式的三条规则：

- deny 规则在每种模式下都拦，包括 `bypassPermissions`；allow 规则在 `bypassPermissions` 下无效。
- 显式 ask 规则、`AskUserQuestion` 等需要用户交互的工具、针对关键路径（如 `rm -rf /`、`rm -rf ~`）的 `rm` / `rmdir`，任何模式都不会自动批准。
- 对受保护路径（`.git`、`.claude`、`.vscode`、`.husky`、`.mcp.json`、shell 启动文件等）的写入：`default` 和 `acceptEdits` 提示，`auto` 交给分类器，`dontAsk` 拒绝，`bypassPermissions` 允许。`permissions.allow` 规则不能预先批准这类写入。

#### 会话从哪个模式启动、如何切换

```text
高  1. --permission-mode 标志，或 --dangerously-skip-permissions
    2. 设置文件中的 permissions.defaultMode（项目或本地设置里的 "auto"、"bypassPermissions" 不生效）
低  3. 内置默认值
```

内置默认值取决于运行方式：`claude -p` 和 Agent SDK 为 `default`；交互式终端和 VS Code 扩展在 v2.1.283 及以上为 `auto`（更早版本仅在 Pro、Max、Ultra、Team 计划上为 `auto`）；任何设置文件把 `disableAutoMode` 设为 `"disable"` 时为 `default`。选中 `auto` 但自动模式不可用时，会话以 Manual 启动。

| 场景 | 做法 |
|---|---|
| CLI 会话中 | 按 `Shift+Tab` 循环：`default` → `acceptEdits` → `plan`，之后是已启用的可选模式（`bypassPermissions` 在前，`auto` 最后） |
| 单条提示进计划模式 | 在提示词前加 `/plan` |
| 启动时指定 | `claude --permission-mode plan`；对 `-p` 同样适用 |
| VS Code | 点提示框底部的模式指示器；固定默认值用 VS Code 用户设置 `claudeCode.initialPermissionMode`（不接受 `auto`），扩展从不读取项目设置里的起始模式 |
| JetBrains | 与 CLI 相同 |
| Desktop / Web | 输入框下方的模式选择器；云端会话提供 Accept edits、Plan、Auto，没有 Bypass permissions |

`dontAsk` 永远不在 `Shift+Tab` 循环里，只能用 `--permission-mode dontAsk` 设置。`bypassPermissions` 只有在启动时启用过才会出现在循环里；`--allow-dangerously-skip-permissions` 只把它加进循环而不激活。

#### plan：先分析再动手

Claude 读文件、跑探索性命令并写出计划，但不编辑源代码。计划就绪后可选 **Yes, and use auto mode**、**Yes, manually approve edits** 或 **No, keep planning**；按 `Ctrl+G` 可在默认文本编辑器里直接改计划。批准提示里按 `Shift+Tab` 是"批准"，想退出要先按 `Esc`。

计划期间的 shell 命令：自动模式可用且 `useAutoModeDuringPlan` 为默认的 `true` 时由分类器审查；否则内置只读命令集之外的命令会提示你。例外：在启用了绕过权限的交互式终端会话里，计划模式的阻止不强制执行。

#### auto：用分类器代替你点"是"

一个独立的分类器模型在操作运行前审查它，阻止超出你请求范围、针对无法识别的基础设施、或看起来受恶意内容驱动的操作。自动模式减少提示，但**不保证安全**。

- **可用条件**：所有计划都可用，管理员可用 `permissions.disableAutoMode` 关闭。模型要求随提供商不同：Anthropic API 上需 Claude Opus 4.6 或更高版本、Sonnet 4.6 或更高版本、Haiku 5.5 或 Fable 模型；Amazon Bedrock、Google Cloud's Agent Platform、Microsoft Foundry 上仅支持 Sonnet 5 或更高版本、Opus 4.7 或更高版本、Haiku 5.5 和 Fable 模型。
- **评估顺序**：先看你的 allow / ask / deny 规则 → 只读操作和工作目录内的文件编辑自动批准 → 其余交给分类器 → 被拦时 Claude 会收到原因。
- **默认阻止**（节选）：`curl | bash` 式的下载并执行、把敏感数据发往外部端点、生产部署和迁移、强制推送、`git reset --hard` 等丢弃未提交更改的命令、`terraform destroy`、授予 IAM 或仓库权限。
- **默认允许**（节选）：工作目录内的本地文件操作、安装锁文件或清单里声明的依赖、只读 HTTP 请求、推送到你正在处理的仓库的分支。完整列表用 `claude auto-mode defaults` 打印。
- **进入自动模式时宽泛的 allow 规则会被丢弃**，如 `Bash(*)`、`Bash(python*)`、`Agent` allow 规则；`Bash(npm test)` 这类窄规则保留，退出后恢复。
- **对话里的边界算数但不持久**：你说"不要推送"，分类器会据此阻止；上下文压缩后可能丢失。要硬保证请加 deny 规则，要保留某步人工确认请加 `permissions.ask` 规则。
- **回退**：分类器连续 3 次或累计 20 次阻止后，自动模式暂停并恢复提示；这两个阈值不可配置。

#### auto 模式的分类器计费

| 情况 | 是否计费 |
|---|---|
| 服务器端检查生效（`/status` 的 `Auto mode server` 行为 `Enabled`） | 检查随会话自身的模型请求完成，没有单独的分类器调用 |
| Pro、Max、Ultra、Team 计划 | 永远不会看到"不符合免费条件"的通知 |
| Enterprise 计划，以及 Claude API、Claude Platform on AWS、Amazon Bedrock、Google Cloud's Agent Platform、Microsoft Foundry 账户 | Claude Code 自己发出的分类器请求计入 token 用量 |

看到 `We're changing auto mode to no longer charge for classifier requests in Claude Code. However, this session isn't eligible.` 说明服务器端检查到不了你的会话，最常见的原因是中间有 LLM 网关或代理丢弃了相关请求头或字段。按 **Enter** 继续（照旧计费），按 **Esc** 取消当前操作。已知网关不支持时，可设置 `CLAUDE_CODE_AUTO_MODE_SERVER=0` 让通知不再出现；这是临时设置，后续版本可能移除。分类器请求默认发往 Claude Sonnet 5，v2.1.302 及以上可用 `CLAUDE_CODE_AUTO_MODE_MODEL` 指定。

#### dontAsk、bypassPermissions，以及何时不该用 bypass

`dontAsk`：所有本该提示的调用一律自动拒绝，只运行 Manual 模式下本来就不用批准的操作，以及 `permissions.allow` 或 PreToolUse hook 批准的大多数调用。命中 ask 规则的调用和 `AskUserQuestion` 也会被拒。

`bypassPermissions`：禁用权限提示和安全检查，工具调用立即执行，包括写受保护路径。`--dangerously-skip-permissions` 与 `--permission-mode bypassPermissions` 等效。下面这些情况不该用它：

- **不在隔离环境里。** 官方要求仅在容器、虚拟机等隔离环境中使用；它对提示词注入和意外操作不提供任何保护。
- **只是嫌提示多。** 这种情况用自动模式，或 Manual 模式加沙箱的自动允许模式。
- **以 root 或 `sudo` 运行。** Linux 和 macOS 上 Claude Code 会拒绝启动：`--dangerously-skip-permissions cannot be used with root/sudo privileges for security reasons`。
- **想通过仓库配置给团队开启。** 项目和本地设置里的 `bypassPermissions` 不生效，云端会话也会忽略设置文件里的 `bypassPermissions` 和 `dontAsk`。

想把自己锁在外面：在任意设置文件里把 `permissions.disableBypassPermissionsMode` 设为 `"disable"`；放托管设置里则无法被覆盖。

### 三套权限配置，实际有什么区别？

#### 场景 A：日常开发，推荐

项目 `.claude/settings.json`，提交给团队：

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": ["Bash(npm run *)"],
    "ask": ["Bash(git push *)"],
    "deny": ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  }
}
```

实际行为：

- 工作目录内的文件编辑和 `mkdir`、`mv`、`cp` 等常见文件系统命令不再逐个提示，事后用 `git diff` 审查；
- npm 脚本直接运行，其他 Bash 命令（内置只读集合之外）仍然提示；
- `git push` 每次确认；env 文件和 `secrets` 目录读不到；写 `.git`、`.claude` 等受保护路径仍然提示。

测试方法：让 Claude "在当前项目创建 `hello.txt`"，应直接完成；让它"运行 `npm run test`"，不提示；让它"推送当前分支"，出现权限提示；让它"读取 `.env`"，被拒绝。

注意：队友要先接受该文件夹的工作区信任对话框，`allow` 才对他们生效；`ask` 和 `deny` 立即生效。VS Code 扩展不从项目设置读取起始模式。

#### 场景 B：只读分析

```bash
claude --permission-mode plan
```

想让某个项目的终端会话默认如此，在它的 `.claude/settings.json` 里把 `permissions.defaultMode` 设为 `"plan"`。

实际行为：

- Claude 读文件、跑探索性命令、写出计划，但在你批准计划之前不编辑源文件；
- 适合代码审查、架构分析和排查问题；
- 自动模式可用时，计划期间的 shell 命令由分类器审查而不是提示你。想改成逐条提示，在用户或本地设置里把 `useAutoModeDuringPlan` 设为 `false`。

测试方法：要求 Claude "分析当前项目但不要修改"，结束后运行 `git status`，应无变化；状态栏应显示 `⏸ plan mode on`。不要和绕过权限一起启动，否则交互式终端里计划模式的阻止不强制执行。

#### 场景 C：CI 自动化

```bash
claude -p "run the test suite" --permission-mode dontAsk --allowedTools "Bash(npm test)" "Read"
```

实际行为：

- Claude 不会停下来问人，凡是本该提示的调用直接被拒绝；
- 只有允许列表里的调用，以及本来就不用批准的操作（工作目录内的读取、只读 Bash 命令）会运行；
- 超出允许列表的动作失败，而不是自动获得更大权限；这不等于 `bypassPermissions`。

测试方法：故意给一个需要未列出命令的任务（比如"安装一个依赖"），确认输出里出现被拒绝的调用，并且工作区没有意外改动。

补充：v2.1.302 及以上，未指定模式的 `claude -p` 和 Agent SDK 运行以 Manual 模式启动，需要批准的调用同样会被拒绝，所以 CI 里要么列出允许规则，要么改用 `--permission-mode auto` 让分类器审查。

#### 为什么不建议直接用 bypassPermissions？

少弹窗不等于配置成功。`bypassPermissions` 把保护完全交给外层隔离边界：没有容器、虚拟机或 sandbox runtime 时，一次提示词注入就能动你的主机。正确顺序是先用 Manual 或 `acceptEdits` 找出任务真正需要的命令和目录，写成 allow 规则；提示仍然太多再考虑自动模式或沙箱的自动允许模式；确实要完全无人值守时，在容器里以非 root 用户运行 `claude -p "<prompt>" --dangerously-skip-permissions`。

### 沙箱化 Bash 工具

Bash 沙箱是操作系统围绕 Claude 运行的 shell 命令强制执行的边界，限制这些命令能访问哪些文件和网络域。它适用于 Bash、PowerShell 和 Monitor 命令及其子进程，默认关闭。

> 权限模式回答"这个调用要不要问你"，沙箱回答"命令跑起来之后够得着什么"。`/sandbox` 不是权限模式，两者独立，可以组合。

#### 平台支持与依赖

| 平台 | 支持情况 | 依赖 |
|---|---|---|
| macOS | 支持 | 无；使用内置的 Seatbelt 框架 |
| Linux / WSL2 | 支持（WSL1 不支持） | `bubblewrap` 和 `socat`；seccomp 过滤器可选 |
| 原生 Windows | 不支持，命令以非沙箱方式运行 | 改在 WSL2、容器或虚拟机里运行 |

Ubuntu/Debian 上用 `sudo apt-get install bubblewrap socat`，Fedora 上用 `sudo dnf install bubblewrap socat`。依赖检查在启动时运行，装完要重启 Claude Code。

#### 启用

1. 在会话里运行 `/sandbox` 打开沙箱面板，它有 **Mode**、**Overrides**、**Config** 三个选项卡。如果只显示 **Dependencies**，说明沙箱还起不来，先装缺失的依赖。
2. 在 **Mode** 里二选一：**自动允许**（沙箱内的命令不提示直接跑）或**常规权限**（进了沙箱也照常走权限提示）。
3. 让 Claude 运行一次构建或测试。

面板里的选择保存到当前项目的 `.claude/settings.local.json`。想在所有项目启用，写进 `~/.claude/settings.json`：

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"],
      "denyRead": ["~/.aws/credentials"]
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

`sandbox.autoAllowBashIfSandboxed` 的默认值是 `false`（v2.1.324 之前为 `true`），所以只写 `"enabled": true` 得到的是常规权限模式。

#### 默认边界：文件系统与网络隔离

| 访问 | 默认值 | 用什么改 |
|---|---|---|
| 写入 | 工作目录、每用户临时目录、你添加的目录；受保护路径始终禁止写 | `filesystem.allowWrite`、`filesystem.denyWrite` |
| 读取 | 机器上的大部分内容，**包括 `~/.ssh` 和 `~/.aws/credentials` 等凭据文件** | `filesystem.denyRead`、`credentials` |
| 网络 | 没有直接出站路由；连接经过本机上的沙箱代理，按域名检查，允许列表初始为空 | `network.allowedDomains`、`network.deniedDomains` |
| 环境变量 | 继承自 Claude Code，包括其中的机密 | `credentials`、`CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` |

- **读取默认几乎不受限。** 没有内置的凭据拒绝列表，要保护的文件必须自己列出来。
- **新域名怎么处理取决于权限模式。** Manual 和 `acceptEdits` 提示你；自动模式下由 Claude 在命令上列出所需主机，交给分类器一并审查；`dontAsk` 拒绝；`bypassPermissions` 无提示放行。
- **只有读取代理变量的工具能连出去。** `curl`、`npm`、基于 HTTPS 的 `git` 可以；普通 `ssh`、大多数数据库驱动以及 UDP、ICMP 流量出不去。
- **沙箱有自己的受保护路径。** 即使在可写目录内，也拒绝写 `.claude` 设置文件、`.claude/skills`、`.claude/hooks`、`.mcp.json`、shell 启动文件、`.git` 里的 `hooks` 和 `config` 等；`allowWrite` 或 `Edit` allow 规则盖不过这层保护。

测试方法：让 **Claude** 运行 `touch ~/sandbox-probe`，在沙箱内应失败（macOS 显示 `Operation not permitted`，Linux 和 WSL2 显示 `Read-only file system`）；再运行 `curl --noproxy '*' https://example.com`，应失败并显示 `Could not resolve host`。你自己在 `!` 提示符下输入的命令通常在沙箱外运行，起不到测试作用。如果 Claude 请求在沙箱外重试，拒绝它。

#### sandbox 设置键

| 键 | 类型/默认 | 目的 |
|---|---|---|
| `sandbox.enabled` | boolean，默认 `false` | 为 shell 命令启用沙箱 |
| `sandbox.autoAllowBashIfSandboxed` | boolean，默认 `false` | 沙箱内的 Bash 命令无需权限提示即可运行（自动允许模式） |
| `sandbox.allowUnsandboxedCommands` | boolean，默认 `true` | 是否允许 Claude 在沙箱外重试；设为 `false` 即严格沙箱模式 |
| `sandbox.failIfUnavailable` | boolean，默认 `false` | 沙箱起不来时拒绝启动，而不是退回无沙箱运行 |
| `sandbox.excludedCommands` | 命令模式数组，语法同 `Bash(...)` 规则 | 匹配的命令在沙箱外运行，如 `"docker compose *"` |
| `sandbox.filesystem.allowWrite` / `denyWrite` | 路径数组 | 额外的可写路径 / 禁止写入的路径 |
| `sandbox.filesystem.denyRead` / `allowRead` | 路径数组 | 禁止读取的路径 / 在 `denyRead` 区域内重新开放的路径 |
| `sandbox.filesystem.disabled` | boolean，默认 `false`；用户或托管 | 关闭文件系统层，只保留网络隔离 |
| `sandbox.network.allowedDomains` / `deniedDomains` | 字符串数组：域名、通配符或 IP，可带 `:port` | 预先允许的出站域名 / 在所有权限模式下拒绝的域名 |
| `sandbox.network.strictAllowlist` | boolean，默认 `false`；用户或托管 | 允许列表之外的主机直接拒绝，而不是提示（v2.1.219+） |
| `sandbox.network.allowManagedDomainsOnly` | boolean，默认 `false`；仅托管 | 把网络允许列表锁定为托管设置定义的内容 |
| `sandbox.credentials` | 含 `files`、`envVars` 等的对象，条目带 `"mode": "deny"` 或 `"mask"` | 声明要对沙箱命令保护的凭据文件和环境变量 |
| `sandbox.enableWeakerNestedSandbox` | boolean，默认 `false` | 在非特权 Docker 容器内运行 Linux 沙箱；大幅削弱安全性 |
| `sandbox.network.tlsTerminate` | 含可选 `caCertPath`、`caKeyPath` 的对象；用户或托管 | 实验性：让沙箱代理终止 TLS，`mask` 凭据条目需要它 |

沙箱路径使用标准约定：`/tmp/build` 是绝对路径，`~/.kube` 相对于主目录，`./output` 或无前缀在项目设置里相对于项目根、在用户设置里相对于 `~/.claude`。多个作用域定义同一个文件系统数组时会合并；读取规则重叠时范围更窄的生效，例如 `"denyRead": ["~/"]` 搭配 `"allowRead": ["~/projects"]` 只留 `~/projects` 可读。

#### 逃生舱

| 逃生舱 | 行为 | 怎么收紧 |
|---|---|---|
| 沙箱外重试 | 命令在沙箱里失败后，Claude 可用 `dangerouslyDisableSandbox` 参数重试。Manual 和 `acceptEdits` 下你会收到标题为 "Bash command (unsandboxed)" 的提示；`bypassPermissions` 下无提示运行；匹配的 allow 规则也会批准重试 | `"allowUnsandboxedCommands": false`；或加 ask 规则 `Bash(dangerouslyDisableSandbox:true)` 强制提示 |
| `excludedCommands` | 匹配的命令以你的全部权限在沙箱外运行 | 写窄模式；能用 `allowWrite` 或 `allowedDomains` 解决就别排除整个工具 |
| 沙箱不可用时静默退化 | 平台不支持或依赖检查失败时，命令在没有沙箱的情况下运行 | `"failIfUnavailable": true` |

自动允许模式下仍然生效的检查：`Bash` deny 规则、限定内容的 ask 规则（如 `Bash(git push *)`）、针对关键路径的 `rm` 和 `rmdir`。单独的 `Bash` ask 规则对沙箱内的命令会被跳过。

#### 局限

- **只管 shell 命令。** Read、Edit、Write、WebFetch 等内置工具遵循权限规则而不是沙箱：`denyRead` 拦不住 Read 工具，`allowedDomains` 也限制不了 WebFetch。命令 hook、本地 MCP 服务器、状态栏命令、`apiKeyHelper` 和 mod 都以你的完整权限在沙箱外运行。
- **网络过滤只看主机名。** 内置代理默认不终止或检查 TLS，允许 `github.com` 这类宽泛域名可能成为数据外泄通道。
- **文件系统和网络两层要同时在。** 没有网络隔离，被入侵的 Agent 能外传 SSH 密钥；没有文件系统隔离，它能植入后门拿到网络访问。
- **放宽开关都是风险点。** `allowUnixSockets`（放行 `/var/run/docker.sock` 等于交出主机）、过宽的 `allowWrite`、`enableWeakerNestedSandbox`、`allowAppleEvents`、`allowNestedUserNamespaces`。
- **沙箱不是完整的隔离边界**，官方定位是降低风险，而不是硬安全控制。

### 选择沙箱环境

内置 Bash 沙箱只是其中一种隔离方式。要把文件工具、MCP 服务器和 hooks 也关进边界，需要隔离整个 Claude Code 进程。

| 方法 | 隔离的内容 | 需要 Docker | 设置工作量 | 什么时候选 |
|---|---|---|---|---|
| Sandboxed Bash tool | Bash、PowerShell 和 Monitor 工具命令及其子进程 | 否 | macOS 上最少；Linux 和 WSL2 上较少 | 在自己机器上日常工作时减少权限提示 |
| Sandbox runtime（测试版研究预览） | 整个 Claude Code 进程，包括文件工具、MCP 服务器和 hooks | 否 | 较少 | 不用 Docker 也想隔离 MCP 服务器和 hooks |
| Dev container | 完整开发环境 | 是 | 中等 | 在团队里统一沙箱环境；无人值守运行 |
| Custom container | 完整开发环境 | 是 | 中等到高 | 已有容器基础设施或 CI 运行器 |
| Virtual machine | 完整操作系统 | 否 | 高 | 安全策略要求内核级分离 |
| Cloud sessions | 默认在 Anthropic 托管的虚拟机里运行 shell 命令和文件编辑 | 否 | 无；需要 Claude 订阅，克隆 GitHub 仓库还需已连接的 GitHub 账户 | 从没有本地环境的设备使用 |

原生 Windows 主机用容器或虚拟机，或在 WSL2 里运行 Bash 沙箱。给组织里每个开发者强制隔离时，内置 Bash 沙箱是 Claude Code 唯一能强制的方法：通过托管设置下发 `sandbox` 键。

#### Sandbox runtime 速览

`@anthropic-ai/sandbox-runtime` 用与内置沙箱相同的 Seatbelt 或 bubblewrap 把整个进程包起来；配置格式可能随包的演进而变化。

```bash
npm install -g @anthropic-ai/sandbox-runtime
srt claude
```

- 配置写在 `~/.srt-settings.json`，或用 `--settings` 传文件。默认拒绝网络，并把写入限制在一小组内置运行时路径，所以**必须先配置再启动**。
- 至少放行写入：项目目录、`~/.claude` 和 `~/.claude.json`、运行时文件目录（Linux 和 WSL2 为 `/tmp`，macOS 为 `/private/tmp`）。至少放行网络：`api.anthropic.com` 或你的提供商端点；用 OAuth 登录还需要 `claude.ai` 和 `platform.claude.com`。
- 设置文件不存在时运行时照样启动（只是什么都不放行），所以"能启动"不能证明你的设置已加载。

#### 隔离与权限模式怎么搭

- `--dangerously-skip-permissions` **必须**搭配容器、虚拟机或 sandbox runtime，并以非 root 用户运行。
- 自动模式的分类器是按操作的控制，不是隔离边界；外层隔离对它是纵深防御，不是必需。
- 内置 Bash 沙箱单独使用不足以支撑任一模式的完全无人值守运行；可以在容器或虚拟机里再叠一层内置沙箱（非特权容器需要 `enableWeakerNestedSandbox`）。
- 隔离不改变发给模型的内容：提示词和 Claude 读到的文件无论是否使用沙箱，都会发往 Anthropic API 或你配置的提供商。

### 环境：云环境还是自托管环境？

"云端会话"指在开发者机器以外运行的会话（从 claude.ai/code、Desktop、移动端、`claude --cloud`、云端定时任务、Claude Tag 启动），每个云端会话都跑在一个"环境"里。终端或 IDE 里的普通会话始终在你自己的机器上运行，与这一节无关。

| 对比项 | 云环境（Anthropic 托管） | 自托管环境 |
|---|---|---|
| 会话在哪执行 | Anthropic 管理的 VM | 你网络内的运行器（runner）主机 |
| 可用套餐 | Pro、Max、Ultra、Team，以及有高级席位或 Chat + Claude Code 席位的 Enterprise | Team、Enterprise；**公开测试，默认关闭** |
| 你要维护什么 | 只配环境，无需基础设施 | 运行器镜像、机群、网络出口、git 凭据 |
| 访问内网服务 | 服务需对公网可达 | 直接从内网访问 |
| 资源上限 | 约 4 vCPU / 16 GB RAM / 30 GB 磁盘（可能变化） | 由你的主机决定 |
| 定制依赖 | 设置脚本 + 环境缓存 | 预装进运行器镜像 |
| 不可用场景 | — | 启用了零数据保留或 HIPAA 配置的组织 |

> 自托管只是把"会话执行"搬进你的网络。队列、编排和 claude.ai 界面仍在 Anthropic，对话内容（提示词、回复、工具结果）仍会发送到 `api.anthropic.com`；留在你这边的是仓库检出、构建产物、密钥和会话写出的文件。

### 云环境

没有环境时，引导流程（例如 `/web-setup`）会给你一个 **Default** 环境：**Limited** 网络访问，没有环境变量和设置脚本。默认允许列表不够用、需要环境变量或要预装依赖时再自己配置。

| 配置项 | 目的 | 要点 |
|---|---|---|
| Network access | 控制会话的出站连接 | **None**：无出站；**Limited**（默认，勾选 **Allow package managers**）：默认允许列表加你列的 **Allowed domains**；**Full**：任何域 |
| Environment variables | 给会话注入环境变量 | `.env` 格式，每行一个 `KEY=value`；使用该环境的任何人都读得到 |
| Network secrets | 让会话调用 API 却看不到密钥 | 密钥由 Anthropic 的代理在请求离开 VM 后附加；仅 Anthropic 托管环境可用 |
| Setup script | Claude Code 启动前装依赖 | Bash 脚本，以 root 在 Ubuntu 24.04 上运行，文件不超过 64 KB |

**Allowed domains** 每行一个域，前导 `*.` 匹配所有子域。GitHub（走专用代理）、已启用的 MCP 连接器、网络机密上列出的主机不受这张列表限制。

```bash
#!/bin/bash
apt update && apt install -y shellcheck
```

设置脚本的四条约束：

- 以零退出，否则会话可能无法启动；非关键命令后加 `|| true`。
- 约五分钟内完成，否则环境不会被缓存；独立安装用 `&` 和 `wait` 并行。
- 安装需要网络，**None** 级别下会失败。
- 脚本运行时环境变量尚未设置，需要的值写在脚本里。

环境缓存：脚本在约五分钟内跑完后，Anthropic 对文件系统做快照，后续会话从快照启动并跳过脚本。装的包、拉的镜像、写的文件会保留；脚本启动的数据库或 `docker compose up` 等进程不会保留。改了脚本或允许的网络主机，或缓存约七天到期后，脚本会重跑。

设置脚本只在云端、Claude Code 启动前运行；SessionStart hook 写在设置文件里（如仓库的 `.claude/settings.json`），本地和云端每个会话都会运行。想让 hook 只在云端执行，就在脚本开头判断 `CLAUDE_CODE_REMOTE` 是否为 `true`。

#### 操作流程

1. 在 claude.ai/code 的新会话页面，点消息框下方的环境选择器。
2. 选 **New environment**，或悬停已有环境点设置图标。
3. 依次设置 **Network access**、**Setup script**、**Environment variables**，需要密钥时在 **Network secrets** 里 **Add secret**。
4. 用 **Add environment** 或 **Save changes** 保存。
5. 从 CLI 创建云端会话时，用 `/remote-env` 选默认环境（保存到用户设置的 `remote.defaultEnvironmentId`）。

边界提醒：

- 改了环境变量，已有会话要等 VM 下次恢复或重建才生效。
- GitHub 代理拒绝分支删除、非分支推送（如标签）和 GraphQL 请求，所以 `gh pr`、`gh issue` 会收到 403，改用 `gh api repos/{owner}/{repo}/...`。
- 命令超时沿用 Bash 工具默认值；可在环境变量里设 `BASH_DEFAULT_TIMEOUT_MS`、`BASH_MAX_TIMEOUT_MS`（毫秒）。
- Anthropic 托管环境只能归档不能删除；用户级 `~/.claude/settings.json` 不会到达云端；目前不支持完全替换基础镜像。

### 自托管环境

> 公开测试：仅 Team 和 Enterprise，默认关闭。所有者需在 **Cloud environments** 管理页面打开 **Allow self-hosted environments**。

#### 架构

```text
┌──────────────── 你的网络 ───────────────┐        ┌──── Anthropic ─────┐
│  运行器（claude self-hosted-runner）    │──轮询─▶│ api.anthropic.com  │
│   ├─ 会话子进程（Claude Code）─────────│─事件流▶│  ├─ 环境队列       │
│   └─ 会话子进程（Claude Code）─────────│──推理─▶│  ├─ 会话流         │
│          │                              │        │  └─ 模型推理       │
│          ▼                              │        └────────────────────┘
│   你的 git 主机 / 内部服务              │   全部是出站 HTTPS，Anthropic 从不连入
└─────────────────────────────────────────┘
```

- **环境**：运行器的命名组，在 claude.ai 管理设置里创建；会话路由到环境而不是某个运行器。环境 ID 形如 `ccpool_...`。
- **环境密钥**：运行器向环境注册用的共享凭证，创建时只显示一次，365 天后过期。
- **运行器**：你部署的长期进程，一次只为一个"所有者"服务，认领第一个会话时锁定；`--capacity` 只控制同一所有者内的并发，所以最小机群规模等于同时活跃的所有者数量。

#### 快速开始

前置条件：Linux 或 macOS 主机（Windows 不支持作为运行器主机）、Claude Code v2.1.224 或更高、Git 2.24 或更高、能出站访问 `api.anthropic.com` 和你的 git 主机。

1. `claude self-hosted-runner --help` 能打印运行器用法，说明版本够新。
2. 在 **Cloud environments** 页面的 **Self-hosted environments** 下点 **New** 创建环境，**Copy environment key**。
3. 把环境密钥写入只有属主可读的文件，再启动运行器：

   ```bash
   mkdir -p /etc/claude
   (umask 077 && cat > /etc/claude/environment-secret)   # 粘贴密钥，回车，再按 Ctrl-D
   claude self-hosted-runner --environment-secret-file '/etc/claude/environment-secret' --base-dir '<writable-dir>'
   ```

4. 日志出现 `Registered: runner_id=<runner-id>`，管理页面状态变为 **Healthy**。
5. 在 claude.ai/code 启动会话并选你的环境；运行器日志出现 `Picked up session <session-id>`。
6. 追加消息：`claude -p "your message" --cloud <session-id>`。

也可以运行 `claude self-hosted-runner setup` 走引导式设置（需用所有者账户 `claude auth login`）。

> `--base-dir` 要用专供运行器的目录：运行器会把 `<writable-dir>/<repo-owner>/<repo>` 当成自己的克隆，可能重置或删除其中文件。这样启动的运行器服务完一个会话就按设计退出，生产环境要放在会自动重启它的编排器下。

#### 生产部署要点

| 要点 | 做法 |
|---|---|
| 每会话一个临时容器 | `--capacity 1` 加默认的 `--drain-grace-sec 0`，进程退出即销毁容器 |
| 镜像里不放宽权限凭据 | 不放长期 SSH 密钥、云凭据、个人访问令牌；会话凭据由包装脚本按会话签发 |
| 保护环境密钥 | 优先用按需运行器：密钥只留在编排器主机，每个运行器拿单次使用的工作单 |
| 默认拒绝出站 | 在你自己的网络边界限制出口，产品本身无法验证或强制；容器内屏蔽 `169.254.169.254` |
| 仓库设置防护 | `--confine-repo-settings`：默认 `warn`，`enforce` 拒绝会话，`off` 关闭扫描 |
| git 身份 | `--configure-git` 写入与托管会话相同的身份和提交签名，但不配置推送凭据 |
| 停止超时 | Kubernetes 的 `terminationGracePeriodSeconds`、Compose 的 `stop_grace_period` 至少设为运行器启动日志给出的时间 |
| 监控与排障 | 默认 8080 端口提供 `GET /healthz` 和 `/metrics`（`--health-port`）；`claude self-hosted-runner doctor` 做引导式诊断 |

Anthropic 不发布预构建的运行器镜像，需要围绕 `claude` 二进制自己构建。分派没有按环境的访问控制：组织内任何成员都能向任何环境分派会话。

#### 会话定制

| 机制 | 怎么启用 | 用途 |
|---|---|---|
| 包装脚本 | `--exec-path` 或 `SELF_HOSTED_RUNNER_EXEC_PATH` | 按会话签发凭据、准备工具链、固定权限模式 |
| 生命周期钩子 | `--hooks-dir` 或 `SELF_HOSTED_RUNNER_HOOKS_DIR` | 目录里名为 `checkout`、`post-session`、`command` 的可执行文件替换对应阶段 |
| 按需运行器 | `claude self-hosted-runner orchestrator` 加 `spawn-runner` 钩子 | 每个排队会话启动一个运行器 |
| MCP 服务器 | 构建镜像时 `claude mcp add --scope user ...` | 每个会话都可用；必须用 `--scope user` |
| 主机配置 | 运行器主机的 `~/.claude/` | `settings.json`、`CLAUDE.md`、hook、skill 作为每个会话的用户级基线 |

包装脚本必须以 `exec` 进入运行器自己的二进制结尾，例如把会话固定为自动模式：

```bash
#!/bin/bash
exec "$CLAUDE_RUNNER_CLAUDE_BIN" "$@" --permission-mode auto
```

生命周期钩子在运行器上围绕会话运行，不是会话内的 Claude Code hooks；`post-session` 是保存未提交工作的唯一机会。运行器只在启动时对 `~/.claude/` 做一次快照，改配置后要重启运行器。

#### 身份校验

每个自托管会话在 `CLAUDE_CODE_SESSION_ACCESS_TOKEN` 里拿到一个 Anthropic 签名的 JWT，内部服务按顺序检查：

1. 值以 `sk-ant-cc-` 开头，去掉前缀后是标准 JWT。
2. 用 `https://api.anthropic.com/v1/code/.well-known/jwks.json` 的密钥验证 `ES256` 签名。
3. `iss` 等于 `ccr`；`aud` 数组包含你的环境 ID（`ccpool_...`）；`ccr:role` 等于 `session_worker`；`exp` 未过期。
4. 从 `act.sub` 读取创建者（`user:<id>` 或 `agent:<id>`）；`act.email` 可能缺失，不要拿它判断身份。

包装脚本里可用内置解码器（只验证签名和过期时间）：

```bash
"$CLAUDE_RUNNER_CLAUDE_BIN" self-hosted-runner decode-token | jq -re '.act.email // .act.sub'
```

> 令牌不证明是哪个进程出示的：会话里 Claude 运行的任何代码都读得到它。由它换出的内部凭证应限定为单个编码会话所需的权限和有效期。

#### 端到端测试

发布新运行器镜像前，从 CI 对测试环境跑一遍完整会话：

1. 在测试运行器主机的 `~/.claude/` 装一个 Stop hook，把每轮最终回复（stdin JSON 的 `last_assistant_message`）写到 `$E2E_REPLY_DIR/<session_id>.txt`，导出 `E2E_REPLY_DIR` 后启动运行器。
2. 创建会话：`claude -p "<prompt>" --environment <environment-id> --output-format json`（在 git 检出目录里运行，可选 `--ref <branch>`）。
3. 等回复文件出现，再用 `claude -p "<message>" --cloud <session_id> --output-format json` 发后续消息并再次等待。

这两个命令只接受 claude.ai OAuth 登录（`claude auth login`），不接受 API 密钥；刷新授权上限为 30 天，目前没有对应的长期 CI 令牌。捕获 hook 只装在测试运行器上。

### 模型配置

#### 模型别名

`model` 可以填别名，也可以填完整模型名称（Bedrock 填推理配置文件 ARN，Foundry 填部署名称，Google Cloud 的 Agent Platform 填版本名称）。

| 别名 | 行为 |
|---|---|
| `default` | 特殊值：清除模型覆盖，回到账户的运行时默认值；本身不是模型别名 |
| `best` | 与 `opus` 使用相同的模型 |
| `fable` | 提供商的 Fable 模型，用于最困难、运行时间最长的任务 |
| `sonnet` | 最新的 Sonnet 模型，日常编码 |
| `opus` | 最新的 Opus 模型，复杂推理 |
| `haiku` | 快速高效的 Haiku 模型，简单任务 |
| `sonnet[1m]` / `opus[1m]` | 带 100 万 token 上下文窗口的 Sonnet / Opus；别名已解析到原生 1M 的模型时无效 |
| `opusplan` | 计划模式用 `opus`，执行时切到 `sonnet` |

别名指向提供商的推荐版本并随时间更新，不同提供商解析结果不同。要固定版本，用完整模型名称（如 `claude-opus-5-5`）或 `ANTHROPIC_DEFAULT_*_MODEL`。Fable 模型在任何套餐或提供商上都不是默认值，需要显式选择，用量可能计入使用额度。

#### 设置方式与优先级

```text
高   1. 会话中      /model <alias|name>          （不带参数打开选择器）
 │   2. 启动时      claude --model <alias|name>
 │   3. 环境变量    ANTHROPIC_MODEL=<alias|name>
 │   4. 设置文件    "model": "opus"
 ▼   5. 新会话默认  ANTHROPIC_DEFAULT_MODEL=<alias|name>
低
```

- `/model` 选择器里 `Enter` 切换并保存为默认值（写入用户设置的 `model`），`s` 仅本次会话生效。
- `--model` 和 `ANTHROPIC_MODEL` 只对用它们启动的会话有效；恢复的会话通常保持保存时的模型。
- 用标志、环境变量或设置填错模型名不会被提前拒绝，到第一个请求才报错。

#### effort 级别与扩展上下文

effort 控制自适应推理：级别越低越快越省，越高推理越深。Fable 模型以及 Opus 5.5、Sonnet 5.5、Haiku 5.5、Opus 5、Sonnet 5、Opus 4.8、Opus 4.7 支持 `low`、`medium`、`high`、`xhigh`、`max`；Opus 4.6 和 Sonnet 4.6 没有 `xhigh`；未列出的模型不支持 effort。设了模型不支持的级别，会回退到不高于它的最高支持级别。

解析顺序（取第一个适用的）：`CLAUDE_CODE_EFFORT_LEVEL` → 会话中的选择（`/effort`、`/model` 选择器）→ `--effort` → 设置（`modelSettings` 或 `effortLevel`）→ 模型默认值。

| 设置方式 | 说明 |
|---|---|
| `/effort` | 不带参数打开滑块；`/effort <级别>` 直接设置；`/effort auto` 清除当前模型的保存级别 |
| `--effort` | 为单个会话设置 |
| `CLAUDE_CODE_EFFORT_LEVEL` | 设为级别名；`auto` 或 `unset` 表示不发送 effort 级别 |
| `effortLevel` 设置 | 取 `low`、`medium`、`high` 或 `xhigh`，不接受 `max` |
| skill / 子代理 frontmatter | `effort` 字段，仅在其运行期间生效 |

在提示词里写 `ultrathink` 可为该轮请求更深推理而不改会话设置。`max` 除非通过环境变量设置，否则只对当前会话生效。

扩展上下文：Fable 模型、Sonnet 5 及更高版本、Haiku 5.5、Opus 4.7 及更高版本默认就是 1M 窗口；Opus 4.6 和 Sonnet 4.6 要加 `[1m]` 后缀（如 `/model opus[1m]`、`/model claude-sonnet-4-6[1m]`），是否需要使用额度取决于套餐。`CLAUDE_CODE_DISABLE_1M_CONTEXT=1` 可保持 200K 窗口；自动压缩窗口用 `/autocompact 500k`、`autoCompactWindow` 设置或 `CLAUDE_CODE_AUTO_COMPACT_WINDOW` 调整。

#### 相关环境变量

| 环境变量 | 目的 |
|---|---|
| `ANTHROPIC_MODEL` | 本次启动使用的模型 |
| `ANTHROPIC_DEFAULT_MODEL` | 没有其他来源选择模型时，新会话的默认模型 |
| `ANTHROPIC_DEFAULT_FABLE_MODEL` | `fable` 别名解析到的模型 |
| `ANTHROPIC_DEFAULT_OPUS_MODEL` | `opus` 使用的模型，以及计划模式下的 `opusplan` |
| `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet` 使用的模型，以及非计划模式下的 `opusplan` |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `haiku` 使用的模型，或后台功能使用的模型 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | 未另行指定模型的子代理、队友和工作流 Agent 的默认模型 |
| `DISABLE_PROMPT_CACHING` | 设为 `1` 禁用所有模型的提示缓存 |
| `DISABLE_PROMPT_CACHING_HAIKU` / `_SONNET` / `_OPUS` / `_FABLE` | 设为 `1` 只禁用对应模型的提示缓存 |

`ANTHROPIC_SMALL_FAST_MODEL` 已弃用，改用 `ANTHROPIC_DEFAULT_HAIKU_MODEL`。通过第三方提供商部署时，应在推出前用这些变量把别名固定到具体版本 ID。`ANTHROPIC_BASE_URL` 改变的是请求发往哪里，不是由哪个模型回答。

#### 组织限制

```json
{
  "model": "claude-sonnet-4-5",
  "availableModels": ["claude-sonnet-4-5", "haiku"],
  "enforceAvailableModels": true
}
```

| 托管设置 | 目的 |
|---|---|
| `availableModels` | 允许列表；条目可以是系列（`sonnet`）、版本前缀或完整模型 ID |
| `enforceAvailableModels` | `true` 时把允许列表扩展到默认选项 |
| `deniedModels`、`availableModelsMatch` | 阻止允许列表本会放行的特定版本 |
| `model` | 只是启动时的初始选择，不是强制 |
| `maxEffortLevel` | 在客户端限制 effort 上限 |

允许列表作用于所有能指定模型的地方（`/model`、`--model`、子代理和 skill 的 `model`、`advisorModel`、`/fast` 的隐式切换等）。Claude Enterprise 的管理员还能在 claude.ai 管理控制台禁用单个模型、设置组织默认模型和按角色的 effort 上限。模型名称变化很快，以 `/model` 选择器当前显示的为准。

### Fast mode：同模型，更快，更贵

快速模式是受支持模型的高速配置：同一个模型、质量和功能不变，速度最多提高 2.5 倍，每 token 成本更高。它不是另一个模型，也不同于降低 effort（那是减少思考）。用 `/fast` 切换（Space 开关，Enter 确认），也可以 `/fast on`、`/fast off`。

| 项目 | 说明 |
|---|---|
| 适用模型（Anthropic API） | Fable 5.2、Opus 5.5、Opus 5、Opus 4.8；Sonnet、Haiku 不支持 |
| 预览状态 | Opus 5 和 Opus 4.8 的快速模式是**研究预览**；Opus 4.8 的快速模式已弃用，将于 2026 年 11 月 30 日移除 |
| 可用平台 | 订阅套餐、Claude Console，以及满足条件时的 Google Cloud Agent Platform；Bedrock、Foundry、Claude Platform on AWS 不可用 |
| 订阅用户计费 | 只通过使用额度提供，即使套餐还有余量也直接计入使用额度 |
| Team / Enterprise | 默认禁用，需所有者先启用 |
| 相关设置 | `fastMode` 保存开关；`"fastModePerSessionOptIn": true` 让每个会话从关闭开始；`CLAUDE_CODE_DISABLE_FAST_MODE=1` 彻底禁用 |

回退行为：开启时若当前模型不支持，Claude Code 会切换到 Opus，提示词旁出现 `↯`；切到不支持的模型时自动关闭；触及快速模式速率限制时回退到标准速度和定价，`↯` 变灰，冷却结束后自动恢复；使用额度用完时以标准速度重试。对话中途首次开启要为整个上下文支付未缓存的输入价格，所以尽量在会话开始时开启。

### Advisor 工具

> 实验性功能。需要 Anthropic API、Claude Platform on AWS 或 Claude Models on Google Cloud (Anthropic managed)；Amazon Bedrock、Google Cloud 的 Agent Platform、Microsoft Foundry 不可用。

顾问让主模型在关键时刻（确定方案前、错误反复出现时、宣布完成前）咨询第二个通常更强的模型。顾问能看到完整对话；何时咨询由 Claude 决定，没有设置可以限制或强制调用。

| 方式 | 作用 |
|---|---|
| `/advisor opus` | 会话中设置并保存为默认值；不带参数打开选择器；`/advisor off` 关闭 |
| `"advisorModel": "opus"` | 设置文件里的持久默认值 |
| `claude --advisor opus` | 仅本次会话 |
| `CLAUDE_CODE_DISABLE_ADVISOR_TOOL=1` | 完全禁用顾问工具 |

限制：顾问的能力排名必须等于或高于主模型，否则不会附加（常见搭配是 Sonnet 主模型加 Opus 顾问）；每次调用都按顾问模型的费率重新处理完整对话，不走缓存；被 `availableModels` 排除的顾问不会被调用。与 `opusplan` 的区别：`opusplan` 在计划模式边界切换模型，顾问在任务中途的决策点介入。

### Code Mode

> 实验性功能，默认关闭，可能在任何版本中变化或被移除。

开启后，Claude 不再逐个请求工具调用，而是写一段 JavaScript 一次完成多次调用（并行运行命令、过滤结果后再读取）。对话里看到的仍是工具调用；按 `Ctrl+O` 可展开 **Code** 行阅读代码。适合在许多文件里重复的修改、要先过滤的搜索、可并行的独立命令。

启用：在 shell 或用户设置里把变量设为 `1`（项目设置里的这个变量会被忽略），首次启动时确认。

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_CODE_MODE": "1"
  }
}
```

要求：Claude Code v2.1.323 或更高的原生构建版本；Claude 订阅或 Console 账户（网关和第三方提供商不可用）；组织未使用 HIPAA 配置或零数据保留；模型为 Claude Opus 5.5、Sonnet 5.5、Haiku 5.5 或 Fable 5.2；只支持终端里的交互式 `claude` 会话。

限制：

- Code Mode 会话只能在终端、且 Code Mode 开启时恢复。
- 自动模式的分类器审查的是整段代码而不是每个工具调用；必须管住的调用请保留 `deny` / `ask` 规则或 `PreToolUse` hook。想先审代码再运行，给 `REPL` 加一条 `ask` 规则。
- 等待中的代码十分钟后停止；无法使用 Remote Control、Claude in Chrome 和计算机使用。
- 组织禁用：在托管设置里把该变量设为 `0`。

### 输出风格

输出风格是一组指令，决定 Claude 在整个会话中的角色、语气和响应格式。它是指令而不是保证。

| 内置风格 | 改变什么 |
|---|---|
| 默认（`default`） | 不加风格指令，使用标准的软件工程系统提示 |
| `Proactive` | 立即动手，对常规决策做合理假设而不是提问；不改变权限模式 |
| `Concise` | 以结果开头，省略前言、叙述和总结 |
| `Explanatory` | 额外输出 `Insight` 块，解释做出选择的原因 |
| `Learning` | 在 `Insight` 之外，用 `TODO(human)` 留下小段代码让你自己写 |

切换：`/output-style <style>`、`/config` 里的输出样式菜单，或设置文件的 `outputStyle`（命令和菜单写到 `.claude/settings.local.json`）。设置文件里的值区分大小写，写成 `explanatory` 会得到默认风格。

```json
{
  "outputStyle": "Explanatory"
}
```

自定义风格是一个 Markdown 文件，文件名即风格名，放在 `~/.claude/output-styles`（用户）、`.claude/output-styles`（项目）或托管设置目录内的 `.claude/output-styles`：

```markdown
---
name: Diagrams first
description: Lead every explanation with a diagram
keep-coding-instructions: true
---

When explaining code, architecture, or data flow, start with a Mermaid diagram showing the structure, then explain in prose.
```

| frontmatter 字段 | 目的 |
|---|---|
| `name` | 风格名称，默认是文件名 |
| `description` | 在 `/config` 选择器里显示的描述 |
| `keep-coding-instructions` | `true` 时保留内置的软件工程指令部分；默认 `false` |
| `force-for-plugin` | 仅插件风格：启用插件时自动应用，覆盖用户的 `outputStyle` |

终端在启动时读取风格文件，会话中新建或修改后要重启 Claude Code。

和其他机制的区别：输出风格作用于每个响应，而且自定义风格默认会省略内置的软件工程指令；`CLAUDE.md` 存放项目知识，无论选哪种风格都保持加载；`--append-system-prompt` 在启动时给系统提示追加内容，不删除任何东西；每次都必须发生的事用 hook。

### 终端配置

Claude Code 在任何终端里都能直接用；只有某个行为不符合预期时才需要这一节。

| 症状 | 处理 |
|---|---|
| 想换行而不提交 | `Ctrl+J`，或输入 `\` 再回车，任何终端都可用 |
| Shift+Enter 直接提交 | VS Code、Cursor、Zed 等运行一次 `/terminal-setup`；gnome-terminal、JetBrains IDE 终端不可用 |
| macOS 上 Option 快捷键无效 | 在终端里启用 "Use Option as Meta Key"（iTerm2 设为 "Esc+"） |
| Claude 完成时没有提醒 | `preferredNotifChannel` 设为 `"terminal_bell"`，或配置 Notification hook |
| Windows 上 Backspace 删掉整个单词 | 设置 `CLAUDE_CODE_BS_AS_CTRL_BACKSPACE=0` |
| 屏幕闪烁、滚动位置跳动 | `/tui fullscreen` 切到全屏渲染 |
| 想换主题 | `/theme` 或 `/config`；自定义主题是 `~/.claude/themes/` 里的 JSON（`name`、`base`、`overrides`） |
| 想用 Vim 键位编辑提示词 | `/config` → Editor mode，或 `editorMode` 设为 `"vim"` |

在 tmux 里 Shift+Enter 和桌面通知默认失效，往 `~/.tmux.conf` 加：

```bash
set -g allow-passthrough on
set -g focus-events on
set -s extended-keys on
set -as terminal-features 'xterm*:extkeys'
```

边界提醒：`/terminal-setup` 要在主机终端里直接运行，不要在 tmux 或 screen 里；Vim 模式的 INSERT 状态下 Enter 仍是提交；粘贴超过 800 个字符或三行会折叠成 `[Pasted text #1 +120 lines]` 这样的占位符，提交时仍发送完整内容。

### 全屏渲染

> 研究预览，行为可能根据反馈改变。

全屏渲染在终端的备用屏幕缓冲区上绘制界面（像 `vim` 或 `htop`），消除闪烁、让长对话内存保持平稳，并支持鼠标。"全屏"与窗口是否最大化无关。`/tui fullscreen` 切换并保存 `tui` 设置，`/tui default` 切回经典渲染器，也可以启动前设 `CLAUDE_CODE_NO_FLICKER=1`。

| 以前 | 现在 |
|---|---|
| `Cmd+f` 或 tmux 搜索 | `Ctrl+o` 进入记录模式，按 `/` 搜索，按 `[` 把对话写回终端原生滚动缓冲区 |
| 终端原生拖选复制 | 应用内选择，松开鼠标自动复制 |
| 直接点击链接 | macOS 上 `Cmd` 点击，其他平台 `Ctrl` 点击 |

鼠标捕获妨碍原生选择时，设 `CLAUDE_CODE_DISABLE_MOUSE=1`（仍保留无闪烁渲染）；只想保留滚轮就设 `CLAUDE_CODE_DISABLE_MOUSE_CLICKS=1`；`CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1` 强制经典渲染器。不要在 iTerm2 的 `tmux -CC` 集成模式里启用；显示错乱时按 `Ctrl+L` 重绘。

### 屏幕阅读器与无障碍

屏幕阅读器模式把终端界面换成线性的纯文本：没有边框字符、没有仅靠颜色的提示、没有原地重绘，菜单变成输入编号选择的列表。它不会因为屏幕阅读器在运行而自动开启。

| 开启方式（优先级从高到低） | 范围 |
|---|---|
| `claude --ax-screen-reader` | 单个会话 |
| `CLAUDE_AX_SCREEN_READER=1` | 从该 shell 启动的会话 |
| `"axScreenReader": true`（用户设置） | 这台机器上的每个会话 |

启动后第一行会确认，例如 `[Screen Reader Mode: on via settings]`。项目设置无法开启该模式。其他无障碍选项：`CLAUDE_CODE_ACCESSIBILITY=1` 让光标对屏幕放大镜保持可见，`prefersReducedMotion` 减少动画，`theme` 可选色盲友好的 `dark-daltonized`、`light-daltonized`。

### 语音听写

用说话代替打字：语音实时转录到提示词输入框，可与键盘输入混用。

| 命令 | 效果 |
|---|---|
| `/voice` | 切换开关，保持当前模式 |
| `/voice hold` | 按住模式（默认）：按住 `Space` 录音，松开停止 |
| `/voice tap` | 点击模式：点一次开始，再点一次发送 |
| `/voice off` | 禁用 |

```json
{
  "voice": {
    "enabled": true,
    "mode": "tap"
  }
}
```

要求与限制：

- 必须用 Claude.ai 账户登录；用 API 密钥、Amazon Bedrock、Google Cloud 的 Agent Platform 或 Microsoft Foundry 时不可用。
- 需要本地麦克风，云会话和 SSH 会话里不可用；音频会传到 Anthropic 服务器转录，不消耗消息或 token。
- 听写语言跟随 `language` 设置，为空或不在支持列表里时用英语。
- `Esc` 或 `Ctrl+C` 取消录音；听写键是 `Chat` 上下文里的 `voice:pushToTalk`，可在 `keybindings.json` 里改绑（如 `meta+k`）。

### 状态栏

状态栏是界面底部的一行自定义内容：Claude Code 运行你配置的命令，通过 stdin 传入会话 JSON，显示它打印到 stdout 的内容。它在本地运行，不消耗 API token。不想手写就用 `/statusline <自然语言描述>` 让 Claude Code 生成。

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2
  }
}
```

| 属性 | 目的 |
|---|---|
| `type` | 设为 `"command"` |
| `command` | 脚本路径或内联 shell 命令 |
| `padding` | 额外的水平间距（字符数），默认 `0` |
| `refreshInterval` | 除事件触发外每 N 秒重跑一次，最小值 `1` |
| `hideVimModeIndicator` | `true` 时隐藏内置的 `-- INSERT --` 文本 |

脚本输入 JSON 的主要字段：

| 字段 | 内容 |
|---|---|
| `model.id`、`model.display_name` | 当前模型标识符和显示名称 |
| `workspace.current_dir`、`workspace.project_dir` | 当前工作目录、启动 Claude Code 时所在目录 |
| `cost.total_cost_usd`、`cost.total_duration_ms` | 预估会话费用（客户端估算）、会话总时长 |
| `context_window.used_percentage`、`context_window.context_window_size` | 上下文已用百分比、窗口大小 |
| `rate_limits.five_hour.used_percentage`、`rate_limits.seven_day.used_percentage` | 5 小时和 7 天速率限制已用百分比 |
| `effort.level`、`fast_mode`、`thinking.enabled` | effort 级别、是否开启快速模式、是否启用思考 |
| `session_id`、`session_name`、`transcript_path`、`version` | 会话标识、名称、会话记录路径、Claude Code 版本 |
| `output_style.name`、`vim.mode`、`pr.number`、`worktree.name` | 输出样式、vim 模式、当前分支的 PR、worktree |

最小脚本，保存为 `~/.claude/statusline.sh` 并 `chmod +x`（需要 `jq`）：

```bash
#!/bin/bash
input=$(cat)

MODEL=$(echo "$input" | jq -r '.model.display_name')
DIR=$(echo "$input" | jq -r '.workspace.current_dir')
PCT=$(echo "$input" | jq -r '.context_window.used_percentage // 0' | cut -d. -f1)

echo "[$MODEL] ${DIR##*/} | ${PCT}% context"
```

边界提醒：

- 很多字段可能缺失或为 `null`（`vim`、`pr`、`rate_limits`、会话早期的 `used_percentage`），要给默认值，如 `// 0`。
- 脚本在会话启动、新的助手消息到达、`/compact` 完成、权限模式或 vim 模式变化时运行，有 300ms 防抖；`git status` 这类慢命令要做缓存。
- 终端宽度读 `COLUMNS` 和 `LINES` 环境变量，`tput cols` 读不到。
- 接受工作区信任对话框之前状态栏保持空白；排错用 `claude --debug`。

### 键位绑定

运行 `/keybindings` 创建或打开 `~/.claude/keybindings.json`，改动会被自动检测并应用，无需重启。文件是一个带 `bindings` 数组的对象；每个块指定一个上下文和"按键 → 操作"的映射。操作采用 `namespace:action` 格式，设为 `null` 表示取消绑定。

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+s": null,
        "meta+g": "command:compact"
      }
    }
  ]
}
```

| 常用上下文 | 适用范围 |
|---|---|
| `Global` | 应用的任何地方 |
| `Chat` | 主聊天输入区域 |
| `Autocomplete` | 自动完成菜单打开时 |
| `Confirmation` | 权限和确认对话框 |
| `Transcript` | 记录查看器 |
| `HistorySearch` | 历史记录搜索模式 |
| `Select` | 通用选择 / 列表组件 |
| `Scroll` | 对话滚动和全屏模式下的文本选择 |

常用操作及默认键：`chat:submit`（Enter）、`chat:newline`（Ctrl+J）、`chat:cycleMode`（Shift+Tab）、`chat:modelPicker`（Meta+P）、`chat:fastMode`（Meta+O）、`app:toggleTranscript`（Ctrl+O）、`command:<name>`（运行 `/<name>`，默认未绑定）。完整的上下文和操作列表见官方页面。

```text
ctrl+k          Ctrl + K
shift+tab       Shift + Tab
meta+p          macOS 上的 Option + P，其他地方的 Alt + P
ctrl+k ctrl+s   和弦：按 Ctrl+K，释放，再按 Ctrl+S（间隔 3 秒内）
```

修饰符有 `ctrl`、`shift`、`alt` / `opt` / `option` / `meta`、`cmd` / `command` / `super` / `win`。`cmd` 组只有少数终端会上报，想在任何终端都生效就用 `ctrl` 或 `meta`。

| 快捷键 | 说明 |
|---|---|
| Ctrl+C、Ctrl+D | 保留：硬编码的中断 / 取消、退出 |
| Ctrl+M、Ctrl+\[、Ctrl+I | 保留：始终被接收为 Enter、Escape、Tab |
| Ctrl+H、Caps Lock | 保留：发送 ASCII 退格字节；不传递到终端应用程序 |
| Ctrl+B、Ctrl+A、Ctrl+Z | 可能冲突：tmux 前缀、GNU screen 前缀、Unix 进程暂停 |

边界提醒：

- Vim 的动作键不能通过这个文件重新映射；把 `jj` 之类映射到 Escape 用 `vimInsertModeRemaps` 设置。
- `ctrl+x` 默认是多个和弦的前缀，要当单键用，必须在定义这些和弦的各个上下文里逐一取消绑定。
- 无效 JSON、拼错的修饰符（如 `ctl+k`）、未知操作名只会写入调试日志，用 `claude --debug` 查看。
- 这个文件改的是 Claude Code 如何响应按键；终端有没有把按键正确发过来，属于"终端配置"。

---

## Part VI：参考

### CLI 参考：命令与启动标志

`claude` 既是交互式会话的入口，也是脚本里的一条普通命令。入门只需记住：直接运行进入对话，`-p` 跑完就退出，`-c` / `-r` 接着上次聊。`claude --help` 不会列出每个标志，没出现在 `--help` 里不代表不可用。

| 命令 | 作用 | 示例 |
|---|---|---|
| `claude` | 启动交互式会话 | `claude` |
| `claude "query"` | 带初始提示词启动交互式会话 | `claude "explain this project"` |
| `claude -p "query"` | 通过 SDK 查询，然后退出 | `claude -p "explain this function"` |
| `cat file \| claude -p "query"` | 处理管道内容 | `cat logs.txt \| claude -p "explain"` |
| `claude -c` | 继续当前目录中最近的对话 | `claude -c` |
| `claude -r "<session>" "query"` | 按 ID 或名称恢复会话 | `claude -r "auth-refactor" "Finish this PR"` |
| `claude update` | 更新到最新版本 | `claude update` |
| `claude auth login` / `logout` / `status` | 登录、登出、查看身份验证状态（`status` 已登录退出码 0，未登录为 1） | `claude auth login --console` |
| `claude setup-token` | 为 CI 和脚本生成长期 OAuth 令牌，只打印不保存；需要 Claude 订阅 | `claude setup-token` |
| `claude doctor` | 不启动会话，打印只读的安装与设置诊断 | `claude doctor` |
| `claude mcp` | 配置 MCP 服务器；`claude mcp login <name>` / `logout <name>` 管理 OAuth | `claude mcp login sentry` |
| `claude plugin` | 管理插件，别名 `claude plugins` | `claude plugin install code-review@claude-plugins-official` |
| `claude agents` | 打开 agent view，监控和分派并行后台会话 | `claude agents --json` |
| `claude attach` / `logs` / `stop` / `respawn` / `rm` | 附加、查看输出、停止、重启、删除后台会话 | `claude attach 7c5dcf5d` |

#### 最常用的启动标志

| 标志 | 作用 | 示例 |
|---|---|---|
| `--print`, `-p` | 打印回复而不进入交互模式 | `claude -p "query"` |
| `--continue`, `-c` | 加载当前目录中最近的对话 | `claude --continue` |
| `--resume`, `-r` | 按 ID 或名称恢复会话，或显示交互式选择器 | `claude --resume auth-refactor` |
| `--fork-session` | 恢复时创建新会话 ID（配合 `--resume` 或 `--continue`） | `claude --resume abc123 --fork-session` |
| `--name`, `-n` | 设置会话显示名称，之后可 `claude --resume <name>` | `claude -n "my-feature-work"` |
| `--model` | 用别名（`sonnet`、`opus`、`haiku`、`fable`）或完整名称设置本次会话模型 | `claude --model claude-sonnet-5` |
| `--fallback-model` | 主模型过载或不可用时回退，接受逗号分隔列表 | `claude --fallback-model sonnet,haiku` |
| `--effort` | effort 级别：`low`、`medium`、`high`、`xhigh`、`max` 或 `ultracode` | `claude --effort high` |
| `--permission-mode` | 以指定权限模式启动：`default`、`acceptEdits`、`plan`、`auto`、`dontAsk`、`bypassPermissions`，或别名 `manual` | `claude --permission-mode plan` |
| `--dangerously-skip-permissions` | 跳过权限提示，等同 `--permission-mode bypassPermissions` | `claude --dangerously-skip-permissions` |
| `--allowedTools`, `--allowed-tools` | 无需提示即可执行的工具（权限规则语法） | `"Bash(git log *)" "Bash(git diff *)" "Read"` |
| `--disallowedTools`, `--disallowed-tools` | 拒绝规则；裸工具名会把该工具从上下文中移除 | `"Bash(git log *)" "Bash(git diff *)" "Edit"` |
| `--tools` | 限制可用的内置工具；`""` 全部禁用，`"default"` 用默认集；不影响 MCP 工具 | `claude --tools "Bash,Edit,Read"` |
| `--add-dir` | 添加额外工作目录（只授予文件访问，不发现其中大多数 `.claude/` 配置） | `claude --add-dir ../apps ../lib` |
| `--settings` | 设置 JSON 文件路径或内联 JSON，本次会话覆盖同名键 | `claude --settings ./settings.json` |
| `--mcp-config` | 从 JSON 文件或字符串加载 MCP 服务器 | `claude --mcp-config ./mcp.json` |
| `--agent` | 为当前会话指定 Agent（覆盖 `agent` 设置） | `claude --agent my-custom-agent` |
| `--plugin-dir` | 仅本次会话从目录或 `.zip` 加载插件，可重复 | `claude --plugin-dir ./my-plugin` |
| `--worktree`, `-w` | 在 `<repo>/.claude/worktrees/<name>` 的隔离 git worktree 中启动 | `claude -w feature-auth` |
| `--bg`, `--background` | 作为后台 Agent 启动并立即返回；不能与 `-p` 结合 | `claude --bg "investigate the flaky test"` |
| `--bare` | 最小模式：跳过 hook、skill、插件、MCP、自动记忆和 CLAUDE.md 的自动发现 | `claude --bare -p "query"` |
| `--safe-mode` | 禁用所有自定义来排查损坏的配置；认证、模型、内置工具和权限正常 | `claude --safe-mode` |
| `--debug` | 调试模式，可按类别过滤 | `claude --debug='mcp,startup'` |
| `--cloud` | 用任务描述创建新的云端会话（`--remote` 是其已弃用别名） | `claude --cloud "Fix the login bug"` |
| `--output-format` | 仅 print 模式。输出格式：`text`、`json`、`stream-json` | `claude -p "query" --output-format json` |
| `--json-schema` | 仅 print 模式。得到符合 JSON Schema 的已验证 JSON 输出 | `claude -p --json-schema '{"type":"object","properties":{...}}' "query"` |
| `--max-turns` | 仅 print 模式。限制 Agent 轮次数，默认无限制 | `claude -p --max-turns 3 "query"` |
| `--max-budget-usd` | 仅 print 模式。估算支出达到此金额即停止（客户端估算，可能与账单不同） | `claude -p --max-budget-usd 5.00 "query"` |
| `--permission-prompts` | 仅 print 模式。无人可答权限提示时传 `none`，改为直接拒绝 | `claude -p --permission-prompts none "query"` |

#### 系统提示词标志对比

共五个标志，交互和非交互模式都可用。

| 标志 | 行为 | 示例 |
|---|---|---|
| `--system-prompt` | 替换整个默认提示词 | `claude --system-prompt "You are a Python expert"` |
| `--system-prompt-file` | 用文件内容替换 | `claude --system-prompt-file ./prompts/review.txt` |
| `--append-system-prompt` | 附加到默认提示词 | `claude --append-system-prompt "Always use TypeScript"` |
| `--append-system-prompt-file` | 将文件内容附加到默认提示词 | `claude --append-system-prompt-file ./style-rules.txt` |
| `--system-prompt-snapshot` | `off` 时每个请求重建提示词；`on`（默认）时重用已记录的提示词 | `claude --append-system-prompt "Draft rules" --system-prompt-snapshot off` |

- **怎么选**：Claude 仍是编码助手、只是多几条规则，用 append 系列，默认的工具指导、安全指令和编码约定都保留。身份、入口或权限模型都不同才用 replace 系列，此时整个默认提示词被删除，缺什么要自己补。两类可以组合。
- **恢复对话时不立刻生效**：系统提示词默认在对话的第一个请求上构建一次并记录。之后用 `--resume` / `--continue` 回来换了标志文本，要等对话被压缩或开始新对话才生效；想立刻生效传 `--system-prompt-snapshot off`。
- 长期角色用输出样式，项目约定用 CLAUDE.md，不要都塞进启动标志。

### 命令速查（`/` 命令）

在会话里输入 `/` 查看可用命令，继续输入字母可筛选。命令只在消息开头被识别，名称后面的文本是参数。

内置命令与 skill 的关系：大多数是**内置命令**，行为写死在 CLI 里；标记为 **Skill** 的是随附 skill，和你自己写的 skill 一样，本质是交给 Claude 的提示词；另有少量随附**工作流**（如 `/deep-research`），把工作分发给多个子代理并在后台运行。想添加自己的命令就写 skill，MCP 服务器公开的 prompts 也会显示为命令。终端里一条消息开头可以叠加多个 skill（`/skill-a /skill-b do XYZ`），最多链接六个。

> 并非每个命令都对每个用户显示，可用性取决于平台、套餐和环境。`<arg>` 为必需参数，`[arg]` 为可选参数。

| 分组 | 命令 | 用途 |
|---|---|---|
| 会话 | `/clear [name]` | 用空上下文开始新对话；别名 `/reset`、`/new` |
| 会话 | `/resume [session]` | 按 ID 或名称恢复对话，或打开会话选择器；别名 `/continue` |
| 会话 | `/branch [name]` | 在此处创建当前对话的分支并切换过去，原对话可用 `/resume` 返回 |
| 会话 | `/rewind` | 把对话和/或代码回退到之前的时间点，或从选定消息开始总结；别名 `/checkpoint`、`/undo` |
| 会话 | `/rename [name]` | 重命名当前会话；不给名称则自动生成 |
| 会话 | `/exit` | 退出 CLI；别名 `/quit` |
| 上下文 | `/compact [instructions]` | 总结到目前为止的对话以释放上下文，可传摘要重点 |
| 上下文 | `/context [all]` | 以彩色网格可视化当前上下文使用情况 |
| 上下文 | `/btw [question]` | 提附带问题，不加入对话历史 |
| 模型 | `/model [model]` | 切换模型并保存为新会话默认值 |
| 模型 | `/effort [level\|auto\|status\|ultracode [on\|off]]` | 设置 effort 级别：`low` 到 `xhigh`、`max` 或 `auto` |
| 模型 | `/plan [description]` | 直接进入计划模式，可带任务描述 |
| 模型 | `/goal [condition\|clear]` | 设置目标，Claude 跨轮次持续工作直到条件满足 |
| 配置 | `/config [key=value ...]` | 打开设置界面，或直接设置某项，如 `/config theme=dark`；别名 `/settings` |
| 配置 | `/init` | 用 `CLAUDE.md` 指南初始化项目 |
| 配置 | `/memory` | 编辑 `CLAUDE.md`，启用或禁用自动记忆 |
| 配置 | `/update-config [request]` | **Skill。** 描述一项设置更改，由 Claude 编辑相应的 `settings.json` |
| 权限 | `/permissions` | 管理工具权限的允许、询问和拒绝规则；别名 `/allowed-tools` |
| 权限 | `/add-dir <path>` | 添加一个工作目录以访问文件 |
| 权限 | `/fewer-permission-prompts` | **Skill。** 扫描会话记录，把常见只读调用加入项目允许列表 |
| 扩展 | `/mcp [reconnect (<server>\|all)\|enable\|disable [<server>\|all]]` | 管理 MCP 服务器连接和 OAuth 身份验证 |
| 扩展 | `/plugin [subcommand]` | 管理插件；改动后用 `/reload-plugins` 免重启生效 |
| 扩展 | `/skills` | 列出可用的 skill |
| 扩展 | `/hooks` | 查看 hook 配置（只读浏览器） |
| 并行 | `/tasks` | 查看和管理当前会话中的后台工作；也可作为 `/bashes` 使用 |
| 并行 | `/background [prompt]` | 把当前会话分离为后台 Agent 并释放终端；别名 `/bg` |
| 并行 | `/fork [prompt]` | 把当前对话复制到新的后台会话，并在此处继续工作 |
| 并行 | `/batch <instruction>` | **Skill。** 把大规模更改分解为 5 到 30 个独立单元，各自在隔离 worktree 中并行执行 |
| 并行 | `/loop [interval] [prompt]` | **Skill。** 会话保持打开期间重复运行提示词；别名 `/proactive` |
| 并行 | `/schedule [description]` | 创建、更新、列出或运行云端定时任务；别名 `/routines` |
| 并行 | `/teleport`、`/remote-control` | 把云端会话拉到此终端（`/tp`）；让此会话可从 claude.ai 远程控制（`/rc`） |
| 审查 | `/diff` | 审查工作树中的更改，包括 Claude 所做的编辑 |
| 审查 | `/code-review [low\|medium\|high\|xhigh\|max\|ultra] [--fix] [--comment] ...` | **Skill。** 审查当前 diff 或指定的 PR、分支、路径，查找正确性 bug；别名 `/review` |
| 审查 | `/security-review` | 分析当前分支上的更改是否存在安全漏洞；需要 `origin` 远程 |
| 审查 | `/simplify [target]` | **Skill。** 审查已更改代码的清理机会并应用修复；不查找正确性 bug |
| 诊断 | `/doctor [prompt-audit [path]]` | **Skill。** 运行设置检查并提出修复建议；别名 `/checkup` |
| 诊断 | `/debug [description]` | **Skill。** 为当前会话启用调试日志并读取日志排查问题 |
| 诊断 | `/status` | 显示版本、模型、账户和连接状态 |
| 诊断 | `/usage` | 显示会话开销、套餐用量限制和活动统计；`/cost` 和 `/stats` 是其别名 |
| 诊断 | `/help`、`/feedback [report]` | 显示帮助和可用命令；发送产品反馈 |
| 账户 | `/login`、`/logout` | 登录或注销 Anthropic 账户 |

- Claude 响应时发送的命令会排队到当前轮次结束再运行；`/status`、`/tasks`、`/usage` 等会立即运行。
- 旧教程里的这些命令已移除：`/vim`（改用 `/config` → Editor mode）、`/pr-comments`（直接让 Claude 查看 PR 评论）、`/ultraplan`（改用计划模式）。

### 环境变量

环境变量控制模型选择、身份验证、请求路由和功能开关。shell 里设置的只在该终端会话有效；写在 `settings.json` 的 `env` 键下则每次运行 `claude` 都会应用：

```bash
export API_TIMEOUT_MS="1200000"
claude
```

```json
{
  "env": {
    "API_TIMEOUT_MS": "1200000",
    "BASH_DEFAULT_TIMEOUT_MS": "300000"
  }
}
```

写在哪个设置文件决定作用范围：`~/.claude/settings.json`（你，所有项目）、`.claude/settings.json`（项目里每个人）、`.claude/settings.local.json`（你，仅此项目）、托管设置（组织里每个人）。`env` 里的值不经过 shell 处理，`~` 和 `$HOME` 保持原样，路径要写绝对路径。

| 分类 | 变量 | 作用 |
|---|---|---|
| 认证 | `ANTHROPIC_API_KEY` | 作为 `X-Api-Key` 标头发送的 API 密钥；设置后代替订阅使用（交互模式需批准一次，`-p` 下始终使用） |
| 认证 | `ANTHROPIC_AUTH_TOKEN` | `Authorization` 标头的自定义值，会加上 `Bearer ` 前缀 |
| 认证 | `CLAUDE_CODE_OAUTH_TOKEN` | claude.ai 身份验证的 OAuth 访问令牌，用 `claude setup-token` 生成；项目和本地设置中被忽略 |
| 模型 | `ANTHROPIC_MODEL` | 要使用的模型设置的名称；会被 `--model` 和 `/model` 覆盖 |
| 模型 | `ANTHROPIC_DEFAULT_OPUS_MODEL` | `opus` 别名解析到的模型 ID |
| 模型 | `ANTHROPIC_DEFAULT_SONNET_MODEL` | `sonnet` 别名解析到的模型 ID |
| 模型 | `ANTHROPIC_DEFAULT_HAIKU_MODEL` | `haiku` 别名解析到的模型 ID，也用于后台功能 |
| 模型 | `CLAUDE_CODE_SUBAGENT_MODEL` | 未另行指定模型的子代理、agent team 队友和工作流 Agent 的默认模型 |
| 模型 | `CLAUDE_CODE_EFFORT_LEVEL` | effort 级别：`low`、`medium`、`high`、`xhigh` 或 `max`；优先于 `--effort` 和 `/effort` |
| 模型 | `MAX_THINKING_TOKENS` | 扩展思考的固定 token 预算 |
| 云厂商 | `CLAUDE_CODE_USE_BEDROCK` | 使用 Amazon Bedrock |
| 云厂商 | `CLAUDE_CODE_USE_VERTEX` | 使用 Google Cloud's Agent Platform |
| 云厂商 | `CLAUDE_CODE_USE_FOUNDRY` | 使用 Microsoft Foundry |
| 云厂商 | `CLAUDE_CODE_USE_ANTHROPIC_AWS` | 使用 Claude Platform on AWS |
| 云厂商 | `AWS_BEARER_TOKEN_BEDROCK` | 用于身份验证的 Amazon Bedrock API 密钥 |
| 网络 | `ANTHROPIC_BASE_URL` | 覆盖 API 端点，通过代理或网关路由请求 |
| 网络 | `ANTHROPIC_CUSTOM_HEADERS` | 添加到请求的自定义标头（`Name: Value` 格式，多个以换行分隔） |
| 网络 | `HTTP_PROXY`、`HTTPS_PROXY` | 为网络连接指定 HTTP / HTTPS 代理服务器 |
| 网络 | `NO_PROXY` | 绕过代理直接发送的域名和 IP 列表 |
| 超时限额 | `API_TIMEOUT_MS` | 每次 API 请求尝试接收响应标头的时间，默认 600000（10 分钟）；不限制之后流式响应的时长 |
| 超时限额 | `BASH_DEFAULT_TIMEOUT_MS` | 前台 Bash 或 PowerShell 命令的默认超时，默认 120000（2 分钟） |
| 超时限额 | `BASH_MAX_TIMEOUT_MS` | 模型可为前台命令设置的最大超时，默认 600000（10 分钟） |
| 超时限额 | `BASH_MAX_OUTPUT_LENGTH` | 读回到命令结果中的输出最大字符数，默认 30000，最大 150000 |
| 超时限额 | `MCP_TIMEOUT` | MCP 服务器启动的超时，默认 30000（30 秒） |
| 超时限额 | `MCP_TOOL_TIMEOUT` | MCP 工具执行的超时，默认 100000000（约 28 小时） |
| 超时限额 | `MAX_MCP_OUTPUT_TOKENS` | MCP 工具响应允许的最大 token 数，默认 25000 |
| 超时限额 | `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | 自动压缩窗口，`100000` 或更大的 token 数 |
| 行为 | `CLAUDE_CONFIG_DIR` | 覆盖配置目录（默认 `~/.claude`），适合同时运行多个账户；项目和本地设置中被忽略 |
| 行为 | `CLAUDE_ENV_FILE` | 一个 shell 脚本路径，其内容在每个 Bash 命令之前运行，用来保持 virtualenv 或 conda 激活 |
| 行为 | `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` | 主会话中每个 Bash 或 PowerShell 命令执行后返回原始工作目录 |
| 行为 | `CLAUDE_CODE_SIMPLE`、`CLAUDE_CODE_SAFE_MODE` | 设为 `1` 分别等同传递 `--bare`、`--safe-mode` |
| 行为 | `ENABLE_TOOL_SEARCH` | 控制 MCP 工具搜索：`true`、`auto`、`auto:N` 或 `false` |
| 行为 | `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION` | 设为 `false` 关闭提示词建议，优先于 `promptSuggestionEnabled` 设置 |
| 行为 | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | 设为 `1` 启用 agent team（实验性，默认禁用） |
| 行为 | `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | 设为 `1` 从 Bash 命令、hook 和 stdio MCP 服务器等子进程环境中剥离凭据 |
| 遥测 | `CLAUDE_CODE_ENABLE_TELEMETRY` | 设为 `1` 启用 OpenTelemetry 数据收集；项目和本地设置中被忽略 |
| 遥测 | `DISABLE_TELEMETRY` | 设为任意非空值退出遥测，同时禁用功能标志获取 |
| 遥测 | `DISABLE_ERROR_REPORTING` | 设为任意非空值退出错误报告 |
| 遥测 | `DO_NOT_TRACK` | 设为 `1` 退出遥测，效果与 `DISABLE_TELEMETRY` 相同 |
| 遥测 | `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 设为任意非空值禁用非必要网络流量：自动更新、遥测、错误报告、`/feedback` 等 |
| 禁用 | `DISABLE_AUTOUPDATER` | 设为 `1` 禁用自动后台更新；手动 `claude update` 仍有效 |
| 禁用 | `DISABLE_UPDATES` | 设为 `1` 阻止所有更新，包括手动 `claude update` 和 `claude install` |
| 禁用 | `DISABLE_AUTO_COMPACT` | 设为 `1` 禁用自动压缩和空闲压缩；手动 `/compact` 仍可用 |
| 禁用 | `DISABLE_COMPACT` | 设为 `1` 禁用所有压缩，包括手动 `/compact` |
| 禁用 | `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | 设为 `1` 禁用自动记忆（设为 `0` 则强制启用） |
| 禁用 | `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` | 设为 `1` 移除 `run_in_background` 参数、关闭 `Ctrl+B` 和自动转后台 |
| 禁用 | `CLAUDE_CODE_DISABLE_FILE_CHECKPOINTING` | 设为 `1` 禁用文件检查点，`/rewind` 将无法恢复代码更改 |
| 禁用 | `CLAUDE_CODE_DISABLE_CLAUDE_MDS` | 设为 `1` 阻止加载任何 CLAUDE.md 记忆文件 |
| 禁用 | `CLAUDE_CODE_DISABLE_WEB_FETCH` | 设为 `1` 关闭 WebFetch 工具；WebSearch 仍可用 |

- `DISABLE_TELEMETRY` 和 `DISABLE_ERROR_REPORTING` 设成 `0` 或 `false` **仍然是退出**，只有取消设置才会重新开启。`DO_NOT_TRACK` 是标准布尔值，`0` 表示保持开启。
- 同一变量在 shell 和 `env` 块里都设置时，大多数会话中设置文件的值生效。设置文件里不能删除变量；要覆盖 shell 里甩不掉的值，设为空字符串，如 `"CLAUDE_CODE_USE_VERTEX": ""`。
- 与标志的优先级不统一：`--model` 和 `/model` 覆盖 `ANTHROPIC_MODEL`，而 `CLAUDE_CODE_EFFORT_LEVEL` 覆盖 `--effort` 和 `/effort`。shell 变量在启动时读取，改完要重启 `claude`。

### 工具参考

工具名称是你在权限规则、子代理工具列表和 hook 匹配器里要写的**确切字符串**。"需要权限"指手动模式下、对工作目录内的路径是否提示：标"否"的 `Read`、`Grep`、`Glob` 对工作目录之外的路径仍会提示；`Bash` 标"是"，但内置的只读命令不提示。

| 工具 | 作用 | 需要权限 |
|---|---|---|
| `Agent` | 生成拥有独立上下文窗口的子代理 | 否 |
| `AskUserQuestion` | 提出多选问题以收集要求或澄清歧义 | 否 |
| `Bash` | 在你的环境中执行 shell 命令 | 是 |
| `Edit` | 对特定文件做有针对性的编辑（精确字符串替换） | 是 |
| `Write` | 创建或覆盖文件 | 是 |
| `Read` | 读取文件内容，含图像、PDF、Jupyter 笔记本 | 否 |
| `NotebookEdit` | 修改 Jupyter notebook 单元格 | 是 |
| `Glob` | 基于模式匹配查找文件；macOS、Linux 和 WSL 上默认不存在 | 否 |
| `Grep` | 在文件内容中搜索模式；macOS、Linux 和 WSL 上默认不存在 | 否 |
| `LSP` | 通过语言服务器提供代码智能，需先安装 code intelligence 插件 | 否 |
| `WebFetch` | 从指定 URL 获取内容 | 是 |
| `WebSearch` | 执行网络搜索 | 是 |
| `Skill` | 在主对话中执行 skill | 部分 skill |
| `ToolSearch` | 工具搜索启用时，搜索并加载延迟工具 | 否 |
| `Monitor` | 后台运行命令并把每行输出反馈给 Claude | 是 |
| `PowerShell` | 本地执行 PowerShell 命令 | 是 |
| `EnterPlanMode` / `ExitPlanMode` | 切换到计划模式 / 呈现计划以供批准并退出 | 否 / 是 |
| `EnterWorktree` / `ExitWorktree` | 创建并进入隔离的 git worktree / 退出并返回原始目录 | 部分情况 |
| `TaskCreate` / `TaskGet` / `TaskList` / `TaskUpdate` | 任务列表的创建、查询、列出、更新 | 否 |
| `TaskStop` | 按 ID 停止运行中的后台任务 | 否 |
| `CronCreate` / `CronDelete` / `CronList` | 会话级定时任务的创建、取消、列出 | 否 |

官方表还有 `Artifact`、`Workflow`（两者需要权限）、`SendMessage`、`ListMcpResourcesTool`、`ReadMcpResourceTool`、`TodoWrite`（默认禁用，由 Task 工具取代）、`EndConversation`、`PushNotification`、`RemoteTrigger`、`ScheduleWakeup`、`SendFeedback`、`SendUserFile` 等，以及只在云端会话或桌面应用出现的 `ListPlugins`、`SearchSkills` 一类工具。确切工具集取决于提供商、平台和设置，想知道当前会话加载了什么，直接问 Claude：`What tools do you have access to?`

权限规则统一写成 `ToolName(specifier)`：`Bash(npm run *)`（也适用于 Monitor）、`Read(~/secrets/**)`（也适用于 Grep、Glob、LSP）、`Edit(/src/**)`（也适用于 Write、NotebookEdit）、`Skill(deploy *)`、`Agent(Explore)`、`WebFetch(domain:example.com)`；`WebSearch` 没有 specifier。Hook 的 `matcher` 用裸工具名，不用带括号的格式。

#### Bash 工具行为要点

Bash 工具在单独的进程中运行每个命令。

| 主题 | 行为 |
|---|---|
| 工作目录 | 主会话里的 `cd` 会传递到后续命令，前提是仍在项目目录或额外工作目录内；越界会被重置并附加 `Shell cwd was reset to <dir>`。子代理会话永远不传递 |
| 环境变量 | **不会保留**。一个命令里的 `export` 在下一个命令中不可用 |
| shell 配置 | 启动非交互式 shell，Zsh 加载 `~/.zshrc`，Bash 加载 `~/.bashrc` |
| 超时 | 默认两分钟，上限十分钟；由 Claude 按需传 `timeout`，你无法设置每个命令的超时 |
| 输出 | 有效结果内联最多约 30,000 个字符，超出则给文件路径加最多前 2,000 个字符的预览；失败结果内联最多约 10,000 个字符 |
| 后台运行 | Claude 可设 `run_in_background: true`；用 `/tasks` 列出和停止 |
| 超时转后台 | 前台命令到达超时会被移到后台而不是停止，除非命令以 `sleep` 开头 |
| 后台时限 | 本地会话（终端、桌面应用、VS Code 扩展）无时限；`-p`、Agent SDK、CI、云端会话默认 30 分钟，最多 2 小时 |

需要 virtualenv 或 conda 时，在启动 Claude Code **之前**激活，或把 `CLAUDE_ENV_FILE` 设为一个 shell 脚本。想让每个命令都从项目目录开始，设 `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1`。

#### 其他工具的关键行为

| 工具 | 要点 |
|---|---|
| `Edit` | `old_string` 必须与文件完全一致且恰好出现一次，否则给更长的上下文或设 `replace_all: true`；不用正则或模糊匹配 |
| `Write` | 用完整内容创建或覆盖文件，不追加不合并 |
| `Read` | 单次调用默认最多返回 25,000 个 token，并拒绝整读超过 256 KB 的文件，大文件用 `offset` 和 `limit` 分页；超过 10 页的 PDF 用 `pages` 按范围读，一次最多 20 页；不读目录 |
| `Glob` | 结果按修改时间排序，最多 100 个文件；默认**不**遵守 `.gitignore` |
| `Grep` | 基于 ripgrep，用 ripgrep 正则语法；遵守 `.gitignore`；输出模式 `files_with_matches`（默认）、`content`、`count` |
| `WebFetch` | 多数页面 Claude 看不到全文，而是针对页面提出问题，由单独的模型调用读取并返回答案 |
| `WebSearch` | 只返回结果标题和 URL；交互式终端会话最多 200 次调用。Amazon Bedrock 不公开服务器端网络搜索工具 |
| `Monitor` | 每个监视默认 5 分钟、最多 30 分钟；使用与 Bash 相同的权限规则；Windows 上需要 Git Bash |
| `Agent` | 父对话只看到子代理的最终结果；启动子代理本身不提示权限，子代理的工具调用照常按权限规则检查 |

- macOS、Linux 和 WSL 上默认没有 `Glob` 和 `Grep`，Claude 改用 Bash 里的 `find` 和 `grep`，这些搜索以 `Bash` 调用的形式到达你的 hook 和权限规则。只针对 `Grep` 写的 hook 在这些平台上默认不会触发。
- `PowerShell` 工具在没有 Git Bash 的 Windows 上自动启用，在 Linux、macOS 和 WSL 上是可选的。检查 shell 命令的 hook 应匹配 `Bash|PowerShell`。
- Task 跟踪工具默认只在 Opus 4.7、Sonnet 4.6、Haiku 4.5 及更早的模型上可用；其他模型用 `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` 选择加入。

### 交互模式

快捷键可能因平台和终端而异。macOS 上 `Alt+B`、`Alt+F`、`Alt+D`、`Alt+Y`、`Alt+P` 需要在终端里把 Option 配置为 Meta。

| 分组 | 快捷键 | 作用 |
|---|---|---|
| 控制 | `Ctrl+C` | 中断正在运行的操作；空闲时第一次清除输入，第二次退出 |
| 控制 | `Ctrl+D` | 退出会话（第一次显示确认，800ms 内再按一次退出） |
| 控制 | `Esc` | 中断 Claude 或关闭对话框；在权限提示上等于拒绝 |
| 控制 | `Esc` + `Esc` | 输入有文本时清除草稿；输入为空时打开回退菜单 |
| 控制 | `Shift+Tab` | 循环权限模式（部分 Windows 环境为 `Alt+M`） |
| 控制 | `Ctrl+B` | 把 Bash 命令和代理放到后台；tmux 用户按两次 |
| 控制 | `Ctrl+Enter` 或 `Ctrl+X Ctrl+S` | 立即发送排队的消息 |
| 视图 | `Ctrl+T` | 切换 Claude 的任务清单 |
| 视图 | `Ctrl+O` | 切换记录查看器 |
| 视图 | `Ctrl+R` | 反向搜索命令历史 |
| 输入 | `Ctrl+G` 或 `Ctrl+X Ctrl+E` | 在默认文本编辑器中编辑提示 |
| 输入 | `Ctrl+V`（iTerm2 为 `Cmd+V`，Windows 和 WSL 为 `Alt+V`） | 从剪贴板粘贴图像 |
| 输入 | `Up/Down arrows` 或 `Ctrl+P`/`Ctrl+N` | 移动光标或导航命令历史 |
| 输入 | `Tab` | 接受自动完成建议，或给权限答案添加注释 |
| 输入 | `Ctrl+A` / `Ctrl+E` | 光标移到当前行开始 / 末尾 |
| 输入 | `Ctrl+K` / `Ctrl+U` | 删除到行尾 / 从光标删除到行开始 |
| 输入 | `Ctrl+W` | 删除回到上一个空格（一次删掉整个路径或 `--flag=value`） |
| 输入 | `Alt+B` / `Alt+F` | 光标向后 / 向前移动一个单词 |
| 切换 | `Option+P`（macOS）或 `Alt+P`（Windows/Linux） | 切换模型 |
| 切换 | `Option+T`（macOS）或 `Alt+T`（Windows/Linux） | 切换扩展思考 |
| 切换 | `Option+O`（macOS）或 `Alt+O`（Windows/Linux） | 切换快速模式 |

#### 多行输入与快速前缀

| 多行方法 | 快捷键 | 适用 |
|---|---|---|
| 快速转义 | `\` + `Enter` | 所有终端 |
| 控制序列 | `Ctrl+J` | 任何终端，无需配置 |
| Shift+Enter | `Shift+Enter` | iTerm2、WezTerm、Ghostty、Kitty、Warp、Windows Terminal 1.25 或更高版本原生支持；其他终端运行 `/terminal-setup` |
| Option 键 | `Option+Enter` | macOS 上启用 Option 作为 Meta 后 |

| 前缀 | 作用 |
|---|---|
| `/` 在开始 | 命令或 skill |
| `!` 在开始 | Shell 模式：直接运行命令，输出加入会话，Claude 对其进行响应 |
| `@` | 文件路径提及，触发路径自动完成 |
| `:` | 表情符号简码，键入完整的 `:name:` |
| `?` 在空输入上 | 切换快捷键帮助面板 |

Shell 模式（如 `! npm test`）的边界：命令不经过 Claude 解释或批准；即使启用了沙箱，通常也在沙箱**外**运行，因为沙箱只管 Claude 运行的命令。输出出现后 Claude 会自动回复，成本与一条普通提示词相同；只想把输出放进上下文就把 `respondToBashCommands` 设为 `false`。

#### Vim 模式

通过 `/config` → Editor mode 启用，或在用户设置里写 `"editorMode": "vim"`。支持 `Esc` 或 `Ctrl+[` 回到 NORMAL，`i` `I` `a` `A` `o` `O` 进入 INSERT，`v` / `V` 可视选择；导航 `h`/`j`/`k`/`l`、`w` `e` `b`、`0` `$` `^`、`gg` `G`、`f{char}` `t{char}`；编辑 `x`、`dd`、`cc`、`yy`、`p`、`u`、`.`；文本对象 `iw`/`aw`、`i"`/`a"`、`i(`/`a(` 等。不支持使用 `Ctrl+V` 的块级可视模式。想用 `jj` 退出 INSERT，设 `"vimInsertModeRemaps": { "jj": "<Esc>" }`（项目设置里写了会被忽略）。

#### 后台任务、排队消息、提示建议与任务列表

| 功能 | 怎么用 | 要点 |
|---|---|---|
| 后台 Bash 命令 | 让 Claude 在后台运行，或按 `Ctrl+B` | 输出写入文件，Claude 用 Read 检索；Claude Code 退出时自动清理；输出超过 5GB 自动终止 |
| 排队消息 | Claude 工作时输入并按 `Enter` | 消息在当前工具调用完成后同一轮次内送达；命令和 `!` 命令留到轮次结束。第一行按 `Up` 可取回 |
| 提示建议 | `Tab` 或 `Right arrow` 采纳，直接输入则关闭 | 用后台请求生成，计入用量；`/config` 里关 **Prompt suggestions** 或设 `promptSuggestionEnabled` 为 `false` |
| 任务列表 | `Ctrl+T` 切换，一次最多显示五个任务 | 是 Claude 的待办清单，不是后台任务视图（那个用 `/tasks`）；任务在上下文压缩中保持 |
| 附带问题 | `/btw <问题>` | 不进入对话历史；只凭上下文中已有内容回答，无工具访问 |
| 会话回顾 | 离开后回来自动显示，或运行 `/recap` | 至少三个轮次才出现；限制在 400 个字符以内 |
| 等待限额重置 | claude.ai 用量限制中断任务时自动等待并继续 | 仅限用 claude.ai 订阅登录的交互式会话；自动继续三次后停止 |

### Checkpointing：回退与摘要

Claude Code 会自动跟踪 Claude 的文件编辑工具所做的更改，让你随时撤销。

- 你发送的每条开启新轮次的提示词都会创建一个新的检查点，捕获此刻的代码状态。
- 为会话中最近的 100 个检查点保留文件快照。
- 检查点与对话一起保存，恢复会话后仍然可以 `/rewind`。
- 文件快照默认约在会话最后一次保存快照后的 30 天被清理；想留更久设 `cleanupPeriodDays`。

#### 回退方式

运行 `/rewind`（别名 `/undo`、`/checkpoint`），或在输入框**为空**时按两次 `Esc`，打开回退菜单，选一条提示词，再选操作。输入框有文本时双击 `Esc` 是清除文本；vim 模式下要在 NORMAL 模式中按。

| 选项 | 效果 |
|---|---|
| **Restore code and conversation** | 代码和对话都恢复到该位置 |
| **Restore conversation** | 回退到该消息，保留当前代码 |
| **Restore code** | 还原文件更改，保留对话 |
| **Summarize from here** | 把该位置往后的对话压缩为摘要 |
| **Summarize up to here** | 把该位置之前的对话压缩为摘要，完整保留之后的消息 |
| **Never mind** | 返回消息列表，不做任何更改 |

两个代码恢复选项只在所选检查点存在可还原的已跟踪文件更改时才显示。摘要不改磁盘上的文件，类似有针对性的 `/compact`；想保留原会话另起一条路，改用 `/branch` 或 `claude --continue --fork-session`。

#### 局限：哪些改动不被跟踪

| 情况 | 结果 |
|---|---|
| 仅由 Bash 命令更改过的文件 | 不被跟踪，保持原样 |
| 子代理的编辑 | 通常不恢复（前台分叉技能除外） |
| Claude Code 外部的手动更改、其他并发会话的编辑 | 通常不被捕获 |
| Claude 工作中途发送的消息 | 可能标为 **No code restore**，没有自己的检查点 |
| 符号链接或硬链接文件 | 跳过，并显示 `Restored the code, but skipped N files` |

> 一旦 Claude 用文件编辑工具编辑过某个文件，`/rewind` 也会撤销之后 Bash 命令对**该文件**的更改。Checkpoint 是会话级的快速恢复，不是版本控制的替代品，提交、分支和长期历史继续用 Git。

### Hooks 参考

Hooks 是用户定义的 shell 命令、HTTP 端点、MCP 工具调用、LLM 提示或子代理，在 Claude Code 生命周期的特定点自动执行。它是确定性的：在固定的点触发，而不是由模型自行决定。

```text
（可选）Setup → SessionStart
      │
      ▼
┌─ 每轮 ─────────────────────────────────────────────────────┐
│ UserPromptSubmit（输入命令时还有 UserPromptExpansion）      │
│   ▼                                                        │
│ ┌─ 代理循环 ─────────────────────────────────────────────┐ │
│ │ PreToolUse → PermissionRequest（自动模式拒绝 → PermissionDenied）
│ │   ▼                                                    │ │
│ │ 工具执行（MCP 工具内：Elicitation → ElicitationResult） │ │
│ │   ▼                                                    │ │
│ │ PostToolUse 或 PostToolUseFailure → PostToolBatch      │ │
│ │ SubagentStart / SubagentStop、TaskCreated / TaskCompleted
│ └────────────────────────────────────────────────────────┘ │
│   ▼                                                        │
│ Stop 或 StopFailure                                        │
└────────────────────────────────────────────────────────────┘
      │
      ▼
TeammateIdle → PreCompact → PostCompact → SessionEnd

独立事件：WorktreeCreate、WorktreeRemove、Notification、ConfigChange、InstructionsLoaded、
          CwdChanged、FileChanged、DirectoryAdded、PreModelSwitch、PostModelSwitch、MessageDisplay
```

#### 全部事件

"退出 2 可阻止"指 hook 以退出码 2 结束时能否拦下该操作。`PermissionRequest` 标"否"，但仍可通过 JSON 的 `decision` 对象拒绝。

| 事件 | 触发时机 | 退出 2 可阻止 | matcher 匹配什么 |
|---|---|---|---|
| `SessionStart` | 会话开始或恢复时 | 否 | 会话如何启动：`startup`、`resume`、`clear`、`compact`、`fork` |
| `Setup` | 用 `--init-only` 启动，或 `-p` 模式下用 `--init` 或 `--maintenance` 时 | 否 | 触发的 CLI 标志：`init`、`maintenance` |
| `UserPromptSubmit` | 提交提示词时，在 Claude 处理之前 | 是 | 不支持 |
| `UserPromptExpansion` | 用户输入的命令扩展为提示词时 | 是 | 命令名称 |
| `PreToolUse` | 工具调用执行之前 | 是 | 工具名称 |
| `PermissionRequest` | 工具调用需要权限决策时 | 否 | 工具名称 |
| `PermissionDenied` | 自动模式拒绝工具调用时 | 否 | 工具名称 |
| `PostToolUse` | 工具调用成功后 | 否（工具已运行） | 工具名称 |
| `PostToolUseFailure` | 工具调用失败后 | 否 | 工具名称 |
| `PostToolBatch` | 一整批并行工具调用解决后、下一次模型调用之前 | 是 | 不支持 |
| `Notification` | Claude Code 发送通知时 | 否 | 通知类型，如 `permission_prompt`、`idle_prompt` |
| `MessageDisplay` | 助手消息文本正在显示时 | 否 | 不支持 |
| `SubagentStart` | 子代理被生成时 | 否 | 代理类型，如 `general-purpose`、`Explore`、`Plan` |
| `SubagentStop` | 子代理完成时 | 是 | 代理类型 |
| `TaskCreated` | 通过 `TaskCreate` 创建任务时 | 是 | 不支持 |
| `TaskCompleted` | 任务被标记为已完成时 | 是 | 不支持 |
| `Stop` | Claude 完成响应时 | 是（让 Claude 继续） | 不支持 |
| `StopFailure` | 轮次因 API 错误而结束时 | 否 | 错误类型，如 `rate_limit`、`server_error` |
| `TeammateIdle` | 代理团队队友即将空闲时 | 是 | 不支持 |
| `InstructionsLoaded` | CLAUDE.md 或 `.claude/rules/*.md` 被加载到上下文时 | 否 | 加载原因，如 `session_start`、`compact` |
| `ConfigChange` | 配置文件在会话期间更改时 | 是（`policy_settings` 除外） | 配置源：`user_settings`、`project_settings`、`local_settings`、`policy_settings`、`skills` |
| `CwdChanged` | 工作目录更改时 | 否 | 不支持 |
| `DirectoryAdded` | 会话中期通过 `/add-dir` 等添加工作目录时 | 否 | 添加方式：`slash_command`、`register_repo_root` |
| `FileChanged` | 监视的文件在磁盘上更改时 | 否 | 要监视的文件名，如 `.envrc\|.env` |
| `WorktreeCreate` | 创建工作树时，替换默认的 git 行为 | 是（任何非零退出码都使创建失败） | 不支持 |
| `WorktreeRemove` | 由 `WorktreeCreate` hook 创建的 worktree 被移除时 | 是（任何非零退出码） | 不支持 |
| `PreCompact` | 上下文压缩之前 | 是 | 触发方式：`manual`、`auto` |
| `PostCompact` | 上下文压缩完成后 | 否 | 触发方式：`manual`、`auto` |
| `PreModelSwitch` | 应用你或客户端请求的模型切换之前 | 是 | 切换到的模型的规范名称 |
| `PostModelSwitch` | 会话的模型更改后 | 否 | 切换到的模型的规范名称 |
| `Elicitation` | MCP 服务器在工具调用期间请求用户输入时 | 是 | MCP 服务器名称 |
| `ElicitationResult` | 用户响应 MCP 引出后、响应发回服务器之前 | 是 | MCP 服务器名称 |
| `SessionEnd` | 会话终止时 | 否 | 结束原因：`clear`、`resume`、`logout`、`prompt_input_exit`、`other` |

matcher 求值规则：`"*"`、`""` 或省略匹配所有；只含字母、数字、`_`、`-`、空格、`,` 和 `|` 时按精确字符串（或用 `|` / `,` 分隔的列表）匹配，如 `Edit|Write`；含其他字符时按**未锚定**的 JavaScript 正则匹配，所以 `Edit.*` 同时匹配 `Edit` 和 `NotebookEdit`，要整串匹配写 `^Edit$`。给不支持 matcher 的事件加 `matcher` 会被静默忽略。

#### 配置结构与处理程序类型

三层嵌套：选一个 **hook 事件** → 加一个**匹配器组** → 定义一个或多个 **hook 处理程序**。可以写在 `~/.claude/settings.json`（所有项目）、`.claude/settings.json`（单个项目，可提交）、`.claude/settings.local.json`、托管策略设置、插件的 `hooks/hooks.json`，以及 skill 或 subagent 的 frontmatter。各级别的 hook 条目是**合并**而不是相互替换；事件触发时匹配的 hook 并行运行。

| `type` | 做什么 | 专有字段 |
|---|---|---|
| `"command"` | 运行 shell 命令；stdin 收 JSON，用退出码和 stdout 传回结果 | `command`（必需）、`args`、`async`、`asyncRewake`、`shell`、`onFailure` |
| `"http"` | 把事件 JSON 作为 POST 请求发送到 URL | `url`（必需）、`headers`、`allowedEnvVars`、`onFailure` |
| `"mcp_tool"` | 调用已配置 MCP 服务器上的工具 | `server`（必需）、`tool`（必需） |
| `"prompt"` | 向 Claude 模型发送提示做单轮评估 | `prompt`（必需，`$ARGUMENTS` 为输入 JSON 的占位符）、`model` |
| `"agent"` | 生成可使用工具的子代理来验证条件；**实验性** | 同 `prompt` |

| 通用字段 | 必需 | 说明 |
|---|---|---|
| `type` | 是 | `"command"`、`"http"`、`"mcp_tool"`、`"prompt"` 或 `"agent"` |
| `if` | 否 | 权限规则语法，如 `"Bash(git *)"` 或 `"Edit(*.ts)"`；只在工具事件上评估，恰好一条规则 |
| `timeout` | 否 | 取消前的秒数。默认：`command`、`http` 和 `mcp_tool` 为 600；`prompt` 为 30；`agent` 为 60 |
| `statusMessage` | 否 | hook 运行时显示的自定义消息 |
| `once` | 否 | 为 `true` 时第一次成功运行后删除 hook；仅在技能 frontmatter 中生效 |

并非所有事件都支持每种类型：`PreToolUse`、`PostToolUse`、`Stop`、`UserPromptSubmit` 等支持全部五种；`PermissionRequest` 不支持 `agent`；`Setup` 仅支持 `command`；`SessionStart` 仅支持 `command` 和 `mcp_tool`；`Notification`、`SessionEnd`、`PreCompact` 等不支持 `prompt` 或 `agent`。`command` 设了 `args` 是 exec 形式（不经过 shell），省略 `args` 是 shell 形式（可用管道和 `&&`）。

#### 输入、退出码与 JSON 输出

命令 hook 从 stdin 读 JSON，HTTP hook 从 POST 请求体读。通用字段有 `session_id`、`prompt_id`、`transcript_path`、`cwd`、`scratchpad_dir`、`permission_mode`、`effort`、`hook_event_name`，在子代理内触发时还有 `agent_id` 和 `agent_type`；工具事件另有 `tool_name`、`tool_input` 和 `tool_use_id`。

| 退出码 | 含义 | 结果 |
|---|---|---|
| 0 | 成功 | 应用 stdout 上的 JSON 字段；没有输出就是"无决定"，走正常权限流程 |
| 2 | 阻止错误 | 在可阻止的事件上停止该操作，stderr 作为原因反馈 |
| 其他 | 非阻止错误 | 操作**继续**，会话记录里出现 `<hook name> hook error` 通知 |

> 退出码 1 在这里是非阻止错误，要强制执行策略请用 `exit 2`。路径写错、脚本崩溃或超时的 hook 默认会放行一切；要改为阻止，在 `command`、`http` 或 `mcp_tool` hook 上设置 `"onFailure": "block"`（需要 v2.1.295 或更高版本）。

每个 hook 二选一：只用退出码发信号，或退出 0 并向 stdout 打印**一个** JSON 对象。所有事件通用的输出字段是 `continue`（为 `false` 时 Claude 完全停止处理）、`stopReason`、`systemMessage`、`terminalSequence`；各事件的决策字段如下。

| 事件 | 决策模式 | 关键字段 |
|---|---|---|
| UserPromptSubmit、UserPromptExpansion、PostToolUse、PostToolUseFailure、PostToolBatch、Stop、SubagentStop、ConfigChange、PreCompact | 顶级 `decision` | `decision: "block"`、`reason` |
| TeammateIdle、TaskCompleted、TaskCreated | 退出码或顶级 `decision` | 退出码 2 或 `decision: "block"` |
| PreToolUse | `hookSpecificOutput` | `permissionDecision`（allow/deny/ask/defer）、`permissionDecisionReason`、`updatedInput`、`additionalContext` |
| PreModelSwitch | `hookSpecificOutput` 或顶级 `decision` | `permissionDecision`（allow/deny/ask）、`permissionDecisionReason` |
| PermissionRequest | `hookSpecificOutput` | `decision.behavior`（allow/deny），`updatedInput` 位于 `decision` 对象内 |
| PermissionDenied | `hookSpecificOutput` | `retry: true` |
| WorktreeCreate | 路径返回 | 命令 hook 在 stdout 上打印路径；HTTP hook 返回 `hookSpecificOutput.worktreePath` |
| WorktreeRemove | 退出码 | JSON 输出被丢弃 |
| Elicitation、ElicitationResult | `hookSpecificOutput` 或顶级 `decision` | `action`（accept/decline/cancel）、`content` |
| MessageDisplay | `hookSpecificOutput` | `displayContent`（只影响显示） |
| SessionStart、SubagentStart、PostModelSwitch | 仅上下文 | `hookSpecificOutput.additionalContext` |
| Setup、Notification、SessionEnd、PostCompact、InstructionsLoaded、StopFailure、CwdChanged、DirectoryAdded、FileChanged | 无 | 只用于记录日志或清理等副作用 |

- 顶级 `decision` 的唯一值是 `"block"`，想放行就省略它。多个 PreToolUse hook 决策不同时，优先级为 `deny` > `defer` > `ask` > `allow`。
- `permissionDecision: "allow"` 只是跳过权限提示，拒绝和询问规则仍会被评估；hook 保持沉默也不等于批准。`updatedInput` 替换**整个**输入对象。
- `additionalContext`、`systemMessage` 和纯 stdout 各限制为 10,000 个字符。`Stop` hook 连续 8 次让轮次继续后会被覆盖，脚本里应检查 `stop_hook_active`。

路径占位符 `${CLAUDE_PROJECT_DIR}`（项目根目录）、`${CLAUDE_PLUGIN_ROOT}`、`${CLAUDE_PLUGIN_DATA}` 同时作为环境变量导出给 hook 进程。`CLAUDE_ENV_FILE` 是一个文件路径，往里追加 `export` 语句可为后续 shell 命令持久化环境变量，仅 SessionStart、Setup、CwdChanged 和 FileChanged hook 可用。没有 `$CLAUDE_MODEL` 环境变量。

#### 最小示例：PreToolUse 拦截 `rm -rf`

配置写进 `.claude/settings.json`。`matcher` 缩小到 Bash 工具调用，`if` 进一步缩小到匹配 `rm *` 的子命令：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "args": []
          }
        ]
      }
    ]
  }
}
```

脚本保存为 `.claude/hooks/block-rm.sh`，并执行 `chmod +x .claude/hooks/block-rm.sh`（依赖 `jq`）：

```bash
#!/bin/bash
# .claude/hooks/block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0  # no decision; normal permission flow applies
fi
```

验证：让 Claude 分别运行 `rm file.txt`（应走正常权限流程）和一条 `rm -rf`（应被拒绝，Claude 看到原因）。Windows 上把匹配器写成 `Bash|PowerShell`，并为每个工具各写一个 `if` 处理程序。

#### 安全注意事项与调试

- 命令 hook 以你的**完整用户权限**执行 shell 命令，加入配置之前先审查并测试。
- 交互式会话在你接受工作区信任对话框之前不运行设置文件里的 hook；但 `-p` 或 SDK 会话从不显示对话框，会直接运行仓库 `.claude/settings.json` 里提交的 hook。对不是自己写的仓库跑 `claude -p` 之前，先审查其 `.claude/` 设置，或用 `--bare`，或传 `--settings '{"disableAllHooks": true}'`。
- `if` 过滤是尽力而为的，硬性的允许或拒绝用权限系统；超时的 `PreToolUse` hook 默认不阻止工具调用。
- 脚本里验证和清理输入、始终引用 shell 变量（`"$VAR"`）、检查路径中的 `..`、用绝对路径、跳过 `.env`、`.git/`、密钥等敏感文件。

| 调试目标 | 做法 |
|---|---|
| 看当前生效了哪些 hook | 运行 `/hooks`（只读浏览器，标注每个 hook 的来源） |
| 看 hook 执行细节 | `claude --debug-file <path>`，或 `claude --debug` 后读 `~/.claude/debug/<session-id>.txt`（`--debug` 不打印到终端） |
| 看匹配细节 | 设 `CLAUDE_CODE_DEBUG_LOG_LEVEL=verbose` |
| 临时关掉所有 hook | 设置文件里写 `"disableAllHooks": true`（无法只禁用单个 hook，也关不掉托管 hook） |
| hook 似乎没运行 | 留意 `<hook name> hook error` 通知里的 `No such file or directory`，多半是路径写错 |

### Channels 参考：协议要点

> Channels 处于**研究预览**阶段。Team 和 Enterprise 组织必须明确启用。

Channel 是一个与 Claude Code 运行在同一台机器上的 MCP 服务器，由 Claude Code 作为子进程生成并通过 stdio 通信，把外部事件（聊天消息、webhook、警报）推送进会话。事件以 `<channel source="..." ...>内容</channel>` 的形式到达 Claude 的上下文；双向频道再公开一个标准 MCP 工具让 Claude 回复。

| 协议元素 | 说明 |
|---|---|
| `capabilities.experimental['claude/channel']` | 必需，始终为 `{}`，用于注册通知侦听器 |
| `capabilities.experimental['claude/channel/permission']` | 可选，设为 `{}` 声明可接收权限中继请求 |
| `capabilities.tools` | 仅双向频道，始终为 `{}` |
| `instructions` | 推荐，告诉 Claude 期望什么事件、是否回复及用哪个工具 |
| `notifications/claude/channel` | 推送事件：`content` 是事件主体；可选的 `meta` 每个条目成为标签属性，键只能含字母、数字和下划线 |
| `notifications/claude/channel/permission_request` | 权限中继的出站通知，字段 `request_id`、`tool_name`、`description`、`input_preview` |
| `notifications/claude/channel/permission` | 服务器发回的判决：`request_id` 加 `behavior`（`'allow'` 或 `'deny'`） |

本地测试自定义频道用 `claude --dangerously-load-development-channels server:webhook`（需要交互式会话，用 `-p` 时该标志被忽略）。

- 未门控的频道是提示注入向量：发出事件前按**发送者**身份检查允许列表，而不是按聊天或房间。
- Claude Code 不确认通知；会话没有把服务器加载为频道或组织政策阻止时，事件会被静默删除。
- 权限中继只涵盖工具使用批准，项目信任和 MCP 服务器同意对话只在本地终端出现。

### 词汇表

| 术语 | 一句话解释 |
|---|---|
| Agentic loop | Claude 为每个任务经历的循环：收集上下文、采取行动、验证结果并重复直到完成 |
| Tool | Claude 可以采取的操作：读文件、编辑代码、运行 shell 命令、搜索 web、生成 subagent |
| Session | 与当前目录相关的对话，具有独立的上下文窗口 |
| Turn | Claude 在一个 session 中的一个完整响应，从你发送消息到 Claude 完成响应 |
| Context window | 会话的工作内存，保存对话历史、文件内容、命令输出、CLAUDE.md 等 |
| Compaction | 上下文窗口接近限制时自动总结对话；`/compact` 手动触发 |
| CLAUDE.md | 你为 Claude 编写的持久指令文件，每个会话开始时加载 |
| AGENTS.md | 为 AI 编码代理编写的项目说明；仓库有它且没有 CLAUDE.md 时会被当作项目说明读取 |
| Auto memory | Claude 根据你的更正和偏好为自己编写的笔记，按 git 仓库存储在 `~/.claude/projects/` 下 |
| Rules | `.claude/rules/` 中的模块化指令文件，可用 `paths:` frontmatter 限定路径范围 |
| Command | 在提示中键入 `/name` 调用的可重用指令 |
| Skill | 一个 `SKILL.md` 文件，包含指令、知识或工作流；是自定义命令的推荐后继 |
| Subagent | 在自己的上下文窗口中运行的专门 AI 助手，向主对话返回摘要 |
| Agent teams | 由团队负责人协调的多个独立 Claude Code 会话；实验性，默认禁用 |
| Hook | 在生命周期特定点自动执行的用户定义处理程序，是确定性的 |
| Plugin | 把 skills、hooks、subagents 和 MCP servers 打包成的单个可安装单元 |
| MCP (Model Context Protocol) | 把 AI 工具连接到外部数据源和服务的开放标准 |
| Permission mode | 会话的基线批准行为：`default`、`acceptEdits`、`plan`、`auto`、`dontAsk` 和 `bypassPermissions` |
| Permission rule | 按工具名称和参数模式允许、询问或拒绝工具调用的设置条目，按 deny→ask→allow 顺序评估 |
| Plan mode | Claude 只研究并提议更改、不编辑源文件的权限模式 |
| Auto mode | 由单独的分类器模型代替你审查操作的权限模式 |
| Sandboxing | Bash 工具的操作系统级文件系统和网络隔离，是与权限规则分开的一层 |
| Project trust | 在 Claude Code 加载目录配置之前接受该目录的对话框 |
| Managed settings | 由 IT 或 DevOps 在组织范围内强制执行的设置，用户和项目设置无法覆盖 |
| Settings layers | 配置层级，优先级从高到低：托管策略、`--settings`、本地设置、项目设置、用户设置 |
| Non-interactive mode | 用 `-p` 或 `--print` 执行单个提示并退出的模式；以前称为 headless mode |
| Cloud session | 在云基础设施上运行、关闭笔记本后继续运行的会话 |
| Remote Control | 通过 claude.ai 从手机或浏览器继续本地会话；代码执行和文件留在你的机器上 |
| Worktree isolation | 在 `.claude/worktrees/` 下的单独 git worktree 中运行 Claude，用 `-w` 或 `isolation: worktree` 启用 |
| System reminder | Claude Code 作为 harness 添加到对话中的消息，如 CLAUDE.md 内容和 hook 返回的 `additionalContext` |

已改名的旧术语：Headless mode → Non-interactive mode；Custom commands → Skills（`.claude/commands/` 文件仍然有效）；Slash commands → Commands；Web session → Cloud session。

---

## Part VII：Agent SDK 与资源

### Agent SDK：先选型，再动手

Agent SDK 把 Claude Code 当作一个库来用：你的 Python / TypeScript 程序获得与 Claude Code 相同的内置工具、agent loop 和上下文管理。它不是薄薄一层 API 封装——SDK 会启动并监管一个 `claude` CLI 子进程，由它执行工具、写会话记录。

```text
你想怎么用 Claude？
│
├─ 在自己运营的 Python / TypeScript 应用里嵌入一个会自己干活的 Agent
│  └─ Agent SDK（query()）
├─ 在终端交互开发，或在脚本 / CI 里跑一次性任务
│  └─ Claude Code CLI（交互式，或 claude -p）
├─ 用其他语言驱动同一个 agent loop
│  └─ 把 CLI 当子进程：claude -p --output-format json
├─ 只想直接调模型、自己写工具循环，或需要 temperature / top_p / max_tokens
│  └─ Client SDK（Messages API）
└─ 不想自己托管 agent loop
   └─ Managed Agents（Anthropic 托管）
```

| 方案 | 谁运行 agent loop | 你得到什么 | 需要注意 |
|---|---|---|---|
| **Agent SDK** | 你的应用（库内启动 Claude Code 二进制） | 内置工具、权限、会话、hooks、MCP、子代理 | 不设 `systemPrompt` 时用**最小系统提示** |
| **`claude -p`** | CLI 进程 | 一条命令，适合脚本和其他语言 | 默认使用 Claude Code 系统提示 |
| **Client SDK（Messages API）** | 你自己写循环（或测试版 tool runner） | 直接访问 Claude API | 工具要自己实现和执行 |
| **Managed Agents** | Anthropic | 托管的 agent loop 与沙箱 | 通过 Claude API 配置 |

一句话记忆：**要 Claude Code 全套能力进自己的程序用 Agent SDK；只要一条命令用 `claude -p`；只要模型用 Messages API。**

### 安装、运行时与认证

| 项目 | TypeScript | Python |
|---|---|---|
| 运行时 | Node.js 18.14+ | Python 3.10+ |
| 安装 | `npm install @anthropic-ai/claude-agent-sdk` | `pip install claude-agent-sdk`（或 `uv add claude-agent-sdk`） |
| 选项类型 | `Options` | `ClaudeAgentOptions` |
| 指定自装的 CLI | `pathToClaudeCodeExecutable` | `cli_path` |

两个 SDK 都捆绑了本地 Claude Code 二进制，大多数安装不需要单独装 Claude Code；升级 SDK 就是升级 CLI。两种例外需要本地另装：Python 装到源码分发而非平台 wheel（例如 ARM64 Windows）；TypeScript 跳过了可选依赖（例如 `npm ci --omit=optional`）。

```bash
export ANTHROPIC_API_KEY=your-api-key
```

SDK 从进程环境读取密钥，**不会自动加载 `.env`**。

| 提供商 | 环境变量（另需配置对应云凭证） |
|---|---|
| Amazon Bedrock | `CLAUDE_CODE_USE_BEDROCK=1` |
| Claude Platform on AWS | `CLAUDE_CODE_USE_ANTHROPIC_AWS=1`、`ANTHROPIC_AWS_WORKSPACE_ID` |
| Google Cloud 的 Agent Platform | `CLAUDE_CODE_USE_VERTEX=1` |
| Microsoft Foundry | `CLAUDE_CODE_USE_FOUNDRY=1` |

> 订阅登录的边界：除非事先获得批准，Anthropic 不允许第三方开发者在自己的产品（包括基于 Agent SDK 的 Agent）中提供 claude.ai 登录或其速率限制，面向他人的产品请用 API 密钥。文档只在一处提到订阅用法：你**亲自**用自己的订阅运行 Agent 时，可把 `claude setup-token` 生成的令牌放进 `CLAUDE_CODE_OAUTH_TOKEN`；该令牌只能发模型请求。

### 快速开始：第一个 Agent

`query()` 是主入口，返回异步迭代器：Claude 思考、调工具、看结果的过程以消息流吐出，最后是一条结果消息。

#### TypeScript

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

try {
  for await (const message of query({
    prompt: "Review utils.py for bugs that would cause crashes. Fix any issues you find.",
    options: {
      allowedTools: ["Read", "Edit", "Glob"], // 预先批准这些工具
      permissionMode: "acceptEdits"           // 自动批准文件编辑
    }
  })) {
    if (message.type === "assistant") {
      for (const block of message.message.content) {
        if (block.type === "text") console.log(block.text);
        else if (block.type === "tool_use") console.log(`Tool: ${block.name}`);
      }
    }
    if (message.type === "result") {
      console.log(message.subtype === "success" ? message.result : `Stopped: ${message.subtype}`);
    }
  }
} catch (error) {
  console.error(`Session ended with an error: ${error}`); // 单次 query() 产出错误结果后会抛出
}
```

新项目先 `npm pkg set type=module` 并 `npm install --save-dev tsx`，然后 `npx tsx agent.ts`。

#### Python

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AssistantMessage, ResultMessage, TextBlock, ToolUseBlock

async def main():
    try:
        async for message in query(
            prompt="Review utils.py for bugs that would cause crashes. Fix any issues you find.",
            options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Glob"], permission_mode="acceptEdits"),
        ):
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)
                    elif isinstance(block, ToolUseBlock):
                        print(f"Tool: {block.name}")
            if isinstance(message, ResultMessage):
                print(message.result if message.subtype == "success" else f"Stopped: {message.subtype}")
    except Exception as error:
        print(f"Session ended with an error: {error}")

asyncio.run(main())
```

- **取内容块**：TypeScript 在 `message.message.content`（原始 API 消息包在 `.message` 里），Python 直接是 `message.content`。
- **判断类型**：TypeScript 比较 `message.type` 字符串，Python 用 `isinstance()`。
- **`allowedTools` 是预先批准，不是白名单**：没列出的工具仍然可用，只是要走权限模式。要真正限制内置工具集合，用 `tools` 选项。

### Agent loop 如何工作

```text
prompt（+ 系统提示、工具定义、历史）
   ▼
SystemMessage  subtype="init"          ← 会话元数据
   ▼
┌──────────────── 一个轮次（turn）────────────────┐
│ Claude 评估 → AssistantMessage（每个内容块一条） │
│   ├─ 含工具调用 → hooks / 权限 → SDK 执行工具    │
│   │      └─ UserMessage（工具结果）→ 喂回 Claude │ ← 重复
│   └─ 只有文本 → 循环结束                         │
└─────────────────────────────────────────────────┘
   │  上下文接近上限 → 自动压缩 → system / compact_boundary
   │  达到 maxTurns / maxBudgetUsd → 提前结束
   ▼
ResultMessage  subtype = success | error_*   （result、usage、total_cost_usd、session_id）
```

| 消息 | 何时出现 |
|---|---|
| `SystemMessage` | 会话生命周期事件：`"init"`、`"compact_boundary"` 等（TypeScript 中除 `"init"` 外各是独立类型） |
| `AssistantMessage` | Claude 每个内容块完成时；同一响应的多条消息共享一个消息 ID |
| `UserMessage` | 每次工具执行后，携带工具结果 |
| `StreamEvent` | 仅在启用部分消息时，包着原始 API 流事件 |
| `ResultMessage` | 循环结束。其后可能还有少量尾随系统事件，**迭代到流结束，不要在结果处 break** |

| 结果 `subtype` | 含义 | `result` 可用 |
|---|---|:-:|
| `success` | 正常完成 | 是 |
| `error_max_turns` | 达到 `maxTurns` | 否 |
| `error_max_budget_usd` | 达到 `maxBudgetUsd` | 否 |
| `error_during_execution` | 错误或中断打断了循环 | 否 |
| `error_max_structured_output_retries` | 重试限制内没有得到有效结构化输出 | 否 |

- `maxTurns` / `max_turns`：每个含工具调用的轮次计一轮，最终纯文本回复再计一轮；默认无限制，`0` 也表示不限制。
- `maxBudgetUsd` / `max_budget_usd`：按预估支出停止，**包含子代理支出**。预算在响应返回后才比较，可能超出一个响应的成本，设上限要留余量；`0` 会被当作无效金额拒绝。
- `effort`：`"low"`、`"medium"`、`"high"`、`"xhigh"`、`"max"`；并非所有模型都支持。
- 只读工具（`Read`、`Glob`、`Grep`）可并发，`Edit`、`Write`、`Bash` 顺序执行；自定义工具设了 `readOnlyHint` 才并行。
- 自动压缩会把旧历史换成摘要，早期指令可能丢失。**长期规则写进 CLAUDE.md**；也可发送 `/compact` 手动压缩，或用 `PreCompact` hook 先归档。

### 核心选项速查表

| TypeScript | Python | 作用 |
|---|---|---|
| `model` / `fallbackModel` | `model` / `fallback_model` | 主模型与备用模型（别名或完整模型名） |
| `cwd` | `cwd` | 工作目录；决定项目设置、skills、会话归属，会话中途不能改 |
| `additionalDirectories` | `add_dirs` | 让工具访问工作目录以外的路径 |
| `tools` | `tools` | 限定出现在上下文中的**内置**工具；`[]` 表示只用 MCP 工具 |
| `allowedTools` | `allowed_tools` | 预先批准的工具或规则，如 `"Bash(npm test *)"` |
| `disallowedTools` | `disallowed_tools` | 拒绝规则；裸工具名会把工具从上下文移除 |
| `permissionMode` | `permission_mode` | 权限模式 |
| `canUseTool` | `can_use_tool` | 审批回调 |
| `systemPrompt` | `system_prompt` | 系统提示：字符串，或 `claude_code` 预设（可带 `append`） |
| `settingSources` | `setting_sources` | 加载哪些文件系统设置：`"user"`、`"project"`、`"local"` |
| `settings` | `settings` | 额外的设置文件路径或内联 JSON |
| `mcpServers` | `mcp_servers` | MCP 服务器 |
| `agents` / `hooks` / `skills` / `plugins` | 同名 | 子代理定义、生命周期回调、可调用的 skills、本地 plugins |
| `outputFormat` | `output_format` | 结构化输出 schema |
| `includePartialMessages` | `include_partial_messages` | 开启流式输出 |
| `maxTurns` / `maxBudgetUsd` | `max_turns` / `max_budget_usd` | 轮次与支出上限 |
| `effort` | `effort` | 努力级别 |
| `continue` | `continue_conversation` | 继续当前目录最近的会话 |
| `resume` / `forkSession` | `resume` / `fork_session` | 按会话 ID 恢复；恢复时分叉 |
| `sessionStore` | `session_store` | 外部会话存储适配器 |
| `enableFileCheckpointing` | `enable_file_checkpointing` | 文件检查点 |
| `env` / `sandbox` | 同名 | 子进程环境变量；工具执行的沙箱行为 |

> **TypeScript 中 `env` 替换整个子进程环境**，要写成 `env: { ...process.env, KEY: "value" }`，否则 `PATH`、`ANTHROPIC_API_KEY` 都会丢；Python 中是合并到继承的环境之上。

流式输入会话可以中途改配置：TypeScript 在 `query()` 返回的对象上调 `setModel()` / `setPermissionMode()`，Python 在 `ClaudeSDKClient` 上调 `set_model()` / `set_permission_mode()`。`temperature`、`top_p`、`max_tokens` 在选项对象上没有字段。

### 在 SDK 中使用 Claude Code 能力

**省略 `settingSources` 等同于 `["user", "project", "local"]`**，与 CLI 行为一致；传 `[]` 则只用代码里配置的内容。

| 源 | 加载的内容 |
|---|---|
| `"project"` | `<cwd>/.claude/` 的 `settings.json` 与 hooks；CLAUDE.md、`.claude/rules/*.md`（含父目录）；项目 skills、commands、subagents；`.mcp.json` |
| `"user"` | `~/.claude/` 的 `settings.json`、CLAUDE.md、规则；用户 skills、commands、subagents |
| `"local"` | `CLAUDE.local.md`、`.claude/settings.local.json` |

| 能力 | 加载条件与入口 |
|---|---|
| CLAUDE.md 与规则 | 设置源含对应级别即自动加载；各级内容累加，没有硬优先级 |
| Skills | 由设置源发现，用 `skills` 选项限定；传了 `tools` 时其中要有 `"Skill"`。**没有编程注册 API**，必须是磁盘上的 `SKILL.md` |
| Slash commands | 把 `/<name>` 当提示发送；`.claude/commands/` 旧格式仍可用 |
| Subagents | `agents` 选项（推荐），或 `.claude/agents/` 文件；同名时代码定义优先 |
| Plugins | `plugins: [{ type: "local", path }]`，只接受本地路径 |
| Hooks | `hooks` 选项里的回调，与 `settings.json` 里的 shell hook 并行运行 |
| Agent teams | CLI 功能，不通过 SDK 选项配置 |

即使 `settingSources` 为 `[]`，这些仍会被读取：托管策略设置、`~/.claude.json` 全局配置（用 `CLAUDE_CONFIG_DIR` 重新定位）、自动内存（`CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` 关闭）、claude.ai MCP 连接器（`strictMcpConfig: true` 关闭）。

> 不要依赖默认 `query()` 选项做多租户隔离。

### 会话：继续、恢复、分叉与外部存储

会话是 SDK 自动写入磁盘的对话历史（`~/.claude/projects/` 下的 JSONL）。它保存的是**对话**，不是文件系统。

| 你要做什么 | 用什么 |
|---|---|
| 一次性任务 | 一个 `query()` 调用即可 |
| 同一进程内多轮对话 | Python `ClaudeSDKClient`；TypeScript 每次传 `continue: true` |
| 进程重启后接着上次 | `continue: true` / `continue_conversation=True` |
| 回到某个特定会话 | 从结果消息读 `session_id`，传给 `resume` |
| 试另一条路线又不丢原路线 | `resume` 加 `forkSession: true` / `fork_session=True` |
| 不想写磁盘 | TypeScript `persistSession: false`；Python 在 `env` 中设 `CLAUDE_CODE_SKIP_PROMPT_HISTORY` |

- `session_id` 在每条结果消息上都有（成功或失败）；达到上限后可以用更高的上限 `resume` 继续。
- 分叉只复制对话历史，**不复制文件**；分叉出的 Agent 改文件是真改。
- 管理函数：TypeScript `listSessions()`、`getSessionMessages()`、`renameSession()`、`tagSession()`；Python `list_sessions()`、`get_session_messages()`、`rename_session()`、`tag_session()`。

#### 把会话存到外部（`SessionStore`）

```text
Claude Code 子进程 ──先写──> 本地磁盘
        └─ SDK 再转发同一批 ──> store.append()      ← 镜像，不是替代
恢复：store.load() ──> 临时配置目录 ──> 子进程 ──> 运行结束后删除本地副本
```

- 适配器必须实现 `append` 和 `load`；可选 `listSessions`、`listSessionSummaries`、`delete`、`listSubkeys`。
- SDK 自带 `InMemorySessionStore` 供开发测试；S3、Redis、Postgres 参考适配器在 SDK 仓库的 examples 目录，**不作为包发布**。
- 镜像是尽力而为：重试后仍失败会发出 `subtype: "mirror_error"` 的系统消息并继续运行。监控它，并在 `append()` 里按 `entry.uuid` 去重。
- 查找键来自工作目录，**恢复时 `cwd` 要与原运行一致**。SDK 从不主动删除存储里的数据。
- 不能与 `persistSession: false` 或文件检查点同时使用，否则启动时抛错。

### 输入与输出

| | 单次输入 | 流式输入 |
|---|---|---|
| 写法 | `prompt` 传字符串 | TypeScript：`prompt` 传 yield `SDKUserMessage` 的 async generator；Python：`ClaudeSDKClient` 的 `client.query()` 加 `receive_response()` |
| 适合 | 单个任务、serverless 函数 | 聊天界面、长驻会话 |
| 图像、`interrupt()`、中途切换模型或权限模式 | 不支持 | 支持 |
| 遇到错误结果 | 产出结果后**抛出** | 会话保持活跃 |

#### 审批与澄清问题

工具调用需要批准，或 Claude 调用 `AskUserQuestion` 提问时，都会触发 `canUseTool` 回调并暂停执行，直到你返回：

| 响应 | TypeScript | Python |
|---|---|---|
| 允许（可改写输入） | `{ behavior: "allow", updatedInput }` | `PermissionResultAllow(updated_input=...)` |
| 拒绝（附原因） | `{ behavior: "deny", message }` | `PermissionResultDeny(message=...)` |

- `AskUserQuestion` 每次 1-4 个问题、每个 2-4 个选项；回答时原样传回 `questions`，并在 `answers` 中以问题文本为键、所选 `label` 为值。传了 `tools` 数组时要把它列进去；子代理中目前不可用。
- 被 `allowedTools`、`acceptEdits`、`bypassPermissions` 提前批准的调用**不会到达 `canUseTool`**；要检查这些调用用 `PreToolUse` hook。
- 用户可能很久才回复时，用返回 `defer` 决定的 `PreToolUse` hook，让进程先退出，之后从持久化会话恢复。

#### 流式输出与结构化输出

- 流式输出：设 `includePartialMessages: true` / `include_partial_messages=True`，额外收到 `StreamEvent`（TypeScript 中 `message.type === "stream_event"`）。在 `content_block_delta` 事件里取 `delta.type === "text_delta"` 的文本，自己拼接。子代理的 token 级增量不会转发。
- 结构化输出：

```python
schema = {
    "type": "object",
    "properties": {"company_name": {"type": "string"}, "founded_year": {"type": "number"}},
    "required": ["company_name"],
}
options = ClaudeAgentOptions(output_format={"type": "json_schema", "schema": schema})
# 读取：isinstance(message, ResultMessage) and not message.is_error and message.structured_output
```

TypeScript 写 `outputFormat: { type: "json_schema", schema }`。**`success` 不等于有输出**：`is_error` 为 `true` 或 `structured_output` 缺失都按失败处理。流里出现的 `StructuredOutput` 工具调用是实现机制，展示工具调用时跳过它。

### 工具扩展：自定义工具、MCP、tool search、子代理

#### 自定义工具（进程内 MCP 服务器）

```typescript
import { query, tool, createSdkMcpServer } from "@anthropic-ai/claude-agent-sdk";
import { z } from "zod";

const getTemperature = tool(
  "get_temperature",
  "Get the current temperature at a location",
  { latitude: z.number().describe("Latitude coordinate"), longitude: z.number().describe("Longitude coordinate") },
  async (args) => ({ content: [{ type: "text", text: `Temperature at ${args.latitude},${args.longitude}: ...` }] })
);
const weatherServer = createSdkMcpServer({ name: "weather", version: "1.0.0", tools: [getTemperature] });
// options: { mcpServers: { weather: weatherServer }, allowedTools: ["mcp__weather__get_temperature"] }
```

```python
from typing import Annotated, Any
from claude_agent_sdk import tool, create_sdk_mcp_server, ClaudeAgentOptions

@tool(
    "get_temperature",
    "Get the current temperature at a location",
    {"latitude": Annotated[float, "Latitude coordinate"], "longitude": Annotated[float, "Longitude coordinate"]},
)
async def get_temperature(args: dict[str, Any]) -> dict[str, Any]:
    return {"content": [{"type": "text", "text": f"Temperature at {args['latitude']},{args['longitude']}: ..."}]}

weather_server = create_sdk_mcp_server(name="weather", version="1.0.0", tools=[get_temperature])
options = ClaudeAgentOptions(mcp_servers={"weather": weather_server}, allowed_tools=["mcp__weather__get_temperature"])
```

工具全名是 `mcp__{server_name}__{tool_name}`，`server_name` 即 `mcpServers` 里的键。出错时返回 `isError: true` 的结果，而不是让原始异常冒出来。

#### 连接外部 MCP

```typescript
options: {
  mcpServers: {
    filesystem: { command: "npx", args: ["-y", "@modelcontextprotocol/server-filesystem", "/Users/me/projects"] },
    "remote-api": { type: "sse", url: "https://api.example.com/mcp/sse",
                    headers: { Authorization: `Bearer ${process.env.API_TOKEN}` } }
  },
  allowedTools: ["mcp__filesystem__*", "mcp__remote-api__*"]
}
```

- 文档给命令就用 stdio，给 URL 就用 HTTP / SSE；也可写进项目根目录的 `.mcp.json`（需要 `project` 设置源）。
- **MCP 工具必须显式授权**，否则 Claude 看得到却调不了。`allowedTools: ["*"]` 或 `["mcp__*"]` 会被忽略，通配符只能跟在 `mcp__<server>__` 之后。
- SDK 不会打开浏览器跑 OAuth：缺令牌时服务器状态为 `needs-auth`，在 `headers` 里传令牌。
- 用 init 消息的 `mcp_servers` 检查状态，`status === "failed"` 即连接失败。

#### Tool search 与子代理

工具搜索默认启用：工具定义不预先进上下文，Claude 需要时再搜索，每次最多加载 5 个最相关的工具；少于约 10 个工具时预先加载通常更快。用 `env` 里的 `ENABLE_TOOL_SEARCH` 控制：未设置或 `true` 为启用，`auto` / `auto:N` 在可延迟的定义达到上下文窗口 10%（或 N%）后启用，`false` 关闭。

```typescript
options: {
  allowedTools: ["Read", "Grep", "Glob", "Agent"],
  agents: {
    "code-reviewer": {
      description: "Expert code review specialist. Use for quality, security, and maintainability reviews.",
      prompt: "You are a code review specialist...",
      tools: ["Read", "Grep", "Glob"],
      model: "sonnet"
    }
  }
}
```

- `description` 和 `prompt` 必填；Claude 通过 `Agent` 工具调用子代理，按 `description` 自动匹配，也可在提示里点名。
- 子代理上下文是全新的：父代理传过去的**只有 Agent 工具的提示字符串**，只有最终消息返回父代理。
- 不定义任何子代理时，Claude 仍可调用内置的 `general-purpose`。
- 限制扩散：`env` 中 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`（默认 `3`）、`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`（默认 `20`），以及 `maxBudgetUsd`。

### 定制行为：系统提示、Skills、Plugins

| 你在构建 | 用法 | 得到什么 |
|---|---|---|
| 有人盯着并指导的 CLI / IDE 类编码工具 | `systemPrompt: { type: "preset", preset: "claude_code" }` | Claude Code 提示，含工具指引和安全规则 |
| 同类工具，再加产品规则 | 预设加 `append: "..."` | 上面全部加你的指令；风险最低 |
| 不同界面、身份、权限模型，或非编码 Agent | 自定义字符串 | 只有你写的内容 |
| 薄工具调用循环 | 不设 `systemPrompt` | 最小默认值：仅工具调用支持 |

| | CLAUDE.md | 输出样式 | 预设 + `append` | 自定义 `systemPrompt` |
|---|---|---|---|---|
| 持久性 | 项目文件 | 保存为文件 | 仅会话 | 仅会话 |
| 工具指引 | 保留 | 保留 | 保留 | 丢失（除非自己写） |
| 内置安全指令 | 保留 | 保留 | 保留 | 必须自己加 |
| 定制程度 | 仅添加 | 添加，可省略编码指令 | 仅添加 | 完全控制 |

- CLAUDE.md 不改系统提示本身：SDK 把它作为项目上下文注入对话。
- 恢复会话时换了 `append` 或自定义提示，默认**不会立刻生效**（系统提示在会话首个请求时被记录，直到压缩）。要中途改指令，放进下一条用户消息，或从 `UserPromptSubmit` / `PostToolUse` hook 返回 `additionalContext`。
- Skills：`.claude/skills/<name>/SKILL.md`，启动时只加载描述，调用时才加载全文。`skills` 省略为全部启用，传名称列表只允许这些（插件 skill 写成 `plugin:skill`），`[]` 全部禁用。用 init 消息的 `skills` 与 `slash_commands` 确认加载结果。
- 命令：发送 `/<name>` 即分派，**不受 `skills` 列表限制**。`/compact` 需要已有历史；`/clear` 对单次 `query()` 没有实际效果。
- Plugins：`plugins: [{ type: "local", path: "./my-plugin" }]`，`type` 只接受 `"local"`，marketplace 的 plugin 要先下载。plugin skill 以 `/plugin-name:skill-name` 调用。TypeScript 可调 `reloadPlugins()` 热加载，Python 改为发送 `/reload-plugins`。

### 控制与可观测

#### 权限：求值顺序与模式

```text
工具调用
  1. Hooks ───────── 可直接拒绝；返回 allow 也跳不过第 2、3 步
  2. 拒绝规则 ────── 命中即拒绝，bypassPermissions 下也一样
  3. 询问规则 ────── 命中则交给 canUseTool
  4. 权限模式 ────── bypassPermissions / acceptEdits / plan / auto 在此决定
  5. 允许规则 ────── allowedTools 与 settings.json 的 allow
  6. canUseTool ──── 前面都没解决才到这里；dontAsk 下跳过并拒绝
```

| 模式 | 行为 | 适合 |
|---|---|---|
| `default` | 需要批准且无允许规则的调用交给 `canUseTool`；**没有回调即拒绝** | 带审批界面的交互式应用 |
| `acceptEdits` | 自动批准工作目录内的文件编辑和常见文件系统命令 | 原型开发、隔离目录 |
| `plan` | 只探索和规划；文件编辑交给 `canUseTool` | 先出计划再动手 |
| `dontAsk` | 从不提示：预先批准的运行，其余会提示的一律拒绝 | 无人值守的 headless Agent |
| `auto` | 模型分类器审查 shell 命令、网络请求等操作 | 想要自主又要防护 |
| `bypassPermissions` | 不请求批准，**包括未列入 `allowedTools` 的工具** | 仅限 CI、容器等隔离环境 |

- 锁定型 Agent 的标准组合：`allowedTools` 加 `permissionMode: "dontAsk"`。
- `bypassPermissions` 下要禁用工具只能用 `disallowedTools`；TypeScript 还需 `allowDangerouslySkipPermissions: true`；Unix 上不能以 root 运行；子代理会继承它。
- 路径规则写成 `Edit(path)`（同时管 `Write` 和 `NotebookEdit`）；`//path` 是绝对路径，`/path` 相对工作目录。

#### Hooks

```typescript
const protectEnvFiles: HookCallback = async (input, toolUseID, { signal }) => {
  const preInput = input as PreToolUseHookInput;
  const filePath = (preInput.tool_input as Record<string, unknown>)?.file_path as string;
  if (filePath?.split("/").pop()?.toLowerCase() === ".env") {
    return { hookSpecificOutput: { hookEventName: preInput.hook_event_name,
             permissionDecision: "deny", permissionDecisionReason: "Cannot modify .env files" } };
  }
  return {}; // 没有意见：走正常权限流程
};
// options: { hooks: { PreToolUse: [{ matcher: "Write|Edit", hooks: [protectEnvFiles] }] } }
```

Python 写法：`hooks={"PreToolUse": [HookMatcher(matcher="Bash", hooks=[my_callback])]}`，回调签名为 `(input_data, tool_use_id, context)`。

| 可用范围 | 事件 |
|---|---|
| Python 与 TypeScript | `PreToolUse`、`PostToolUse`、`PostToolUseFailure`、`UserPromptSubmit`、`Stop`、`SubagentStart`、`SubagentStop`、`PreCompact`、`PermissionRequest`、`Notification` |
| 仅 TypeScript | `SessionStart`、`SessionEnd`、`Setup`、`PostToolBatch`、`UserPromptExpansion`、`MessageDisplay`、`StopFailure`、`PostCompact`、`PreModelSwitch`、`PostModelSwitch`、`PermissionDenied`、`TeammateIdle`、`TaskCreated`、`TaskCompleted`、`Elicitation`、`ElicitationResult`、`ConfigChange`、`InstructionsLoaded`、`WorktreeCreate`、`WorktreeRemove`、`CwdChanged`、`FileChanged`、`DirectoryAdded` |

- **回调抛错或没有返回值时，默认操作会继续**——崩溃的守卫等于放行。要失败即阻止，在匹配器上设 `onFailure: "block"`（Python 为 `on_failure="block"`）。
- `Write|Edit` 匹配器拦不住 Bash 写文件，那需要沙箱配置。Python 中 `SessionStart` / `SessionEnd` 只能用设置文件里的 shell 命令 hook。

#### 文件检查点、成本、OpenTelemetry、Todo

| 主题 | 要点 |
|---|---|
| 文件检查点 | 开启 `enableFileCheckpointing` 并加 `extraArgs: { 'replay-user-messages': null }`，记下用户消息的 `uuid`，回退时调 `rewindFiles()` / `rewind_files()`。只跟踪 `Write`、`Edit`、`NotebookEdit`；**Bash 改动和子代理编辑不撤销，也不回退对话** |
| 成本 | `total_cost_usd` 是**客户端估算**，不是计费依据。同一响应的多条 assistant 消息共享 ID 和 `usage`，按 ID 去重；真实输出 token 读结果消息；含子代理的全量数据读 `modelUsage` / `model_usage`；恢复的会话读最新结果，不要求和 |
| OpenTelemetry | SDK 本身不产生遥测，由 CLI 子进程导出。设 `CLAUDE_CODE_ENABLE_TELEMETRY=1`，再选 `OTEL_METRICS_EXPORTER`、`OTEL_LOGS_EXPORTER`；Traces（测试版）需 `OTEL_TRACES_EXPORTER` 加 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1`。导出失败默认静默；提示文本和工具输入默认不导出，`OTEL_LOG_USER_PROMPTS=1` 等是显式选择加入 |
| Todo 跟踪 | `TodoWrite`、`TaskCreate`、`TaskUpdate` 等只在 Claude Opus 4.7、Claude Sonnet 4.6、Claude Haiku 4.5 及更早的模型上默认可用；其他模型需在 `allowedTools` / `tools` 中列出，或设 `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` |

### 部署：托管形态与安全

托管 Agent SDK 不像托管无状态 API：**一个会话对应一个 `claude` 子进程**，它拥有 shell、工作目录和本地会话文件。

| 会话模式 | 做法 | 适合 |
|---|---|---|
| Ephemeral sessions | 每个任务一个容器，完成即销毁 | bug 修复、票据提取、文档翻译 |
| Long-running sessions | 常驻容器，通常托管多个 SDK 进程 | 邮件 Agent、站点构建器、聊天机器人 |
| Hybrid sessions | 临时容器，启动时从 `SessionStore` 恢复 | 间歇交互：深度研究、客服 |
| Multi-agent container | 一个容器内多个 SDK 子进程 | 必须紧密协作的多 Agent 模拟 |

- 起步资源：每个 Agent 1 GiB RAM、5 GiB 磁盘、1 个 CPU，这是下限。
- 会话记录、`CLAUDE.md` 内存文件、工作目录产物默认都在容器本地磁盘，重启即丢；`SessionStore` **只镜像会话记录**。
- 会话没有顶层超时：用 `maxTurns` 兜底，并限制会话长度或定期回收子进程。

```text
┌──── 隔离边界（sandbox-runtime / 容器 / gVisor / VM）────┐
│  你的应用 + Agent SDK + claude 子进程                    │
│  代码只读挂载 · 环境里没有真实凭证                       │
└──────────────────────┬──────────────────────────────────┘
                       ▼  不带凭证的请求
        边界外的转发代理（proxy）：域名允许列表 · 注入凭证 · 审计日志
                       ▼
               Claude API / 外部服务
```

| 隔离技术 | 隔离强度 | 性能开销 | 复杂性 |
|---|---|---|---|
| Sandbox 运行时 | 良好（安全默认值） | 非常低 | 低 |
| 容器（Docker） | 取决于设置 | 低 | 中等 |
| gVisor | 优秀（正确设置） | 中等 / 高 | 中等 |
| VM（Firecracker、QEMU） | 优秀（正确设置） | 高 | 中等 / 高 |

- **凭证不进 Agent 环境**：由边界外的 proxy 注入。`ANTHROPIC_BASE_URL` 只路由采样请求，`HTTP_PROXY` / `HTTPS_PROXY` 是系统范围。
- **入站认证放在网关**，Agent 容器只接收已认证的请求。
- **多租户**：`settingSources: []`、`CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`、每租户独立的 `CLAUDE_CONFIG_DIR` 和 `cwd`、每租户独立的出站规则。

### 迁移、V2 预览与常见排错

| 方面 | 旧（Claude Code SDK） | 新（Claude Agent SDK） |
|---|---|---|
| TS / JS 包 | `@anthropic-ai/claude-code` | `@anthropic-ai/claude-agent-sdk` |
| Python 包 / 导入 | `claude-code-sdk` / `claude_code_sdk` | `claude-agent-sdk` / `claude_agent_sdk` |
| Python 选项类型 | `ClaudeCodeOptions` | `ClaudeAgentOptions` |

- **破坏性变更（v0.1.0）**：系统提示不再默认是 Claude Code 的；要旧行为就显式设 `claude_code` 预设。
- **设置源默认值**：v0.1.0 曾短暂改为不加载文件系统设置，随后已恢复，无需迁移。Python SDK 0.1.59 及更早版本把空列表当作省略，依赖 `setting_sources=[]` 前先升级。
- **TypeScript V2 session API（已移除）**：TypeScript Agent SDK 0.3.142 移除了 `unstable_v2_createSession`、`unstable_v2_resumeSession`、`unstable_v2_prompt` 以及 `SDKSession`、`SDKSessionOptions` 类型；0.2.x 是最后一个包含 V2 的版本。新代码用 `query()`：多轮传 `AsyncIterable<SDKUserMessage>`，恢复用 `options.resume`。

| 症状 | 原因与处理 |
|---|---|
| `Not logged in`、`Invalid API key` | 运行 Agent 的 shell 里没有 `ANTHROPIC_API_KEY`；SDK 不读 `.env` |
| `CLINotFoundError`（Python） | 找不到 `claude`：检查安装、`cli_path` 和服务进程的 `PATH` |
| `Native CLI binary for <platform>-<arch> not found`（TypeScript） | 安装时跳过了可选依赖；重装或设 `pathToClaudeCodeExecutable` |
| `exists but failed to launch` | 二进制与容器架构或 libc 不匹配，或丢了执行权限 |
| `CLIJSONDecodeError: JSON message exceeded maximum buffer size` | Python 单条消息超过 `max_buffer_size`；调大后用 `continue_conversation=True` 接续 |
| 循环在结果之后抛错 | 单次 `query()` 产出错误结果后抛出是有意设计；用 try 包住循环 |
| MCP 工具看得到却不调用 | 没有用 `allowedTools` 授权 |

### Agent SDK 上线前检查表

- [ ] 面向他人的产品使用 API 密钥或云厂商凭证认证，而不是 claude.ai 登录。
- [ ] 显式决定了 `systemPrompt`，没有无意中落到最小默认提示。
- [ ] 显式设置了 `settingSources`；多租户场景为 `[]`，并配好 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`、每租户的 `CLAUDE_CONFIG_DIR` 与 `cwd`。
- [ ] 权限按最小集配置：无人值守用 `allowedTools` 加 `dontAsk`；`bypassPermissions` 只出现在隔离环境。
- [ ] 设置了 `maxTurns` 和留有余量的 `maxBudgetUsd`，并限制了子代理的深度与并发。
- [ ] 读取 `result` / `structured_output` 前检查了 `subtype` 和 `is_error`，并用 try 包住 `query()` 循环。
- [ ] 作为守卫的 hook 设置了 `onFailure: "block"`，并测试过允许与拒绝路径。
- [ ] 需要跨主机恢复的会话接入了 `SessionStore`，监控 `mirror_error`，并有自己的保留策略。
- [ ] Agent 运行在沙箱、容器或 VM 内，出站经过带域名允许列表的 proxy，凭证不在 Agent 环境中。
- [ ] 不把 `total_cost_usd` 当计费依据；TypeScript 中传 `env` 的地方都展开了 `...process.env`。

### 资源：法律与合规

| 你的身份 | 适用条款 |
|---|---|
| Team、Enterprise、Claude API 用户 | 商业条款（Commercial Terms） |
| Free、Pro、Max、Ultra 用户 | 消费者服务条款（Consumer Terms） |
| 通过 Amazon Bedrock、Google Cloud 的 Agent Platform 访问 | 现有商业协议继续适用，除非另有约定 |
| 使用 Claude Agent SDK（含为自己的客户提供产品） | Anthropic 商业服务条款；个别组件以其 LICENSE 文件为准 |

#### 在自己的产品里提供 Claude Code

除非另有约定，在产品或服务中预装或运行 Claude Code 需要同意商业条款，并满足：

- **二进制不得修改**：按 Anthropic 发布的方式安装运行，不得删除、禁用或限制内置的任何认证方式。
- **不得代付、转售或中介用量**：每个最终用户用自己的 API 密钥、订阅凭证或第三方推理提供商凭证认证，用量直接向该用户计费。
- **名称与徽标**：可以用纯文本如实说明产品预装或运行 Claude Code；不能把 Claude Code 或 Anthropic 的名称、徽标用进自己的产品名、功能名、公司名，也不能暗示 Anthropic 认可或合作。
- 集成 Agent SDK 时品牌使用是可选的：允许 "Claude Agent"、"{YourAgentName} Powered by Claude"；不允许 "Claude Code"、"Claude Code Agent" 或模仿 Claude Code 的视觉元素。

#### 认证方式的使用边界

| 认证方式 | 给谁用 |
|---|---|
| OAuth（订阅登录） | 仅供 Free、Pro、Max、Ultra、Team、Enterprise 订阅购买者，用于 Claude Code 和其他原生 Anthropic 应用的普通使用 |
| API 密钥（Claude Console 或受支持的云提供商） | 构建产品或服务的开发者，包括使用 Agent SDK 的 |

- 第三方开发者不得在自己的应用中提供 Claude.ai 登录，不得代表用户通过 Free、Pro、Max、Ultra 套餐凭据路由请求，也不得收集、存储或中介 Claude.ai 凭据或会话令牌。
- Pro、Max、Ultra 套餐公布的用量限制以 Claude Code 和 Agent SDK 的普通个人使用为前提。
- Anthropic 保留执行这些限制的权利，且可能不事先通知；拿不准时联系销售。

#### 医疗合规、使用政策与安全

- **BAA**：与 Anthropic 签有业务关联协议的客户，BAA 在两种配置下扩展到 Claude Code——HIPAA 配置（Claude Code CLI 或 Claude Desktop 的 Code 标签页，配合已启用 HIPAA 的 Claude for Enterprise 账户），或启用了零数据保留（ZDR）的 Claude Code。ZDR 并非对所有组织可用。
- HIPAA 配置**不涵盖**云端会话、Remote Control、Claude 移动应用中的 Claude Code，以及通过第三方云提供商或网关路由的 Claude Code。
- **可接受使用**：Claude Code 的使用受 Anthropic 使用政策（AUP）约束。
- **信任与安全**：见 Anthropic 信任中心（trust.anthropic.com）和透明度中心。
- **安全漏洞**：通过 HackerOne 的表单报告。SDK 的 bug 报告到 `anthropics/claude-agent-sdk-typescript` 与 `anthropics/claude-agent-sdk-python` 的 GitHub issues。

---

## 常见场景速查

| 需求 | 推荐做法 |
|---|---|
| 每次修改后必须跑测试 | `CLAUDE.md` 写明命令 + `Stop`/`PostToolUse` hook 或 CI 强制 |
| 绝不能执行某类命令 | `permissions.deny` 规则；需要硬隔离时加沙箱或 `PreToolUse` hook |
| 每次发版执行同一套检查 | Skill（有副作用时加 `disable-model-invocation: true`） |
| 读取 GitHub / 设计稿 / 工单 | MCP 服务器，或打包了连接器的插件 |
| 安全、测试、性能同时审查 | Subagents 或 `/code-review` |
| 团队共享 Skill + MCP + Hook | 打包成插件，放进团队 marketplace |
| 并行开发互不干扰 | `claude --worktree` / agent view |
| 定时自动运行 | 云端定时任务（Routines）/ 桌面计划任务 / `/loop` |
| 在 CI 或脚本中运行 | `claude -p`，陌生仓库加 `--bare` |
| 在自己的产品里嵌入 | Agent SDK |

## 安全与维护清单

- [ ] 必须遵守的规则用 hook、`deny` 规则或托管设置强制，`CLAUDE.md` 只作为上下文。
- [ ] `CLAUDE.md` 保持精简，子目录规则放在离代码近的位置，用 `/context` 确认加载。
- [ ] 秘密不写进提交的 `settings.json`、`.mcp.json` 或 `CLAUDE.md`；会话记录是明文，按需调整 `cleanupPeriodDays`。
- [ ] `bypassPermissions` 只在隔离环境（容器、VM）中使用。
- [ ] Bash 权限规则不是安全边界；需要硬保证时启用沙箱。
- [ ] 插件、MCP 服务器、hooks 以完整用户权限运行，只安装可信来源。
- [ ] `claude -p` 处理陌生仓库前使用 `--bare` 或禁用 hooks。
- [ ] 子代理和并行会话的写入范围互不重叠，必要时用 worktree 隔离。
- [ ] 修改配置后用 `/status`、`/permissions`、`/doctor` 实际验证。

## 常见问题

### `CLAUDE.md` 和 Skill 有什么区别？

`CLAUDE.md` 每次会话都会加载，适合“始终成立”的项目约定；Skill 只在相关任务或你显式调用 `/skill-name` 时才加载完整内容，适合步骤固定的流程。

### 为什么写在 `CLAUDE.md` 里的规则偶尔被违反？

`CLAUDE.md` 是给模型的上下文，不是强制配置。必须保证的事情用权限 `deny` 规则、hook 或托管设置实现。

### 我的仓库只有 `AGENTS.md`，Claude Code 会读吗？

默认只在工作目录及其上方没有 `CLAUDE.md`、`.claude/CLAUDE.md`、`CLAUDE.local.md` 时读取。两者并存时，可以在 `CLAUDE.md` 中写 `@AGENTS.md` 导入，或在 `/config` 调整项目说明设置。

### 修改设置后没有生效？

先用 `/status` 查看 `Setting sources`，确认生效的是哪个文件；检查 JSON 是否合法（不允许注释和尾逗号）；确认没有更高优先级的托管设置、`--settings` 或 `settings.local.json` 覆盖。`~/.claude.json` 与 `~/.claude/settings.json` 是两个不同文件，权限、hooks、env 写进前者会被忽略。

### MCP 服务器没出现？

项目级配置在仓库根的 `.mcp.json`，用户和本地作用域在 `~/.claude.json`；Claude Code 不读 `settings.json` 里的 `mcpServers`。用 `claude mcp list` 和会话内 `/mcp` 检查状态。

### 订阅用户为什么被按 API 计费？

环境里设置了 `ANTHROPIC_API_KEY` 时，它优先于 `/login` 的订阅登录。用 `/status` 确认，必要时 `unset ANTHROPIC_API_KEY`。

### `/rewind` 能替代 Git 吗？

不能。检查点只跟踪 Claude 通过文件编辑工具做的修改，Bash 命令改的文件和远程副作用都不会恢复。

### 子代理是不是越多越快？

不是。只有互相独立的任务适合并行；每个子代理都会独立消耗 token，同时写同一批文件还会冲突。

> 文档与功能会随 Claude Code 更新。复制配置前，请以当前版本和官方文档为准。
