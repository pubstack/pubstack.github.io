---
layout: post
title: "From User Stories to AI Prompts: Bridging Agile and Agentic Development"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - engineering
  - AI
  - agentic
  - agile
  - spec-driven
favorite: false
commentIssueId: 105
refimage: '/static/agentic_ai/agile_meets_agentic.svg'
---

Agile development transformed how teams build software by emphasizing
iteration, user stories, and continuous delivery. Now, agentic AI is
transforming how code gets written within those iterations. But how do
these two paradigms fit together? Can user stories serve as specifications
for AI agents? Where does agile end and agentic begin? This post explores
the convergence of agile methodology and AI-assisted development.

## The Agile-Agentic Connection

At its core, agile development is about delivering value incrementally
through short feedback loops. Agentic AI accelerates these loops by
compressing the time between "here's what I want" and "here's a working
implementation."

![](/static/agentic_ai/agile_meets_agentic.svg)

The alignment is natural:

| Agile Concept | Agentic Equivalent |
|---------------|-------------------|
| User story | Agent prompt / specification |
| Sprint | Agent session |
| Acceptance criteria | Verification tests |
| Code review | Agent output review |
| Retrospective | Prompt refinement |

But there are important differences. Agile assumes human developers
who understand context implicitly. AI agents need explicit context.
This gap is where spec-driven development bridges the two worlds.

## User Stories as Agent Specifications

A traditional user story follows the format:

```
As a [user type],
I want [functionality],
So that [benefit].
```

This format works well for communicating intent between humans, but it's
too ambiguous for AI agents. Consider:

```
As a user,
I want to reset my password,
So that I can regain access to my account.
```

A human developer reads this and brings years of context: they know
about email verification, token expiration, password strength rules,
and rate limiting. An AI agent needs these details spelled out.

### Enriched User Stories

To bridge the gap, enrich user stories with the specifics an agent needs:

```markdown
## User Story
As a registered user who has forgotten their password,
I want to reset it via email verification,
So that I can regain access to my account.

## Acceptance Criteria
- User enters email and receives a reset link within 30 seconds
- Reset token expires after 1 hour
- Token is single-use (invalidated after first use)
- New password must meet strength requirements:
  minimum 8 chars, 1 uppercase, 1 number, 1 special char
- Failed reset attempts are rate-limited: max 3 per hour per email
- Successful reset invalidates all existing sessions

## Technical Constraints
- Token: cryptographically random, 256 bits, stored as SHA-256 hash
- Email: sent via existing SendGrid integration
- Database: update users table, add password_reset_tokens table
- API: POST /auth/forgot-password, POST /auth/reset-password

## Edge Cases
- What if the email doesn't exist? Return 200 (no information leak)
- What if the user requests multiple resets? Only latest token valid
- What if the token is tampered with? Return 400, log attempt
```

This enriched story preserves the agile format while providing enough
detail for an AI agent to generate a correct implementation.

## Sprint Planning with AI Agents

Sprint planning changes when AI agents are part of the team:

### Estimation

Story points become less about implementation effort and more about
specification effort and review effort. A complex feature might take
2 hours to specify, 5 minutes for the agent to implement, and 1 hour
to review — compared to 2 days of implementation without the agent.

### Capacity

Team capacity is no longer just human hours. It's a combination of:
- Human hours for specification writing and review
- Agent compute budget (API costs, execution time)
- Review pipeline capacity

### Task Breakdown

Instead of breaking stories into implementation sub-tasks (design DB
schema, write API handler, add validation, write tests), the breakdown
becomes:

1. Write specification for the feature
2. Generate implementation via agent
3. Review and verify the output
4. Integrate with existing code
5. Manual testing and edge case verification

## The Daily Standup in an Agentic World

The daily standup questions evolve:

**Traditional**:
- What did you do yesterday?
- What will you do today?
- Any blockers?

**Agentic**:
- What did you specify and verify yesterday?
- What specifications are you writing today?
- Are any agent outputs failing verification?
- Do any specs need team input on edge cases?

The focus shifts from "what code did you write" to "what behaviors
did you define and validate."

## Retrospectives and Prompt Refinement

Agile retrospectives become even more valuable in an agentic context.
The team reflects on:

- **Spec quality**: Which specifications produced good implementations
  on the first try? Which required multiple iterations? Why?
- **Agent effectiveness**: Are there task types where the agent
  consistently struggles? Should we adjust our approach for those?
- **Review efficiency**: Are reviewers catching the same types of
  issues repeatedly? Can those be added to the spec template?
- **Prompt patterns**: Which prompt structures work best for different
  task types?

These insights feed back into better specifications, creating a
virtuous cycle of improvement.

## The Role of the Product Owner

The product owner's role becomes more important, not less. In an
agentic world, the quality of the specification determines the quality
of the output. The product owner must:

1. **Write clear acceptance criteria**: Vague criteria lead to
   vague implementations
2. **Prioritize ruthlessly**: Agents can implement features faster,
   which means more features compete for review bandwidth
3. **Define "done" precisely**: The definition of done must be
   specific enough for automated verification
4. **Manage the spec backlog**: Specifications need review and
   refinement just like code

## Anti-Patterns to Avoid

### 1. "Just Let the Agent Figure It Out"

Skipping the specification and asking the agent to "build a login
page" produces lowest-common-denominator results. Agents are powerful
but not psychic.

### 2. Over-Specification

Writing 20-page specs for every feature defeats the purpose. Match
the spec detail to the task complexity. A one-line config change
doesn't need a spec.

### 3. Skipping Review

"The tests pass, ship it." Passing tests verify behavior, not design.
A human still needs to review the agent's architectural choices,
naming conventions, and integration points.

### 4. Treating Agents Like Junior Developers

Agents aren't learning on the job. They don't benefit from code
reviews the way humans do. Feedback should go into the spec and
prompt, not into comments on the generated code.

## Conclusion

Agile and agentic development are complementary, not competing.
Agile provides the framework — short iterations, user-centered design,
continuous feedback. Agentic AI provides the execution engine —
fast implementation from specifications, automated verification,
and tireless iteration.

The teams that thrive will be those who integrate AI agents into
their agile workflows thoughtfully: writing better specifications,
reviewing agent output critically, and continuously refining their
processes through retrospection. The sprint isn't going away — it's
getting supercharged.
