# Mobile Tests Guidance

Apply this guidance when writing, reviewing or maintaining WebdriverIO mobile tests.

## WebdriverIO Rules

### Async Model

- Use async/await

---

### Selectors

- Prefer accessibility-first selectors as your primary choice: 
  - iOS: accessibility id 
  - Android: resource id or accessibility id
  - Cross-platform: accessibility id

**Primary (preferred):**

```javascript
$("~login-button"); // accessibility id
```

**Fallback:**

If no accessibility id exists, use a platform-native selector strategy. Several libraries are exposed through Appium:

- iOS: XCUITest predicate strings (`-ios predicate string`) and class chain (`-ios class chain`)
- Android: UiAutomator2 (`android=new UiSelector()...`) 

---

### Waiting

Do not use:

```javascript
driver.pause(1000);
```

- Use `waitForDisplayed`, `waitForEnabled` to wait on element state, or `driver.waitUntil` for custom conditions

```javascript
await $("~element").waitForDisplayed();
```

---

### Assertions

- Use `expect-webdriverio` matchers for test assertions

```javascript
await expect($("~username-input")).toExist();
await expect($("~password-input")).toBeDisplayed();
```

---

### Screen Objects

- Mobile apps are organised into screens, however the Page Object Model principles should still be applied.
- Define each screen's selectors and functions in its own class to encourage re-usability across tests

**Example:**

```javascript
class LoginScreen {
  get usernameInput() {
    return $("~username-input");
  }
  get passwordInput() {
    return $("~password-input");
  }
  get submitButton() {
    return $("~submit-button");
  }

  async login(username, password) {
    await this.usernameInput.setValue(username);
    await this.passwordInput.setValue(password);
    await this.submitButton.click();
  }
}
```

---

### Gestures

- Prefer WebdriverIO's gesture commands (`tap`, `longPress`, `swipe`, `pinch`, `zoom`, `dragAndDrop`, `scrollIntoView`) 
- Fall back to `driver.execute("mobile: ...")` only when no native WebdriverIO command exists
- Anchor gestures to an element rather than hard-coded screen coordinates — coordinates can break across different device screen sizes

**Example:**

```javascript
class InboxScreen {
  async swipeMessages() {
    await driver.swipe({
      direction: "up",
      scrollableElement: $("~message-list"),
    });
  }
}
```
