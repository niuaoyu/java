MCP（Model Context Protocol）是一个**标准化的进程间通信协议**，基于 JSON-RPC

流程：
Claude Code (MCP Client) <--JSON-RPC(stdio 或 HTTP/SSE)--> MCP Server (独立进程/服务)