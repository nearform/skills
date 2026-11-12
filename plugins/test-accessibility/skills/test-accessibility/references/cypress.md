# Accessibility scanning with Cypress (`cypress-axe`)

```javascript
import "cypress-axe";

beforeEach(() => {
  cy.visit("/checkout");
  cy.injectAxe();
});

it("has no critical or serious violations on load", () => {
  cy.checkA11y(null, {
    runOnly: { type: "tag", values: ["wcag2a", "wcag2aa", "wcag21aa", "wcag22aa"] },
    includedImpacts: ["critical", "serious"],
  });
});

it("has no violations after opening the modal", () => {
  cy.get('[data-testid="open-modal"]').click();
  cy.checkA11y('[role="dialog"]', { includedImpacts: ["critical", "serious"] });
});
```

- Call `cy.injectAxe()` after each `cy.visit()` / full reload — the injected script does not survive navigation.
- Scope `cy.checkA11y(selector)` to the changed region after an interaction instead of re-scanning the page.
- Pass a violation callback (third argument) to log rule id, impact, and nodes, so failures are triageable.
