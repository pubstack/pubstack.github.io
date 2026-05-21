---
layout: post
title: "The Spec-Code-Verify Loop: A New Development Paradigm"
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
commentIssueId: 102
refimage: '/static/agentic_ai/spec_code_verify.svg'
---

Software development has always been cyclical — write, test, fix, repeat.
But the emergence of AI agents introduces a fundamentally new cycle:
**Spec → Code → Verify**. This loop represents a paradigm shift in how
software is built, where human intent drives machine execution and
automated verification closes the feedback loop.

## The Three Phases

### Phase 1: Spec

The developer writes a specification that describes the desired behavior,
constraints, and acceptance criteria. The spec is the primary artifact —
it captures *what* the system should do without prescribing *how*.

![](/static/agentic_ai/spec_code_verify.svg)

A good spec answers these questions:

- What does this component do?
- What are its inputs and outputs?
- What are the constraints and invariants?
- What should happen in error cases?
- How do we know it's correct?

### Phase 2: Code

An AI agent consumes the specification and generates an implementation.
The agent brings knowledge of programming patterns, libraries, and best
practices. It translates the *what* into the *how*.

This phase is where the leverage of AI agents becomes clear. A
specification that takes 30 minutes to write might describe a component
that would take a full day to implement manually. The agent compresses
implementation time dramatically.

### Phase 3: Verify

The generated code is verified against the specification through a
combination of:

- **Automated tests** derived from the spec
- **Static analysis** checking types, contracts, and constraints
- **Human review** for architectural fit and non-functional requirements
- **Runtime validation** in staging or preview environments

If verification passes, the code is accepted. If it fails, the loop
continues — either by refining the spec (if the spec was incomplete)
or by regenerating the code (if the implementation was incorrect).

## How It Differs from TDD

Test-driven development (TDD) follows a Red-Green-Refactor cycle:

```
Write failing test → Write minimal code to pass → Refactor
```

The Spec-Code-Verify loop shares TDD's emphasis on verification-first
thinking but differs in key ways:

| Aspect | TDD | Spec-Code-Verify |
|--------|-----|------------------|
| Primary artifact | Test cases | Specification |
| Who writes code? | Human developer | AI agent |
| Granularity | Method/function level | Feature/component level |
| Iteration speed | Minutes | Seconds to minutes |
| Verification scope | Unit tests | Tests + analysis + review |

The SCV loop operates at a higher level of abstraction. Instead of
testing individual methods, you're specifying and verifying complete
behaviors. This doesn't replace unit testing — it layers on top of it.

## A Worked Example

Let's walk through a concrete example: building a rate limiter.

### Step 1: Write the Spec

```markdown
## Rate Limiter

### Behavior
Implement a sliding window rate limiter that tracks requests
per client IP address.

### Configuration
- window_size: duration (default: 60 seconds)
- max_requests: integer (default: 100)

### Interface
- check(ip: string) -> {allowed: bool, remaining: int, reset_at: timestamp}
- reset(ip: string) -> void

### Requirements
- Thread-safe for concurrent access
- Memory-efficient: clean up expired entries
- Time complexity: O(1) for check operations
- Must handle clock skew gracefully

### Edge Cases
- First request from a new IP is always allowed
- Exactly max_requests in a window should be allowed
- Request max_requests + 1 should be denied
- After window expires, full quota is restored
```

### Step 2: Generate Code

The AI agent reads the spec and generates an implementation — perhaps
using a sorted set to track timestamps, with a background cleanup
routine for expired entries.

### Step 3: Verify

From the spec, we derive verification:

```python
def test_first_request_allowed():
    limiter = RateLimiter(max_requests=5, window_size=60)
    result = limiter.check("192.168.1.1")
    assert result.allowed is True
    assert result.remaining == 4

def test_at_limit():
    limiter = RateLimiter(max_requests=2, window_size=60)
    limiter.check("10.0.0.1")
    result = limiter.check("10.0.0.1")
    assert result.allowed is True
    assert result.remaining == 0

def test_over_limit():
    limiter = RateLimiter(max_requests=2, window_size=60)
    limiter.check("10.0.0.1")
    limiter.check("10.0.0.1")
    result = limiter.check("10.0.0.1")
    assert result.allowed is False

def test_window_expiry():
    limiter = RateLimiter(max_requests=1, window_size=1)
    limiter.check("10.0.0.1")
    time.sleep(1.1)
    result = limiter.check("10.0.0.1")
    assert result.allowed is True

def test_thread_safety():
    limiter = RateLimiter(max_requests=100, window_size=60)
    results = parallel_execute(
        [lambda: limiter.check("10.0.0.1")] * 200
    )
    allowed = sum(1 for r in results if r.allowed)
    assert allowed == 100
```

If all tests pass, the rate limiter is verified. If the thread safety
test fails, the agent regenerates with explicit locking. If the spec
didn't mention thread safety, we'd discover it during review and
update the spec.

## The Economics of the Loop

The SCV loop changes the economics of software development:

- **Spec writing** is high-value human work that AI can't fully
  automate — it requires understanding business context, user needs,
  and system constraints
- **Code generation** is increasingly commoditized — AI agents can
  produce competent implementations from good specs
- **Verification** is partially automatable but requires human
  judgment for architectural and design decisions

This means the highest-leverage activity for a developer shifts from
writing code to writing specifications and reviewing generated code.
The bottleneck moves from implementation capacity to specification
quality.

## Making the Loop Faster

Several practices accelerate the SCV loop:

1. **Template your specs**: Create reusable templates for common
   component types (APIs, data models, workers, etc.)
2. **Automate verification**: The more of the verification step
   that's automated, the faster the loop turns
3. **Cache intermediate results**: If only the spec's error handling
   section changed, you might not need to regenerate the entire
   implementation
4. **Parallelize across components**: Different components can be
   in different phases of the loop simultaneously
5. **Tighten feedback**: Show verification results immediately,
   not in a batch at the end

## Conclusion

The Spec-Code-Verify loop isn't just a process improvement — it's a
fundamental rethinking of who does what in software development.
Developers become architects and reviewers. AI agents become
implementers. And specifications become the connective tissue that
holds it all together.

The developers who master this loop will ship faster, with fewer
bugs, and with better documentation than those who continue writing
every line by hand. The loop is the future of software development
— and it's available today.
