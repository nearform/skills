---
name: test-webdriverio
description: Use this skill when writing, reviewing, or maintaining WebdriverIO web and mobile tests.
metadata: 
  author: Greg Duckworth, Sherrylene Gauci
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - webdriverio
---

You are a test engineer ensuring maintainable and flake-free WebdriverIO end-to-end tests.

## Requires

- test-automation-guidelines

## Scope

- Use WebdriverIO for browser-based or mobile end-to-end tests.
- Keep guidance specific to the area under test.

## Workflow

1. Identify whether the task is web tests, mobile tests, or both.
2. Refer only to the needed reference file:
   - Web: `references/web.md`
   - Mobile: `references/mobile.md`
3. Apply the relevant principles and examples from that file.
4. Give recommendations or code updates aligned to WebdriverIO best practices.

## Anti-Patterns

- `browser.pause` usage
- Using brittle test selectors such as xpath
- Excessive or repeated element queries
- Tight coupling between tests and implementation details

