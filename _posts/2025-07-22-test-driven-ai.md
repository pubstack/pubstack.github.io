---
layout: post
title: "Test-Driven AI: How Specifications Become Validation"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - cloud
  - engineering
  - AI
  - agentic
  - testing
  - spec-driven
favorite: false
commentIssueId: 100
refimage: '/static/agentic_ai/test_driven_ai.svg'
---

When AI agents generate code, how do you know the code is correct?
Traditional testing requires a human to write test cases — but in a
spec-driven world, the specification itself becomes the source of
truth for validation. This post explores how to turn specifications
into automated tests that verify AI-generated code.

## The Trust Problem

AI-generated code has a fundamental trust problem. An LLM can produce
code that looks correct, compiles without errors, and even passes a
cursory review — but still contains subtle bugs. The model optimizes
for plausibility, not correctness.

![](/static/agentic_ai/test_driven_ai.svg)

This isn't unique to AI. Human developers write bugs too. The
difference is that experienced developers have intuition about where
bugs hide — edge cases, boundary conditions, concurrency issues. AI
agents need a more systematic approach: **specification-based testing**.

## From Spec to Test: The Transformation

A well-written specification contains everything needed to generate
tests. Consider this spec fragment:

```markdown
## Email Validation

The system must validate email addresses according to RFC 5322.

- Valid: user@example.com, user+tag@domain.co.uk
- Invalid: @example.com, user@, user@.com
- Maximum length: 254 characters
- Must not contain spaces
- Domain must have at least one dot
```

From this spec, we can derive multiple test categories:

### 1. Positive Tests (Happy Path)

```python
def test_valid_standard_email():
    assert validate_email("user@example.com") is True

def test_valid_plus_addressing():
    assert validate_email("user+tag@domain.co.uk") is True
```

### 2. Negative Tests (Invalid Input)

```python
def test_missing_local_part():
    assert validate_email("@example.com") is False

def test_missing_domain():
    assert validate_email("user@") is False

def test_domain_starts_with_dot():
    assert validate_email("user@.com") is False
```

### 3. Boundary Tests

```python
def test_max_length_email():
    local = "a" * 64
    domain = "b" * 185 + ".com"
    assert validate_email(f"{local}@{domain}") is True

def test_exceeds_max_length():
    email = "a" * 255 + "@example.com"
    assert validate_email(email) is False
```

### 4. Constraint Tests

```python
def test_no_spaces_allowed():
    assert validate_email("user @example.com") is False

def test_domain_has_dot():
    assert validate_email("user@localhost") is False
```

## The Spec-Test-Generate Loop

In test-driven AI, the workflow becomes:

```
Write Spec → Generate Tests from Spec → Generate Code → Run Tests
     ↑                                                       │
     └───────────── Refine Spec if Tests Reveal Gaps ────────┘
```

This loop has several advantages over traditional TDD:

1. **Tests come from the spec, not the implementation**: This prevents
   the common TDD trap of writing tests that mirror the implementation
   rather than testing the behavior.

2. **AI can generate both tests and code**: But from different
   perspectives. The test-generating agent focuses on "what could go
   wrong" while the code-generating agent focuses on "how to make it
   work."

3. **Spec gaps become visible**: When you try to generate tests from
   a spec, you quickly discover what's missing. "What happens when
   the email contains unicode characters?" If the spec doesn't say,
   the test can't be written, and the gap becomes visible.

## Property-Based Testing from Specs

Beyond example-based tests, specifications often imply properties that
should hold for all inputs. These are ideal for property-based testing:

```python
from hypothesis import given, strategies as st

@given(st.emails())
def test_valid_emails_are_accepted(email):
    """Any well-formed email should be accepted."""
    assert validate_email(email) is True

@given(st.text().filter(lambda x: '@' not in x))
def test_strings_without_at_are_rejected(text):
    """No string without @ can be a valid email."""
    assert validate_email(text) is False
```

Property-based tests are particularly valuable for AI-generated code
because they explore the input space much more broadly than hand-written
examples.

## Contract Testing for APIs

When your spec defines an API contract, you can generate contract tests
that verify both the request and response schemas:

```python
def test_lookup_returns_correct_schema():
    response = client.post("/api/v1/users/lookup",
                          json={"email": "test@example.com"})
    assert response.status_code == 200
    data = response.json()
    assert "user_id" in data
    assert "department" in data
    assert "manager_email" in data
    assert isinstance(data["active"], bool)

def test_lookup_invalid_email_returns_422():
    response = client.post("/api/v1/users/lookup",
                          json={"email": "not-an-email"})
    assert response.status_code == 422
    assert response.json()["error"] == "invalid_email_format"
```

## Measuring Coverage Against the Spec

Traditional code coverage measures which lines of code were executed.
In spec-driven testing, we care about **spec coverage** — which
requirements from the specification have been tested.

A simple approach is to tag each test with the spec requirement it
validates:

```python
@spec("EMAIL-001: Valid emails are accepted")
def test_valid_email():
    ...

@spec("EMAIL-002: Maximum length is 254 characters")
def test_max_length():
    ...
```

Then generate a coverage report that maps requirements to tests:

```
EMAIL-001: ✓ Covered (3 tests)
EMAIL-002: ✓ Covered (2 tests)
EMAIL-003: ✗ NOT COVERED
EMAIL-004: ✓ Covered (1 test)
```

This gives you confidence not just that the code works, but that every
aspect of the specification has been verified.

## Practical Recommendations

1. **Write specs before asking AI to generate code**. The spec is your
   leverage. Without it, you're reviewing code without knowing what
   "correct" means.

2. **Use a different AI agent for test generation**. Having the same
   model generate both code and tests creates correlated failures.
   Use a different model, or at minimum a different prompt strategy.

3. **Run tests automatically**. The verification step should be
   automated in your CI pipeline. When tests fail, the spec and
   implementation are out of sync.

4. **Treat test failures as spec signals**. A failing test might mean
   the code is wrong — or it might mean the spec is incomplete. Both
   are valuable signals.

5. **Start with the critical path**. You don't need 100% spec coverage
   on day one. Start with the features where bugs would be most costly
   and expand from there.

Test-driven AI isn't about replacing human judgment. It's about making
AI-generated code as trustworthy as human-written code — and often
more so, because the validation is systematic rather than ad hoc.
