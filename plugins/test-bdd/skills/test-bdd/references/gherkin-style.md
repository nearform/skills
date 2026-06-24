# Gherkin Style Reference

A focused style guide for writing declarative, behaviour-first Gherkin. Pair it
with the framework skill (`test-cypress`, `test-playwright`, `test-webdriverio`)
for everything that happens inside a step definition.

## Declarative vs Imperative

The single most important habit in BDD. Write intent, not mechanics.

### Good (declarative)

```gherkin
Scenario: Out-of-stock items cannot be ordered
  Given the "Aeron Chair" is out of stock
  When Ada tries to add it to her basket
  Then she is told the item is unavailable
```

### Bad (imperative)

```gherkin
Scenario: Out-of-stock items cannot be ordered
  Given I open "/product/aeron-chair"
  When I click "#add-to-basket"
  Then I see ".alert.alert-danger" containing "Unavailable"
```

The imperative version breaks when the route, the CSS class, or the button id
changes — none of which are the behaviour under test.

---

## One Behaviour Per Scenario

### Good

```gherkin
Scenario: Empty basket shows a prompt to start shopping
  Given Ada has an empty basket
  When she views her basket
  Then she is prompted to start shopping
```

### Bad

```gherkin
Scenario: Basket
  Given Ada has an empty basket
  Then she is prompted to start shopping
  When she adds a chair
  Then the basket shows 1 item
  And the total updates
  And a discount banner appears
```

Split the bad example into separate, independently named scenarios.

---

## Background

### Good — shared context only (Given)

```gherkin
Background:
  Given the "Standard" catalogue is loaded
  And Ada is signed in
```

### Bad — actions and assertions leak in

```gherkin
Background:
  Given Ada is signed in
  When she opens the basket   # action belongs in a scenario
  Then it is empty            # assertion belongs in a scenario
```

---

## Scenario Outline

### Good — same behaviour, varying data

```gherkin
Scenario Outline: Free delivery threshold
  Given Ada has a basket worth <total>
  When she proceeds to checkout
  Then delivery is <delivery>

  Examples:
    | total | delivery |
    | 40    | charged  |
    | 75    | free     |
```

### Bad — different behaviours forced into one outline

```gherkin
Scenario Outline: Checkout
  Given Ada has a basket worth <total>
  When she <action>
  Then <outcome>

  Examples:
    | total | action            | outcome              |
    | 75    | proceeds          | delivery is free     |
    | 0     | proceeds          | she sees an error    |
    | 75    | applies a voucher | the total decreases  |
```

---

## Step Phrasing

### Good

- `Given Ada is a registered customer`
- `When she signs in with valid credentials`
- `Then she lands on her dashboard`

Reusable, business-readable, parameterisable.

### Bad

- `Given the user with email ada@example.com and password hunter2 exists in the DB`
- `When she types her email and clicks submit and waits 2 seconds`
- `Then the div with id dashboard is visible`

Leaks data, mechanics, and waits into the specification.

---

## State

### Good — per-scenario World

```javascript
Given("Ada has a basket worth {int}", function (total) {
  this.basket = createBasketWorth(total);
});
```

### Bad — module global shared across scenarios

```javascript
let basket; // leaks between scenarios, breaks parallelism

Given("Ada has a basket worth {int}", function (total) {
  basket = createBasketWorth(total);
});
```

---

## Quick Checklist

- Does each scenario name one behaviour, with no "and"?
- Could a non-engineer read the feature file without seeing a selector or URL?
- Is every step reusable, in business language, in a consistent tense?
- Is `Background` only `Given` context?
- Does `Scenario Outline` vary data, not behaviour?
- Is per-scenario state on the World, never a module global?
- Are mechanics (selectors, waiting, network, assertions) delegated to the framework skill?
