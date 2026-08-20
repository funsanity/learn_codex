# Codex Configuration 配置全景指南

> 面向第一次配置 Codex 的用户：看完后应能找到配置入口、理解加载优先级，知道模型、权限、沙箱、网络、自定义、Skills、MCP 与子代理等配置的作用和操作流程。
>
> 主入口：[OpenAI 官方 Configuration 文档](https://learn.chatgpt.com/docs/configuration)。最后核对：2026-08-20。

## 阅读导航

- 第一次配置：从“**小白先走这条路线**”和“**一份安全的起步配置**”开始。
- 看不懂配置格式：阅读“**先认识 TOML**”。
- 不知道权限怎么选：阅读“**三套权限配置，实际有什么区别？**”。
- 想直接照做：跳到“**五个完整实战：从需求到验证**”。
- 想查属性：阅读“**核心 `config.toml` 属性**”和“**高级配置分组索引**”。
- 想让 Codex 更懂项目：阅读 `AGENTS.md`、Memories 和 Skills。
- 想连接外部系统：阅读 Plugins 与 MCP。
- 想并行处理任务：阅读 Subagents。
- 配置不生效：阅读文末“**常见问题**”。

## Configuration 到底管什么？

Configuration 的目标是：设置默认行为、提供长期上下文，并让 ChatGPT 与 Codex 在不同聊天、仓库和机器上的工作方式尽量一致。

```text
Codex Configuration
│
├─ 个人界面设置
│  ├─ General / Profile / Appearance / Voice
│  ├─ Configuration / Personalization
│  └─ Keyboard shortcuts / Personality
│
├─ config.toml
│  ├─ 模型、推理、表达风格
│  ├─ 审批、沙箱、文件系统、网络
│  ├─ Tools、Feature flags、TUI、日志
│  ├─ MCP、Apps、Plugins、Hooks
│  └─ Profiles、Provider、Telemetry
│
├─ 长期上下文与复用
│  ├─ AGENTS.md / Memories
│  └─ Skills / Plugins
│
├─ Agent 配置
│  ├─ Subagents / Speed
│  └─ Rules
│
└─ 平台与扩展
   ├─ MCP / Record & Replay
   ├─ Linux desktop
   └─ Windows desktop / Sandbox / WSL
```

### 最常用的四个界面配置

Configuration 页面直接展示了四项最影响 Codex 行为的设置：

| 界面属性 | 对应配置 | 目的 | 新手建议 |
|---|---|---|---|
| Approval policy | `approval_policy` | 决定 Codex 何时需要请求批准 | 从 `on-request` 开始 |
| Sandbox settings | `sandbox_mode` 或 Permission Profile | 决定命令能读写哪些文件、能否越出工作区 | 从 `workspace-write` 开始 |
| Allow network access | `sandbox_workspace_write.network_access` | 在 workspace-write 沙箱中是否允许命令访问网络 | 默认关闭，按需开启 |
| Personality | `personality` | 设置支持模型的默认沟通风格 | `pragmatic` 或 `friendly` |

> 审批策略回答“什么时候问你”，沙箱回答“即使获准前，命令能做什么”。两者是独立维度，不要混为一谈。

## 小白先走这条路线

如果你第一次使用 Codex，先不要阅读全部高级属性。按下面路线完成第一次配置：

```text
打开 config.toml
      │
      ▼
粘贴“安全起步配置”
      │
      ▼
重启 Codex
      │
      ▼
用只读任务测试：解释项目结构
      │
      ▼
用写入任务测试：创建一个测试文件
      │
      ▼
观察审批提示和文件写入范围
      │
      ▼
再按需求增加 AGENTS.md / Skill / MCP / Subagents
```

完成后你应该看到这些效果：

- Codex 默认使用你设置的沟通风格；
- 普通命令在工作区内运行，不能随意写工作区外文件；
- 需要更高权限时 Codex 会说明原因并请求批准；
- 命令默认不能联网；
- 项目规则通过 `AGENTS.md` 自动生效。

如果结果与上述不一致，先跳到文末“常见问题”，不要继续堆更多配置。

## 先认识 TOML：只需掌握四件事

`config.toml` 使用 TOML 格式。小白最容易踩的坑不是 Codex，而是把配置写到了错误的表中。

### 顶层键

```toml
personality = "pragmatic"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
```

格式是 `属性 = 值`。字符串使用双引号，布尔值是没有引号的 `true` 或 `false`。

### 表

```toml
[features]
memories = true
multi_agent = true
```

`[features]` 表示后面的键属于 `features`。上例相当于 `features.memories` 和 `features.multi_agent`。

### 嵌套表

```toml
[mcp_servers.docs]
url = "https://example.com/mcp"
enabled = true
```

这里 `docs` 是你给 MCP Server 起的名称，不是固定值。

### 对象数组

```toml
[[skills.config]]
path = "/absolute/path/to/skill/SKILL.md"
enabled = false

[[skills.config]]
path = "/absolute/path/to/another-skill/SKILL.md"
enabled = true
```

双中括号表示可以重复出现多项。

### 常见错误

错误一：把顶层键放在表后面，以为已经“回到顶层”。

```toml
[features]
memories = true

# 错误理解：这一行仍属于 [features]，不会自动回到顶层
personality = "pragmatic"
```

正确做法：把所有顶层键放在第一个 `[table]` 之前。

错误二：把布尔值写成字符串。

```toml
# 错误
enabled = "false"

# 正确
enabled = false
```

错误三：复制示例后忘记替换占位符，例如 `<server-name>`、`<model-id>`。尖括号内容通常只是说明，不是可以原样运行的值。

## 配置入口与文件位置

### 图形界面

- ChatGPT 桌面端：打开 **Settings**，在个人、Configuration、Personalization、MCP servers 等页面调整。
- Codex IDE 扩展：右上角齿轮 → **Codex Settings > Open config.toml**。
- MCP：桌面端或 IDE 的 **Settings > MCP servers**。

### 文件与目录

```text
~/.codex/                         # 默认 CODEX_HOME
├── config.toml                   # 用户级持久配置
├── <profile>.config.toml         # 命名 Profile
├── AGENTS.md                     # 个人全局指令
├── agents/*.toml                 # 个人自定义 Agent
├── rules/*.rules                 # 用户命令规则
├── memories/                     # 本地生成的 Memories
├── auth / logs / sessions ...    # 认证、日志和会话状态
└── ...

repo-root/
├── .codex/
│   ├── config.toml               # 项目级配置；仅可信项目加载
│   ├── agents/*.toml             # 项目自定义 Agent
│   └── rules/*.rules             # 项目命令规则
├── AGENTS.md                     # 项目指令
└── .agents/skills/               # 项目 Skills
```

CLI、IDE 扩展与桌面端在同一个 Codex Host 上会共享相应配置。ChatGPT Web 不读取本地 `config.toml`。

## 配置优先级

同一个属性出现在多处时，按下面顺序取值，越上面优先级越高：

```text
高  1. CLI flags 与 -c / --config 一次性覆盖
    2. 项目 .codex/config.toml
       项目根 → 当前目录逐层加载，离当前目录最近者胜出
    3. --profile 选中的 ~/.codex/<profile>.config.toml
    4. 用户 ~/.codex/config.toml
    5. 系统 /etc/codex/config.toml（Unix）
低  6. Codex 内置默认值
```

项目被标记为不可信时，项目内 `.codex/` 的 config、hooks 和 rules 都不会加载；用户级与系统级配置仍然加载。组织管理的设备还可能通过 `requirements.toml` 限制可选值，这类限制不是普通配置可以覆盖的。

出于机器本地安全边界，项目级 `.codex/config.toml` 不能覆盖 Provider、认证、通知、Profile 选择和 Telemetry 路由等键。下列键写在项目配置中会被忽略，应放在用户级配置：`openai_base_url`、`chatgpt_base_url`、`apps_mcp_product_sku`、`model_provider`、`model_providers`、`notify`、`profile`、`profiles`、`experimental_realtime_ws_base_url`、`otel`。

### 一次性覆盖

```bash
# 专用 CLI 参数
codex --model <model-id>

# 通用 TOML 值覆盖；右侧必须是合法 TOML，不是 JSON
codex -c 'personality="pragmatic"'
codex -c 'sandbox_workspace_write.network_access=true'
```

### Profile

创建 `~/.codex/deep-review.config.toml`：

```toml
model_reasoning_effort = "high"
approval_policy = "on-request"
sandbox_mode = "read-only"
```

使用：

```bash
codex --profile deep-review
codex exec --profile deep-review "审查当前改动"
```

Profile 文件直接使用顶层键，不要再套 `[profiles.deep-review]`。

## 一份安全的起步配置

只复制你理解并需要的键，不要直接启用所有实验功能：

```toml
# ~/.codex/config.toml

# 模型行为
model_reasoning_effort = "medium"
personality = "pragmatic"
web_search = "cached"

# 批准与隔离
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false

# 常用稳定能力
[features]
apps = true
goals = true
hooks = true
multi_agent = true
personality = true
remote_plugin = true
shell_snapshot = true
shell_tool = true
fast_mode = true

# 子代理
[agents]
enabled = true
max_concurrent_threads_per_session = 4
interrupt_message = true

# 对 shell 环境中的 KEY / SECRET / TOKEN 等名称应用默认过滤
[shell_environment_policy]
ignore_default_excludes = false

[shell_environment_policy.filters]
"PATH" = "include"
"HOME" = "include"
```

验证流程：

1. 保存文件并重启 Codex 或新开 TUI 会话。
2. 运行 `codex status` 核对工作区和权限。
3. 在 TUI 中检查 `/permissions`、`/personality`、`/mcp` 等当前状态。
4. 先执行只读任务，再执行一次工作区内的小改动，确认审批和沙箱行为符合预期。

## 核心 `config.toml` 属性

### 模型、推理与回答风格

| 属性 | 类型/常见值 | 目的 |
|---|---|---|
| `model` | string | CLI/IDE 默认模型 |
| `review_model` | string，可选 | `/review` 专用模型；未设置时使用当前会话模型 |
| `model_provider` | string；默认 `openai` | 从 `model_providers` 选择模型提供方 |
| `model_context_window` | number | 覆盖模型上下文窗口大小；通常让模型目录自动决定 |
| `model_auto_compact_token_limit` | number | 达到阈值时自动压缩上下文 |
| `model_reasoning_effort` | string | 支持模型的推理强度，如 `low`、`medium`、`high` |
| `plan_mode_reasoning_effort` | string | Plan mode 单独使用的推理强度 |
| `model_reasoning_summary` | string | 控制是否及如何生成推理摘要 |
| `model_verbosity` | string | 控制回答详略程度 |
| `personality` | `friendly` / `pragmatic` / `none` | 默认沟通风格；会话中可用 `/personality` 覆盖 |
| `service_tier` | string | 选择模型服务层，例如支持时使用 `fast` |
| `model_catalog_json` | path | 使用自定义模型目录；高级用途 |
| `model_instructions_file` | path | 从文件加载模型基础指令；高级用途 |
| `instructions` | string | 附加用户指令 |
| `developer_instructions` | string | 附加开发者级指令；通常更适合用 `AGENTS.md` 管项目规范 |

模型可用值会变化，应从当前客户端或 [Models 文档](https://learn.chatgpt.com/docs/models) 选择，不要长期复制过时模型名。

### 审批、沙箱与网络

| 属性 | 类型/常见值 | 目的 |
|---|---|---|
| `approval_policy` | `untrusted` / `on-request` / `never`，或 granular 表 | 控制何时请求用户批准 |
| `approvals_reviewer` | reviewer 配置 | 选择审批请求的审查方式 |
| `auto_review.policy` | policy | 控制自动审查策略 |
| `sandbox_mode` | `read-only` / `workspace-write` / `danger-full-access` | 控制命令的文件系统和网络隔离范围 |
| `sandbox_workspace_write.writable_roots` | path 数组 | 给 workspace-write 增加明确可写根目录 |
| `sandbox_workspace_write.network_access` | boolean | workspace-write 下允许或禁止命令联网 |
| `sandbox_workspace_write.exclude_tmpdir_env_var` | boolean | 是否排除环境变量指向的临时目录 |
| `sandbox_workspace_write.exclude_slash_tmp` | boolean | 是否排除 `/tmp` |
| `default_permissions` | Profile 名 | 选择内置或自定义 Permission Profile |
| `windows.sandbox` | `elevated` / `unelevated` | Windows 原生沙箱实现；官方推荐可用时选 `elevated` |
| `windows.sandbox_private_desktop` | boolean | Windows 沙箱是否使用私有桌面 |

内置 Permission Profiles：`:read-only`、`:workspace`、`:danger-full-access`。自定义 Profile 使用 `[permissions.<name>]`，可精确配置工作区根目录、文件路径/Glob 与网络策略。

Granular approval 可分别控制：

```toml
approval_policy = { granular = {
  sandbox_approval = true,
  rules = true,
  mcp_elicitations = true,
  request_permissions = false,
  skill_approval = false
} }
```

| Granular 属性 | 目的 |
|---|---|
| `sandbox_approval` | 沙箱外执行是否需要批准 |
| `rules` | 命令规则触发时是否走批准流程 |
| `mcp_elicitations` | MCP Server 向用户请求额外输入时是否批准 |
| `request_permissions` | 动态请求权限时是否批准 |
| `skill_approval` | Skill 相关审批是否启用 |

### 三套权限配置，实际有什么区别？

#### 场景 A：第一次使用，推荐

```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false
```

实际行为：

- Codex 可以读取项目并修改工作区中的普通文件；
- 命令不能默认联网；
- 想写工作区外目录或执行需要额外权限的动作时，会先解释原因并请求批准；
- 适合日常开发和学习。

测试方法：让 Codex“在当前项目创建 `hello.txt`”，应该可以在工作区内完成；再让它“下载一个在线文件”，应该遇到网络限制或请求额外权限。

#### 场景 B：只读审查

```toml
approval_policy = "on-request"
sandbox_mode = "read-only"
```

实际行为：

- 适合代码审查、架构分析和排查问题；
- Codex 可以读文件，但写文件需要改变权限或请求批准；
- 可以降低“只想分析，却意外修改代码”的风险。

测试方法：要求 Codex“分析当前项目但不要修改”，随后检查文件状态应无变化。

#### 场景 C：自动化任务

```toml
approval_policy = "never"
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = false
```

实际行为：

- Codex 不会停下来向人询问；
- 超出当前沙箱权限的动作会失败，而不是自动获得更大权限；
- 适合 CI 或确定性很高的非交互流程；
- 不等于 `danger-full-access`。

测试方法：使用 `codex exec` 跑一个只需要当前工作区的任务，并确认失败时能从退出状态和输出中定位权限问题。

#### 为什么不建议新手直接使用 Full Access？

`danger-full-access` 会显著放宽隔离边界。它可能是某些受控环境的合理选择，但新手容易把“少弹窗”误认为“配置成功”。正确顺序是：先用 `workspace-write` 找出任务真正需要的目录和网络，再只开放必要能力。

### 搜索、工具与 Apps

| 属性 | 类型/默认 | 目的 |
|---|---|---|
| `web_search` | `cached`（默认）/ `indexed` / `live` / `disabled` | 控制 Web Search 数据来源与是否联网 |
| `tools.view_image` | boolean | 启用本地图像查看工具 |
| `apps.<id>.enabled` | boolean | 启停指定 App/Connector |
| `apps._default.enabled` | boolean | Apps 默认启停策略 |
| `apps.<id>.destructive_enabled` | boolean | 是否允许该 App 暴露破坏性动作 |
| `apps.<id>.open_world_enabled` | boolean | 是否允许该 App 暴露开放世界/外部访问动作 |
| `apps.<id>.default_tools_enabled` | boolean | 该 App 工具默认是否启用 |
| `apps.<id>.tools.<tool>.enabled` | boolean | 单工具启停覆盖 |
| `apps.<id>.default_tools_approval_mode` | `auto` / `prompt` / `writes` / `approve` | App 工具默认审批模式 |
| `apps.<id>.tools.<tool>.approval_mode` | 同上 | 单工具审批覆盖 |
| `tool_suggest.discoverables` | 配置集合 | 控制可由 Tool Search 发现的工具 |
| `tool_suggest.disabled_tools` | 工具列表 | 从 Tool Search 中排除工具 |

`cached` 使用 OpenAI 维护的预索引结果；`indexed` 通过搜索索引门控外部访问；`live` 获取最新网页；`disabled` 关闭搜索。无论哪种模式，网页内容都应视为不可信输入。

### 会话、历史、日志与通知

| 属性 | 目的 |
|---|---|
| `history.persistence` | 控制会话历史是否持久化 |
| `history.max_bytes` | 限制历史存储大小 |
| `log_dir` | 指定本地日志目录；显式设置后也启用纯文本 `codex-tui.log` |
| `sqlite_home` | 指定 SQLite 状态目录；优先于 `CODEX_SQLITE_HOME` |
| `notify` | 配置外部通知命令 |
| `tui.notifications` | 控制 TUI 内部通知 |
| `tui.notification_method` | 通知实现方式 |
| `tui.notification_condition` | 何时触发通知 |
| `check_for_update_on_startup` | 启动时是否检查更新 |
| `feedback.enabled` | 是否启用反馈入口 |
| `analytics.enabled` | 是否启用产品分析 |
| `tool_output_token_limit` | 限制工具输出进入上下文的 Token 数 |
| `background_terminal_max_timeout` | 后台终端最长超时 |

日志排错示例：

```bash
RUST_LOG=debug codex -c log_dir=./.codex-log
tail -F ./.codex-log/codex-tui.log
```

### TUI

| 属性 | 目的 |
|---|---|
| `tui.notifications` | TUI 通知开关或筛选 |
| `tui.animations` | 动画开关 |
| `tui.alternate_screen` | 是否使用终端 alternate screen |
| `tui.resume_cwd` | 恢复会话时如何处理工作目录 |
| `tui.vim_mode_default` | 默认启用 Vim 模式 |
| `tui.raw_output_mode` | 原始输出模式 |
| `tui.show_tooltips` | 是否显示快捷提示 |
| `tui.status_line` | 状态栏内容 |
| `tui.terminal_title` | 是否设置终端标题 |
| `tui.theme` | 主题 |
| `tui.keymap.<context>.<action>` | 给特定上下文动作绑定快捷键；空数组表示解绑 |

```toml
[tui.keymap.global]
open_transcript = "ctrl-t"

[tui.keymap.composer]
submit = ["enter", "ctrl-m"]

[tui.keymap.chat]
interrupt_turn = "f12"
```

### Shell 环境变量转发

`[shell_environment_policy]` 控制 Codex 启动的命令能看到哪些环境变量：

| 属性 | 目的 |
|---|---|
| `inherit` | 控制继承范围 |
| `ignore_default_excludes` | 是否忽略默认秘密名称过滤；想过滤 `KEY`/`SECRET`/`TOKEN` 时设为 `false` |
| `filters` | 按变量名逐项 include/exclude |
| `exclude` | 旧式排除列表 |
| `include_only` | 旧式仅允许列表 |
| `set` | 给子命令固定设置变量 |
| `experimental_use_profile` | 实验性：是否加载 shell profile |

密钥优先通过进程环境注入，并尽量只在需要的命令范围内存在。

## Feature flags

写在 `[features]` 下；未写的键沿用默认值。

| Key | 默认 | 成熟度 | 作用 |
|---|:---:|---|---|
| `apps` | `true` | Stable | App/Connector 集成 |
| `goals` | `true` | Stable | 持久目标与自动继续 |
| `hooks` | `true` | Stable | 生命周期 Hooks |
| `fast_mode` | `true` | Stable | Fast 服务层选择 |
| `memories` | `false` | Experimental | 本地 Memories |
| `multi_agent` | `true` | Stable | 子代理协作工具 |
| `personality` | `true` | Stable | Personality 选择 |
| `remote_plugin` | `true` | Stable | 远程 Plugin 目录 |
| `shell_snapshot` | `true` | Stable | 缓存 shell 环境以加速重复命令 |
| `shell_tool` | `true` | Stable | 默认 Shell 工具 |
| `unified_exec` | Windows 外默认 `true` | Stable | PTY 支持的统一执行工具 |
| `web_search` | `true` | Deprecated | 旧开关；改用顶层 `web_search` |
| `web_search_cached` | `false` | Deprecated | 旧开关；映射到 `web_search = "cached"` |
| `web_search_request` | `false` | Deprecated | 旧开关；映射到 `web_search = "live"` |

启用方式：

```toml
[features]
memories = true
```

```bash
codex --enable memories
codex --enable feature_a --enable feature_b
```

实验性与 under-development 功能可能变化；普通用户应优先使用官方列出的 Stable 功能。

## 环境变量

`config.toml` 适合持久设置；环境变量适合临时覆盖、自动化秘密、安装器行为和诊断。

| 变量 | 默认/范围 | 目的 |
|---|---|---|
| `CODEX_HOME` | `~/.codex` | Codex 配置、认证、日志、会话、技能与包状态根目录；自定义目录必须已存在 |
| `CODEX_SQLITE_HOME` | `CODEX_HOME` | SQLite 状态目录；`sqlite_home` 配置优先 |
| `CODEX_NON_INTERACTIVE` | `false` | 安装脚本跳过询问并采用默认答案，适合无人值守安装 |
| `CODEX_INSTALL_DIR` | 平台默认 bin 目录 | 改变 `codex` 命令安装目录，不改变包缓存位置 |
| `CODEX_API_KEY` | 仅 `codex exec` | 给单次非交互运行提供 API Key；应内联、缩小作用域 |
| `CODEX_ACCESS_TOKEN` | CLI/app-server/可信自动化 | 提供 ChatGPT/Codex Access Token |
| `CODEX_CA_CERTIFICATE` | PEM 路径 | 企业 TLS 拦截或私有 CA；优先于 `SSL_CERT_FILE` |
| `SSL_CERT_FILE` | PEM 路径 | CA Bundle 回退路径 |
| `RUST_LOG` | `error` 等 | 控制 CLI/app-server 日志级别，可用 `warn`、`info`、`debug`、`trace` |

Provider API Key 的变量名由 `model_providers.<id>.env_key` 自行指定，不是固定 Codex 环境变量。

## 高级配置分组索引

以下是 Configuration Reference 中的高级配置族。普通用户通常不需要逐键修改；遇到明确需求时再进入官方 Reference 搜索属性名。

| 配置族 | 解决的问题 | 关键前缀/属性 |
|---|---|---|
| 自定义 Model Provider | 接入兼容 Provider、Bedrock、Ollama、LM Studio | `model_providers.<id>.*`、`oss_provider` |
| Provider 重试与流式连接 | HTTP/stream 重试、空闲超时、WebSocket | `request_max_retries`、`stream_max_retries`、`stream_idle_timeout_ms` |
| Provider 认证 | 外部命令生成或刷新认证 | `model_providers.<id>.auth.*` |
| Named Permissions | 可复用文件系统与网络策略 | `default_permissions`、`permissions.<name>.*` |
| Hooks | 在生命周期事件运行脚本 | `hooks.<Event>[].hooks[]` |
| Network Proxy | 沙箱网络代理与域名/Socket 控制 | `features.network_proxy.*` |
| MCP | 本地/远程 MCP 与工具审批 | `mcp_servers.<id>.*` |
| Apps | Connector 启停、工具权限与审批 | `apps.<id>.*` |
| Plugins | Plugin 内 MCP 的策略覆盖 | `plugins.<plugin>.mcp_servers.*` |
| Skills | 单个 Skill 启停 | `skills.config[]` |
| Agents | 子代理默认值与自定义角色 | `agents.*`、`.codex/agents/*.toml` |
| Memories | 提取、使用、归并和额度阈值 | `memories.*` |
| OTel | 日志、Trace、Metrics 导出 | `otel.*` |
| Desktop file handlers | 自定义打开文件的应用与参数 | `desktop.custom_file_handlers.<id>.*` |
| TUI | 通知、主题、状态栏、快捷键 | `tui.*` |
| Project discovery | 项目根和指令文件发现 | `project_root_markers`、`project_doc_*` |
| History / State | 历史、SQLite、日志目录 | `history.*`、`sqlite_home`、`log_dir` |
| Managed requirements | 管理员强制许可、模型、功能和来源 | `requirements.toml` 中的 `allowed_*`、`features.*` 等 |

完整、可搜索的键表见 [Configuration Reference](https://learn.chatgpt.com/docs/config-file/config-reference)，概念与示例见 [Advanced Configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)。

## 五个完整实战：从需求到验证

下面的例子故意使用普通项目和安全权限。每个案例都包含目标、步骤、预期结果和失败排查。

### 实战一：第一次创建个人配置

目标：让 Codex 默认使用务实风格，只能写当前工作区，并在需要额外权限时询问。

#### 第一步：打开配置文件

最简单的方法：

- IDE 扩展：右上角齿轮 → **Codex Settings > Open config.toml**；
- 桌面端：Configuration 中选择 **Open config.toml**；
- macOS/Linux 路径：`~/.codex/config.toml`；
- Windows 路径：`%USERPROFILE%\.codex\config.toml`。

如果文件不存在，先创建 `.codex` 目录，再用文本编辑器新建 `config.toml`。如果文件已有内容，只增加或修改对应键，不要整文件覆盖；同一个 TOML 表或键重复声明会导致解析问题。

#### 第二步：写入最小配置

```toml
personality = "pragmatic"
model_reasoning_effort = "medium"
approval_policy = "on-request"
sandbox_mode = "workspace-write"
web_search = "cached"

[sandbox_workspace_write]
network_access = false
```

每一行的意义：

- `personality`：回答更直接、偏执行；
- `model_reasoning_effort`：使用均衡推理强度；
- `approval_policy`：必要时向你询问；
- `sandbox_mode`：允许修改工作区，限制工作区外写入；
- `web_search`：搜索使用预索引缓存结果；
- `network_access`：Shell 命令默认不直接联网。

#### 第三步：重启并验证

新开 Codex 会话，输入：

```text
请说明当前工作区、沙箱模式、审批策略和回答风格。不要修改任何文件。
```

然后输入：

```text
请在当前工作区创建 codex-config-test.txt，写入“配置成功”，完成后告诉我文件路径。
```

预期结果：文件可以在当前工作区创建；如果动作需要越出权限边界，Codex 会请求批准。

失败排查：

- 完全没生效：检查是否修改了当前 `CODEX_HOME` 下的文件，并重启会话；
- TOML 报错：检查重复键、缺失引号或把顶层键放进了错误表；
- 无法写工作区：检查是否误设为 `read-only`，或项目目录不在当前工作区；
- 没有联网：这是本例预期行为，需要时再把 `network_access` 改为 `true`。

### 实战二：给 Node.js 项目增加团队规则

目标：无论谁使用 Codex，都优先使用 `pnpm`，修改后运行测试，并且不直接修改生成目录。

#### 第一步：仓库根目录创建 `AGENTS.md`

```md
# Project instructions

## 项目概况

- 这是 Node.js 项目，包管理器统一使用 pnpm。
- 源代码位于 `src/`，测试位于 `tests/`。
- `dist/` 是构建产物，不允许手工修改。

## 修改流程

1. 修改前先定位相关源码和现有测试。
2. 修改业务逻辑后运行 `pnpm test`。
3. 提交结果前运行 `pnpm lint`。
4. 如果测试失败，报告失败信息和可能原因，不要隐藏失败。

## 依赖规则

- 新增生产依赖前先说明用途、体积和替代方案，并请求确认。
```

#### 第二步：可选的项目配置

创建 `.codex/config.toml`：

```toml
approval_policy = "on-request"
sandbox_mode = "workspace-write"
model_reasoning_effort = "medium"
```

项目配置只有在项目被信任时加载。不要在项目文件中放 Token、用户认证或机器私有 Provider 配置。

#### 第三步：验证规则来源

从仓库根目录启动 Codex，然后输入：

```text
总结你加载的项目指令，并告诉我：包管理器、测试命令、禁止修改的目录分别是什么。不要执行命令。
```

预期回答应包含 `pnpm`、`pnpm test` 和 `dist/`。如果 Codex 没读到：

- 确认文件名精确为 `AGENTS.md`；
- 确认文件非空；
- 检查更近目录是否存在 `AGENTS.override.md`；
- 重启 Codex，因为指令链在会话开始时构建。

### 实战三：连接一个文档 MCP Server

目标：让 Codex 能通过 MCP 查询开发文档，并确认连接是否成功。

#### 方法 A：CLI 添加 STDIO Server

这个官方示例依赖本机已经安装 Node.js/npm，首次通过 `npx` 获取包时还需要网络访问。如果你尚未安装 Node.js，可以先跳过本例，或改用已经获得 URL 的 Streamable HTTP Server。

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
codex mcp list
```

这条命令的结构：

```text
codex mcp add  context7  --  npx -y @upstash/context7-mcp
│              │            └─ 启动 Server 的命令与参数
│              └─ 你为 Server 取的本地名字
└─ Codex MCP 管理命令
```

#### 方法 B：手工写 `config.toml`

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
enabled = true
startup_timeout_sec = 20
tool_timeout_sec = 60
default_tools_approval_mode = "prompt"
```

#### 验证

1. 重启 Codex。
2. 在 TUI 输入 `/mcp`，应看到 `context7`。
3. 提问：

```text
请使用 context7 查询我当前使用的框架中，如何配置一个最小测试，并说明你调用了哪个 MCP 工具。
```

常见失败：

- Server 不出现：确认配置表名是 `[mcp_servers.context7]`，然后重启；
- 找不到 `npx`：先安装 Node.js/npm，并确认终端执行 `npx --version` 有输出；
- 启动超时：确认 `npx` 在 `PATH` 中，必要时增大 `startup_timeout_sec`；
- 工具执行超时：增大 `tool_timeout_sec`，同时检查 Server 自身日志；
- 需要认证：远程 OAuth Server 使用 `codex mcp login <server-name>`；
- Token 问题：用 `bearer_token_env_var` 指向环境变量，不要把秘密直接写进 TOML。

### 实战四：创建第一个项目 Skill

目标：把“修改 README 后检查链接和格式”变成一个可重复调用的流程。

目录：

```text
repo-root/
└── .agents/
    └── skills/
        └── docs-check/
            └── SKILL.md
```

`SKILL.md`：

```md
---
name: docs-check
description: 检查 Markdown 文档的结构、链接和代码块。用户要求检查 README、文档质量、Markdown 链接或文档发布前检查时使用；不要用于修改业务代码。
---

1. 找出本次修改涉及的 Markdown 文件。
2. 检查标题层级是否跳级。
3. 检查围栏代码块是否闭合。
4. 检查本地链接目标是否存在。
5. 如项目已有 Markdown lint 命令，运行该命令。
6. 只修复有明确依据的问题。
7. 输出：修改内容、验证命令、仍无法验证的外部链接。
```

验证：

```text
$docs-check 请检查 README.md，但先报告问题，得到确认后再修改。
```

预期结果：Codex 加载 Skill 的完整步骤，检查范围限于文档，不会漂移到业务代码。

如果不能自动触发，重点改 `description`：同时写清用途、触发词和“不适用”的边界。更新后通常会自动检测；仍未出现时重启 Codex。

### 实战五：三个子代理并行审查

目标：让不同子代理分别检查安全、测试和可维护性，主线程只接收归纳后的结果。

```text
请并行审查当前分支：
1. 一个 explorer 只读梳理受影响的调用链；
2. 一个子代理检查安全与权限风险；
3. 一个子代理检查测试缺口和边界条件。
不要让多个代理修改文件。等待全部完成后，按严重度汇总，
每条发现给出文件路径、依据和建议的验证方法。
```

开始前使用保守配置：

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 3
default_subagent_reasoning_effort = "medium"
```

预期结果：三个线程可以独立工作，主代理等待后统一汇总。若结果重复，下一次提示中把三个职责边界写得更窄；若消耗过高，减少并发数或只委派最嘈杂的探索任务。

---

## Part II：Customization 与 Agent 配置

## 一句话理解 Codex 自定义

Codex 的自定义不是一个配置文件，而是五个互补层：

```text
                         你的任务
                            │
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ AGENTS.md  ── 每次都要遵守的项目规则、命令和约定             │
├──────────────────────────────────────────────────────────────┤
│ Memories   ── 从过去工作中提取、供未来参考的本地上下文        │
├──────────────────────────────────────────────────────────────┤
│ Skills     ── 可复用的标准作业流程、脚本、资料和模板           │
├──────────────────────────────────────────────────────────────┤
│ MCP        ── GitHub、Figma、浏览器、内部系统等外部工具与数据  │
├──────────────────────────────────────────────────────────────┤
│ Subagents  ── 把独立、并行或专业任务交给不同子代理             │
└──────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    更稳定、可复用的结果
```

它们解决的问题不同：

| 配置层 | 核心目的 | 典型例子 | 是否应提交到仓库 |
|---|---|---|---|
| `AGENTS.md` | 约束 Codex 在项目中的长期行为 | 测试命令、代码规范、评审要求 | 项目规则建议提交 |
| Memories | 记住过去工作中的有用背景 | 用户偏好、近期项目上下文 | 否，本地生成状态 |
| Skills | 封装可重复执行的工作流 | 发版、修复 CI、更新文档 | 项目 Skill 建议提交 |
| MCP | 连接仓库以外的工具和数据 | GitHub、Figma、Linear、内部文档 | 按团队安全策略决定 |
| Subagents | 分工、并行、隔离噪声 | 安全审查、测试、代码探索 | 自定义 Agent 可提交 |
| Plugins | 安装和分发 Skills + MCP 等能力 | 安装成熟工作流及连接器 | 通常通过插件目录安装 |

## 我应该用哪一种？

```text
开始
 │
 ├─ 这是“每次进入仓库都必须遵守”的规则吗？
 │      └─ 是 → AGENTS.md
 │
 ├─ 这是从过去对话中自然积累、允许偶尔不完整的背景吗？
 │      └─ 是 → Memories
 │
 ├─ 这是会重复执行、步骤相对固定的流程吗？
 │      ├─ 已有成熟插件 → 安装 Plugin
 │      └─ 没有合适插件 → 创建 Skill
 │
 ├─ 执行流程需要访问外部服务或数据吗？
 │      └─ 是 → MCP；通常再配合 Skill 描述“怎么用”
 │
 └─ 任务能拆成独立并行部分，或需要不同专业角色吗？
        └─ 是 → Subagents
```

最重要的边界：

- 必须稳定执行的团队规范放进 `AGENTS.md`，不要只依赖 Memories。
- “怎么做”放进 Skill，“连接什么系统”交给 MCP。
- 能用 lint、类型检查、测试、pre-commit hook 强制执行的规则，应同时落到工程基础设施里。
- 子代理更适合读取、搜索、测试、归纳等独立工作；多个代理同时修改同一批文件容易冲突。

## 推荐落地顺序

官方建议按以下顺序逐层建设：

```text
1. AGENTS.md
   写清仓库约定，并用测试、lint、hooks 强制关键规则
        │
        ▼
2. Plugin / Skill
   有现成插件先安装；没有再创建自己的可复用流程
        │
        ▼
3. MCP
   工作流需要外部系统时，再接入工具与数据
        │
        ▼
4. Subagents
   流程成熟后，把独立、嘈杂或专业工作并行委派
```

不要一开始就把所有能力都打开。先让最小配置跑通，再逐层增加。

---

## `AGENTS.md`：持久项目指令

### 目的

`AGENTS.md` 是 Codex 开始工作前读取的持久指令。适合写：

- 构建、测试、lint、格式化命令；
- 目录结构和优先阅读路径；
- 代码、文档、提交和评审约定；
- 某个目录特有的限制；
- 团队反复提出的审查意见。

保持短小。发生重复错误、反复读取无关文件或多次出现相同 PR 反馈时，再补规则。

### 放在哪里

```text
~/.codex/
├── AGENTS.md                 # 个人全局偏好
└── AGENTS.override.md        # 临时全局覆盖；存在时优先于 AGENTS.md

repo-root/
├── AGENTS.md                 # 全仓库规则
└── services/
    └── payments/
        ├── AGENTS.md         # 若同目录有 override，则此文件不加载
        └── AGENTS.override.md# payments 目录的覆盖规则
```

### 加载与优先级

Codex 每次启动时构建一次指令链：

```text
全局 AGENTS.override.md（若存在）
否则读取全局 AGENTS.md
                │
                ▼
仓库根目录 AGENTS.override.md / AGENTS.md / fallback
                │
                ▼
中间目录 AGENTS.override.md / AGENTS.md / fallback
                │
                ▼
当前工作目录 AGENTS.override.md / AGENTS.md / fallback
                │
                ▼
越靠近当前目录的内容越晚合并，因此优先级越高
```

每个目录最多加载一个文件，检查顺序为：

1. `AGENTS.override.md`
2. `AGENTS.md`
3. `project_doc_fallback_filenames` 中声明的备用文件名

空文件会被忽略。合并内容默认最多为 32 KiB，达到 `project_doc_max_bytes` 后停止追加。

### 相关属性

| 属性 | 类型/默认值 | 作用 | 什么时候改 |
|---|---|---|---|
| `CODEX_HOME` | 环境变量；默认 `~/.codex` | 改变 Codex 配置、状态和全局 `AGENTS.md` 的根目录 | 自动化账号或隔离配置时 |
| `project_doc_fallback_filenames` | 字符串数组 | 把已有文件名也当作项目指令，如 `TEAM_GUIDE.md` | 团队已有规范文件时 |
| `project_doc_max_bytes` | 字节数；默认 32 KiB | 限制合并后项目指令的总大小 | 指令被截断时；更推荐拆分到子目录 |

示例 `~/.codex/config.toml`：

```toml
project_doc_fallback_filenames = ["TEAM_GUIDE.md", ".agents.md"]
project_doc_max_bytes = 65536
```

### 最小可用示例

```md
# AGENTS.md

## 常用命令

- 安装依赖：`pnpm install`
- 单元测试：`pnpm test`
- 静态检查：`pnpm lint`

## 修改要求

- 修改业务逻辑后必须补充或更新测试。
- 不要直接修改生成文件。
- 新增生产依赖前先说明原因并请求确认。

## Code Review Rules

- 优先报告会导致错误行为、安全风险或测试缺失的问题。
- 每条问题必须给出文件位置和可验证依据。
```

### 操作流程

1. 在仓库根目录创建 `AGENTS.md`，先只写最重要的命令与约束。
2. 某个子目录规则不同时，在该目录增加 `AGENTS.md` 或 `AGENTS.override.md`。
3. 重新启动 Codex；指令链在每次运行或 TUI 会话启动时重建。
4. 验证：

```bash
codex --ask-for-approval never "总结当前加载的指令"
codex --cd services/payments --ask-for-approval never "列出当前生效的指令来源"
```

5. 把重复犯错、重复评审意见持续沉淀进去。

详细规则见 [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)。

---

## Memories：跨会话本地记忆

### 目的与边界

Memories 让 Codex 从符合条件的历史对话中提取有用上下文，并在未来会话中使用。它适合“有帮助但不要求绝对可靠”的背景，不适合承载必须执行的团队规则。

- 本地 Codex Memories 与 ChatGPT Web 的 Memory 是两套不同机制。
- 本地记忆默认关闭，存放在 `~/.codex/memories/`。
- 记忆在后台生成，不保证对话结束后立刻更新。
- Codex 会对生成的记忆字段做秘密信息脱敏，但仍不要主动把密钥写进记忆；分享 `CODEX_HOME` 前应检查相关文件。
- 记忆目录是生成状态，不建议把手工编辑当成主要控制方式。

### 开启与按会话控制

桌面端：`Settings > Personalization > Enable memories`。

配置文件方式：

```toml
[features]
memories = true

[memories]
generate_memories = true
use_memories = true
disable_on_external_context = true
min_rate_limit_remaining_percent = 25
```

在 ChatGPT 桌面端或 Codex TUI 中使用 `/memories`，可以单独决定当前会话：

- 是否读取已有本地记忆；
- 是否允许当前会话成为未来记忆的生成输入。

会话级选择不会修改全局设置。

### 记忆配置属性

| 属性 | 类型/默认值 | 目的 |
|---|---|---|
| `features.memories` | boolean；默认 `false` | 打开或关闭本地 Memories 功能总开关 |
| `memories.generate_memories` | boolean；默认 `true` | 是否让新会话成为记忆生成输入 |
| `memories.use_memories` | boolean；默认 `true` | 是否把已有记忆注入未来会话 |
| `memories.disable_on_external_context` | boolean；默认 `false` | 为 `true` 时，使用过 MCP、Web Search 或 Tool Search 的会话不参与记忆生成 |
| `memories.no_memories_if_mcp_or_web_search` | 旧别名 | `disable_on_external_context` 的兼容名称，新配置优先用新名称 |
| `memories.max_raw_memories_for_consolidation` | number；默认 `256`，最大 `4096` | 全局归并时保留的近期原始记忆上限 |
| `memories.max_unused_days` | number；默认 `30`，范围 `0–365` | 多久未使用的记忆不再参与归并 |
| `memories.max_rollout_age_days` | number；默认 `30`，范围 `0–90` | 允许参与记忆生成的历史会话最大天数 |
| `memories.max_rollouts_per_startup` | number；默认 `16`，最大 `128` | 每次启动最多处理多少候选会话 |
| `memories.min_rollout_idle_hours` | number；默认 `6`，范围 `1–48` | 会话空闲多久后才可生成记忆 |
| `memories.min_rate_limit_remaining_percent` | number；默认 `25`，范围 `0–100` | 剩余额度低于该百分比时不启动记忆生成 |
| `memories.extract_model` | string；可选 | 覆盖单会话记忆提取使用的模型 |
| `memories.consolidation_model` | string；可选 | 覆盖全局记忆归并使用的模型 |

详细说明见 [Memories](https://learn.chatgpt.com/docs/customization/memories)。

---

## Skills：可复用工作流

### 目的

Skill 把一个重复工作封装成可发现、可复用的能力。一个 Skill 通常由必需的 `SKILL.md` 和可选的脚本、参考资料、资产组成：

```text
my-skill/
├── SKILL.md                 # 必需：元数据 + 操作指令
├── scripts/                 # 可选：确定性脚本或校验工具
├── references/              # 可选：按需读取的详细资料
├── assets/                  # 可选：模板、图片、资源
└── agents/
    └── openai.yaml          # 可选：UI、调用策略、工具依赖
```

Codex 使用渐进式加载：先读取所有 Skill 的 `name` 和 `description`；匹配任务后才加载完整 `SKILL.md`；引用资料和脚本只在需要时使用。因此，`description` 是否清晰直接影响自动触发。

### `SKILL.md` 必需属性

```md
---
name: release-check
description: 发布前执行构建、测试和变更日志检查。用户要求发版、发布检查或 release checklist 时使用。
---

1. 读取项目发布说明。
2. 执行测试和构建。
3. 汇总失败项，不自动发布。
```

| 属性 | 是否必需 | 目的 | 写法建议 |
|---|---:|---|---|
| `name` | 是 | Skill 的唯一名称和显式引用名 | 简短、稳定、表达任务 |
| `description` | 是 | 告诉 Codex 何时应该或不应该调用 | 前置核心用途、触发词和边界 |
| 正文 instructions | 是 | Skill 被选中后要执行的完整步骤 | 使用祈使句，明确输入、输出和验证 |

### Skill 放置位置

| 范围 | 路径 | 适用场景 |
|---|---|---|
| 当前目录/仓库 | `$CWD/.agents/skills` 及从当前目录到仓库根的各级 `.agents/skills` | 模块或仓库专用流程 |
| 用户 | `$HOME/.agents/skills` | 个人跨仓库流程 |
| 管理员 | `/etc/codex/skills` | 共享机器或容器的统一能力 |
| 系统 | Codex 内置 | OpenAI 随产品提供的通用能力 |

同名 Skill 不会自动合并，可能同时出现在选择器中。Codex 支持指向 Skill 目录的符号链接。

### 调用方式

- 显式调用：CLI/IDE 输入 `$skill-name`，或通过 `/skills` 选择；ChatGPT 中可输入 `@` 选择。
- 隐式调用：任务匹配 `description` 时，Codex 可自动选择。

### `agents/openai.yaml` 可选属性

| 属性 | 目的 |
|---|---|
| `interface.display_name` | UI 中展示的友好名称 |
| `interface.short_description` | UI 短描述 |
| `interface.icon_small` | 小图标路径 |
| `interface.icon_large` | 大图标路径 |
| `interface.brand_color` | 品牌色，如 `#3B82F6` |
| `interface.default_prompt` | 调用 Skill 时建议使用的默认提示 |
| `policy.allow_implicit_invocation` | 是否允许隐式触发；默认 `true`；设为 `false` 后仍可显式 `$skill` 调用 |
| `dependencies.tools[].type` | 依赖类型，例如 `mcp` |
| `dependencies.tools[].value` | 依赖工具或 MCP Server 的标识 |
| `dependencies.tools[].description` | 依赖用途说明 |
| `dependencies.tools[].transport` | 连接方式，例如 `streamable_http` |
| `dependencies.tools[].url` | 依赖服务地址 |

示例：

```yaml
interface:
  display_name: "Release Check"
  short_description: "发布前检查构建、测试和变更日志"
  icon_small: "./assets/icon.svg"
  brand_color: "#3B82F6"
  default_prompt: "请执行发布前检查并汇总阻塞项"

policy:
  allow_implicit_invocation: true

dependencies:
  tools:
    - type: "mcp"
      value: "projectDocs"
      description: "项目文档 MCP Server"
      transport: "streamable_http"
      url: "https://docs.example.com/mcp"
```

### 启用或停用单个 Skill

不删除 Skill 也可以在 `~/.codex/config.toml` 中禁用：

```toml
[[skills.config]]
path = "/absolute/path/to/my-skill/SKILL.md"
enabled = false
```

| 属性 | 类型 | 目的 |
|---|---|---|
| `skills.config` | object 数组 | 保存逐个 Skill 的启停覆盖项 |
| `skills.config[].path` | 路径 | 指向包含定义的 `SKILL.md` |
| `skills.config[].enabled` | boolean | 启用或禁用该 Skill |

修改后重启 Codex。详细流程见 [Build skills](https://learn.chatgpt.com/docs/build-skills)。

### 创建流程

1. 先查插件目录；已有成熟工作流时优先安装插件。
2. 没有合适插件时，在 Codex 中调用 `$skill-creator`，或手工创建 Skill 目录和 `SKILL.md`。
3. 一开始优先写纯指令；只有需要确定性行为或外部工具时才增加脚本。
4. 用明确匹配和明确不匹配的提示测试 `description`。
5. 若 Skill 未出现，重启 Codex。
6. 需要跨团队分发或同时携带连接器时，把 Skill 打包成 Plugin。

---

## Plugins：安装与分发单元

Plugin 不是另一种工作流语言。Skill 仍是工作流的创作格式，而 Plugin 是可安装、可分享的分发单元，可以同时打包：

- 一个或多个 Skills；
- 已注册的 MCP 连接或 MCP 配置；
- 展示资源等配套内容。

安装流程：

1. 在 ChatGPT Web 或桌面端打开 **Plugins**。
2. 搜索或浏览插件并打开详情。
3. 点击加号安装。
4. 如插件需要连接器，按提示登录并审查权限。
5. 新建聊天，明确要求 ChatGPT/Codex 使用该插件。

判断原则：本地编写和仓库专用工作流用 Skill；跨项目分发、组合多个 Skills 或捆绑连接器时用 Plugin。详情见 [Plugins](https://learn.chatgpt.com/docs/plugins)。

---

## MCP：连接外部工具与上下文

### 目的与结构

MCP（Model Context Protocol）把 Codex 连接到仓库以外的系统。

```text
┌──────────────┐      MCP Client       ┌──────────────────────┐
│ Codex / Host │ ────────────────────► │ MCP Server           │
└──────────────┘                       │ Tools     可执行动作  │
                                       │ Resources 可读取数据  │
                                       │ Prompts   提示模板    │
                                       └──────────────────────┘
```

常见用法：读取内部文档、查询 issue、操作 GitHub、获取 Figma 设计、控制浏览器。MCP Server 的动作权限可能很强，应使用最小权限、工具白名单和审批策略。

### 两种连接类型

| 类型 | 适合场景 | 核心配置 |
|---|---|---|
| STDIO | 本地命令启动的 Server | `command`、`args`、`env` |
| Streamable HTTP | 远程地址提供的 Server | `url`、OAuth/Bearer、Headers |

配置默认写在 `~/.codex/config.toml`；可信项目也可以写 `.codex/config.toml`。桌面端、CLI 和 IDE 扩展共享同一个 Codex Host 上的 MCP 配置。

### STDIO 属性

每个 Server 使用 `[mcp_servers.<server-name>]` 表：

| 属性 | 必需 | 目的 |
|---|---:|---|
| `command` | 是 | 启动 MCP Server 的命令 |
| `args` | 否 | 传给命令的参数数组 |
| `env` | 否 | 明确设置给 Server 的环境变量映射 |
| `env_vars` | 否 | 允许从本地或远程执行环境转发的变量名 |
| `cwd` | 否 | 启动 Server 时的工作目录 |
| `experimental_environment` | 否 | 设为 `remote` 时，在可用的远程执行环境启动 STDIO Server |

```toml
[mcp_servers.context7]
command = "npx"
args = ["-y", "@upstash/context7-mcp"]
env_vars = ["LOCAL_TOKEN"]

[mcp_servers.context7.env]
MY_ENV_VAR = "MY_ENV_VALUE"
```

`env_vars` 也可以声明来源：

```toml
env_vars = ["LOCAL_TOKEN", { name = "REMOTE_TOKEN", source = "remote" }]
```

字符串和 `source = "local"` 从本地环境读取；`source = "remote"` 需要远程 MCP STDIO 支持。

### Streamable HTTP 属性

| 属性 | 必需 | 目的 |
|---|---:|---|
| `url` | 是 | MCP Server 地址 |
| `auth` | 否；默认 `oauth` | 认证方式；可信第一方来源可用 `chatgpt`，并以已存 OAuth 为回退 |
| `bearer_token_env_var` | 否 | 保存 Bearer Token 的环境变量名；避免把 Token 直接写进 TOML |
| `http_headers` | 否 | 静态 HTTP Header 映射；不要在仓库配置中放秘密 |
| `env_http_headers` | 否 | Header 名到环境变量名的映射 |

```toml
[mcp_servers.design]
url = "https://mcp.example.com/mcp"
bearer_token_env_var = "DESIGN_MCP_TOKEN"
http_headers = { "X-Region" = "cn-east" }
```

### 所有 Server 通用属性

| 属性 | 默认值 | 目的 |
|---|---|---|
| `startup_timeout_sec` | `10` | Server 启动超时秒数 |
| `tool_timeout_sec` | `60` | 单次工具执行超时秒数 |
| `enabled` | 启用 | 不删除配置的情况下停用 Server |
| `required` | `false` | 为 `true` 时，启用的 Server 初始化失败会导致启动失败 |
| `enabled_tools` | 未限制 | 工具允许列表 |
| `disabled_tools` | 空 | 工具拒绝列表；在 `enabled_tools` 之后应用 |
| `default_tools_approval_mode` | 可选 | Server 内工具的默认审批模式：`auto`、`prompt`、`writes`、`approve`；`writes` 会对非只读工具询问 |
| `tools.<tool>.approval_mode` | 可选 | 覆盖某一个工具的审批模式 |
| `mcp_oauth_callback_port` | 临时端口 | 顶层配置；OAuth 必须固定回调端口时使用 |
| `mcp_oauth_callback_url` | 本地回调 | 顶层配置；远程 Devbox 或自定义回调地址时使用 |

建议从工具白名单和 `prompt`/`writes` 开始，确认工具行为后再放宽。

### 配置流程

CLI 快速添加 STDIO Server：

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
codex mcp list
codex mcp --help
```

OAuth Server：

```bash
codex mcp login <server-name>
```

桌面端：

1. `Settings > MCP servers > Add server`。
2. 填写名称，选择 STDIO 或 Streamable HTTP。
3. 填写命令或 URL，保存并重启。
4. 需要 OAuth 时点击 **Authenticate**。
5. 在输入框使用 `/mcp` 检查连接。

IDE 扩展流程相同，保存后选择 **Restart extension**。ChatGPT Web 不读取本地 `config.toml`，需通过 Plugin 使用远程 MCP 工具。

详细说明见 [Model Context Protocol](https://learn.chatgpt.com/docs/extend/mcp)。

---

## Subagents：专业分工与并行执行

### 目的

子代理把探索日志、测试输出、堆栈信息等噪声留在独立线程，只把结论汇总回主线程。适合：

- 并行代码探索；
- 安全、质量、测试分别审查；
- 大文档分片归纳；
- 使用不同工具或 MCP 的专业角色。

代价是每个子代理都会独立消耗模型与工具资源，因此通常比单代理使用更多 Token。写密集型并行任务还可能产生文件冲突。

### 内置角色与触发方式

Codex 内置：

| Agent | 用途 |
|---|---|
| `default` | 通用回退角色 |
| `worker` | 实现和修复 |
| `explorer` | 以读取、搜索和代码探索为主 |

明确告诉 Codex 如何拆分、是否等待全部结果、最终返回什么：

```text
并行审查这个分支：启动一个子代理检查安全风险，一个检查测试缺口，
一个检查可维护性。等待全部完成后，按类别汇总，并附文件位置。
```

### 全局属性

写入 `~/.codex/config.toml` 或可信项目的 `.codex/config.toml`：

```toml
[agents]
enabled = true
max_concurrent_threads_per_session = 4
default_subagent_reasoning_effort = "medium"
interrupt_message = true
```

| 属性 | 类型/默认值 | 目的 |
|---|---|---|
| `agents.enabled` | boolean；默认 `true` | 启用或禁用多代理工具 |
| `agents.max_concurrent_threads_per_session` | number；未设置时由 Codex 决定 | 限制同时打开的子代理线程数，不含主线程 |
| `agents.max_threads` | number；旧别名 | 兼容旧配置，新配置使用上面的完整名称 |
| `agents.default_subagent_model` | string；可选 | 子代理默认模型；显式 spawn 值优先 |
| `agents.default_subagent_reasoning_effort` | string；可选 | 子代理默认推理强度；显式 spawn 值优先 |
| `agents.interrupt_message` | boolean；默认 `true` | 中断代理时，是否在其上下文中记录模型可见的中断消息 |

子代理默认继承父代理的模型、推理强度、沙箱和审批策略。显式 spawn 参数、`[agents]` 默认值和自定义 Agent 文件可以进一步覆盖；权限模式应在委派前确认。

### 自定义 Agent 文件

位置：

```text
~/.codex/agents/*.toml       # 个人自定义 Agent
.codex/agents/*.toml         # 项目自定义 Agent
```

每个文件定义一个 Agent，必需属性如下：

| 属性 | 必需 | 目的 |
|---|---:|---|
| `name` | 是 | Codex 识别和引用 Agent 的真实名称；文件名只是惯例 |
| `description` | 是 | 告诉 Codex 什么时候应该使用它 |
| `developer_instructions` | 是 | 定义角色行为、边界和输出要求 |

还可使用普通 `config.toml` 属性，例如：

| 可选属性 | 目的 |
|---|---|
| `model` | 为该角色指定模型 |
| `model_reasoning_effort` | 为该角色指定推理强度 |
| `sandbox_mode` | 限制该角色的文件和命令权限，例如只读 |
| `mcp_servers` | 只给该角色配置需要的外部工具 |
| `skills.config` | 控制该角色可使用的 Skills |

示例 `.codex/agents/security-reviewer.toml`：

```toml
name = "security_reviewer"
description = "只读安全审查员；在用户要求安全检查、威胁分析或权限审查时使用。"
sandbox_mode = "read-only"
model_reasoning_effort = "high"
developer_instructions = """
只进行安全审查，不修改文件。
优先检查认证、授权、秘密信息、输入校验和依赖风险。
每条发现必须给出文件位置、风险说明和验证依据。
"""
```

如果自定义名称与 `explorer` 等内置 Agent 相同，自定义版本优先。详细说明见 [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)。

### 操作流程

1. 先确认任务能否拆成互不依赖的子任务。
2. 简单分工直接写进提示词；反复使用的专业角色再创建自定义 Agent。
3. 给读取型 Agent 设置 `sandbox_mode = "read-only"`。
4. 限制并发数，避免资源浪费和写冲突。
5. 要求主代理等待所有结果，并只汇总证据和结论。
6. 若多个 Agent 必须写文件，让它们负责互不重叠的文件或目录。

---

## Rules：控制哪些命令能在沙箱外运行

Rules 是实验性能力，用来给命令前缀设置沙箱外执行决策。它与 `approval_policy`、`sandbox_mode` 配合，而不是替代它们。

文件位置：

```text
~/.codex/rules/*.rules          # 用户级
<repo>/.codex/rules/*.rules     # 项目级；仅可信项目加载
```

`prefix_rule()` 属性：

| 属性 | 必需/默认 | 目的 |
|---|---|---|
| `pattern` | 必需 | 非空命令参数前缀；元素可为字面量或同位置候选值集合 |
| `decision` | 默认 `allow` | `allow` 直接允许、`prompt` 每次询问、`forbidden` 直接阻止 |
| `justification` | 可选 | 在审批或拒绝信息中展示规则原因；禁止时建议写替代方案 |
| `match` | 默认 `[]` | 应当命中的内联测试样例 |
| `not_match` | 默认 `[]` | 不应命中的内联测试样例 |

多条规则命中时采用最严格结果：`forbidden > prompt > allow`。

```python
# ~/.codex/rules/default.rules
prefix_rule(
    pattern = ["gh", "pr", ["view", "list"]],
    decision = "prompt",
    justification = "读取远程 PR 前请求确认",
    match = ["gh pr view 123", "gh pr list"],
    not_match = ["gh issue list"],
)
```

操作流程：

1. 在激活配置层旁创建 `rules/` 与 `.rules` 文件。
2. 写 `pattern`、决策和正反样例。
3. 先测试规则：

```bash
codex execpolicy check --pretty \
  --rules ~/.codex/rules/default.rules \
  -- gh pr view 123
```

4. 重启 Codex。
5. 对复合 Shell 命令保持谨慎：Codex 只会在能安全解析简单命令链时逐段检查；包含重定向、变量、通配符或控制流时，会把整段脚本作为单个调用保守匹配。

详见 [Rules](https://learn.chatgpt.com/docs/agent-configuration/rules)。

## Speed：速度与成本

Fast mode 是同一受支持模型的加速服务层，不是另一个模型；速度更快，但会消耗更多 Credits。Codex-Spark 则是单独的快速模型选择，两者不要混淆。

```toml
service_tier = "fast"

[features]
fast_mode = true
```

会话内操作：

```text
/fast on
/fast off
/fast status
```

具体支持模型、倍率、资格与价格容易变化，请始终查看 [Speed](https://learn.chatgpt.com/docs/agent-configuration/speed) 和 [Pricing](https://learn.chatgpt.com/docs/pricing) 当前说明。

## Record & Replay：把演示变成 Skill

Record & Replay 当前用于 macOS，且需要 Computer Use 可用并启用。适合“流程固定、偏好明确、演示比文字描述容易”的操作。

```text
你演示一次完整流程
        │
        ▼
Codex 观察必要的窗口与动作
        │
        ▼
生成 Skill：触发条件 + 输入 + 步骤 + 验证
        │
        ▼
新会话提供本次变量，复用该 Skill
```

操作流程：

1. 桌面端选择 ChatGPT Work 或 Codex，打开 **Plugins**。
2. 点击 **+ > Record a skill**。
3. 补充目标与可变输入，提交并批准录制。
4. 在 Mac 上完整演示一次，完成后停止录制。
5. 审查生成的 Skill，补充隐藏偏好、决策点和成功标准。
6. 新建聊天，调用该 Skill 并提供本次不同的输入。

录制时避免秘密和敏感数据。需要跨团队稳定分发、捆绑多个 Skills 或连接器时，再打包为 Plugin。详见 [Record & Replay](https://learn.chatgpt.com/docs/extend/record-and-replay)。

## Linux 与 Windows

### Linux 桌面端

Linux 桌面端有独立的安装与更新流程，入口见 [ChatGPT desktop app for Linux](https://learn.chatgpt.com/docs/linux/linux-app)。`config.toml`、Skills、MCP 和 Agent 的核心概念与其他本地 Codex Host 一致。

### Windows 原生沙箱

```toml
[windows]
sandbox = "elevated"       # 官方推荐；可用管理员权限时
# sandbox = "unelevated"   # 无管理员权限或 elevated 初始化失败时回退
```

原生 Windows 运行时应先尝试 `elevated`；私有桌面等高级设置见 [Windows sandbox](https://learn.chatgpt.com/docs/windows/windows-sandbox)。桌面应用安装见 [ChatGPT desktop app for Windows](https://learn.chatgpt.com/docs/windows/windows-app)。

### WSL2

WSL2 中 Codex 运行在 Linux 环境，使用 Linux 沙箱而不是 Windows 原生沙箱。适合依赖 Linux 工具链或代码本来就位于 WSL 的项目。WSL1 从 Codex 0.115 起不再受支持。

```powershell
wsl --install
wsl
```

进入 WSL 后：

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex
```

仓库尽量放在 `~/code/...`，避免 `/mnt/c/...` 的 I/O、符号链接和权限问题。详见 [WSL](https://learn.chatgpt.com/docs/windows/wsl)。

## 组合使用：Skill + MCP + Subagents

单独使用已经有价值，组合后更适合真实团队流程：

```text
用户："审查这个 PR，并核对 Linear 需求与 Figma 设计"
                          │
                          ▼
       AGENTS.md：项目规范、测试命令、评审标准
                          │
                          ▼
       Skill：定义 PR 审查的固定步骤和输出格式
                          │
                ┌─────────┴─────────┐
                ▼                   ▼
      MCP：Linear / Figma      Subagents：并行分工
      获取外部需求和设计       ├─ 代码风险
                               ├─ 测试缺口
                               └─ 文档/API 核对
                └─────────┬─────────┘
                          ▼
                 主代理汇总最终报告
```

如果 Skill 依赖 MCP，应在 `agents/openai.yaml` 的 `dependencies.tools` 中声明依赖。这样 Skill 负责流程，MCP 负责能力边界，子代理负责并行执行。

## 常见场景速查

| 需求 | 推荐配置 |
|---|---|
| 每次修改 JS 后必须跑测试 | `AGENTS.md` + CI |
| 记住个人偏好和近期工作背景 | Memories |
| 每次发版都执行同一套检查 | Skill |
| 工作流需要读取 GitHub/Figma/Linear | MCP 或包含连接器的 Plugin |
| 安全、测试、性能同时审查 | Subagents |
| 团队共享“发版流程 + GitHub 工具” | Skill + MCP，打包成 Plugin |
| 某子目录必须使用不同测试命令 | 嵌套 `AGENTS.override.md` |
| 临时停用一个 Skill/MCP Server | `skills.config[].enabled = false` / MCP 的 `enabled = false` |

## 安全与维护清单

- [ ] 必须遵守的规则已经写入 `AGENTS.md`，而非只存在于 Memories。
- [ ] `AGENTS.md` 保持精简，规则离适用目录尽量近。
- [ ] 测试、lint、类型检查、hooks 能自动执行关键规则。
- [ ] Skill 的 `description` 写清“何时使用”和边界。
- [ ] Skill 脚本只用于需要确定性的步骤，并能独立验证。
- [ ] MCP Token 使用环境变量，不直接写入仓库 TOML。
- [ ] MCP 使用最小工具白名单和合适的审批模式。
- [ ] 外部上下文不应进入记忆时，开启 `disable_on_external_context`。
- [ ] 共享 `CODEX_HOME` 前检查 Memories、认证与会话状态。
- [ ] 子代理并发数受控，写入范围互不重叠。
- [ ] 配置修改后已重启 Codex，并实际验证加载结果。

## 常见问题

### `AGENTS.md` 和 Skill 有什么区别？

`AGENTS.md` 是进入项目就应遵守的持续规则；Skill 是完成某类任务时才加载的完整工作流。

### Skill 和 Plugin 有什么区别？

Skill 是工作流的创作格式，适合本地或仓库内迭代；Plugin 是安装和分发单位，可同时携带多个 Skills、MCP 连接及资源。

### Skill 和 MCP 为什么经常一起用？

Skill 告诉 Codex“按什么步骤完成任务”，MCP 提供“访问哪个外部系统、能调用哪些动作”。

### Memories 能替代 `AGENTS.md` 吗？

不能。Memories 是有帮助的回忆层，生成和使用都可能受设置、空闲时间、额度及会话条件影响；强制规则必须放在 `AGENTS.md` 或工程检查中。

### 为什么修改配置后没有生效？

先确认编辑的是当前 `CODEX_HOME` 下的配置，再重启 Codex 或新开 TUI 会话。项目级 `.codex/config.toml` 只在项目被信任后加载。

### 子代理是不是越多越快？

不是。只有能独立并行的任务才容易提速；依赖关系强或同时写相同文件时，协调和冲突成本可能更高，而且会消耗更多 Token。

## 官方资料

- [Configuration 总览](https://learn.chatgpt.com/docs/configuration)
- [Config basics](https://learn.chatgpt.com/docs/config-file/config-basic)
- [Advanced Configuration](https://learn.chatgpt.com/docs/config-file/config-advanced)
- [Configuration Reference](https://learn.chatgpt.com/docs/config-file/config-reference)
- [Environment variables](https://learn.chatgpt.com/docs/config-file/environment-variables)
- [Customization 总览](https://learn.chatgpt.com/docs/customization/overview)
- [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)
- [Memories](https://learn.chatgpt.com/docs/customization/memories)
- [Build skills](https://learn.chatgpt.com/docs/build-skills)
- [Plugins](https://learn.chatgpt.com/docs/plugins)
- [Model Context Protocol](https://learn.chatgpt.com/docs/extend/mcp)
- [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents)
- [Rules](https://learn.chatgpt.com/docs/agent-configuration/rules)
- [Speed](https://learn.chatgpt.com/docs/agent-configuration/speed)
- [Record & Replay](https://learn.chatgpt.com/docs/extend/record-and-replay)
- [Linux desktop app](https://learn.chatgpt.com/docs/linux/linux-app)
- [Windows sandbox](https://learn.chatgpt.com/docs/windows/windows-sandbox)
- [WSL](https://learn.chatgpt.com/docs/windows/wsl)

> 文档和属性会随 Codex 更新。复制配置前，请以对应官方链接中的当前版本为准。
