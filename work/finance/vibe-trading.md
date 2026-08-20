
```mermaid
flowchart TD
    U[User] --> FE[Frontend Web UI]
    U --> CLI[CLI]
    U --> MCP[MCP Server]
    U --> CH[IM Channels]

    FE --> API[FastAPI API Server]
    CLI --> API
    MCP --> SVC[SessionService]
    CH --> SVC
    API --> SVC[SessionService]

    SVC --> STORE[SessionStore]
    SVC --> BUS[EventBus]
    SVC --> ATTEMPT[Attempt Lifecycle]
    BUS --> SSE[SSE Stream]
    SSE --> FE

    SVC --> AGENT[AgentLoop]
    AGENT --> CTX[ContextBuilder]
    AGENT --> LLM[ChatLLM]
    AGENT --> MEM[WorkspaceMemory]
    AGENT --> PM[PersistentMemory]
    AGENT --> REG[ToolRegistry]
    AGENT --> TRACE[TraceWriter]
    AGENT --> GROUND[GroundingLedger]

    REG --> TOOLS[Finance Tools]
    TOOLS --> DATA[Market Data Loaders]
    TOOLS --> BT[Backtest Tool]
    TOOLS --> DOC[Document/Web Tools]
    TOOLS --> QUANT[QuantLib/Factor Tools]
    TOOLS --> TRADE[Trading Connectors]

    BT --> RUNNER[backtest.runner]
    RUNNER --> VALID[Config + Signal Validation]
    RUNNER --> LOADER[Loader Registry/Fallback]
    RUNNER --> ENGINE[Market Engine]
    ENGINE --> ARTIFACTS[Metrics/Reports/Run Card]

    STORE --> PERSIST[Sessions/Attempts]
    TRACE --> PERSIST
    ARTIFACTS --> PERSIST
    GROUND --> PERSIST
```




之前研究问题：以某种策略，估值英伟达，结果如何？
1. 先找股票，查当前价格，历史价格
2. 找数据，去yahoo，tushare，akshare，ccxt找，下载下来，清洗，对齐，处理缺失值，转换格式
3. 写分析代码，计算收益，波动，回撤，夏普
4. 通过历史数据，写策略，模拟交易，看收益和风险
5. 最后整理报告，画图，保存结果

vibe-trading把原本，人找数据，下载，清洗，写代码，分析，回测，整理报告，变成了，一个自然语言，就可以找数据，分析，回测，然后保存结果了。

==我觉得这些过程都很碎，但是不是最困难的点，困难的点是如何写策略？这个工具只是解决碎的问题，降低了操作成本，但核心策略还是靠人的，相当于给你一个好的工具，如何能用好看个人经验了，我做的就是这个事情，如何熟练的搭建起来工具，然后给那些策略的人用！！！==




一句话定义

> **这是一个给"有交易想法但不会编程的投资者/交易员"解决"如何把交易想法快速变成可验证的量化策略并回测验证"的个人交易智能体系统。**


**Vibe-Trading = LLM Agent (推理引擎) + 74个金融技能(知识) + 74个工具(能力) + 29个DAG多Agent团队(协作流程) + 7个回测引擎(验证) + 20+数据源(情报)**






价值
1. 自然语言得到回测代码，这跟掌柜问数项目（text2sql）类似，聚焦业务而非代码
2. 内置的74个金融技能（因子分析，期权定价，风险审计...），就是简单的规则skill，这里有问题，怎么评判到底评估的好与不好？74个工具注册表又是什么？预设的29个swarm DAG引擎？
3. 上下文组装系统提示词是什么东西？
4. 多agent团队，这里跟金融技能差不多，怎么评判结果好与坏？
5. 多数据源，从哪来的？如果是免费的怎么保证准确性？
6. 回测的验证怎么判断的？是什么逻辑？




前置知识
1. **Swarm 配置化** ： 没有硬编码，需要变得参数写在yaml、json里面
2. DAG：有向无环图，制定数据流转流程
3. 







# 核心流程

提出问题——agent理解——决定用什么能力——调用数据，搜索，分析，回测等工具——得到结果，整理——保存研究过程和结果

数据流动路径永远是：**用户输入 → 入口层 → SessionService → AgentLoop(ReAct循环) → (Skills / Tools / Swarm / Backtest) → 结果聚合 → SSE推送 → 用户看到报告**

AgentLoop 是整个系统的"大脑"——它决定什么时候调用什么工具、什么时候启动多Agent团队、什么时候写代码跑回测、什么时候生成报告。其他所有模块都是它的"手和脚"，被它按需调用。

典型场景：用户说"帮我做一个高股息低估值的中证500选股策略，回测2024年"

1. 用户post
2. api-server/cli/mcp-server做鉴权处理，是否符合一些规范
3. 鉴权通过送到session管理处
	1. 如果当前session已经存在，返回409 并发控制
	2. 否则就是新的，attempt记录，然后提交到线程池里（这里大概是有线程控制，有调度的，不能无限制创建实例），并把当前事件sse传到客户端，然后听线程池调度thread_pool的管理，到时候创建agentloop实例
4. AgentLoop (src/agent/loop.py) — ReAct 主循环 
	1. context上下文管理：系统提示词+74知识skills+74工具+记忆memory+加载策略发现路由（这个是啥？）+用户role的信息
	2. LLM大模型获取到这些上下文信息，开始思考决定使用什么能力（因为有提示词，上下文）
		1. 调用load_skill工具找有哪些策略，找有哪些工具，哪些模板
		2.  swarm_tool.launch("quant_strategy_desk", params={market: "A-share", objective: "高股息低估值"})  ，大致是开始走既定流程，yaml已经定义好的策略，股票怎么筛选？因子怎么挖掘？策略怎么跑？风险怎么回测？报告怎么聚合？得到的结果依次往回返回

这里有疑问：数据怎么插入的？





# 框架


```text
                    Vibe-Trading
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ↓                 ↓                 ↓
   ① 用户入口        ② Agent大脑       ③ 工具能力
       │                 │                 │
    Web / CLI         Agent Loop        Tools
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ↓
                  ④ 金融数据与计算
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          Market Data            Backtest
              │                     │
              └──────────┬──────────┘
                         ↓
                  ⑤ 研究结果/记忆
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
           Memory                Reports
                         
                         +
                         
                  ⑥ 多 Agent / MCP
```


## 交互层 presentation

web（react19）、CLI（richTUI）、MCP（Stdio）、Rest API（FastAPI）
这几个都是前端可以用的东西，可以浏览器web，也可以命令行cli，也可以ai直接调用mcp，这个rest api 好像是说前后端通信的规则！不像是一个输入框，还是说就想调用大模型的api这种key就行？给个url+key联网就可以访问这个服务了？好像不是



MCP：这里代码提供的是mcp-server，也就是llm可以访问这些server，那么就要了解这里的server是如何写的？假设某个server可以调用天气预报，那么是怎么实现的？这个很细，先不考虑



## 会话管理 session

会话怎么创建的？怎么恢复？怎么查询？

消息怎么路由到AgentLoop？

SSE 事件怎么推送到EventBus的？

并发怎么控制的？


`SessionStore` 是**硬盘**，`EventBus` 是**喇叭**，`SessionService` 是**大脑**。
```
SessionService (服务层/大脑)
    │
    ├── 用 SessionStore 读写磁盘 (持久化)
    │
    └── 用 EventBus 广播事件给前端 (实时推送)
```
SessionStore：**把 Session/Message/Attempt 存到磁盘上，以及读回来**

`EventBus` — SSE 事件总线：**当 AgentLoop 在后台线程执行时，把进度推送给前端的 HTTP 长连接。**

**重点追 `EventBus` 的 `set_loop()` 是在哪里用到的。** 那样你就能真正理解为什么这里要把 `loop` 塞进去，而不是只停留在“loop 是异步循环”这个概念上。

==这里实现是用队列做的，也就是阻塞后就用下一个==







## Agent 

agent core = context知道什么？ + loop下一步干什么？ + tools能干什么？+ memory记得什么？

ReAct怎么设计的？怎么思考？怎么调用工具？怎么看结果？

5层压缩对话怎么压缩的（长对话自动压缩避免超 token 限制）？

工具怎么批量处理的？

这里的会话持久保存怎么做的？（FTS5 全文搜索？）












## 工具执行

7个回测引擎是什么？

20个数据是怎么加载的？

265个quantlib量化计算库是什么？

13个交易连接器怎么做到的？

4种组合优化器是什么？

风险审计（VaR、MC、Bootstrap）？反正就是三个计算亏损概率的方法













# 细节问题

```
Vibe-Trading/
├── agent/                          # Python 后端
│   ├── api_server.py / cli.py / mcp_server.py  # 入口点
│   ├── backtest/                   # 回测引擎 + 数据加载器 + 优化器
│   ├── src/
│   │   ├── agent/                  # AgentLoop 推理核心
│   │   ├── tools/                  # 74个工具（每个一个文件）
│   │   ├── skills/                 # 74个金融技能（每个一个目录）
│   │   ├── swarm/                  # DAG 多 Agent 编排
│   │   ├── providers/              # LLM 提供商抽象层
│   │   ├── session/                # 会话管理
│   │   ├── memory/                 # 持久化内存
│   │   ├── quantlib/               # 量化计算库
│   │   ├── trading/                # 交易连接器
│   │   ├── live/                   # 实盘交易相关
│   │   ├── governance/             # 模型风控
│   │   ├── goal/                   # 交易目标管理
│   │   └── strategy_discovery/     # 策略发现
│   └── tests/                      # 测试
├── frontend/                       # React 19 前端
├── wiki/                           # 文档


```
```
agent/
├── api_server.py          # 入口1: FastAPI (≈400行)
├── cli/main.py            # 入口2: CLI TUI
├── mcp_server.py          # 入口3: MCP 服务器
│
├── src/
│   ├── api/               # API 路由 + 安全 + 状态管理
│   │   ├── routes.py
│   │   ├── security.py
│   │   └── state.py       # 全局状态（session_service 等）
│   │
│   ├── session/
│   │   ├── service.py     # SessionService (≈600行) ★
│   │   ├── store.py       # 会话持久化存储
│   │   ├── models.py      # Session/Attempt/Message 模型
│   │   └── events.py      # SSE 事件总线
│   │
│   ├── agent/             # AgentLoop 推理核心
│   │   ├── loop.py        # AgentLoop ReAct 主循环 (≈2500行) ★★★
│   │   ├── context.py     # ContextBuilder 上下文组装 (≈430行) ★
│   │   ├── tools.py       # ToolRegistry 工具注册表 74个工具
│   │   ├── skills.py      # SkillsLoader 技能加载 74个skills
│   │   ├── memory.py      # WorkspaceMemory 运行内存
│   │   ├── grounding.py   # 数据"落地"层
│   │   └── frontmatter.py # YAML 元数据解析器
│   │
│   ├── swarm/             # DAG 多 Agent 编排
│   │   ├── models.py      # DAG 数据模型
│   │   ├── runtime.py     # DAG 运行时引擎 ★★
│   │   ├── worker.py      # 单个 Agent worker 执行
│   │   └── presets/       # 29 个预设 YAML ★
│   │
│   ├── tools/             # 74 个工具 ★
│   │   ├── backtest_tool.py
│   │   ├── swarm_tool.py
│   │   ├── market_data_tool.py
│   │   ├── load_skill_tool.py
│   │   └── ... (每个文件一个工具)
│   │
│   ├── skills/            # 74 个金融技能 (SKILL.md)
│   │
│   ├── providers/         # LLM 提供商抽象层
│   │   ├── llm.py         # ChatLLM 抽象层
│   │   ├── chat.py        # LLM 调用实现
│   │   └── capabilities.py
│   │
│   ├── memory/
│   │   └── persistent.py  # 持久化内存
│   │
│   ├── trading/           # 交易连接器
│   ├── live/              # 实盘交易相关
│   ├── governance/        # 模型风控
│   ├── goal/              # 交易目标管理
│   ├── strategy_discovery/# 策略发现
│   └── quantlib/          # 量化计算库 (265 函数)
│
├── backtest/
│   ├── engines/           # 7 个回测引擎 ★★
│   │   ├── base.py        # BaseEngine
│   │   ├── china_a.py     # A股引擎
│   │   ├── global_equity.py
│   │   ├── crypto.py
│   │   └── ...
│   ├── loaders/           # 20+ 数据加载器 ★
│   │   ├── base.py        # DataLoader Protocol
│   │   ├── registry.py    # LoaderRegistry + 自动降级
│   │   ├── tushare.py
│   │   ├── yfinance_loader.py
│   │   └── ...
│   ├── runner.py          # 回测运行器
│   ├── metrics.py         # 绩效指标
│   └── validation.py      # 蒙特卡洛/WalkForward
frontend/                       # React 19 前端
wiki/                           # 文档
```

**组织方式的优点：**
1. **关注点分离** — agent（推理）+ tools（能力）+ skills（知识）三层解耦
2. **插件化设计** — 新增一个工具 = 在 tools/ 加一个文件，自动注册
3. **策略与实现分离** — 技能用 SKILL.md（Markdown）描述，LLM 通过 `load_skill` 按需加载
4. **Swarm 配置化** — 团队协作流程用 YAML 定义，无需改代码




tips：有些知识点知道了也别删

1. 数据来源碎，多，杂，如何统一处理输入模型的？
2. 为了解决可信，如何持久化保存记忆的？让来源可溯源
3. 任务多，如何解决并发，session如何处理？eventbus，sse，都是什么？
4. 
5. 
6. 
7. 
8. 
9. 
10. 这里agent分析的步骤写死了，这些策略工具够用吗？








```mermaid
sequenceDiagram
    participant User as 用户
    participant FE as Agent.tsx
    participant API as FastAPI
    participant SVC as SessionService
    participant Store as SessionStore
    participant Bus as EventBus
    participant Agent as AgentLoop
    participant LLM as ChatLLM
    participant Registry as ToolRegistry
    participant BT as BacktestTool
    participant Runner as BacktestRunner
    participant Loader as LoaderRegistry
    participant Engine as BacktestEngine
    participant FS as RunArtifacts

    User->>FE: 输入自然语言问题
    FE->>API: POST sessions id messages
    API->>SVC: send_message
    SVC->>Store: 保存 user message
    SVC->>Store: 创建 Attempt
    SVC->>Bus: attempt.created
    SVC-->>API: 返回 message_id 和 attempt_id
    API-->>FE: 请求立即返回

    SVC->>SVC: asyncio.create_task run_attempt
    SVC->>SVC: build_registry
    SVC->>LLM: 创建模型适配器
    SVC->>Agent: 创建 AgentLoop

    FE->>API: GET sessions id events
    API->>Bus: subscribe
    Bus-->>FE: SSE attempt.started

    Agent->>Agent: 创建 run_dir
    Agent->>FS: 写入 req.json trace manifest
    Agent->>Agent: ContextBuilder.build_messages
    Agent->>LLM: stream_chat

    LLM-->>Agent: tool call load_skill
    Agent->>Registry: 执行 load_skill
    Registry-->>Agent: 返回技能内容
    Agent->>Bus: tool_call 和 tool_result
    Bus-->>FE: SSE 工具事件

    Agent->>LLM: 继续推理
    LLM-->>Agent: tool call write config
    Agent->>Registry: 执行写文件工具
    Registry-->>Agent: 返回写入结果

    Agent->>LLM: 继续推理
    LLM-->>Agent: tool call backtest
    Agent->>BT: execute run_dir
    BT->>Runner: subprocess execute

    Runner->>Runner: 校验 config 和 signal_engine
    Runner->>Loader: 获取 BTC 数据
    Loader-->>Runner: 返回 OHLCV 数据
    Runner->>Engine: run_backtest
    Engine->>Engine: 生成信号
    Engine->>Engine: 逐 bar 执行
    Engine->>Engine: 计算指标
    Engine->>FS: 写入 metrics.csv
    Engine->>FS: 写入 reports
    Engine->>FS: 写入 run_card

    Runner-->>BT: 返回执行结果
    BT-->>Agent: JSON 工具结果
    Agent->>Bus: tool_result
    Bus-->>FE: SSE 工具完成事件

    Agent->>LLM: 请求最终总结
    LLM-->>Agent: 文本最终答案

    Agent->>FS: 写入 state.json
    Agent->>FS: 写入 trace
    Agent-->>SVC: success 和 run_dir 和 content

    SVC->>Store: 保存 assistant message
    SVC->>Bus: attempt.completed
    Bus-->>FE: SSE 完成事件

    FE->>API: GET sessions id messages
    API-->>FE: 返回完整消息和 tool trail
```


