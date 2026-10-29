# Testing Principles Reference

## Determinism

A deterministic test produces the same result every time given the same inputs.

### Good

- Wait for API response
- Assert DOM state

### Bad

- Fixed delays
- Timing assumptions

---

## Independence

Each test must run in isolation.

### Good

- Fresh data per test
- Independent setup

### Bad

- Reusing state from previous tests

---

## Behaviour vs Implementation

### Good

- "User sees success message"

### Bad

- "Function X was called"

---

## Selector Strategy

### Preferred

- data-testid
- accessibility roles

### Avoid

- CSS classes
- DOM structure

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
