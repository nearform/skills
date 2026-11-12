# Testing Principles Reference

These principles are layer-agnostic: the same reasoning applies whether the
test drives a browser, calls an API, or exercises a single function.
Layer-specific mechanics (selectors, request/response shape, etc.) live in the
relevant framework or layer skill, not here.

## Determinism

A deterministic test produces the same result every time given the same
inputs, run in any order, on any machine.

### Good

- Wait for a real signal: an API response, an emitted event, a resolved
  promise, a state change.
- Assert against the resulting state once that signal has arrived (a response
  body, a database row, a UI element, a returned value).

### Bad

- Fixed delays (`sleep(1000)`, `cy.wait(1000)`, `page.waitForTimeout(1000)`).
- Timing or ordering assumptions ("the request will finish within 500ms",
  "this always runs after that").

---

## Independence

Each test must run in isolation, in any order, alone or in parallel with
others.

### Good

- Create the data a test needs inside that test (or its setup) and clean up
  afterwards.
- Use independent setup/teardown per test so failures don't cascade.

### Bad

- Reusing state, fixtures, or side effects left behind by a previous test.
- Tests that only pass when run in a specific order.

---

## Behaviour vs Implementation

Assert what the system does, not how it does it internally. Implementation
details change frequently and for reasons unrelated to correctness; behaviour
is what users and consumers actually depend on.

### Good

- "The user sees a success message."
- "The API returns a 201 with the created resource."

### Bad

- "Function X was called."
- "Component re-rendered twice."

---

## Readable and Maintainable

A failing test should say what broke without requiring a read of the
implementation. Optimise for the person diagnosing a failure months from now.

### Good

- Name the test after the behaviour it verifies ("returns 404 when the user
  does not exist").
- Keep each test focused on a single behaviour, with a clear
  arrange/act/assert shape.
- Make the expected outcome obvious from the assertion itself.

### Bad

- Vague names ("test1", "it works") that say nothing when they fail.
- Multiple unrelated behaviours in one test, so a failure doesn't point to a
  cause.
- Logic (loops, conditionals, shared mutable helpers) that forces the reader to
  reconstruct what is actually being verified.

---

## Failing with the Cause

A failing test should say why it failed. A timeout says only that something didn't happen.

### Good

- Setup helpers check each response and throw with its status and body
- Test doubles stop and report a call that was refused or unexpected
- A poll stops on every final state (success and failure) and reports the state it found and its error

### Bad

- Ignoring an error result and carrying on with the rest of the script
- Polling only for the success state, so a failure shows up as a timeout

---

## Test Doubles

A test double is only as useful as the code it leaves in place.

### Good

- Replacing a dependency at the seam the codebase's own tests replace it
- Reusing the codebase's existing fake or harness instead of copying it
- Keeping the guards around that seam (permission checks, input validation, environment scrubbing) so the tests still exercise them

### Bad

- A double that skips the code path it stands in for (writing a file directly instead of through the permission check)
- Giving the code under test credentials or config that production strips out, so a regression in that stripping goes unseen
- A copied harness that drifts from the original

---

## Time Budgets

### Good

- Each wait's timeout comes from the slowest realistic case, including queueing behind parallel tests
- Waits that run one after another fit inside the test's own timeout together
- Independent setup runs in parallel

### Bad

- Several long waits chained in series that can exceed the test timeout on a slow machine

---

## Proving the Outcome

### Good

- After an update, assert the updated content is shown

### Bad

- Asserting only that an indicator appeared ("updated", "revision 2"), which also passes when the content is stale

---

## Dead Test Code

### Good

- Every helper method, fixture branch and export is used by a test

### Bad

- Helpers kept "for later" and exported constants no test reads

---

## Handling a Failing Test

### Good

- Reproduce the failure, then decide: application bug, test bug, or environment issue
- Application bug in a test-only task: stop, report the failing test, the command, and the observed vs expected behaviour
- Test bug: fix the test so it asserts the intended behaviour, not whatever the app currently does

### Bad

- Changing an expected value to match the app's current (wrong) output
- Adding `.skip`, deleting the test, or raising a timeout until it passes
- Editing production code to make a test pass when the task was to write or fix tests
