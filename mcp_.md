# 模型上下文协议 MCP
## 概念
采用客户端-服务器的设计架构

由三个部分构成：
- 主机（Host）：比如 Cursor、Claude Desktop
- MCP Client：运行于主机内部，相当于插件。它仅负责以正确的格式和协议与各种服务提供商通信。处理通信协议、数据格式转换和连接管理等底层问题
- MCP Server

一个 Host 中可以有多个 Client ，那么怎么知道要用哪个呢？

## 推荐阅读
- [MCP 官方的 python-sdk](https://github.com/modelcontextprotocol/python-sdk)
- [MCP world](https://www.mcpworld.com/mcp): 可用于 pdf 解析、知识图谱构建
  - [pdf 转换为 markdown](https://www.mcpworld.com/zh/detail/c3b50c4a44b9b96bd3a6c0a59d21b4f1)
## 疑问
MCP 它扩展了大模型，那 MCP + LLM 相当于智能体吗，它和智能体之间的差距在哪？
MCP 的作用和 RAG 类似，都是让 LLM 获取外部知识，避免它仅通过自身的预训练参数形式知识来推理
- RAG
- MCP
- Agent

1. MCP 是 Agent 工具调用的统一协议。它旨在标准化大型语言模型（LLM）与外部工具和服务的交互方式
2. 原先的痛点是：每个 LLM 的调用结构都不一样，而每个工具的 API 也不同。这导致它们对接的时候不统一，需要专门为了适配而调整
![](https://pic-gino-prod.oss-cn-qingdao.aliyuncs.com/zhangli2025/20251027004351179-paste.png)

## 本地开发 MCP Server
目前MCP Server 官方给出了两种语言的SDK，NodeJS 以及 Python
创建一个项目
```
pdm new mcp_demo
pdm add "mcp[cli]"
```
`MCP Inspector`是一个 MCP 官方提供的 MCP Server 调试和测试工具
```
uv run mcp dev mcp_server1.py
```
MCP Server 中一般存在的 3 种操作
- Resources：加载环境中的资源到 LLM 上下文中
- Tools：该函数执行计算操作
- Prompts
