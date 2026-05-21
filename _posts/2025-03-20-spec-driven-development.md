---
layout: post
title: "Spec-Driven Development: Writing the What Before the How"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - engineering
  - AI
  - agentic
  - spec-driven
favorite: false
commentIssueId: 98
refimage: '/static/agentic_ai/spec_driven_dev.svg'
---

In the age of AI-assisted coding, the most powerful thing you can write
isn't code — it's a **specification**. Spec-driven development (SDD) is
a methodology where the specification is the primary artifact, and code
is a derived output. This approach becomes particularly powerful when
AI agents are the ones generating the code from your specs.

## What is Spec-Driven Development?

Spec-driven development is an approach where you start by writing a clear,
detailed specification of what a system should do — its inputs, outputs,
behaviors, constraints, and edge cases — before any code is written. The
specification serves as both the blueprint and the acceptance criteria.

![](/static/agentic_ai/spec_driven_dev.svg)

This isn't a new idea. Formal methods, design-by-contract, and even
test-driven development (TDD) share DNA with SDD. What's new is the
execution model: **AI agents can now consume specifications and produce
working implementations**.

## The Traditional Approach vs. SDD

### Traditional Development

```
Idea → Code → Test → Fix → Deploy
```

In traditional development, the specification often lives informally in
the developer's head, in scattered Jira tickets, or in meetings. The code
becomes the de facto specification, and understanding the system means
reading the implementation.

### Spec-Driven Development

```
Idea → Spec → Generate → Verify → Deploy
```

In SDD, the specification is explicit, structured, and machine-readable.
It captures not just *what* the system does but *why* it does it, what the
constraints are, and how success is measured.

## Anatomy of a Good Specification

A well-written specification for SDD includes:

### 1. Purpose and Context

```markdown
## Purpose
A REST API endpoint that validates user email addresses
against a corporate directory and returns their department
and manager information.

## Context
This endpoint will be consumed by the onboarding flow
in the HR dashboard. It must handle 500 concurrent requests
and respond within 200ms at p95.
```

### 2. Interface Contract

```markdown
## API Contract
POST /api/v1/users/lookup

Request:
  - email: string (required, valid email format)

Response 200:
  - user_id: string
  - department: string
  - manager_email: string
  - active: boolean

Response 404:
  - error: "user_not_found"

Response 422:
  - error: "invalid_email_format"
```

### 3. Behavioral Requirements

```markdown
## Behaviors
- Email matching is case-insensitive
- Inactive users are returned with active: false
- Results are cached for 5 minutes
- Rate limiting: 100 requests per minute per client IP
```

### 4. Edge Cases and Constraints

```markdown
## Edge Cases
- Users with multiple email aliases should resolve to
  their primary email
- Distribution lists should return 404, not a user record
- If the directory service is unavailable, return 503
  with a retry-after header
```

## Why SDD Works Well with AI Agents

The combination of SDD and AI agents creates a powerful feedback loop:

1. **Specifications are natural language**: AI models excel at understanding
   and acting on natural language descriptions. A well-written spec is
   essentially a sophisticated prompt.

2. **Verification is built in**: Because the spec defines expected behaviors,
   you can automatically verify whether the generated code meets the spec.

3. **Iteration is cheap**: If the generated code doesn't match the spec,
   you can refine the spec or ask the agent to try again — both are
   faster than manual coding.

4. **Knowledge is preserved**: The spec persists as documentation even
   after the code is generated. When you need to modify the system,
   you update the spec first.

## Practical Tips for Writing Specs

- **Be specific about boundaries**: "Fast" isn't a spec. "Responds in
  under 200ms at p95 under 500 concurrent connections" is.
- **Include examples**: Concrete input/output examples eliminate ambiguity
  more effectively than abstract descriptions.
- **State what should NOT happen**: Negative requirements ("must not
  expose internal IDs") prevent a class of bugs that positive
  requirements miss.
- **Version your specs**: Specs evolve with your system. Track changes
  the same way you track code changes.
- **Start small**: You don't need to spec an entire system. Start with
  a single endpoint, a single component, or a single workflow.

## The Spec as a Living Document

One of the most powerful aspects of SDD is that the specification becomes
a living document that evolves with the system. Unlike comments that rot
or documentation that diverges from reality, a spec that is actively
used to generate and verify code stays current by necessity.

When a bug is found, the first question isn't "what went wrong in the
code?" but "what's missing from the spec?" This reframing shifts the
team's attention from implementation details to system behavior — which
is ultimately what matters.

## Getting Started

If you're interested in adopting SDD, start with your next feature:

1. Write a one-page spec before touching any code
2. Share it with your team for review
3. Use an AI agent to generate the implementation
4. Verify the output against the spec
5. Iterate on both the spec and the implementation

The initial investment in writing a good spec pays dividends throughout
the entire lifecycle of the feature — from implementation to testing
to maintenance to eventual replacement.

Spec-driven development isn't about replacing developers with AI. It's
about elevating the developer's role from code writer to system designer.
And that's a much more valuable position to be in.
