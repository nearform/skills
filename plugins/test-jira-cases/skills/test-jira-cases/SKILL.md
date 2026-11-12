---
name: test-jira-cases
description: Use this skill only when the user explicitly asks for test cases, QA coverage, or validation scenarios from a Jira issue. Produces risk-prioritised, traceable, reviewer-ready manual test cases via a Jira MCP server.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/generation
    - domain/engineering
    - domain/testing
    - tool/jira
    - tool/mcp
  compatibility:
    - all
---

# Test Jira Cases

Generate comprehensive, risk-based, reviewer-ready test cases from Jira issues.

Do **not** use for test automation code (use the skill map in `test-automation-guidelines`), refactoring, or Jira summaries without testing intent.

## Prerequisites

A Jira MCP server is available. If several are, use the most appropriate one.

## Input

- Required: a Jira key or URL (story, bug, task, subtask, or epic). If none is given, ask for one.
- Optional: scope focus (API, UI, permissions, reporting…), risk focus (security, payments, performance, accessibility…), maximum number of cases.

## Process

1. Retrieve the issue: summary, description, acceptance criteria, comments, labels, components, status, priority, linked issues, subtasks, relevant custom fields. Follow links and subtasks only when they materially affect testing scope.
2. Identify functional behaviour, business rules, workflows, validation, state transitions, roles and permissions, integrations, error handling, security, boundaries and edge cases, and business risks.
3. If acceptance criteria are missing, derive explicit numbered requirements, label them **Derived Requirements**, and lower confidence.
4. Generate concise, executable, risk-prioritised cases covering, where applicable: happy path, negative, boundary, edge, validation, permission, integration, error recovery, regression.
5. Merge duplicate coverage; each case validates one primary behaviour.

If the issue cannot be retrieved, ask the user for its contents, continue with only that, and mark confidence reduced. If requirements are ambiguous, list the gaps, separate facts from assumptions, and ask clarifying questions.

## Validation

Before presenting, check that every AC or derived requirement maps to at least one case, major workflows and negative/edge cases are covered, steps are executable, expected results are observable, and any uncovered requirement is listed.

## Output

Coverage Summary, Traceability Matrix, Test Cases (TC-001…), Open Questions, Risks & Gaps, Uncovered Requirements, and a Reviewer Recommendation — fields and wording in [references/output-format.md](references/output-format.md).

## Guardrails

- Never invent acceptance criteria, business rules, or requirements.
- Clearly separate facts from assumptions; prefer clarification over speculation.
- Never claim complete coverage when requirements are incomplete.
- Do not expose internal reasoning.
