
思路：用自己做得agent开发agent就ok了，也别求别人用，自己首先能用得下去就完了  
先把最通用得coding agent做掉，剩下的估摸着也好弄了


[‌​​​‬​​‬​​⁠‍‌‬​​‌​‍‍​⁠‌‬​​​​​⁠⁠​﻿​‬⁠​​‬​​‌⁠‌​​‬‌‬​从后端转AI Agent这条路，我踩了半年坑才摸清门道 - 飞书云文档](https://my.feishu.cn/wiki/YeXfw5N1eic2cEkRfMkcLYZUnVi)

# 2026年程序员转型Agent工程师 已上岸-完整经验分享【贡献者：鱼塘-南京-77红姐】

来源：「码农互助」微信群友「鱼塘-77」的分享整理  
整理日期：2026年6月

# 一、分享人背景

鱼塘-77，【搜索索引缺失：此处可能有初始专业或职业经历】后转Java后端开发（黑马Java培训），再转型AI Agent开发。

目前在南京一家公司负责广告业务的Agent化，薪资30K，六险一金，包三餐，七月入职。

简历投递情况：小红书没过，希音、360、汇丰银行都过了。

大厂不太好过，但中小厂目前Agent岗位处于风口期，面试相对好面。

# 二、为什么转 Agent

“再干土木就饿死了”——这是很多传统行业程序员的真实写照。

Agent开发目前处于风口期，市场上做Agent的人很少，学完可以直接冲中大厂。

求职市场上Agent开发经验要求最多2年，外加5年左右后端经验，就可以做Agent架构师。

学得好的甚至可以直接面Agent架构师岗位。

Boss直聘上目前还没有专门的“Agent应用开发”Title，大多挂在Java主方向下，简历写“2年后端 + 2年Agent开发”即可，没有外包。

# 三、学习路线

## 3.1 前置准备

**Python：** 花两天过一下Python基础就够了，Python上手很快。

**TypeScript：** 目前做Agent最多的是Python和TS，Python框架多，TS语言比较适合做Agent，自研很多都是TS（比如Cursor/CC就是用TS写的）。

但TS没学也问题不大，Python生态已经足够。

## 3.2 核心课程推荐

**首选：** 吴恩达老师的课（YouTube上免费），如果能自己整理学习路线的话。

**次选：** 尚硅谷的Agent课（闲鱼49元可以买到），目前有点过时了，但最近有新项目更新。

置顶里有分享人学的尚硅谷课程目录，详见下方课程学习建议。

**不推荐：** 黑马的Agent课，分享人明确表示“不要看”，黑马现在不行了。

## 3.3 尚硅谷课程学习建议（26年1月开课版本）

| 序号 | 课程模块 | 学习建议 |
|---|---|---|
| 01 | 大模型概述 / 低代码智能体开发 / 企业级LM部署 | 2.0倍速看，如果语速跟得上的话 |
| 02 | LangChain / LangGraph / MCP | 1.2倍速看，基本函数名记住，基本概念原理记住 |
| 03 | 项目 - 掌柜问数 | 1.2倍速看，记住整体流程，各种中间件数据库等作用 |
| 04 | 多智能体项目 - DeepAgents | 1.2倍速看，记住整体流程，多Agent交互方式 |
| 05 | 项目 - 掌柜智库 | 1.0倍速看，记住整体流程，记住各个模块主要功能，各个中间件数据库等作用，看一下里面用到的技术栈的官网 |
| 06 | 微调 | 可看可不看，看自己选择（面试问得很少） |
| 07 | 项目 - 智能客服系统 | 如果作为项目写到简历里就好好看；不写到简历的话知道整体架构即可 |
| 08 | 线上大模型智能体班就业指导 | 准备面试之前看 |
| 09 | 大厂老师直播课资料 | 不用看，想看也行 |

整体学习周期大约2个月。

学完之后再看看 Loop Agent，就知道Agent是干啥的了。

## 3.4 额外推荐

**Harness Engineering：** 兄弟们可以看看这个，理解Agent的本质。

**多Agent通信协议（A2A）：** 也要学，这是进阶内容。

# 四、技术栈概览

## 4.1 主流框架

**LangChain / LangGraph：** 最常规的Agent框架，Python生态。

**自研框架：** 很多公司完全自研，分享人自己就用Java写了一个工作流Agent框架（开源，见附件）。

**Hermes Agent：** 通用Agent框架。

**Coding Agent：** 如Codex、Cursor（CC）等。

## 4.2 核心技术概念

面试主要考这几个方向：

**大模型相关：** LLM基础、Prompt Engineering。

**工具相关：** Function Calling、MCP（Model Context Protocol）。

**记忆相关：** 短期记忆、长期记忆、向量存储。

**模型边界和护栏：** 安全控制、输出约束。

Agent可以理解为“驾驭LLM的 + 外加LLM缺少的能力”。

Agent基本上一个月冒出一个新概念，在职的话没有太多时间关注新东西，所以面试也不会问得太深。

## 4.3 可落地的Agent架构

目前Agent可落地的架构不多，主要有这几个方向：

- DeepResearch（深度研究）
- 智能客服
- 内部提效
- 云平台
- Coding Agent
- RAG / NL2SQL
- 多Agent协作（DAG工作流）

## 4.4 技术栈详情（来自简历）

**AI / Agent 方向：** LangChain、LangGraph、DeepAgents、RAG、GraphRAG、ReAct Agent、Function Calling、MCP、Prompt、SSE / WebSocket、Milvus、Elasticsearch、Qdrant、Neo4j、MongoDB、MinIO、BGE-M3、BGE Reranker、Hybrid Search、RRF、rerank。

**后端方向：** Java、Spring Boot、Spring Cloud、MyBatis-Plus、FastAPI、Pydantic、SQLAlchemy、MySQL、Redis、RabbitMQ、Docker、Jenkins、Linux、Nginx。

# 五、项目经验要点

分享人的项目经验（写在简历上的）：

## 项目一：AI Agent 工作流平台（2024/03 - 至今）

基于Spring Boot + Spring AI + Milvus + Elasticsearch + Redis + MySQL，核心是自研的DAG Agent工作流引擎。

支持20+节点类型（Start、Input、Output、Condition、LLM、Agent、RAG、Tool、Code、SQL、SubWorkflow等），使用CompletableFuture / ForkJoinPool实现并行执行，GraphState管理LLM上下文。

集成ReAct Agent和MCP协议，RAG使用Milvus + Elasticsearch双引擎RRF融合 + rerank排序，WebSocket / SSE实现token级流式输出和trace追踪。

成果：50+企业用户，15个团队，3个月落地，CPU利用率从30%提升到80%，响应时间200ms以内，支持100MB Excel文件处理（EasyExcel）。

## 项目二：PDF 智能解析（2024/09 - 2025/04）

FastAPI + Pydantic后端，MinIO存储，MinerU将PDF转Markdown，chunk后用BGE-M3生成dense vector / sparse vector存入Milvus。

支持Markdown VLM理解，query rewrite / HyDE增强检索，RRF + BGE Reranker重排，Neo4j知识图谱。

chunk + rerank Top5方案将准确率从15%提升到25%。

## 项目三：NL2SQL（2024/04 - 2024/08）

基于schema理解自动生成SQL，Qdrant + Elasticsearch做语义检索，SQL Prompt + EXPLAIN验证SQL正确性。

覆盖200+ SQL场景，准确率从80%提升到90%。

# 六、面试经验

## 6.1 面试节奏

投了两天，面了2周，平均一天3-4个面试。

全是主动找过来的，Agent开发目前处于风口期。

## 6.2 面试要点

**不会问大模型底层：** 做Agent开发面试不会问大模型底层原理，微调也问得很少。

**重点问架构：** 面试Agent岗位主要问架构问题，项目基本都是vibe coding做的，但底层架构要自己能讲清楚。

**会吹就行：** 分享人直言“现在外面面试官都是水货，会吹牛逼就好了”，当然底层架构要真写过。

**项目可以包装：** 项目基本上对内提效的，想咋忽悠咋忽悠，但架构要能自圆其说。

## 6.3 简历建议

最好自己搞两个Agent项目，整一个牛逼的Agent项目。

简历写“2年后端 + 2年Agent开发”。

大厂不太好忽悠，但中小厂机会很多。

小厂做Agent的很少，学完直接冲中大厂。

# 七、关键认知

1. **Agent做出来很简单，做好很难** ——落地非常难，但面试不需要你落地过。
2. **风口期抓紧转** ——主要面试好面，现在没有那么卷。
3. **Agent不如我** ——分享人自信地表示，Agent本质上就是驾驭LLM，有后端经验的人转过来有优势。
4. **开源精神** ——分享人开源了自己的工作流Agent框架，“兄弟好我也开心”。
5. **实名上网** ——分享人表示自己是实名上网，内容可信。

# 八、附件

## 附件一：尚硅谷 AI 大模型智能体课程（百度网盘）

**26版AI大模型智能体 + 往期AI大模型**

链接：  
https://pan.baidu.com/s/1bCzCn2wDc7oy2doTQiAXrQ?pwd=8888

提取码：8888

说明：复制到百度网盘APP即可获取。

包含9个模块，从大模型概述到就业指导全覆盖。

闲鱼49元也可购买。

## 附件二：自研开源 Agent 工作流框架

GitHub 地址：  
https://github.com/wangyingqi-x/workflow-showcase-platform/

语言：Java

说明：群友们想学图编排和Agent的可以看看。

这是一个自研的工作流Agent框架，支持DAG编排、ReAct Agent、MCP协议、RAG检索等核心能力。

开源项目，欢迎Star。

## 附件三：简历模板

文件名：大模型应用开发.pdf（442.8K）

说明：分享人实际使用的简历，已通过希音、360、汇丰银行等公司筛选。

简历突出AI Agent项目经验，技术栈覆盖LangChain/LangGraph/RAG/GraphRAG/ReAct/MCP等主流方向。

可以参考其项目描述写法和技术栈组织方式。

