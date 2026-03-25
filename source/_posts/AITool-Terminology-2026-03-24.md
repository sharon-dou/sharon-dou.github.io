---
title: AI Terminology
date: 2026-03-24 14:20:28
tags: 
    - AI
    - Tool
categories: 
    - Tool
cover: https://tenor.com/zh-CN/view/dictionary-simpsons-hmmmm-homer-simpson-gif-15407882.gif
---

| Terminology | Description |
|:---------|:--------|
| LLM | Large Language Model |
| Token | The basic processing unit of the LLM <br> &nbsp;&nbsp;&nbsp; - 1 token ~ 0.75 English word, ~ 1-2 Chinese word|
| Context | The infomation available to the model for the current task (its working memory) |
| Context Window | The mamimum #tokens the model can process at once <br> &nbsp;&nbsp;&nbsp; - e.g., 1M Claude Opus 4.6, 1.05M GPT 5.4 |
| RAG | Retrieval Augmented Generation <br> &nbsp;&nbsp;&nbsp; - A method that augments the model with external knowledge retrieved at runtime |
| Prompt | The input to the model, including intructions and task-specific context |
| System Prompt | A high-level instruction that defines the model's behavior, persona, and constraints (typically confiugred backend, not visible to user) |
| User Prompt | The user-provided input that specifies the task |
| Tool | External function that model can invoke to perform actions <br> &nbsp;&nbsp;&nbsp; - e.g., user, LLM, Tool (weather query tool), Platform (MCP host, e.g., Cline) <br> &nbsp;&nbsp;&nbsp; - LLM: choose tool & summarize <br> &nbsp;&nbsp;&nbsp; - MCP server: query weather <br> &nbsp;&nbsp;&nbsp; - MCP host: connect E2E workflow |
| MCP | Model Context Protocol <br> &nbsp;&nbsp;&nbsp; - A standard protocol for tool integration (MCP server → MCP host) <br> &nbsp;&nbsp;&nbsp; - [MCP Official Doc](https://modelcontextprotocol.io/docs/getting-started/intro)|
| Agent | An automous system that plans and invokes tools to fulfill user requests <br> &nbsp;&nbsp;&nbsp; - e.g., Claude Code, Codex, Gemini CLI |
| Agent Skill | Instructions that defines how the agent should behave, including task requirements and output format |

# Reference
- https://www.youtube.com/watch?v=7qO8-kx3gW8
