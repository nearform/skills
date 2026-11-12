# Accessibility scanning with Playwright (`@axe-core/playwright`)

```javascript
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("checkout has no critical or serious a11y violations", async ({ page }) => {
  await page.goto("/checkout");

  const results = await new AxeBuilder({ page })
    .withTags(["wcag2a", "wcag2aa", "wcag21aa", "wcag22aa"])
    .analyze();

  const blocking = results.violations.filter((v) =>
    ["critical", "serious"].includes(v.impact),
  );
  expect(blocking).toEqual([]); // the diff shows rule id, impact and nodes
});
```

- `.include(selector)` / `.exclude(selector)` scope a scan to a changed region.
- `.withTags([...])` pins the WCAG level under test rather than every rule.
- Filter by `impact` before asserting — asserting the raw array fails on `moderate`/`minor` too. Log the filtered-out violations instead of discarding them.
