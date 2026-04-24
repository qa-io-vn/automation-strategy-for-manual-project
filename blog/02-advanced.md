# Part 2 — Advanced: Converting Manual Tests to Automation with AI

> **Series:** AI-Assisted QA — From Manual Tester to Automation Strategist  
> **Level:** Advanced  
> **Prev:** [Part 1 — Beginner: Your First Week](./01-beginner.md)  
> **Next:** [Part 3 — Expert: Strategy, Risk, and Scale](./03-expert.md)

---

## The Shift: From Doing Tests to Building a Test Suite

In the Beginner phase, AI helped you do manual testing faster. Now it's time to eliminate the most repetitive parts entirely.

In this phase, you will:

- Learn how to decide **what to automate** (and what not to)
- Use AI to **convert manual test cases into Playwright scripts**
- Build a **basic CI/CD integration** with AI guidance
- Understand the **3-layer architecture** (Spec → Flow → Page) without needing a computer science degree

You don't need to be a developer. You need to be willing to learn and experiment.

---

## Step 1: Decide What to Automate First

Not every test case should be automated. The key is to pick the right ones first, so you get the most return on your investment.

### Use the ROI Scoring Model

For each manual test case, score it on two dimensions:

**Business Priority Score (BPS) — 1 to 5**

| Score | What it means |
|---|---|
| 5 | Mission-critical: payment, core workflow, high defect history |
| 4 | High impact, frequently used, blocks release when broken |
| 3 | Common path, medium impact |
| 2 | Some usage but not critical |
| 1 | Edge case, rarely used, low impact |

**Automation Complexity Score (ACS) — 1 to 5**

| Score | What it means |
|---|---|
| 1 | Very easy: stable UI, simple data, clear selectors |
| 2 | Easy: some data setup, few pages |
| 3 | Medium: multiple roles, pages, some async complexity |
| 4 | Hard: unstable UI, heavy permissions, external dependencies |
| 5 | Very hard: CAPTCHA, OTP, hardware, unstable requirements |

**ROI Score = BPS ÷ ACS** — Automate highest ROI first.

### Ask Zen to Score Your Test Cases

```
Here are my top 20 manual test cases. Score each one for:
- Business Priority (1–5)
- Automation Complexity (1–5)
- ROI Score (BPS/ACS)
- Decision: "Automate now" / "Automate later" / "Keep manual"

Test cases:
[paste your test cases]
```

Zen will produce a prioritized table. Use it to build your first automation sprint backlog.

### Automation Decision Rules

**Automate NOW if:**
- BPS is 4–5 AND ACS is 1–3
- Repeated every single release
- Deterministic expected result (not subjective)

**Keep Manual if:**
- ACS is 5 (OTP, CAPTCHA, external hardware)
- UI changes every sprint (unstable)
- Expected result is subjective (visual layout, "looks good")

---

## Step 2: Understand the Architecture (Without the Jargon)

Before writing any automation, you need to understand *how* good automation is structured. Think of it like organizing a kitchen:

```
Spec File (Chef's Recipe)
  └── describes WHAT to cook and in what order

Flow File (Cooking Technique)
  └── describes HOW to do each cooking step

Page Object (Kitchen Tools)
  └── knows WHERE everything is (the stove, the knife, the bowl)
```

In Playwright terms:

```
spec/checkout.spec.js       ← The test scenario ("create order and verify")
flows/checkout.flow.js      ← Business steps ("createOrder()", "applyPromoCode()")
pages/CheckoutPage.js       ← UI interactions ("clickSubmit()", locators)
```

This structure means:
- If the UI changes, you only update `CheckoutPage.js`
- If the business logic changes, you update `checkout.flow.js`
- Your test scenario stays clean and readable

---

## Step 3: Convert Your First Manual Test Case

### Pick a Simple Candidate

Good first automation: a high-priority, stable, simple test case.
Example: **"Login with valid credentials"**

Manual test case:
```
Title: Login with valid credentials
Preconditions: User account exists with email test@example.com / password Test@1234
Steps:
  1. Navigate to https://app.example.com/login
  2. Enter email: test@example.com
  3. Enter password: Test@1234
  4. Click Login button
Expected: Dashboard page loads, user name "John Doe" is visible in the header
```

### Prompt Zen to Convert It

```
Convert this manual test case to a Playwright JavaScript test.
Use the Spec → Flow → Page Object architecture.
Use data-testid selectors where possible.
Follow this naming convention:
  - Spec: login.spec.js
  - Flow: auth.flow.js
  - Page: LoginPage.js

Manual test case:
[paste the test case]
```

### What Zen Generates

**`pages/LoginPage.js`**
```javascript
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.getByTestId('email-input');
    this.passwordInput = page.getByTestId('password-input');
    this.loginButton = page.getByTestId('login-button');
  }

  async navigate() {
    await this.page.goto('/login');
  }

  async fillEmail(email) {
    await this.emailInput.fill(email);
  }

  async fillPassword(password) {
    await this.passwordInput.fill(password);
  }

  async clickLogin() {
    await this.loginButton.click();
  }
}

module.exports = { LoginPage };
```

**`flows/auth.flow.js`**
```javascript
const { LoginPage } = require('../pages/LoginPage');

async function loginAs(page, { email, password }) {
  const loginPage = new LoginPage(page);
  await loginPage.navigate();
  await loginPage.fillEmail(email);
  await loginPage.fillPassword(password);
  await loginPage.clickLogin();
}

module.exports = { loginAs };
```

**`spec/login.spec.js`**
```javascript
const { test, expect } = require('@playwright/test');
const { loginAs } = require('../flows/auth.flow');

test.describe('Login', () => {
  test('should show dashboard when login with valid credentials @smoke @regression', async ({ page }) => {
    await loginAs(page, {
      email: 'test@example.com',
      password: 'Test@1234'
    });

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByTestId('user-name')).toHaveText('John Doe');
  });
});
```

Review it, adjust selectors to match your actual app, and run it.

---

## Step 4: Ask Dev for data-testid Attributes

If your app doesn't have `data-testid` attributes on elements, the automation will be fragile. This is the moment to start collaborating with developers.

### Create a Task for Your Dev Team

Ask Zen to help you write the request:

```
Help me write a clear developer request for adding data-testid attributes 
to our login page elements: email input, password input, login button, 
error message container.
```

Zen generates a clean Jira/GitHub ticket you can file immediately.

---

## Step 5: Set Up a Basic CI Pipeline

You don't need to build the full CI from scratch. Ask Zen:

```
Set up a GitHub Actions workflow that:
1. Runs Playwright tests on every pull request
2. Only runs @smoke tagged tests on PR (fast)
3. Runs @regression tests nightly
4. Uploads test reports as artifacts
5. Notifies on failure

Our project uses Node.js and Playwright.
```

Zen generates a complete `.github/workflows/test.yml` file. You review it, your team merges it, and you have automated testing in CI — without needing deep DevOps knowledge.

---

## Step 6: Debug Failing Tests with AI

When an automated test fails, don't panic. Use Zen:

```
My Playwright test is failing with this error:
[paste the error message and stack trace]

Here is the test code:
[paste the test]

What are the most likely causes and how do I fix it?
```

Common issues Zen will help you diagnose:
- **Selector not found** — element's testid changed, or page hasn't loaded
- **Timing issues** — network request still pending when assertion runs
- **Test data problems** — test account doesn't exist in current environment
- **State pollution** — previous test left the app in a bad state

---

## Step 7: Build Your First Regression Suite

Once you have 10–20 automated tests, organize them into a suite:

```
Here are my 15 automated test cases with their tags.
Help me organize them into:
1. Smoke suite (run on every PR, <5 minutes)
2. Regression suite (run nightly, <20 minutes)
3. Extended suite (run before releases)

Include recommended Playwright config settings for each suite.
```

---

## Advanced Habits: Sprint Cadence

| When | Action |
|---|---|
| Sprint start | Score new test cases for automation potential |
| During sprint | Convert top 2–3 candidates to automation |
| During sprint | Request data-testid additions from devs |
| End of sprint | Review flaky tests and fix or quarantine |
| Monthly | Refactor test files, remove duplicates |

---

## Flaky Test Policy

A flaky test is a bug. Treat it like one.

If a test fails intermittently:
1. Paste the failure to Zen: *"This test fails 1 in 5 runs. Here's the code and the error. What's causing the flakiness?"*
2. Fix within 48 hours, or **quarantine it** (mark non-blocking) until fixed
3. Never use retries to hide flakiness — fix the root cause

---

## Checkpoint: Are You Ready for Expert?

- [ ] You can score test cases using BPS/ACS and pick automation candidates
- [ ] You've converted at least 5 manual tests to Playwright scripts
- [ ] Your automation runs in CI (even if basic)
- [ ] You have a smoke suite and a regression suite
- [ ] You've filed at least one data-testid request to your dev team

→ **[Part 3: Expert — Strategy, Risk Analysis, and Scaling](./03-expert.md)**

---

*Part of the series: **AI-Assisted QA — From Manual Tester to Automation Strategist***
