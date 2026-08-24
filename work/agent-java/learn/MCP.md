MCP（Model Context Protocol）是一个**标准化的进程间通信协议**，基于 JSON-RPC

流程：
Claude Code (MCP Client) <--JSON-RPC(stdio 或 HTTP/SSE)--> MCP Server (独立进程/服务)


**MCP（Model Context Protocol，模型上下文协议）** 是一个**标准化协议**，用来让 AI 模型（如 Claude、GPT）和外部工具/数据源进行通信。

用人话说就是：**给 AI 模型装了一个“万能插头”**，让它可以按统一的方式调用各种外部工具（数据库、文件系统、API 等）。

## 为什么要 MCP？

在没有 MCP 之前，你要让 AI 调用一个工具（比如查天气 API），需要写**定制代码**：

```python
# 每个工具都要单独写一套集成代码
if tool == "weather":
    call_weather_api()
elif tool == "database":
    call_database()
elif tool == "filesystem":
    call_filesystem()
```

问题：每加一个新工具，就要改代码、重部署，非常麻烦。

**有了 MCP**：工具提供方（Server）和 AI（Client）都遵循同一个协议，**加新工具就像插拔 U 盘一样简单**，AI 自动发现并使用，无需改代码。

---

## MCP 客户端和服务端之间的关系

```mermaid
flowchart LR
    subgraph Client["MCP 客户端"]
        AI[AI 模型<br>如 Claude / GPT]
    end

    subgraph Servers["MCP 服务端（可插拔）"]
        S1[文件系统服务]
        S2[数据库服务]
        S3[GitHub 服务]
        S4[天气 API 服务]
    end

    Client <-->|MCP 协议<br>JSON-RPC| Servers
```

- **MCP 客户端**：AI 模型那一侧，负责向服务端发送请求（"帮我读文件"、"查数据库"）。
- **MCP 服务端**：工具/数据源那一侧，暴露能力（"我能读文件"、"我能查天气"），接收并执行客户端请求。

**一个客户端可以连接多个服务端，一个服务端也可以服务于多个客户端。**

## 它们怎么通信？（两种方式）

### 方式一：进程内通信（stdio）

**客户端启动服务端作为一个子进程**，通过标准输入/输出（stdin/stdout）交换消息。

```mermaid
flowchart LR
    AI[AI 模型] -->|启动子进程| Server[MCP 服务端]
    AI <-->|stdin/stdout<br>JSON-RPC 消息| Server
```

特点：
- 简单、轻量，不需要网络
- 适用于**同一个机器**上的本地工具（如文件系统、本地数据库）

MCP的两种通信！
1. stdio(标准输入输出)：主要用在本地服务上，操作你本地的软件或者本地的文件。比如Blender这种就只能用Stdio因为他没有在线服务。是MCP默认通信方式
	1. 优点：这种方式适用于客户端和服务器在同一台机器上运行的场景，简单。stdio模式无需外部网络依赖，通信速度快，适合快速响应的本地应用。可靠性高，且易于调试
	2. 缺点：Stdio的配置比较复杂，我们需要做些准备工作，你需要提前安装需要的命令行工具。stdio模式为单进程通信，无法并行处理多个客户端请求，同时由于进程资源开销较大，不适合在本地运行大量服务。（限制了其在更复杂分布式场景中的使用）
2. SSE(Server-Sent Events)：主要用在远程通信服务上，这个服务本身就有在线的API，比如访问你的谷歌邮件，天气情况等。SSE方式适用于客户端和服务器位于不同物理位置的场景。适用于实时数据更新、消息推送、轻量级监控和实时日志流等场景，对于分布式或远程部署的场景，基于HTTP和SSE的传输方式则更为合适。


### 方式二：网络通信（HTTP/WebSocket）

**客户端和服务端通过网络连接**，比如通过 HTTP 或 WebSocket 通信。

```mermaid
flowchart LR
    AI[AI 模型] <-->|HTTP / WebSocket<br>JSON-RPC 消息| Server[MCP 服务端]
```

特点：
- 支持**远程部署**（服务端可以跑在另一台机器上）
- 适合云端工具、微服务架构

## MCP 消息格式（JSON-RPC）

所有 MCP 消息都使用 **JSON-RPC 2.0** 格式。举个例子：

**客户端请求：**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": { "path": "/tmp/test.txt" }
  }
}
```

**服务端响应：**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": "Hello, World!"
  }
}
```

这种格式的好处是：
- **语言无关**：任何语言都能生成/解析 JSON
- **有请求 ID**：可以区分"哪个请求对应哪个响应"
- **标准化**：所有 MCP 服务端都遵守同一套规范

## 核心概念：Tool（工具）

MCP 里最重要的概念是 **Tool**。服务端注册 Tool，客户端调用 Tool。

**服务端注册 Tool：**
```json
{
  "tools": [
    {
      "name": "read_file",
      "description": "读取指定路径的文件内容",
      "inputSchema": {
        "path": { "type": "string", "description": "文件路径" }
      }
    }
  ]
}
```

**客户端调用 Tool：**
```json
{
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": { "path": "/tmp/test.txt" }
  }
}
```


MCP的C/S架构， 5个核心概念，MCP遵循客户端-服务器架构（client-server）
1. MCP主机(MCP Hosts)：作为运行MCP的主应用程序，例如Claude Desktop、Cursor、Cline或AI工具。为用户提供与LLM交互的接口，同时集成MCPClient以连接MCPServer。
2. MCP客户端(MCP Clients)
3. MCP服务器(MCP Servers)
4. 本地资源(Local Resources)
5. 远程资源(Remote Resources)

mcp hosts可以理解就是这些ai工具，这些工具包含llm+tools 其中tools里有mcp clients 也就是客户端，具体实现的地方在mcp 




## 一句话总结

> **MCP 是一个让 AI 模型（客户端）和外部工具（服务端）之间按统一格式（JSON-RPC）通信的标准化协议。通信方式可以是进程内管道（stdio）或网络（HTTP/WebSocket），核心是让“加新工具”像插拔 U 盘一样简单。**




