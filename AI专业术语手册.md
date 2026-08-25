# AI 专业术语手册

**模型 · Agent · MCP · Skill · RAG · Harness · Loop · Coding · Eval · Governance · AGI**

*2026 实用版｜面向 AI 技术交流、Agent 工程、AI 编程与项目评审*

> **使用方法：先记“术语属于哪一层”，再记“它和相邻概念有什么区别”。专业交流的关键不是堆缩写，而是能准确说出边界、依赖关系、工程取舍和验证方法。**

**核心心智模型：** Model（模型能力） → Context（给什么信息） → Tool/MCP（能做什么动作） → Skill（会怎么做） → Harness（如何可靠运行） → Loop（如何持续闭环） → Eval/Governance（如何验证与治理）。

## 一页先懂：最重要的 16 组关系

| 概念关系 | 一句话判断 |
| --- | --- |
| LLM vs Agent | LLM 是“脑”；Agent 是“脑 + 工具 + 状态 + 循环 + 规则”。 |
| Prompt vs Context | Prompt 是指令；Context 是模型本次能看到的全部信息。 |
| Context vs Memory | Context 是当前输入；Memory 是可持久保存、以后再注入 Context 的信息。 |
| Tool vs Skill | Tool = 可执行接口；Skill = 完成某类任务的方法包/知识包。 |
| Plugin vs Tool | Plugin = 宿主扩展方式；一个插件可能提供多个 Tools。 |
| MCP vs API | API = 软件接口；MCP = AI 客户端连接外部能力的标准协议层。 |
| Function Calling vs Tool Execution | 模型生成调用意图和参数；宿主程序才真正执行。 |
| RAG vs Fine-tuning | RAG 运行时取知识；Fine-tuning 改模型权重/行为。 |
| RAG vs Memory | RAG 解决知识召回；Memory 解决跨步骤/跨会话状态和经验。 |
| Workflow vs Agent | Workflow 偏固定流程；Agent 偏动态决策。 |
| Harness vs Loop | Harness 让一次 Agent 运行可靠；Loop 让多次 Agent 运行自动闭环。 |
| Reviewer vs Verifier | Reviewer 可做主观/综合审查；Verifier 更强调客观可执行验证。 |
| Eval vs Test | Test 往往是确定性工程检查；Eval 更广，包含模型评分、质量、安全、成本。 |
| KV Cache vs Prompt Cache | KV Cache 是模型内部 attention 缓存；Prompt Cache 是重复输入前缀的复用。 |
| Vibe Coding vs Engineering | Vibe Coding 偏自然语言驱动快速产出；工程化要求规范、测试、审查、可维护和治理。 |
| Autonomy vs Governance | 自治越强，越需要权限边界、预算、审计、停止条件和人工升级。 |

## 1. 基础模型与核心概念 Foundation Models & Core Concepts

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| AI — Artificial Intelligence | 人工智能 | 让机器完成通常需要人类智能的感知、推理、生成、决策等任务的总称。 | “这是 AI 系统层问题，不只是模型层问题。” | ★ |
| AGI — Artificial General Intelligence | 通用人工智能 | 能跨领域学习、推理和完成广泛任务的通用智能目标；目前没有统一行业判定标准。 | “AGI 更像能力边界与自主学习能力的组合，而不是单一 benchmark。” | ★ |
| ASI — Artificial Superintelligence | 超级人工智能 | 假设中在多数认知任务上显著超过人类的智能形态。 | 交流时宜标明这是前瞻概念，而非已实现产品。 | ★ |
| LLM — Large Language Model | 大语言模型 | 以大规模文本/多模态数据训练、能够理解与生成语言的模型。 | “LLM 是 Agent 的推理核心，但 Agent 不等于 LLM。” | ★ |
| Foundation Model | 基础模型 | 在广泛数据上预训练，可适配多个下游任务的大模型。 |  |  |
| Multimodal Model | 多模态模型 | 同时处理文字、图像、音频、视频等多种模态的模型。 |  |  |
| VLM — Vision-Language Model | 视觉语言模型 | 联合理解图像与语言的模型，常用于看图、OCR、GUI 理解、视觉问答。 |  |  |
| Reasoning Model | 推理模型 | 针对复杂推理、规划、数学、代码等任务优化的模型类型。 |  |  |
| Model Family | 模型家族 | 同一技术路线或品牌下的多个模型版本/尺寸/能力档。 |  |  |
| Checkpoint | 模型检查点 | 训练过程中特定时刻保存的模型参数状态。 |  |  |
| Weights | 模型权重/参数 | 神经网络中通过训练得到的数值参数。 |  |  |
| Parameters | 参数量 | 模型中可训练参数的数量；参数更多不自动等于更聪明。 |  |  |
| Active Parameters | 激活参数 | MoE 模型一次前向计算实际参与计算的参数子集。 | “总参数决定容量上限的一部分，激活参数更直接影响单次计算成本。” | ★ |
| Dense Model | 稠密模型 | 每次前向基本使用全部主要参数的模型。 |  |  |
| MoE — Mixture of Experts | 混合专家模型 | 通过路由器选择部分专家网络参与计算，以更低激活成本扩展总容量。 |  |  |
| Expert Routing | 专家路由 | MoE 中选择哪些专家处理当前 token 的机制。 |  |  |
| Inference | 推理/推断 | 模型训练完成后，接收输入并生成输出的计算过程。 |  |  |
| Training | 训练 | 通过数据和优化算法更新模型参数。 |  |  |
| Pre-training | 预训练 | 在大规模通用数据上学习语言/世界规律的基础训练阶段。 |  |  |
| Post-training | 后训练 | 预训练之后进行指令遵循、对齐、推理、偏好等能力优化的阶段。 |  |  |
| Fine-tuning | 微调 | 在特定数据/任务上继续训练，使模型适配特定目标。 |  |  |
| SFT — Supervised Fine-Tuning | 监督微调 | 使用输入-理想输出样本进行监督训练。 |  |  |
| RLHF — Reinforcement Learning from Human Feedback | 基于人类反馈的强化学习 | 利用人类偏好信号优化模型行为。 |  |  |
| RLAIF — Reinforcement Learning from AI Feedback | 基于 AI 反馈的强化学习 | 用 AI 评估/偏好替代或补充人工反馈。 |  |  |
| DPO — Direct Preference Optimization | 直接偏好优化 | 直接用偏好对数据优化模型，而不显式训练传统奖励模型的一类方法。 |  |  |
| Distillation | 知识蒸馏 | 用强模型产生的知识/分布训练较小模型，以降低成本或部署难度。 |  |  |
| Quantization | 量化 | 降低权重/激活数值精度（如 FP16→INT8/INT4）以减少显存和计算。 |  |  |
| Context Window | 上下文窗口 | 单次模型调用能够处理的输入与输出 token 总容量范围。 | “上下文窗口是容量上限，不等于模型永久记忆。” | ★ |
| Token | 词元 | 模型处理文本的基本离散单位，不等同于中文字符或英文单词。 |  |  |
| Tokenizer | 分词器 | 把文本编码为 token ID、再解码回文本的组件。 |  |  |
| Embedding | 向量嵌入 | 把文本、图像等映射成高维向量，用于语义检索、聚类、相似度等。 |  |  |
| Latent Space | 潜空间 | 模型内部以向量形式表示信息和特征的抽象空间。 |  |  |
| Logits | 未归一化输出分数 | 模型对候选 token 给出的原始分数，随后通常经 softmax 形成概率。 |  |  |
| Temperature | 温度参数 | 控制采样随机性；通常越高越发散，越低越稳定。 |  |  |
| Top-p / Nucleus Sampling | 核采样 | 只从累计概率达到阈值 p 的候选 token 集合中采样。 |  |  |
| Determinism | 确定性 | 相同输入在相同设置下得到一致输出的程度；生成式模型通常并非绝对确定。 |  |  |
| Latency | 延迟 | 请求到首 token 或完整响应所需时间。 |  |  |
| TTFT — Time to First Token | 首 Token 延迟 | 从请求发出到返回第一个 token 的时间。 |  |  |
| Throughput | 吞吐量 | 单位时间内可处理的 token 或请求量。 |  |  |
| TPS — Tokens Per Second | 每秒 Token 数 | 常用于衡量生成速度。 |  |  |
| Hallucination | 幻觉 | 模型生成看似可信但缺乏事实依据或与给定信息冲突的内容。 |  |  |
| Grounding | 事实落地/依据约束 | 让模型输出绑定可靠数据源、工具结果或可验证证据。 |  |  |
| Alignment | 对齐 | 让模型行为符合人类意图、安全要求和任务规范。 |  |  |
| Capability Elicitation | 能力激发 | 通过合适提示、工具、上下文或推理预算把模型已有能力发挥出来。 |  |  |

## 2. Prompt、Context 与 Memory 提示词/上下文/记忆

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Prompt | 提示词 | 发送给模型的自然语言或结构化指令。 |  |  |
| System Prompt | 系统提示词 | 最高优先级的行为、角色、规则与安全约束之一。 |  |  |
| Developer Prompt | 开发者指令 | 由应用开发者提供的产品/工作流级行为约束。 |  |  |
| User Prompt | 用户提示词 | 用户当前任务请求。 |  |  |
| Instruction Hierarchy | 指令层级 | 不同来源指令的优先级与冲突解决机制。 |  |  |
| Prompt Engineering | 提示词工程 | 通过设计指令、示例、格式和约束提高单次/少数轮输出质量。 |  |  |
| Context | 上下文 | 当前调用中模型可见的指令、历史、文件、工具定义、检索结果等信息总和。 |  |  |
| Context Engineering | 上下文工程 | 系统性选择、组织、压缩、注入模型真正需要的信息。 | “Prompt Engineering 关注怎么问；Context Engineering 关注给模型看什么。” | ★ |
| Context Management | 上下文管理 | 控制历史保留、摘要、检索、裁剪、优先级与 token 预算。 |  |  |
| Context Packing | 上下文装配 | 把任务相关文档、规则、示例、工具说明组织进有限上下文。 |  |  |
| Context Compression | 上下文压缩 | 对历史/工具输出进行摘要、去重或结构化以降低 token 消耗。 |  |  |
| Context Rot | 上下文腐化/上下文污染 | 长上下文中无关、过时或冲突信息累积，导致模型判断变差的现象。 |  |  |
| Lost in the Middle | 中间信息遗失效应 | 模型在长上下文中对中部信息利用率可能低于首尾信息。 |  |  |
| Prompt Injection | 提示词注入 | 不可信内容试图诱导模型忽略原规则或泄露/执行不应执行的行为。 |  |  |
| Jailbreak | 越狱提示 | 试图绕过模型安全或行为限制的提示策略。 |  |  |
| Few-shot Prompting | 少样本提示 | 在提示中给少量示例，让模型模仿任务格式/规律。 |  |  |
| Zero-shot Prompting | 零样本提示 | 不提供示例，仅凭任务描述完成任务。 |  |  |
| In-context Learning | 上下文学习 | 模型利用当前上下文中的示例/规律临时适配，而不更新模型权重。 |  |  |
| CoT — Chain of Thought | 思维链 | 将复杂问题拆成多步推理过程的能力/方法。 | 专业交流中不要把“看不到内部推理过程”误解为“模型没有推理”。 | ★ |
| Scratchpad | 草稿区/推理草稿 | 供模型或 Agent 临时记录中间状态、计划和计算的工作区概念。 |  |  |
| Memory | 记忆 | 跨步骤或跨会话保存并可再次调用的信息机制；通常由外部系统实现。 |  |  |
| Short-term Memory | 短期记忆 | 当前任务/会话内保留的状态和近期信息。 |  |  |
| Long-term Memory | 长期记忆 | 跨会话持久化的用户偏好、经验、事实或 Agent 状态。 |  |  |
| Episodic Memory | 情景记忆 | 保存过去任务/事件经历，用于以后类比或复用。 |  |  |
| Semantic Memory | 语义记忆 | 保存较稳定事实、概念和知识。 |  |  |
| Working Memory | 工作记忆 | 当前推理和执行所需的临时任务状态。 |  |  |
| KV Cache — Key-Value Cache | 键值缓存 | Transformer 推理时缓存已计算 attention 的 Key/Value，减少重复计算。 | KV Cache 是推理加速缓存，不等于产品层长期记忆，也不等于 Prompt Cache。 | ★ |
| Prompt Cache / Prefix Cache | 提示前缀缓存 | 对重复输入前缀的计算结果复用，以降低延迟/成本。 |  |  |
| Cache Hit | 缓存命中 | 请求中可复用既有缓存的部分被成功命中。 |  |  |
| Cache Miss | 缓存未命中 | 无法复用缓存，需要重新计算。 |  |  |

## 3. Agent、规划、执行、审核、决策与自治

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Agent / AI Agent | 智能体/代理 | 以模型为核心，能观察环境、规划、调用工具、执行动作并根据结果迭代的系统。 | “Agent = Model + Context + Tools + State + Loop + Guardrails。” | ★ |
| Agentic AI | 智能体式 AI | 强调自主执行、工具使用、多步迭代和目标驱动的 AI 系统范式。 |  |  |
| Agentic Workflow | 智能体工作流 | 含一个或多个 Agent 的任务流，可能部分固定、部分由模型动态决策。 |  |  |
| Workflow | 工作流 | 按预定义步骤流转的过程；可以完全没有 Agent。 | Agent 强调动态决策；Workflow 强调流程编排。 | ★ |
| Planner | 规划器/规划 Agent | 把目标拆成步骤、依赖、里程碑和完成条件。 |  |  |
| Executor | 执行器/执行 Agent | 根据计划执行代码、工具调用、文件修改、检索等动作。 |  |  |
| Reviewer / Critic | 审核器/批评器 | 独立检查输出是否符合需求、规范与质量门槛。 |  |  |
| Evaluator | 评估器 | 通过规则、测试、模型评分或人工评价判断结果质量。 |  |  |
| Verifier | 验证器 | 验证结果是否满足可客观检查的条件，如测试通过、文件存在、接口返回正确。 |  |  |
| Decision Agent | 决策 Agent | 根据证据、约束和目标在多个选项中选择下一步行动。 |  |  |
| Supervisor Agent | 监督 Agent | 负责分配任务、检查子 Agent 结果、控制终止或升级。 |  |  |
| Orchestrator | 编排器 | 调度多个 Agent、工具、队列、状态与依赖的系统组件。 |  |  |
| Subagent | 子代理/子智能体 | 由主 Agent 派生或调用、处理特定子任务的 Agent。 |  |  |
| Multi-Agent System | 多智能体系统 | 多个 Agent 分工、协作、竞争或互相验证的系统。 |  |  |
| Agent Team | Agent 团队 | 按角色划分的多 Agent 协作方式，如 Planner/Coder/Reviewer/Tester。 |  |  |
| Delegation | 任务委派 | 主 Agent 将子任务交给子 Agent 或工具执行。 |  |  |
| Handoff | 交接 | 一个 Agent 将任务、上下文、状态移交给另一个 Agent。 |  |  |
| Routing | 路由 | 根据任务类型选择合适模型、Agent、工具或技能。 |  |  |
| Model Routing | 模型路由 | 高难任务用强模型，简单任务用低成本模型的动态选择机制。 |  |  |
| Tool Routing | 工具路由 | 根据意图与权限选择应调用的工具。 |  |  |
| Plan-and-Execute | 规划-执行模式 | 先生成计划，再按步骤执行；适合复杂长任务。 |  |  |
| ReAct — Reason + Act | 推理-行动范式 | 模型交替进行推理、工具行动、观察结果，再继续下一步。 |  |  |
| Observe-Think-Act Loop | 观察-思考-行动循环 | Agent 持续读取环境状态、决策、行动、再观察的基本循环。 |  |  |
| Reflection | 反思 | Agent 对已有输出/失败进行复盘并调整策略。 |  |  |
| Self-Correction | 自我修正 | 根据错误、测试或反馈自动修复自身输出。 |  |  |
| Self-Verification | 自验证 | 模型/Agent 对自己的输出进行检查；最好结合外部可执行验证。 |  |  |
| Generator-Evaluator Pattern | 生成器-评估器模式 | 一个组件生成候选，另一个独立评分/选择/退回修改。 |  |  |
| Human-in-the-Loop — HITL | 人在回路 | 关键步骤由人审批、纠错、补充判断。 |  |  |
| Human-on-the-Loop | 人在环上监督 | 系统大多自治运行，人主要监控并在异常时介入。 |  |  |
| Human-out-of-the-Loop | 无人参与闭环 | 系统无需人工审批即可持续执行，高风险场景通常需要更强控制。 |  |  |
| Bounded Autonomy | 有界自治 | 通过权限、成本、次数、时间、范围、停止条件约束 Agent 自治边界。 |  |  |
| Escalation | 升级/人工介入 | 当置信度低、风险高或多次失败时把任务交还人类。 |  |  |
| Stop Condition | 停止条件 | 明确 Agent 何时判定完成、失败或需要升级。 |  |  |
| Retry Policy | 重试策略 | 失败后重试次数、退避时间、是否换模型/换方法的规则。 |  |  |
| Budget / Token Budget | 预算/Token 预算 | 限制一次任务可使用的 token、金额、调用次数或时间。 |  |  |
| State | 状态 | Agent 当前任务进度、变量、计划、历史动作与外部环境的结构化表示。 |  |  |
| State Machine | 状态机 | 以状态与转移规则显式管理 Agent 工作流。 |  |  |
| Idempotency | 幂等性 | 相同动作重复执行不会造成额外副作用，Agent 自动化中非常重要。 |  |  |

## 4. Tool、Function Calling、MCP、Skill、Plugin 与常用集成

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Tool | 工具 | 模型/Agent 可调用的外部能力，如搜索、Shell、文件、数据库、浏览器。 |  |  |
| Tool Use | 工具使用 | 模型根据任务选择并调用外部工具完成动作或获取实时数据。 |  |  |
| Tool Call | 工具调用 | 一次具体的工具请求，通常包含工具名与结构化参数。 |  |  |
| Function Calling | 函数调用 | 模型输出结构化函数名和参数，由宿主程序真正执行函数/接口。 | Function Calling 不是“模型直接运行 API”；模型负责选择/填参，宿主负责执行。 | ★ |
| Structured Output | 结构化输出 | 约束模型输出 JSON/Schema 等可机器解析格式。 |  |  |
| JSON Schema | JSON 模式 | 描述结构化参数字段、类型、必填项等的规范，常用于 Tool/Function Calling。 |  |  |
| MCP — Model Context Protocol | 模型上下文协议 | 连接 AI 应用与外部数据、工具、工作流的开放标准。 | 可以理解为 AI 工具生态的“统一接口层”，但不是所有 Tool 都必须通过 MCP。 | ★ |
| MCP Host | MCP 宿主 | 承载 AI 应用并发起 MCP 连接的应用，如 AI IDE/助手。 |  |  |
| MCP Client | MCP 客户端 | 宿主内部连接 MCP Server、处理协议通信的组件。 |  |  |
| MCP Server | MCP 服务器 | 通过 MCP 暴露 Tools、Resources、Prompts 等能力的服务。 |  |  |
| MCP Tool | MCP 工具 | MCP Server 向模型暴露的可执行能力。 |  |  |
| MCP Resource | MCP 资源 | MCP 提供的可读取上下文数据，如文件、数据库内容。 |  |  |
| MCP Prompt | MCP 提示模板 | MCP Server 暴露的预定义交互模板。 |  |  |
| Skill / Agent Skill | 技能 | 把特定领域的指令、脚本、资源打包成可发现、可加载的专业能力模块。 | Skill 通常更像“方法与知识包”；Tool 更像“可执行能力”。 | ★ |
| Plugin | 插件 | 扩展宿主应用能力的安装式集成概念；不同产品对 Plugin 的定义并不统一。 | 不要把所有 MCP Server、Tool、Skill 都笼统叫插件。 | ★ |
| Connector | 连接器 | 连接外部 SaaS、数据源或企业系统的集成组件。 |  |  |
| API — Application Programming Interface | 应用程序接口 | 软件之间交换数据和调用能力的接口。 |  |  |
| SDK — Software Development Kit | 软件开发工具包 | 封装 API、类型、鉴权和常用功能的开发库。 |  |  |
| CLI — Command-Line Interface | 命令行接口 | 通过终端命令操作软件或平台。 |  |  |
| Shell Tool | Shell 工具 | 允许 Agent 执行终端命令、脚本、构建、测试等。 |  |  |
| Filesystem Tool | 文件系统工具 | 读取、搜索、创建、修改文件的工具。 |  |  |
| Browser Tool / Web Tool | 浏览器/网页工具 | 搜索互联网、打开网页、点击、抓取信息或执行浏览器操作。 |  |  |
| Computer Use | 计算机操作 | 模型通过视觉/GUI 控制鼠标、键盘和桌面应用。 |  |  |
| Playwright | 浏览器自动化框架 | 常用于 Agent 自动操作网页、E2E 测试、截图和 DOM 交互。 | Playwright 本身是框架；接给 Agent 后才成为它可用的工具能力。 | ★ |
| Git Tool | Git 工具 | 让 Agent 查看 diff、提交、分支、worktree、历史等。 |  |  |
| GitHub Integration | GitHub 集成 | 让 Agent 访问仓库、Issue、PR、Actions 等。 |  |  |
| Database Tool | 数据库工具 | 执行 SQL/查询数据库、读取结构化业务数据。 |  |  |
| Vector Database | 向量数据库 | 存储/检索 embedding 向量，常用于语义搜索与 RAG。 |  |  |
| Search Tool | 搜索工具 | 面向网页、代码、文档或内部知识的检索能力。 |  |  |
| Code Interpreter | 代码解释器 | 让模型执行 Python 等代码进行计算、数据处理、文件生成。 |  |  |
| Sandbox | 沙箱 | 隔离代码/工具运行环境，限制文件、网络、系统权限。 |  |  |
| Permission Model | 权限模型 | 规定 Agent 能读/写什么、是否需批准、哪些操作禁止。 |  |  |
| Approval Gate | 审批门 | 在高风险写操作、部署、删除、付款等动作前要求人工批准。 |  |  |
| Webhook | Webhook 回调 | 事件发生时由外部系统主动通知另一个系统的 HTTP 机制。 |  |  |
| Cron / Scheduler | 定时调度 | 按时间周期触发 Agent 或工作流。 |  |  |
| LangChain | LangChain 框架 | 用于构建 LLM 应用、Agent、Tool、RAG 等组件的开源框架生态。 |  |  |
| LangChain Tool | LangChain 工具 | 在 LangChain 中封装并暴露给 Agent 的可调用函数/接口。 |  |  |
| LangGraph | LangGraph | 以图/状态机方式构建有状态 Agent 工作流的框架。 |  |  |
| LlamaIndex | LlamaIndex | 聚焦数据连接、索引、RAG 与 Agent 数据层的框架。 |  |  |
| Cocos CLI | Cocos 命令行工具 | 面向 Cocos 项目自动化、构建或工程操作的 CLI；是否能直接做 UI/Prefab 取决于具体命令能力。 |  |  |
| Figma Integration | Figma 集成 | 将设计稿、节点、样式或资源提供给 Agent，辅助 UI 还原/生成。 |  |  |
| Slack/Teams Connector | 协作平台连接器 | 让 Agent 搜索消息、发通知或自动化团队流程。 |  |  |
| Calendar/Gmail Connector | 日历/邮件连接器 | 让 Agent 查询日程、邮件并在授权下执行操作。 |  |  |

## 5. RAG、检索、知识库与数据层

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| RAG — Retrieval-Augmented Generation | 检索增强生成 | 先从外部知识库检索相关信息，再把结果作为上下文交给生成模型。 | RAG 解决“临时取知识”；Memory 解决“保存并复用状态/经验”，二者可组合。 | ★ |
| Retriever | 检索器 | 根据查询从知识库中召回相关文档/片段。 |  |  |
| Index | 索引 | 为加速检索建立的数据结构，如倒排索引、向量索引。 |  |  |
| Chunking | 分块 | 把长文档切成适合 embedding 和检索的片段。 |  |  |
| Chunk Size | 分块大小 | 单个 chunk 的长度，影响召回粒度与上下文完整度。 |  |  |
| Overlap | 分块重叠 | 相邻 chunk 保留部分重复文本，以降低语义被切断的风险。 |  |  |
| Semantic Search | 语义搜索 | 按向量语义相似度而非纯关键词进行检索。 |  |  |
| Keyword Search | 关键词搜索 | 基于词项匹配的传统检索。 |  |  |
| Hybrid Search | 混合检索 | 结合关键词与向量语义检索。 |  |  |
| BM25 | BM25 排序算法 | 经典关键词相关性打分方法，常与向量检索混合。 |  |  |
| ANN — Approximate Nearest Neighbor | 近似最近邻 | 在大规模向量库中快速寻找相似向量的算法类。 |  |  |
| Reranker | 重排模型 | 对初步召回结果再次排序，提高最终上下文相关性。 |  |  |
| Top-k Retrieval | Top-k 检索 | 取相关性最高的 k 个结果。 |  |  |
| Metadata Filtering | 元数据过滤 | 按时间、来源、权限、标签等过滤检索结果。 |  |  |
| Query Rewriting | 查询改写 | 把用户问题改写为更适合检索的查询。 |  |  |
| Query Expansion | 查询扩展 | 生成同义词/子问题/多查询以提高召回率。 |  |  |
| Multi-hop Retrieval | 多跳检索 | 一个问题分多步检索，后一步依赖前一步结果。 |  |  |
| Knowledge Base | 知识库 | 组织企业/项目文档、FAQ、代码、规范等供检索使用的数据集合。 |  |  |
| Corpus | 语料库 | 用于训练、检索或评测的文本/多模态数据集合。 |  |  |
| Provenance | 来源追溯 | 记录答案由哪些文档、数据或工具结果支持。 |  |  |
| Citation Grounding | 引用依据约束 | 让模型给出可追溯引用并尽量只基于来源作答。 |  |  |
| Freshness | 数据新鲜度 | 知识库/检索数据更新到什么时间。 |  |  |
| Data Ingestion | 数据摄取 | 把文档/数据库/网页清洗、切分、向量化并导入知识系统的流程。 |  |  |
| ETL — Extract, Transform, Load | 抽取-转换-加载 | 把源数据加工后装载到目标系统的传统数据工程流程。 |  |  |

## 6. Harness Engineering、Loop Engineering 与 Agent 工程

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Harness | Agent 执行支架/运行框架 | 包围模型的执行环境：提示、上下文、工具、权限、日志、测试、状态、规则等。 | “模型决定能力上限，Harness 决定能力能不能稳定落地。” | ★ |
| Harness Engineering | Harness 工程/智能体支架工程 | 系统设计 Agent 的上下文、工具、权限、测试、状态、观测和运行环境，使单次 Agent 会话可靠。 |  |  |
| Loop | 循环 | 围绕目标反复执行“行动→观察→评估→调整”的迭代过程。 |  |  |
| Agent Loop | 智能体循环 | Agent 为完成目标不断感知、推理、调用工具、验证、修正的内循环。 |  |  |
| Loop Engineering | 循环工程 | 设计让 Agent 自动反复运行的外层系统：目标、触发、验证、重试、停止、状态与人类升级机制。 | 2026 年快速流行的新术语；可理解为 Prompt→Context→Harness 之后更上层的自动闭环工程。 | ★ |
| Outer Loop | 外循环 | 调度完整 Agent 运行、验证结果、持久化状态并决定是否开启下一轮的系统。 |  |  |
| Inner Loop | 内循环 | 一次 Agent 会话内部的 observe/reason/act/verify 迭代。 |  |  |
| Verification Loop | 验证循环 | 产出→测试/检查→失败→修复→再次验证，直到满足条件。 |  |  |
| Feedback Loop | 反馈回路 | 系统根据输出质量或环境反馈调整下一次行动。 |  |  |
| Closed Loop | 闭环 | 结果能被检测并反馈到下一轮动作，而不是只生成一次输出。 |  |  |
| Open Loop | 开环 | 执行后没有自动反馈/纠错机制。 |  |  |
| Guardrail | 护栏 | 约束 Agent 不越权、不超预算、不违反安全/业务规则的机制。 |  |  |
| Policy | 策略/政策规则 | 系统必须遵循的行为、权限、安全和业务规则。 |  |  |
| Governance | 治理 | 围绕模型/Agent 的权限、审计、责任、风险、数据、安全、成本的组织机制。 |  |  |
| Observability | 可观测性 | 通过日志、trace、指标、事件了解 Agent 在做什么、为什么失败、成本多少。 |  |  |
| Tracing | 链路追踪 | 记录每次模型调用、Tool Call、输入输出、耗时和依赖关系。 |  |  |
| Telemetry | 遥测数据 | 系统运行时采集的指标、日志和事件数据。 |  |  |
| Audit Log | 审计日志 | 记录谁在何时触发什么 Agent、调用了什么工具、修改了什么。 |  |  |
| Checkpointing | 状态检查点 | 在长任务中保存中间状态，失败后可恢复而非从头开始。 |  |  |
| Persistence | 持久化 | 将 Agent 状态、记忆、任务结果写入稳定存储。 |  |  |
| Queue | 任务队列 | 缓冲并调度待执行任务，支持并发、重试、优先级。 |  |  |
| Concurrency | 并发 | 多个 Agent/任务同时运行。 |  |  |
| Parallelism | 并行 | 多个独立步骤同时执行以提高速度。 |  |  |
| Isolation | 隔离 | 不同 Agent/任务在文件、分支、容器、权限上相互不干扰。 |  |  |
| Worktree / Git Worktree | Git 工作树 | 同一 Git 仓库同时检出多个分支到不同目录，适合多个 Coding Agent 并行。 |  |  |
| Factory Model / Agent Factory | Agent 工厂模型 | 把软件交付视为由 Agent、检查、队列和流水线组成的生产系统。 |  |  |
| Dark Factory | 黑灯工厂（AI 工程比喻） | 指极少人工值守、Agent 可长时间自主生产与验证的软件工程形态。 |  |  |

## 7. AI Coding、Vibe Coding 与工程化开发术语

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| AI Coding | AI 辅助编程 | 使用 LLM/Agent 生成、修改、解释、测试和审查代码。 |  |  |
| Coding Agent | 编程智能体 | 能读取仓库、修改代码、运行命令/测试、迭代修复的 Agent。 |  |  |
| Vibe Coding | 氛围编程/凭感觉编程 | 主要以自然语言描述意图，让 AI 大量生成代码，人更关注结果而非逐行实现的开发方式。 | 专业场合可说：“Vibe Coding 适合原型，但生产工程需要 Harness、测试、审查和可维护性约束。” | ★ |
| Spec-driven Development | 规格驱动开发 | 先形成明确需求、验收标准和接口契约，再让人/Agent 实现。 |  |  |
| AI-native Development | AI 原生开发 | 从流程设计之初就把 Agent、自动验证、上下文与工具链作为核心生产方式。 |  |  |
| Repo-level Agent | 仓库级 Agent | 能够理解并跨多个文件/模块修改完整代码仓库的 Agent。 |  |  |
| Codebase Context | 代码库上下文 | 与当前任务相关的代码、架构、规范、历史、依赖和测试信息。 |  |  |
| AGENTS.md / Rules File | Agent 规则文件 | 向 Coding Agent 提供仓库级约束、命令、风格、验证方式等。 |  |  |
| PR Context | Pull Request 上下文 | 当前 PR 的需求背景、diff、讨论、相关文件、测试与变更范围。 |  |  |
| Diff | 差异 | 版本之间新增、删除、修改的代码内容。 |  |  |
| Patch | 补丁 | 一组可应用到代码/文件的修改。 |  |  |
| Apply Patch | 应用补丁 | 把结构化 diff/patch 写入工作区。 |  |  |
| Refactor | 重构 | 不改变外部行为前提下改善代码结构。 |  |  |
| Technical Debt | 技术债 | 为快速交付做出的折中导致未来维护/扩展成本增加。 |  |  |
| Scaffolding | 脚手架 | 快速生成项目、模块、目录、模板代码的工具/过程。 |  |  |
| Code Generation | 代码生成 | 由模板、编译器或 AI 自动产生代码。 |  |  |
| Static Analysis | 静态分析 | 不运行程序即可检查类型、风格、潜在错误和安全问题。 |  |  |
| Lint / Linter | 代码规范检查 | 检查格式、风格、常见错误的工具。 |  |  |
| Type Check | 类型检查 | 验证类型系统约束，如 TypeScript tsc。 |  |  |
| Build | 构建 | 把源代码转换为可运行/可发布产物。 |  |  |
| Artifact | 构建产物/制品 | 编译、打包、测试产生的 APK/AAB/二进制/包等。 |  |  |
| CI — Continuous Integration | 持续集成 | 提交代码后自动执行构建、测试、检查。 |  |  |
| CD — Continuous Delivery/Deployment | 持续交付/部署 | 自动把通过验证的版本交付或部署到环境。 |  |  |
| Pipeline | 流水线 | 按阶段自动执行构建、测试、审查、部署等步骤。 |  |  |
| Unit Test | 单元测试 | 验证最小函数/类/模块行为。 |  |  |
| Integration Test | 集成测试 | 验证多个模块/服务组合后的行为。 |  |  |
| E2E — End-to-End Test | 端到端测试 | 从用户入口到后端/结果的完整链路测试。 |  |  |
| Regression Test | 回归测试 | 确认新改动没有破坏已有功能。 |  |  |
| Smoke Test | 冒烟测试 | 快速验证关键路径是否基本可用。 |  |  |
| Golden Test / Golden File | 黄金测试/基准文件 | 用已确认的期望输出与当前输出对比。 |  |  |
| Snapshot Test | 快照测试 | 保存输出快照，后续改动时自动比较差异。 |  |  |
| Visual Regression | 视觉回归测试 | 通过截图像素/结构对比发现 UI 视觉偏差。 |  |  |
| Benchmark | 基准测试 | 用统一任务/指标比较模型或系统表现。 |  |  |
| Eval / Evaluation | 评测 | 系统化评价模型/Agent 的正确性、稳定性、安全性、成本等。 |  |  |
| Eval Harness | 评测框架 | 自动运行测试集、收集结果、评分和生成报告的系统。 |  |  |
| Pass@k | k 次通过率指标 | 生成 k 个候选时至少一个正确的概率，常见于代码评测。 |  |  |
| RCA — Root Cause Analysis | 根因分析 | 从表面故障追到真正原因，并提出防止复发的措施。 | Agent RCA 不只是“修 bug”，还要解释触发条件、根因、修复、验证和预防。 | ★ |
| Postmortem | 事故复盘 | 对故障过程、影响、根因、改进措施进行结构化总结。 |  |  |
| Acceptance Criteria | 验收标准 | 任务完成必须满足的可验证条件。 |  |  |
| Definition of Done — DoD | 完成定义 | 团队统一认定“真正完成”需要满足的一组质量门槛。 |  |  |
| Green / Test Green | 测试全绿 | 所有要求的自动化检查均通过。 |  |  |
| Red-Green-Refactor | 红-绿-重构 | TDD 循环：先失败测试，再实现通过，再重构。 |  |  |

## 8. 评测、可靠性、安全、治理与企业术语

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Reliability | 可靠性 | 系统在重复任务、异常条件和长期运行下稳定完成目标的能力。 |  |  |
| Robustness | 鲁棒性 | 面对噪声、变化输入、工具失败时仍保持合理表现。 |  |  |
| Accuracy | 准确率 | 预测/回答正确的比例；不同任务定义不同。 |  |  |
| Precision | 精确率 | 预测为正的样本中真正为正的比例。 |  |  |
| Recall | 召回率 | 真正为正的样本中被识别出来的比例。 |  |  |
| F1 Score | F1 分数 | Precision 与 Recall 的调和平均。 |  |  |
| Ground Truth | 标准答案/真值 | 用于训练或评测的可信目标结果。 |  |  |
| Judge Model / LLM-as-a-Judge | 裁判模型/模型评审 | 让另一个 LLM 按规则评分输出；方便但可能有偏差。 |  |  |
| Rubric | 评分量表 | 把质量要求拆成明确评分维度和标准。 |  |  |
| A/B Test | A/B 测试 | 将不同模型/提示/产品方案随机分流比较真实效果。 |  |  |
| Offline Eval | 离线评测 | 在固定数据集上离线比较模型/Agent。 |  |  |
| Online Eval | 在线评测 | 在真实流量或运行环境中衡量效果。 |  |  |
| Canary Release | 金丝雀发布 | 先向小比例用户/流量发布，确认稳定后逐步扩大。 |  |  |
| Guardrails | 安全护栏 | 输入/输出/工具/权限/成本层的安全与业务约束。 |  |  |
| Policy Engine | 策略引擎 | 集中判断哪些动作允许、需要审批或禁止。 |  |  |
| Least Privilege | 最小权限 | 只给 Agent 完成任务所需的最低权限。 |  |  |
| RBAC — Role-Based Access Control | 基于角色的访问控制 | 根据角色分配系统权限。 |  |  |
| Auditability | 可审计性 | 系统行为可以被追踪、还原与问责。 |  |  |
| Compliance | 合规 | 满足法律、行业、公司数据和安全规范。 |  |  |
| PII — Personally Identifiable Information | 个人可识别信息 | 可识别具体个人的数据，AI 系统处理时需谨慎。 |  |  |
| Data Residency | 数据驻留 | 数据必须存储/处理在特定国家或区域的要求。 |  |  |
| Data Governance | 数据治理 | 数据质量、权限、血缘、隐私、生命周期管理。 |  |  |
| Model Governance | 模型治理 | 模型选择、版本、风险、验证、监控和退役等管理机制。 |  |  |
| AI Governance | AI 治理 | 涵盖模型、Agent、数据、安全、风险、责任与组织政策的整体治理。 |  |  |
| Red Teaming | 红队测试 | 主动寻找模型/Agent 安全漏洞、越权路径和极端失败模式。 |  |  |
| Adversarial Testing | 对抗测试 | 用恶意或极端输入检验系统鲁棒性与安全。 |  |  |
| Rate Limit | 限流 | 限制单位时间请求/token 数，避免过载或滥用。 |  |  |
| Quota | 配额 | 账号/项目在周期内允许使用的资源上限。 |  |  |
| SLA — Service Level Agreement | 服务等级协议 | 对可用性、响应时间、支持等服务指标的正式承诺。 |  |  |
| SLO — Service Level Objective | 服务等级目标 | 内部设定的服务可靠性目标。 |  |  |
| Cost per Task | 单任务成本 | 完成一次完整任务所消耗的模型、工具、算力和人工成本。 |  |  |
| Token Economics | Token 经济性 | 围绕输入/输出/缓存 token、模型价格和任务成功率的成本分析。 |  |  |
| Unit Economics | 单位经济模型 | 每个用户/任务/Agent 的收入、成本与毛利结构。 |  |  |

## 9. 推理、训练与高级技术黑话

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Inference-time Compute | 推理时计算 | 在回答阶段投入更多计算/迭代以换取更高推理质量。 |  |  |
| Test-time Compute / TTC | 测试时计算 | 广义上指推理/测试阶段额外计算，如多采样、搜索、验证。 |  |  |
| Reasoning Effort | 推理强度/推理预算 | 控制模型在复杂任务上投入多少内部推理资源的产品/接口概念。 |  |  |
| Self-Consistency | 自洽采样 | 生成多个推理路径并以一致性/投票选择答案。 |  |  |
| Best-of-N | N 选一 | 生成 N 个候选，再由评分器选择最佳。 |  |  |
| Beam Search | 束搜索 | 保留若干高概率序列候选进行搜索的解码方法。 |  |  |
| Speculative Decoding | 推测解码 | 用小模型/草稿模型先预测，再由大模型验证，以提高推理速度。 |  |  |
| Attention | 注意力机制 | Transformer 用于根据上下文动态聚合信息的核心机制。 |  |  |
| Self-Attention | 自注意力 | 序列内部 token 之间互相计算关联。 |  |  |
| Transformer | Transformer 架构 | 现代 LLM 的核心神经网络架构，以注意力机制为基础。 |  |  |
| RoPE — Rotary Positional Embedding | 旋转位置编码 | 常见的 Transformer 位置信息编码方法。 |  |  |
| FlashAttention | FlashAttention | 优化 attention 内存访问与计算效率的算法/实现。 |  |  |
| RL — Reinforcement Learning | 强化学习 | 通过奖励信号学习行为策略。 |  |  |
| Reward Model | 奖励模型 | 对候选输出质量/偏好打分，用于 RLHF 等。 |  |  |
| Policy Model | 策略模型 | 强化学习中根据状态选择动作的模型。 |  |  |
| Synthetic Data | 合成数据 | 由模型、程序或仿真自动生成的训练/评测数据。 |  |  |
| Data Flywheel | 数据飞轮 | 产品使用→产生反馈/数据→提升模型→提升产品→产生更多数据的循环。 |  |  |
| Curriculum Learning | 课程学习 | 按难度或能力阶段组织训练数据，让模型循序学习。 |  |  |
| Continual Learning | 持续学习 | 模型在部署/长期过程中持续吸收新知识与经验，同时尽量避免遗忘。 |  |  |
| Catastrophic Forgetting | 灾难性遗忘 | 模型学习新任务后显著丢失旧能力的现象。 |  |  |
| Self-Improvement | 自我改进 | AI 利用自身生成、评估、研究或数据生产能力提升后续系统。 |  |  |
| Recursive Self-Improvement | 递归自我改进 | 系统改进自身能力后，改进后的系统继续加速下一轮改进的理论概念。 |  |  |
| World Model | 世界模型 | 对环境状态、变化规律和行动后果进行内部建模的模型/系统概念。 |  |  |
| Embodied AI | 具身智能 | AI 通过机器人/设备在物理世界中感知、决策、行动并学习。 |  |  |
| Digital Twin | 数字孪生 | 现实对象/系统在数字环境中的动态映射，可用于仿真与 Agent 控制。 |  |  |
| Simulation | 仿真 | 在虚拟环境中模拟真实系统，用于训练、测试或决策。 |  |  |
| Scaling Laws | 缩放定律 | 模型规模、数据、计算量与性能之间的经验规律。 |  |  |
| Compute-optimal Training | 计算最优训练 | 在给定计算预算下平衡模型参数与训练数据以获得更高性能。 |  |  |
| Emergent Capability | 涌现能力 | 模型规模/训练达到一定程度后表现出的新能力；具体“涌现”定义在研究中有争议。 |  |  |
| Capability Overhang | 能力悬置 | 模型已有能力未被产品、提示、工具或评测充分激发/发现。 |  |  |

## 10. 行业常见产品/框架/模型命名与易误用词

| **English / Acronym** | **中文** | **专业解释（说人话）** | **专业交流用法/边界** | **★** |
| --- | --- | --- | --- | --- |
| Sol | Sol（产品/模型档位名） | 某些 AI 产品中用于区分模型/推理配置的命名；不是全行业统一技术术语。 | 交流时应补全厂商与版本，例如“GPT-5.6 Sol”，避免只说 Sol。 | ★ |
| Terra / Luna | Terra / Luna（产品/模型档位名） | 与 Sol 类似，属于具体产品体系内的模型/能力档位命名时，应以对应厂商文档为准。 |  |  |
| Model Tier | 模型档位 | 同一产品内按能力、速度、成本区分的模型层级。 |  |  |
| Reasoning Tier | 推理档位 | 同一模型使用不同推理预算/强度的配置。 |  |  |
| Plugin vs Tool | 插件 vs 工具 | Plugin 是宿主扩展形态；Tool 是模型可调用的动作能力。 | 一个 Plugin 可以暴露多个 Tools。 | ★ |
| Tool vs Skill | 工具 vs 技能 | Tool 偏执行接口；Skill 偏领域方法、说明、脚本和资源集合。 |  |  |
| MCP vs API | MCP vs API | API 是具体软件接口；MCP 是让 AI 客户端标准化发现/调用外部能力的协议层。 |  |  |
| Agent vs Workflow | Agent vs 工作流 | Agent 动态决定下一步；Workflow 更多由预定义流程控制。 |  |  |
| RAG vs Fine-tuning | RAG vs 微调 | RAG 运行时取外部知识；微调修改模型权重/行为。 |  |  |
| RAG vs Memory | RAG vs 记忆 | RAG 检索知识；Memory 保存并恢复任务/用户/经验状态。 |  |  |
| Context vs Memory | 上下文 vs 记忆 | Context 是这次调用模型可见内容；Memory 是可持久化并在需要时重新注入 Context 的信息。 |  |  |
| KV Cache vs Prompt Cache | KV Cache vs 提示缓存 | KV Cache 是 Transformer 推理计算缓存；Prompt/Prefix Cache 是产品层对重复前缀计算的复用机制。 |  |  |
| Function Calling vs API Call | 函数调用 vs API 调用 | Function Calling 通常是模型生成结构化调用意图；真正 API 请求由宿主程序执行。 |  |  |
| Langchat Tool | “Langchat Tool”（常见误写） | 行业主流通常是 LangChain Tool。若你看到“Langchat”，需确认对方是否指某个具体产品或只是拼写错误。 |  |  |
| Kappay / Kappa | Kappay / Kappa（需确认语境） | “kappay”不是当前主流通用 AI 术语。Kappa 可能指 Kappa Architecture（流式数据架构）、某项目名或内部代号。 | 专业交流中不要不加语境直接使用，先确认来源。 | ★ |
| RCA | 根因分析 | 不是 AI 专属词，但在 Agent 工程里非常常见，用来要求 Agent 不只修复表象，还要给根因证据。 |  |  |

## 11. AI 工程发展脉络：从 Prompt 到 Loop

| 阶段 | 中文 | 核心对象 | 一句话问题 | 关键指标 |
| --- | --- | --- | --- | --- |
| Prompt Engineering | 提示词工程 | 优化一次指令 | “怎么问模型？” | 单轮质量 |
| Context Engineering | 上下文工程 | 优化模型看到的信息 | “给模型什么？” | 相关性、完整性、token 效率 |
| RAG / Memory | 检索与记忆 | 补充外部知识和长期状态 | “信息从哪里来、怎么记住？” | 召回、事实性、持续性 |
| Tool / MCP / Skill | 能力接入 | 让模型能行动并掌握专业方法 | “它能做什么、会怎么做？” | 工具成功率、权限、可组合性 |
| Harness Engineering | 支架工程 | 让一次 Agent 会话稳定可靠 | “怎样把模型装进可控工程环境？” | 可靠性、测试、日志、权限 |
| Loop Engineering | 循环工程 | 让 Agent 自动反复完成目标 | “怎样无人盯守仍能持续推进？” | 停止条件、验证、重试、状态、成本 |
| Agent Factory / AI-native SDLC | AI 原生交付系统 | 多 Agent + Pipeline + Governance | “怎样规模化生产结果？” | 吞吐、质量、成本、审计 |

## 12. 梁文锋公开交流中提到的 AGI“阶梯”路线（2026 公开报道整理）

**重要说明：**公开报道对“几阶”计数存在两种口径：有的从“语言模型”开始算六阶；有的把语言模型视为前置基础，从 CoT 开始算五阶。核心顺序基本一致。以下按完整六阶表达。

| 阶梯 | English | 中文 | 核心含义 | 阶段判断 |
| --- | --- | --- | --- | --- |
| 1 | Language Model | 语言模型 | 基础语言能力与知识表示 | 前置基础 |
| 2 | CoT — Chain of Thought | 思维链 | 通过推理提升智能上限 | 报道中称前一阶段重点 |
| 3 | Agent | 智能体 | 从回答问题扩展到调用工具、执行任务 | 2026 年重点；优先 Coding/General Agent |
| 4 | Continual Learning | 持续学习 | 像员工一样在长期工作中积累经验，而非每次都靠完整上下文 | 被视为关键瓶颈 |
| 5 | Self-Iteration / AI accelerates AI R&D | 自我迭代 / AI 加速 AI 研发 | AI 参与研发下一代 AI，形成渐进式加速 | 接近“渐进奇点”的概念 |
| 6 | Embodied Intelligence | 具身智能 | 把智能扩展到物理世界的感知与行动 | 更后期阶段 |

**专业表达示例：**“按梁文锋 2026 年公开交流的阶梯式判断，Agent 不是终点；真正影响 AI 能否像员工一样长期工作的下一道门槛是 Continual Learning，之后才可能出现 AI 加速 AI 研发的递归改进。”

## 13. 交流时显得专业的 30 句“高信息密度表达”

01. 这个问题不要只看模型能力，要拆成 Model、Context、Tool、Harness 和 Eval 五层。

02. LLM 是推理核心，Agent 是一个系统，不要把二者当同义词。

03. 先问有没有可验证的 Done Condition，否则 Agent 很容易“自认为完成”。

04. 长任务真正难点不是一次生成，而是状态持久化、失败恢复和验证闭环。

05. 这里更像 Context Engineering 问题，不是再改一版 Prompt 就能根治。

06. 如果知识变化快，优先考虑 RAG/Tool Grounding，而不是把所有知识 Fine-tune 进权重。

07. Function Calling 只是模型产生结构化调用意图，执行和权限仍在宿主系统。

08. MCP 解决的是工具/资源接入标准化，不自动解决工具质量、权限和业务语义。

09. Skill 是把领域 SOP、脚本和资源模块化；Tool 是执行动作，两者层次不同。

10. 生产级 Agent 的核心不是“能跑”，而是可重复、可验证、可观测、可恢复。

11. 多 Agent 并不天然更强，先看任务是否真的可拆、通信成本是否值得。

12. Planner 和 Executor 分离的价值，是降低一边执行一边改目标导致的漂移。

13. Reviewer 最好与 Coder 解耦，否则容易出现同源偏差。

14. 能用 deterministic test 验证的，就不要只靠 LLM-as-a-Judge。

15. Agent 自验证可以作为第一层，但最终还是要有外部 verifier 或真实环境反馈。

16. 这个失败应该做 RCA：触发条件、根因、修复、回归验证、预防机制都要闭环。

17. Worktree 的价值是隔离并行 Agent，避免多个执行器互相覆盖工作区。

18. Prompt → Context → Harness → Loop，是近几年 AI 工程关注点逐层上移的一个理解框架。

19. Loop Engineering 关注的不只是“再跑一次”，而是何时重试、如何验证、什么时候停止。

20. 自治程度越高，Governance、权限最小化、预算和审计就越重要。

21. 别只看 benchmark，要看真实任务成功率、重试率、人工接管率和 cost per task。

22. 模型选型应该看 capability/cost/latency 三角，不是永远选最强模型。

23. 低成本模型做路由和简单执行，高推理模型处理规划、异常和高价值决策，是常见的混合架构。

24. 缓存命中影响成本和延迟，但“会话变笨”通常还要看上下文装配、摘要和状态恢复。

25. KV Cache、Prompt Cache、Memory 是三种完全不同层次的东西。

26. Vibe Coding 能提高原型速度，但生产代码需要 Spec、Tests、Review 和维护边界。

27. 真正成熟的 AI Coding 不是“让模型写代码”，而是让 Agent 在 CI 和验收标准下完成闭环。

28. RAG 的瓶颈很多时候不在生成，而在 chunking、retrieval、rerank 和数据新鲜度。

29. 如果没有 observability，你很难知道 Agent 是模型能力不足，还是 Tool/Context/Harness 出问题。

30. 专业不是缩写多，而是知道每个术语的边界、依赖和什么时候不该用。

## 14. 关键术语核验来源与备注

| 主题 | 核验说明 | 来源 |
| --- | --- | --- |
| MCP | Model Context Protocol 官方文档：MCP 是连接 AI 应用与外部系统的数据、工具、工作流的开放标准；官方将 Server 能力分为 Prompts、Resources、Tools 等。 | https://modelcontextprotocol.io/docs/getting-started/intro |
| Agent Skills | Anthropic Engineering（2025-10）：Agent Skills 将 instructions、scripts、resources 组织成可发现、动态加载的专业能力模块。 | https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills |
| Function Calling / Tools | OpenAI API 文档：模型可选择 function/custom/MCP/built-in tools，并用 JSON Schema 描述参数。 | https://platform.openai.com/docs |
| RAG | Lewis et al., 2020, Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks。 | https://arxiv.org/abs/2005.11401 |
| Loop Engineering | IBM Think（2026-07）：定义为设计 Agentic workflows/loops，使 Agent 能行动、观察、决策、迭代直至完成目标。 | https://www.ibm.com/think/topics/loop-engineering |
| 梁文锋 AGI 阶梯 | 2026-07 多家媒体对投资者交流会内容的公开整理：核心顺序为语言模型→CoT→Agent→持续学习→自我迭代/AI 加速 AI 研发→具身智能；不同报道对“5 阶/6 阶”计数口径不同。 | 公开报道整理，需以 DeepSeek 后续正式发布原文为最高依据 |

**版本说明：**AI 行业术语变化很快，尤其是模型档位名、Agent 框架、插件/连接器名称。本文把“稳定通用术语”和“产品特定名称”分开处理；遇到新词时先判断它是行业标准、论文术语、厂商命名，还是社区黑话。
