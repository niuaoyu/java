
Human-in-the-Loop 怎么实现？ 
LangGraph 的理解？之上的 Deep Agents 封装框架？ 上下文管理的手段？除了裁剪压缩还有啥？ 
项目介绍 
RAG 介绍 
重排序介绍 
项目中你主要负责啥模块？ 
A2A 协议了解吗？ 
记忆管理怎么用的？ 
大模型怎么预测 token 的？temperature 采样？ 

【翻车一：HITL】 答：hil是啥，不知道是Human-in-the-Loop。。。 
【翻车二：Deep Agents】 问：LangGraph 之上的 Deep Agents 封装框架？ 我：LangGraph 熟，封装框架……不清楚。 复盘：LangGraph 是引擎（节点/状态/边），Deep Agents 是上层封装——主管式多 Agent 编排，不用自己写循环。 
【翻车三：上下文管理】 回答了压缩裁剪应用，追问还有哪些方法，复盘：还能答：摘要、记忆外置到向量库、分层记忆、关键信息提取、Prompt Cache、路由分流。。。 
【翻车四：A2A】 问：A2A 协议了解吗？ 我：知道是 Agent 互操作协议，但聊到金融场景调用就接不上了。 复盘：A2A = Agent 连 Agent（MCP 是 Agent 连工具）。金融的点：跨机构协作、任务委托、合规留痕。我停在概念层，没到场景层。 
【翻车五：token 预测】 问：大模型怎么预测下一个 token？采样知道吗？ 我：只知道是概率预测的分布。 复盘：自回归 next-token 预测：softmax 出概率分布 → temperature 缩放 logits → 采样。



