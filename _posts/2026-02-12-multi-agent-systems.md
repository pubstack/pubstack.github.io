---
layout: post
title: "Multi-Agent Systems: When One AI Isn't Enough"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - engineering
  - AI
  - agentic
  - architecture
favorite: false
commentIssueId: 103
refimage: '/static/agentic_ai/multi_agent_systems.svg'
---

A single AI agent can write a function, fix a bug, or answer a question.
But what about redesigning an entire microservice architecture? Or
migrating a million-line codebase to a new framework? These tasks exceed
what any single agent can handle — not because the model isn't smart
enough, but because the context is too large, the concerns too diverse,
and the coordination too complex. Enter **multi-agent systems**.

## Why Multiple Agents?

The case for multi-agent systems comes down to three factors:

![](/static/agentic_ai/multi_agent_systems.svg)

### 1. Context Window Limitations

Even the largest language models have finite context windows. A single
agent working on a large codebase quickly runs out of context space.
Multiple agents can each focus on a subset of the problem, keeping
their context focused and relevant.

### 2. Specialization

Different tasks require different expertise. A security audit agent
needs different knowledge than a performance optimization agent. By
specializing agents, you can give each one a focused system prompt,
relevant tools, and domain-specific knowledge.

### 3. Parallelism

Some tasks are embarrassingly parallel. Reviewing 50 files doesn't
require reviewing them in sequence — 50 agents can review them
simultaneously, reducing wall-clock time from hours to minutes.

## Multi-Agent Topologies

### Flat Collaboration

All agents are peers with equal authority. They communicate through
shared state (a file system, a database, a message queue) and coordinate
through conventions rather than hierarchy.

```
Agent A ←→ Agent B ←→ Agent C
    ↕                    ↕
         Shared State
```

**Pros**: Simple, no single point of failure.
**Cons**: Coordination is difficult, agents can conflict.

### Hierarchical (Supervisor-Worker)

A supervisor agent manages a team of worker agents. The supervisor
decomposes the task, assigns sub-tasks, collects results, and handles
conflicts.

```
        Supervisor
       /    |     \
  Worker  Worker  Worker
```

**Pros**: Clear coordination, handles complex tasks well.
**Cons**: Supervisor is a bottleneck and single point of failure.

### Assembly Line

Each agent handles one phase of a multi-phase process. Output from
one agent feeds directly into the next.

```
Agent A → Agent B → Agent C → Agent D
(Plan)   (Implement) (Test)  (Review)
```

**Pros**: Clear responsibilities, easy to debug.
**Cons**: Rigid, slow if phases can't overlap.

### Debate / Adversarial

Multiple agents tackle the same problem independently, then a judge
agent evaluates their solutions and picks the best one (or synthesizes
a hybrid).

```
Agent A ──┐
Agent B ──┼──→ Judge → Result
Agent C ──┘
```

**Pros**: Reduces individual bias, catches more edge cases.
**Cons**: Expensive (multiple full implementations), needs a good
judge.

## Communication Patterns

How agents communicate determines the system's reliability and
scalability:

### Message Passing

Agents send structured messages to each other. Each message includes
the sender, recipient, content, and metadata. This is the most
flexible pattern but requires a message routing layer.

```json
{
  "from": "planner",
  "to": "implementer",
  "type": "task_assignment",
  "content": {
    "task": "implement_rate_limiter",
    "spec": "...",
    "constraints": ["must be thread-safe", "O(1) lookup"]
  }
}
```

### Shared Workspace

Agents read from and write to a shared workspace (typically the file
system). Coordination happens through file locks, conventions, or a
task board.

This is how systems like Claude Code work — the agent reads files,
makes changes, runs tests, and the file system serves as both the
communication channel and the state store.

### Event-Driven

Agents publish events (e.g., "file modified", "test failed",
"review complete") to a central bus. Other agents subscribe to
relevant events and react accordingly.

```
Agent A publishes: FileModified("src/auth.py")
Agent B (test runner) reacts: runs tests for auth module
Agent C (reviewer) reacts: reviews changes to auth.py
```

## Real-World Example: AI-Powered Code Migration

Consider migrating a Python 2 codebase to Python 3. Here's how a
multi-agent system might handle it:

**Supervisor Agent**: Scans the codebase, identifies all Python 2
files, creates a task board, and monitors progress.

**Analyzer Agents** (3-5): Each takes a batch of files and identifies
Python 2 patterns that need migration (print statements, unicode
handling, dict methods, etc.).

**Migrator Agents** (5-10): Each takes assigned files and performs
the actual migration, applying Python 3 patterns.

**Test Agent**: Runs the test suite after each batch of changes,
reports failures back to the supervisor.

**Conflict Resolver**: When two migrator agents change the same
file or when changes in one file break another, this agent detects
and resolves the conflict.

**Report Agent**: Generates a migration progress report with
statistics on files migrated, tests passing, and remaining work.

This system can migrate thousands of files in parallel while
maintaining consistency and catching regressions.

## Challenges

### Coordination Overhead

More agents means more communication, more state management, and
more potential for miscommunication. The overhead can dominate
the actual work for small tasks.

### Conflict Resolution

When agents modify the same files or make contradictory decisions,
you need a strategy for resolution. Options include:
- Locking (pessimistic concurrency)
- Merge with conflict detection (optimistic concurrency)
- Supervisor arbitration

### Cost

Running multiple agents in parallel multiplies API costs. A
10-agent system costs roughly 10x a single agent. This is
acceptable for large, high-value tasks but prohibitive for
routine work.

### Debugging

When something goes wrong in a multi-agent system, tracing the
cause requires understanding the interactions between agents. Good
logging, tracing, and replay capabilities are essential.

## When to Use Multi-Agent Systems

Use multi-agent systems when:

- The task is too large for a single context window
- Different parts of the task require different expertise
- Sub-tasks can be parallelized for speed
- You need redundancy or adversarial verification

Stick with a single agent when:

- The task fits in one context window
- The task is sequential by nature
- The coordination overhead would exceed the time savings
- Cost is a primary constraint

Multi-agent systems are a powerful tool in the agentic AI toolkit,
but they're not always the right tool. Start with a single agent
and scale to multiple agents only when the problem demands it.
