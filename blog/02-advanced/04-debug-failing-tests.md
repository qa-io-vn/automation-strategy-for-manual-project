# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · 04: Debugging Failing Playwright Tests with AI

> **Series Navigation**
> [← 03 · First Test](./03-convert-first-test.md) | **04 · Debugging** | [05 · Regression Suite →](./05-build-regression-suite.md)

---

## Purpose

Tests fail. That's not a problem — it's the point. A failing test is information. The skill is knowing how to extract that information quickly, fix the root cause, and move on.

This guide covers:
- The most common Playwright error types and their root causes
- How to use Playwright's trace viewer, debug mode, and codegen
- Exactly how to prompt Zen with error messages to get useful fixes
- How to identify and eliminate flaky tests
- A systematic debugging workflow that works every time

---

## The Debugging Mindset

Before diving into tools, internalize this: **A failing test is either a product bug or a test bug.** Both outcomes are valuable.

| Test Fails Because... | Action |
|-----------------------|--------|
| Product has a real bug | File the bug. The test did its job. |
| Selector changed | Update the Page Object selector |
| Test data is wrong/missing | Fix your data setup |
| Timing/async issue | Add proper waits |
| Test was wrong all along | Fix the assertion logic |
| Environment issue | Fix the CI/CD config |

Always determine which category you're in before touching any code.

---

## Common Error Types: Causes and Fixes

### Error Type 1: Timeout — Waiting for Locator

```
TimeoutError: locator.click: Timeout 30000ms exceeded.
=========================== logs ===========================
waiting for locator('[data-testid="checkout-submit-btn"]')
  - locator resolved to <button data-testid="checkout-submit-btn" disabled="">...</button>
```

**What it means:** Playwright found the element but couldn't click it because it was disabled, hidden, or outside the viewport within the timeout window.

**Common causes:**

| Cause | Clue in Error | Fix |
|-------|---------------|-----|
| Element is disabled | `disabled=""` in resolved locator | Wait for it to become enabled, or check prerequisite steps |
| Element hidden by CSS | `display: none` or `visibility: hidden` | Check if a previous step should have revealed it |
| Wrong selector | Element never found | Use `--debug` to inspect actual DOM |
| Page not loaded | Selector appears before page ready | Add `await page.waitForLoadState('networkidle')` |
| Selector returns multiple elements | Multiple matches | Use `.first()`, `.nth(0)`, or a more specific selector |

**Fix pattern:**

```javascript
// ❌ Assumes element is immediately clickable
await page.getByTestId('checkout-submit-btn').click();

// ✅ Wait for it to be enabled first
await expect(page.getByTestId('checkout-submit-btn')).toBeEnabled();
await page.getByTestId('checkout-submit-btn').click();

// ✅ Or wait for a network response that triggers the enable
await page.waitForResponse('**/api/cart/validate');
await page.getByTestId('checkout-submit-btn').click();
```

---

### Error Type 2: Strict Mode Violation

```
Error: strict mode violation: getByTestId('product-card') resolved to 6 elements:
  1) <div data-testid="product-card" class="...">Product A</div>
  2) <div data-testid="product-card" class="...">Product B</div>
  ...
```

**What it means:** Your selector matched more than one element. Playwright's default strict mode requires a unique match.

**Fix:**

```javascript
// ❌ Matches all product cards
await page.getByTestId('product-card').click();

// ✅ Option 1: Filter by text content
await page.getByTestId('product-card').filter({ hasText: 'Product A' }).click();

// ✅ Option 2: Use nth (only if order is guaranteed)
await page.getByTestId('product-card').first().click();

// ✅ Option 3: Ask devs to add unique IDs
// <div data-testid="product-card-123">Product A</div>
await page.getByTestId('product-card-123').click();
```

---

### Error Type 3: Navigation Timeout

```
TimeoutError: page.waitForURL: Timeout 30000ms exceeded.
waiting for url matching /.*\/dashboard/

Call log:
  - navigating to "https://staging.myapp.com/login", waiting until "load"
  - "load" event fired
  - navigating to "https://staging.myapp.com/login?error=invalid_credentials", waiting until "load"
```

**What it means:** The page navigated, but to the wrong URL. In this example, login failed and the user wasn't redirected to `/dashboard` — they stayed on `/login` with an error query parameter.

**Debugging steps:**
1. Is this a product bug (login failing that shouldn't)?
2. Is the test using wrong credentials?
3. Is the environment down?

**Fix:**

```javascript
// ❌ Hard navigation wait with no fallback info
await page.waitForURL('/dashboard');

// ✅ Better: check what's actually on the page if navigation fails
try {
  await page.waitForURL('**/dashboard', { timeout: 10000 });
} catch (e) {
  const currentUrl = page.url();
  const errorMessage = await page.getByTestId('login-error').textContent().catch(() => 'none');
  throw new Error(`Login failed. Current URL: ${currentUrl}. Error: ${errorMessage}`);
}
```

---

### Error Type 4: Assertion Failure

```
Error: expect(received).toHaveText(expected)

Expected string: "Welcome back, Test User"
Received string: "Welcome back, Test user"
```

**What it means:** The assertion is wrong. This is case sensitivity — "user" vs "User."

**This is very often a product issue** — the developer shipped with the wrong capitalization. File a bug first. If it's intentional, update your assertion.

```javascript
// ❌ Strict string match
await expect(page.getByTestId('welcome-message')).toHaveText('Welcome back, Test User');

// ✅ Case-insensitive if capitalization is intentionally flexible
await expect(page.getByTestId('welcome-message')).toHaveText('Welcome back, Test User', {
  ignoreCase: true,
});

// ✅ Partial match if the full string varies
await expect(page.getByTestId('welcome-message')).toContainText('Welcome back');
```

---

### Error Type 5: Element Detached From DOM

```
Error: locator.fill: Error: Element is detached from DOM

Call log:
  - fill("testuser@example.com")
```

**What it means:** The element existed when Playwright found it, but was removed from the DOM before the action completed. This happens when:
- A React/Vue component re-renders and replaces the element
- An animation or transition temporarily removes the element
- A modal or overlay closed and removed the element

**Fix:**

```javascript
// ❌ Storing a reference and using it after a potential re-render
const emailInput = page.locator('#email');
await page.click('#show-login-form');  // This might re-render the form
await emailInput.fill('test@example.com');  // emailInput is now detached

// ✅ Always query fresh before interacting
await page.click('#show-login-form');
await page.waitForSelector('#email', { state: 'attached' });
await page.locator('#email').fill('test@example.com');

// ✅ Or use getByTestId which re-queries each time
const loginPage = new LoginPage(page);
// Locators in the constructor are lazy — they re-query when used
await loginPage.fillEmail('test@example.com');
```

---

### Error Type 6: Network Request Failure

```
Error: page.waitForResponse: Timeout 30000ms exceeded.
waiting for response matching "https://api.staging.myapp.com/auth/login"
```

**What it means:** An API call your test was waiting for never happened or never returned.

**Common causes:**
- Wrong URL pattern in the matcher
- API is down in the test environment
- Auth token expired before the request was made
- CORS issue blocking the request

**Fix:**

```javascript
// ❌ Waiting for exact URL (fragile if URL format changes)
await page.waitForResponse('https://api.staging.myapp.com/auth/login');

// ✅ Glob pattern (more flexible)
await page.waitForResponse('**/auth/login');

// ✅ With status code check
const response = await page.waitForResponse(
  (resp) => resp.url().includes('/auth/login') && resp.status() === 200
);

// ✅ Checking for failure case
const response = await page.waitForResponse('**/auth/login');
if (!response.ok()) {
  const body = await response.json();
  console.log('Auth API returned error:', body);
}
```

---

## How to Use Playwright's Debug Mode

Debug mode pauses test execution and lets you inspect the browser state, step through actions, and check selectors in real time.

### Starting Debug Mode

```bash
# Debug a specific file
npx playwright test tests/specs/auth/login.spec.js --debug

# Debug a specific test by name
npx playwright test --debug -g "valid credentials"

# Debug and run headed
npx playwright test tests/specs/auth/login.spec.js --debug --headed
```

### What You See in Debug Mode

1. **The Playwright Inspector** opens alongside the browser
2. A **step-through toolbar** lets you advance one action at a time
3. The **action log** shows every Playwright command and its status
4. You can use the **Locator** tab to test selectors against the live page

### Debug Mode Workflow

```
Start debug mode
      ↓
Click "Resume" or "Step Over" to advance
      ↓
When a step fails, inspect the browser
      ↓
Use the Locator tab to test your selector
      ↓
Find the correct selector
      ↓
Update your Page Object and re-run
```

### Adding Breakpoints in Code

```javascript
test('login test', async ({ page }) => {
  const auth = new AuthFlow(page);

  await auth.loginPage.navigate();
  await auth.loginPage.fillEmail('testuser@example.com');

  // Pause here — Playwright Inspector will stop and wait
  await page.pause();

  await auth.loginPage.fillPassword('TestPass123!');
  await auth.loginPage.submit();
});
```

`await page.pause()` drops you into interactive debug mode at that exact point.

---

## How to Use the Playwright Trace Viewer

The trace viewer is your most powerful debugging tool. It records everything — screenshots at every step, network requests, console logs, DOM snapshots — and lets you replay the test execution interactively.

### Enable Traces

```javascript
// playwright.config.js — Recommended settings
use: {
  trace: 'on-first-retry',      // Record on first retry (CI-friendly)
  // trace: 'retain-on-failure', // Keep traces for all failed tests
  // trace: 'on',                // Record every test (expensive, for debugging sessions)
}
```

### Open a Trace After a Failure

```bash
# Open from a file
npx playwright show-trace test-results/auth-login-chromium/trace.zip

# Open from the HTML report (click on the test → Traces tab)
npx playwright show-report
```

### What the Trace Viewer Shows

| Panel | What You Can See |
|-------|-----------------|
| **Actions** | Every Playwright action with timing |
| **Screenshots** | Browser state before/after each action |
| **Network** | All HTTP requests with request/response bodies |
| **Console** | Browser console output at each step |
| **Source** | The test code with the failing line highlighted |
| **DOM Snapshots** | The actual HTML at any point in time |

### Trace Viewer Debugging Workflow

```
Test fails in CI
      ↓
Download trace.zip artifact from CI
      ↓
npx playwright show-trace trace.zip
      ↓
Find the failing action in the Actions panel
      ↓
Look at the screenshot BEFORE the failure
  → Is the expected element visible?
  → Is there a modal/overlay blocking it?
  → Did the previous step complete successfully?
      ↓
Check the Network tab
  → Did an API call fail?
  → Was there a redirect?
      ↓
Check the Console tab
  → Any JavaScript errors?
  → Any 404/500 logged?
```

---

## How to Use Playwright Codegen

Codegen records your manual interactions and generates Playwright code. It's the fastest way to:
- Find selectors for new elements
- Understand what actions Playwright would generate for a flow
- Quickly scaffold new page objects

```bash
# Basic codegen
npx playwright codegen https://staging.myapp.com

# With authentication state (start logged in)
npx playwright codegen --save-storage=auth.json https://staging.myapp.com/login
# Log in manually, then press "Record" to start generating for authenticated pages

# On a specific browser
npx playwright codegen --browser=firefox https://staging.myapp.com
```

**Workflow:**
1. Run codegen pointing at your staging environment
2. Perform the user actions you want to automate
3. Copy the generated selectors into your Page Object
4. Clean up any fragile selectors Playwright suggested

---

## How to Prompt Zen with Error Messages

This is the highest-leverage debugging technique available to you. Zen can diagnose errors, suggest fixes, and explain exactly what's wrong.

### The Golden Prompt Template

```
I'm debugging a failing Playwright test. Here's everything I know:

**Error message:**
[paste the full error message and stack trace]

**The test code:**
[paste the failing test block]

**The relevant Page Object:**
[paste the Page Object class]

**What the test is supposed to do:**
[describe in plain English what you're testing]

**What actually happened:**
[describe what you observed — did the page load? did you see an error? did it redirect?]

**Environment:**
- Playwright version: [run npx playwright --version]
- Browser: chromium / firefox / webkit
- Running: locally / in CI (GitHub Actions / GitLab CI)

Please:
1. Explain what's causing this error
2. Suggest a fix
3. Explain how to prevent this class of error in the future
```

### Real Example — Using Zen for a Timeout Error

**Your prompt to Zen:**

```
I'm debugging a failing Playwright test. Here's everything I know:

**Error message:**
TimeoutError: locator.click: Timeout 30000ms exceeded.
=========================== logs ===========================
waiting for locator('[data-testid="checkout-submit-btn"]')
  - locator resolved to <button data-testid="checkout-submit-btn" disabled="">Place Order</button>
============================================================

**The test code:**
test('complete checkout', async ({ page }) => {
  const checkout = new CheckoutFlow(page);
  await checkout.fillShippingAddress();
  await page.getByTestId('checkout-submit-btn').click();
  await expect(page).toHaveURL('/order-confirmation');
});

**What the test is supposed to do:**
User fills in shipping address, clicks Place Order, and is taken to order confirmation.

**What actually happened:**
The button was found but was disabled. The test timed out waiting for it to become clickable.

**Environment:**
- Playwright version: 1.44.0
- Browser: chromium
- Running: locally
```

**Zen will tell you something like:**

The button is disabled because a prerequisite step hasn't completed. This is almost always one of:
1. A payment method hasn't been selected yet
2. An API call to validate the cart hasn't resolved
3. A form field is still invalid

It will suggest adding a wait for the button to be enabled, checking your flow for missing steps, and verifying the cart validation API call is completing.

---

## Identifying and Eliminating Flaky Tests

A flaky test is one that passes sometimes and fails sometimes without any code changes. Flakiness is the #1 trust destroyer for automation suites.

### Common Sources of Flakiness

| Source | Symptom | Fix |
|--------|---------|-----|
| **Timing** | Test passes when run alone, fails in parallel | Add explicit waits, avoid `waitForTimeout` |
| **Shared test data** | Tests pass in isolation, fail when run together | Each test creates + owns its data |
| **Hardcoded waits** | Tests pass on fast CI, fail on slow CI | Replace `waitForTimeout` with `waitForSelector` |
| **Animation interference** | Test fails clicking element during animation | Wait for animation to complete |
| **Network variability** | Test flaky only in CI, not locally | Add `waitForResponse` for critical API calls |
| **Session/cookie bleed** | Test fails when run after certain other tests | Ensure proper test isolation / fresh context |
| **Date/time sensitivity** | Test fails on specific dates | Mock the system clock |

### Finding Flaky Tests

Run your suite multiple times with `--repeat-each`:

```bash
# Run each test 5 times — flaky tests will show mixed pass/fail results
npx playwright test --repeat-each=5

# Run a specific file many times
npx playwright test tests/specs/auth/login.spec.js --repeat-each=10
```

### The Flaky Test Quarantine Strategy

Don't delete flaky tests — quarantine them while you investigate:

```javascript
// Add a tag to quarantine without removing the test
test('@flaky @regression TC-CHECKOUT-003: ...', async ({ page }) => {
  // test body
});
```

Then exclude flaky tests from your main run:

```bash
# Run all regression tests EXCEPT flaky ones
npx playwright test --grep @regression --grep-invert @flaky
```

Investigate each quarantined test, fix the root cause, remove the `@flaky` tag.

### Ask Zen to Identify Flakiness Causes

```
This Playwright test is flaky — it passes about 70% of the time and fails 30%
of the time with this error:

[paste error]

Here's the test code:
[paste test]

The test environment uses a shared staging database. Multiple QA engineers
run tests simultaneously sometimes.

What are the most likely causes of flakiness here, and how should I fix each?
```

---

## Systematic Debugging Workflow

When a test fails, follow this checklist every time. Don't skip steps.

```
1. READ THE ERROR MESSAGE
   □ What is the error type? (Timeout, Assertion, Navigation, etc.)
   □ What element/URL/value is involved?
   □ What was Playwright waiting for?

2. CHECK THE TRACE OR SCREENSHOT
   □ What did the page look like at the moment of failure?
   □ Was the expected element present?
   □ Was there an unexpected overlay, modal, or error state?

3. REPRODUCE LOCALLY
   □ npx playwright test [file] --headed
   □ Does it fail consistently or intermittently?
   □ If intermittent → likely flaky → see Flaky Test section

4. USE DEBUG MODE
   □ npx playwright test [file] --debug
   □ Step through to the failing action
   □ Inspect the element in the browser DevTools

5. CLASSIFY THE FAILURE
   □ Is it a product bug? → File a bug report
   □ Is it a test issue? → Fix the test
   □ Is it an environment issue? → Fix the environment

6. ASK ZEN (if stuck)
   □ Use the Golden Prompt Template
   □ Include: error, code, page object, environment, observed behavior

7. FIX AND VERIFY
   □ Make the fix
   □ Run the test 3+ times to confirm it's stable
   □ If it was flaky, run with --repeat-each=5
```

---

## Quick Reference: Debug Commands

```bash
# Run with full verbose output
npx playwright test --reporter=list

# Run with debug mode on a single test
npx playwright test -g "TC-AUTH-001" --debug

# Show HTML report from last run
npx playwright show-report

# Open a specific trace file
npx playwright show-trace test-results/auth-login/trace.zip

# Generate test code from browser interactions
npx playwright codegen https://your-staging.com

# Run with headed browser (watch the test)
npx playwright test --headed

# Slow down execution (helps see what's happening)
npx playwright test --headed --slow-mo=1000

# Run test and keep browser open after failure (no --headed needed in some versions)
PWDEBUG=1 npx playwright test tests/specs/auth/login.spec.js

# Print all console logs during test
# (Add this to playwright.config.js)
# use: { launchOptions: { slowMo: 100 } }
```

---

## Summary

| Situation | Tool to Use |
|-----------|-------------|
| Test fails in CI, need to understand why | Trace Viewer |
| Selector is wrong, need to find the right one | Codegen + Debug Mode |
| Error is confusing, need an explanation | Zen (Golden Prompt Template) |
| Test is flaky, need to find the cause | `--repeat-each`, Trace Viewer |
| Need to step through a test | `--debug` with `await page.pause()` |
| Test passes locally, fails in CI | Check environment vars, parallelism, timing |

Debugging is a skill that improves with practice. Every failure you investigate makes you better at writing tests that don't fail.

---

**Next:** [05 · Build Your Regression Suite →](./05-build-regression-suite.md) — How to grow from a handful of tests into a full, organized, maintainable regression suite.
