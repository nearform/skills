---
name: test-jira-cases
description: Generate reviewer-ready, traceable test cases from Jira issues using an available Jira MCP server.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - category/generation
    - domain/testing
    - tool/jira
    - tool/mcp
  compatibility:
    - all
---

# Test Jira Cases

Generate comprehensive, risk-based, reviewer-ready test cases from Jira issues.

Invoke only when the user explicitly requests:

- Test cases
- QA coverage
- Validation scenarios

Do not invoke for:

- Test automation implementation
- Code refactoring
- Jira summarisation without testing intent

## Prerequisites

An MCP server providing Jira issue access is available.

If multiple Jira MCP servers are available, use the most appropriate one.

## Input

Accept:

- Jira key
- Jira URL
- Story
- Bug
- Task
- Subtask
- Epic

Optional:

- Scope focus (API, UI, permissions, reporting, etc.)
- Risk focus (security, payments, performance, accessibility, etc.)
- Maximum number of test cases

If no ticket reference is provided, ask for one.

## Process

1. Retrieve the Jira issue using an available Jira MCP tool.
2. Use available issue information, including:
   - Summary
   - Description
   - Acceptance Criteria
   - Comments
   - Labels
   - Components
   - Status
   - Priority
   - Linked issues
   - Subtasks
   - Relevant custom fields
3. Review linked issues or subtasks only when they materially affect testing scope.
4. Identify:
   - Functional behaviour
   - Business rules
   - User workflows
   - Validation rules
   - State transitions
   - Roles and permissions
   - Integrations and dependencies
   - Error handling
   - Security considerations
   - Boundary and edge cases
   - Business risks
5. If acceptance criteria are missing:
   - Derive explicit numbered requirements from the available information.
   - Label them as **Derived Requirements**.
   - Reduce confidence accordingly.
6. Generate concise, executable, risk-prioritised test cases covering, where applicable:
   - Happy path
   - Negative
   - Boundary
   - Edge case
   - Validation
   - Permission
   - Integration
   - Error recovery
   - Regression
7. Merge duplicate coverage and ensure each test validates one primary behaviour.

If the issue cannot be retrieved:

- Ask the user to provide the ticket contents.
- Continue using only user-provided information.
- Mark confidence as reduced.

If requirements are incomplete or ambiguous:

- Identify gaps.
- Clearly distinguish facts from assumptions.
- Ask clarifying questions when necessary.
- Never invent requirements.

## Validation

Before presenting results, ensure:

- Every acceptance criterion or derived requirement is covered.
- Major workflows are represented.
- Appropriate negative and edge-case testing is included.
- Each test case maps to at least one requirement.
- Steps are concise and executable.
- Expected results are observable.
- Assumptions and gaps are explicit.
- Any uncovered requirement is listed.

## Output

### Coverage Summary

Include:

- Acceptance criteria count
- Derived requirement count
- Test case count
- Coverage areas
- Key risks
- Confidence

Confidence:

- High — Requirements are complete and testable.
- Medium — Minor assumptions required.
- Low — Significant ambiguity or missing information.

### Traceability Matrix

| Requirement / AC | Test Cases |
| ---------------- | ---------- |
| AC-1             | TC-001     |

### Test Cases

For each test case include:

- ID
- Title
- Coverage Area
- Priority (P0–P3)
- Type
- Preconditions
- Steps
- Expected Result
- Traceability
- Notes / Assumptions

Formatting:

- Sequential IDs (TC-001, TC-002...)
- Numbered steps
- Concise reviewer-friendly language
- Observable expected results

### Completion

Include:

- Coverage Summary
- Open Questions
- Risks & Gaps
- Uncovered Requirements (if any)
- Reviewer Recommendation

Reviewer Recommendation:

- Ready for review
- Ready for review with assumptions
- Clarification required before review
- Insufficient requirements for reliable test generation

## Guardrails

- Never invent acceptance criteria, business rules or requirements.
- Clearly separate facts from assumptions.
- Prefer clarification over speculation.
- Never claim complete coverage when requirements are incomplete.
- Use linked issues only when relevant to testing scope.
- Do not expose internal reasoning.
