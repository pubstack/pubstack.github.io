---
layout: post
title: "Tool Use in AI Agents: Extending LLM Capabilities"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - engineering
  - AI
  - agentic
favorite: false
commentIssueId: 101
refimage: '/static/agentic_ai/tool_use_agents.svg'
---

Large language models are remarkable at understanding and generating text,
but text alone can't deploy a server, query a database, or fix a failing
build. **Tool use** is the mechanism that transforms an LLM from a
conversational partner into an autonomous agent capable of acting in the
real world. This post explores how tool use works, why it matters, and
how to design effective tool interfaces for AI agents.

## What is Tool Use?

Tool use — also called function calling — allows an AI model to invoke
external functions during a conversation. Instead of generating a text
response, the model outputs a structured request to call a specific tool
with specific parameters. The tool executes, returns a result, and the
model continues its reasoning with the new information.

![](/static/agentic_ai/tool_use_agents.svg)

A typical tool use flow looks like this:

```
User: "What's the current CPU usage on prod-server-1?"

Model thinks: I need to check server metrics.
Model calls: get_server_metrics(host="prod-server-1", metric="cpu")
Tool returns: {"cpu_percent": 73.2, "timestamp": "2025-10-05T14:30:00Z"}

Model responds: "prod-server-1 is currently at 73.2% CPU usage."
```

The model decides *when* to use a tool, *which* tool to use, and *what
parameters* to pass — all based on the conversation context and the
tool descriptions it has been given.

## Categories of Tools

### Information Retrieval

Tools that fetch data the model doesn't have access to:

- **Web search**: Finding current information beyond the training cutoff
- **Database queries**: Looking up records, aggregations, and relationships
- **File reading**: Accessing source code, configuration files, and logs
- **API calls**: Retrieving data from external services

### Environment Interaction

Tools that modify state or take actions:

- **File writing/editing**: Creating and modifying source code
- **Shell execution**: Running commands, scripts, and build processes
- **API mutations**: Creating resources, updating records, deploying services
- **Git operations**: Committing, branching, and managing version control

### Computation

Tools that perform calculations the model can't reliably do in its head:

- **Code execution**: Running Python, JavaScript, or other code in a sandbox
- **Mathematical computation**: Precise arithmetic, statistics, and modeling
- **Data transformation**: Parsing, filtering, and reformatting data

## Designing Good Tool Interfaces

The quality of an agent's tool use depends heavily on how the tools are
designed. Here are principles for creating effective tool interfaces:

### 1. Clear, Descriptive Names

```json
// Good
{"name": "search_codebase", "description": "Search for files and symbols in the project repository"}

// Bad
{"name": "search", "description": "Searches for things"}
```

The model uses the name and description to decide when to use a tool.
Vague descriptions lead to incorrect tool selection.

### 2. Well-Typed Parameters

```json
{
  "name": "create_issue",
  "parameters": {
    "title": {"type": "string", "description": "Issue title, max 100 chars"},
    "body": {"type": "string", "description": "Detailed description in markdown"},
    "labels": {"type": "array", "items": {"type": "string"}, "description": "Labels to apply"},
    "priority": {"type": "string", "enum": ["low", "medium", "high", "critical"]}
  }
}
```

Constrained parameter types (enums, bounded ranges, required fields)
reduce the chance of malformed tool calls.

### 3. Informative Return Values

Return enough context for the model to reason about the result:

```json
// Good: includes context
{"status": "success", "file": "src/auth.py", "lines_changed": 15, "warnings": ["unused import on line 3"]}

// Bad: minimal information
{"status": "ok"}
```

### 4. Graceful Error Handling

Tools should return structured errors that help the model self-correct:

```json
{"error": "file_not_found", "message": "src/auth.py does not exist", "suggestion": "Did you mean src/authentication.py?"}
```

## The Tool Selection Problem

One of the biggest challenges in tool use is **tool selection** — choosing
the right tool for the job when many tools are available. Models must
consider:

- Which tools are relevant to the current task?
- What's the most efficient sequence of tool calls?
- When should the model reason internally vs. call a tool?

Research shows that models perform better with **focused toolsets** (5-15
tools relevant to the current task) rather than massive toolsets (100+
tools). If your agent needs many tools, consider organizing them into
categories and loading only the relevant category based on the task.

## Security Considerations

Tool use introduces security concerns that don't exist in pure text
generation:

- **Injection attacks**: User input might contain instructions that
  manipulate tool calls (e.g., injecting shell commands)
- **Permission scope**: An agent should have the minimum permissions
  needed for its task — not root access to everything
- **Data exfiltration**: An agent with read access to sensitive data
  and write access to external services could leak information
- **Unintended side effects**: A tool call that modifies production
  data can't be undone by regenerating the model's response

Design your tool layer with the principle of least privilege. Sandbox
execution environments. Log every tool call. And always keep a human
in the loop for high-impact actions.

## The Future of Tool Use

Tool use is evolving rapidly:

- **Self-generated tools**: Agents that write their own tools when
  existing ones don't fit the task
- **Tool learning**: Models that improve their tool use strategies
  through experience
- **Standardization**: Protocols like MCP (Model Context Protocol)
  that standardize how tools are described and invoked across
  different platforms
- **Composable tools**: Micro-tools that agents combine dynamically
  to handle novel situations

As tool ecosystems mature, the distinction between "what the model
knows" and "what the model can do" will continue to blur. The most
capable agents won't be the ones with the largest models — they'll
be the ones with the best tools.
