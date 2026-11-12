# Accessibility scanning with WebdriverIO (`@axe-core/webdriverio`)

```javascript
import AxeBuilder from "@axe-core/webdriverio";

it("has no critical or serious violations", async () => {
  await browser.url("/checkout");

  const results = await new AxeBuilder({ client: browser })
    .withTags(["wcag2a", "wcag2aa", "wcag21aa", "wcag22aa"])
    .analyze();

  const blocking = results.violations.filter((v) =>
    ["critical", "serious"].includes(v.impact),
  );
  expect(blocking).toEqual([]);
});
```

- Same scoping, tagging, and impact filtering as Playwright: `include`/`exclude`, `withTags`, filter by `impact`.
