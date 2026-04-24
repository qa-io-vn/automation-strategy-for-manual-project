# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · 03: Converting Your First Manual Test to Playwright

> **Series Navigation**
> [← 02 · Architecture](./02-architecture-explained.md) | **03 · First Test** | [04 · Debugging →](./04-debug-failing-tests.md)

---

## Purpose

This is the hands-on chapter. You'll take a real manual test case and convert it, step by step, into three working Playwright files — a Page Object, a Flow, and a Spec. Every step includes the exact code to write and the exact Zen prompts to use when you get stuck.

By the end, you'll have a running Playwright test you can execute right now.

---

## The Manual Test We're Converting

Here's the manual test case we'll automate. This is representative of what you'd find in TestRail, Jira, or a spreadsheet.

```
Test Case ID: TC-AUTH-001
Title: Login with valid credentials
Priority: Critical
Module: Authentication

Preconditions:
  - Application is running at staging URL
  - Test user exists: testuser@example.com / TestPass123!

Steps:
  1. Navigate to the login page (/login)
  2. Enter "testuser@example.com" in the Email field
  3. Enter "TestPass123!" in the Password field
  4. Click the "Sign In" button

Expected Results:
  - User is redirected to /dashboard
  - Page title shows "Dashboard — MyApp"
  - Welcome message is visible: "Welcome back, Test User"
  - User's name appears in the top navigation bar
```

We'll also automate its negative test:

```
Test Case ID: TC-AUTH-002
Title: Login with invalid password
Priority: High
Module: Authentication

Steps:
  1. Navigate to the login page (/login)
  2. Enter "testuser@example.com" in the Email field
  3. Enter "wrongpassword" in the Password field
  4. Click the "Sign In" button

Expected Results:
  - User remains on /login page
  - Error message appears: "Invalid email or password. Please try again."
  - Password field is cleared
  - Email field retains the entered value
```

---

## Step 1: Set Up Your Project

If you haven't already, initialize Playwright in a new project:

```bash
mkdir my-qa-project && cd my-qa-project
npm init -y
npm init playwright@latest
```

Choose these options when prompted:
- Language: **JavaScript**
- Test directory: `tests`
- GitHub Actions: **Yes** (we'll use this in the CI section)
- Install browsers: **Yes**

Then create the folder structure:

```bash
mkdir -p tests/specs/auth tests/flows tests/pages
```

Your structure should now look like:

```
my-qa-project/
├── playwright.config.js
├── package.json
└── tests/
    ├── specs/
    │   └── auth/
    ├── flows/
    └── pages/
```

---

## Step 2: Configure playwright.config.js

Open `playwright.config.js` and update it:

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/specs',

  // Fail the build if any test is accidentally left as test.only
  forbidOnly: !!process.env.CI,

  // Retry twice on CI, zero locally
  retries: process.env.CI ? 2 : 0,

  // Use 4 workers on CI, default locally
  workers: process.env.CI ? 4 : undefined,

  // Reporter: HTML for CI artifacts, list for local terminal output
  reporter: process.env.CI
    ? [['html', { open: 'never' }], ['github']]
    : [['html', { open: 'on-failure' }], ['list']],

  use: {
    // Base URL — set in environment or default to staging
    baseURL: process.env.BASE_URL || 'https://staging.myapp.com',

    // Collect traces on first retry (very useful for debugging)
    trace: 'on-first-retry',

    // Take screenshot on failure
    screenshot: 'only-on-failure',

    // Record video on first retry
    video: 'on-first-retry',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    // Mobile testing
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
  ],
});
```

---

## Step 3: Find Your Selectors

Before writing any code, identify stable selectors for the elements you'll interact with.

### Ask Zen to Help Identify Good Selectors

```
I'm writing a Playwright test for a login page. I need to interact with:
1. Email input field
2. Password input field  
3. "Sign In" submit button
4. Error message that appears on failed login

Which selector strategies should I use? My HTML looks like this:
<form class="login-form">
  <input type="email" id="email" name="email" placeholder="Email address" />
  <input type="password" id="password" name="password" placeholder="Password" />
  <button type="submit" class="btn-primary">Sign In</button>
  <div class="alert alert-danger hidden" role="alert">
    Invalid email or password. Please try again.
  </div>
</form>

Are there better selectors I should ask our developers to add?
```

**Zen's guidance will tell you:**
- Use `page.getByLabel()` or `page.getByPlaceholder()` when ARIA labels exist
- Use `page.getByRole('button', { name: 'Sign In' })` for buttons — robust and accessible
- Ask developers to add `data-testid` attributes for long-term stability

### Recommended Selector Priority

| Priority | Selector Type | Example | When to Use |
|----------|--------------|---------|-------------|
| 1st | `data-testid` | `[data-testid="email-input"]` | Best — stable, explicit |
| 2nd | ARIA role + name | `getByRole('button', { name: 'Sign In' })` | Great for interactive elements |
| 3rd | Label text | `getByLabel('Email address')` | Good for form fields |
| 4th | Placeholder | `getByPlaceholder('Email address')` | Acceptable fallback |
| 5th | Test ID pattern | `getByTestId('email-input')` | Same as #1 (Playwright shorthand) |
| Avoid | CSS class | `.btn-primary` | Changes for styling reasons |
| Avoid | XPath | `//button[1]` | Fragile, hard to read |
| Never | Position | `nth-child(3)` | Breaks when DOM changes |

### Using Playwright Codegen to Find Selectors

Playwright can record interactions and suggest selectors automatically:

```bash
npx playwright codegen https://staging.myapp.com/login
```

This opens a browser. Click the elements you want to target. Playwright shows you the recommended selectors in real time. This is the fastest way to get started.

---

## Step 4: Write the Page Object — LoginPage.js

Now write the Page Object for the login page.

```javascript
// tests/pages/LoginPage.js

export class LoginPage {
  /**
   * @param {import('@playwright/test').Page} page
   */
  constructor(page) {
    this.page = page;

    // Locators — update these to match your actual selectors
    // Best case: use data-testid attributes your devs have added
    this.emailInput = page.getByTestId('email-input');
    this.passwordInput = page.getByTestId('password-input');
    this.submitButton = page.getByRole('button', { name: 'Sign In' });
    this.errorMessage = page.getByTestId('login-error');
    this.forgotPasswordLink = page.getByRole('link', { name: 'Forgot password?' });
    this.rememberMeCheckbox = page.getByLabel('Remember me');

    // If your app doesn't have data-testid yet, use these fallbacks:
    // this.emailInput = page.getByPlaceholder('Email address');
    // this.passwordInput = page.getByPlaceholder('Password');
    // this.submitButton = page.getByRole('button', { name: 'Sign In' });
    // this.errorMessage = page.locator('[role="alert"]');
  }

  /**
   * Navigate to the login page
   */
  async navigate() {
    await this.page.goto('/login');
    // Wait for the form to be ready
    await this.emailInput.waitFor({ state: 'visible' });
  }

  /**
   * Fill the email input
   * @param {string} email
   */
  async fillEmail(email) {
    await this.emailInput.clear();
    await this.emailInput.fill(email);
  }

  /**
   * Fill the password input
   * @param {string} password
   */
  async fillPassword(password) {
    await this.passwordInput.clear();
    await this.passwordInput.fill(password);
  }

  /**
   * Click the Sign In button
   */
  async submit() {
    await this.submitButton.click();
  }

  /**
   * Complete the login form in one call
   * @param {string} email
   * @param {string} password
   */
  async login(email, password) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.submit();
  }

  /**
   * Get the text content of the error message
   * @returns {Promise<string>}
   */
  async getErrorMessage() {
    await this.errorMessage.waitFor({ state: 'visible' });
    return this.errorMessage.textContent();
  }

  /**
   * Check if the error message is visible
   * @returns {Promise<boolean>}
   */
  async hasError() {
    return this.errorMessage.isVisible();
  }

  /**
   * Get the current value of the email input
   * @returns {Promise<string>}
   */
  async getEmailValue() {
    return this.emailInput.inputValue();
  }

  /**
   * Get the current value of the password input
   * @returns {Promise<string>}
   */
  async getPasswordValue() {
    return this.passwordInput.inputValue();
  }

  /**
   * Click "Forgot password?" link
   */
  async clickForgotPassword() {
    await this.forgotPasswordLink.click();
  }
}
```

### Ask Zen to Write This For You

If you have the HTML of your login page, paste it to Zen:

```
Here is the HTML of my login page:
[paste your login page HTML here]

Please write a Playwright Page Object class called LoginPage.js.
Requirements:
- Use the best available selectors (prefer data-testid, then ARIA roles)
- Define all locators in the constructor
- One method per user action
- Include a navigate() method
- Include a login(email, password) convenience method
- No assertions in the page object
- Add JSDoc comments for each method
```

---

## Step 5: Write the Flow File — auth.flow.js

```javascript
// tests/flows/auth.flow.js
import { LoginPage } from '../pages/LoginPage.js';

// Test credentials — in a real project, load these from environment variables
const TEST_USERS = {
  standard: {
    email: process.env.TEST_USER_EMAIL || 'testuser@example.com',
    password: process.env.TEST_USER_PASSWORD || 'TestPass123!',
    name: 'Test User',
  },
  admin: {
    email: process.env.ADMIN_USER_EMAIL || 'admin@example.com',
    password: process.env.ADMIN_USER_PASSWORD || 'AdminPass123!',
    name: 'Admin User',
  },
  readonly: {
    email: 'readonly@example.com',
    password: 'ReadOnly123!',
    name: 'Read Only User',
  },
};

export class AuthFlow {
  /**
   * @param {import('@playwright/test').Page} page
   */
  constructor(page) {
    this.page = page;
    this.loginPage = new LoginPage(page);
  }

  /**
   * Log in as the standard test user (valid credentials)
   */
  async loginWithValidUser() {
    const user = TEST_USERS.standard;
    await this.loginPage.navigate();
    await this.loginPage.login(user.email, user.password);
    // Wait for navigation to complete
    await this.page.waitForURL('**/dashboard');
  }

  /**
   * Attempt login with a valid email but wrong password
   */
  async loginWithInvalidPassword() {
    await this.loginPage.navigate();
    await this.loginPage.fillEmail(TEST_USERS.standard.email);
    await this.loginPage.fillPassword('this-is-the-wrong-password');
    await this.loginPage.submit();
    // Wait for error to appear (don't wait for navigation since login fails)
    await this.page.waitForTimeout(500); // brief pause for error to render
  }

  /**
   * Log in as a specific role
   * @param {'standard' | 'admin' | 'readonly'} role
   */
  async loginAs(role) {
    const user = TEST_USERS[role];
    if (!user) {
      throw new Error(`Unknown role: "${role}". Valid roles: ${Object.keys(TEST_USERS).join(', ')}`);
    }
    await this.loginPage.navigate();
    await this.loginPage.login(user.email, user.password);
    await this.page.waitForURL('**/dashboard');
  }

  /**
   * Log in with custom credentials (for edge case testing)
   * @param {string} email
   * @param {string} password
   */
  async loginWithCredentials(email, password) {
    await this.loginPage.navigate();
    await this.loginPage.login(email, password);
  }

  /**
   * Log out the current user
   */
  async logout() {
    await this.page.getByTestId('user-menu').click();
    await this.page.getByRole('menuitem', { name: 'Sign out' }).click();
    await this.page.waitForURL('**/login');
  }

  /**
   * Check if a user is currently logged in
   * @returns {Promise<boolean>}
   */
  async isLoggedIn() {
    return this.page.getByTestId('user-menu').isVisible();
  }
}
```

---

## Step 6: Write the Spec File — login.spec.js

```javascript
// tests/specs/auth/login.spec.js
import { test, expect } from '@playwright/test';
import { AuthFlow } from '../../flows/auth.flow.js';

test.describe('Authentication — Login', () => {
  // ============================================================
  // SMOKE TESTS — Run on every PR (fast, critical path only)
  // ============================================================

  test('@smoke TC-AUTH-001: valid credentials → redirected to dashboard', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithValidUser();

    await expect(page).toHaveURL(/.*\/dashboard/);
    await expect(page).toHaveTitle(/Dashboard/);
  });

  // ============================================================
  // REGRESSION TESTS — Run nightly and on release branches
  // ============================================================

  test('@regression TC-AUTH-001b: welcome message visible after login', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithValidUser();

    await expect(page.getByTestId('welcome-message')).toBeVisible();
    await expect(page.getByTestId('welcome-message')).toContainText('Welcome back, Test User');
  });

  test('@regression TC-AUTH-001c: user name in navigation after login', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithValidUser();

    await expect(page.getByTestId('nav-user-name')).toHaveText('Test User');
  });

  test('@regression TC-AUTH-002: invalid password → error message shown', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithInvalidPassword();

    await expect(page).toHaveURL(/.*\/login/);
    await expect(page.getByTestId('login-error')).toBeVisible();
    await expect(page.getByTestId('login-error')).toHaveText(
      'Invalid email or password. Please try again.'
    );
  });

  test('@regression TC-AUTH-002b: email retained after failed login', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithInvalidPassword();

    const loginPage = auth.loginPage;
    await expect(loginPage.emailInput).toHaveValue('testuser@example.com');
  });

  test('@regression TC-AUTH-002c: password cleared after failed login', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithInvalidPassword();

    const loginPage = auth.loginPage;
    await expect(loginPage.passwordInput).toHaveValue('');
  });

  test('@regression TC-AUTH-003: empty email → validation error', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginPage.navigate();
    await auth.loginPage.fillPassword('somepassword');
    await auth.loginPage.submit();

    await expect(page.getByTestId('email-validation-error')).toBeVisible();
    await expect(page.getByTestId('email-validation-error')).toContainText(
      'Email is required'
    );
  });

  test('@regression TC-AUTH-004: malformed email → validation error', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginPage.navigate();
    await auth.loginWithCredentials('not-an-email', 'somepassword');

    await expect(page.getByTestId('email-validation-error')).toContainText(
      'Please enter a valid email address'
    );
  });

  // ============================================================
  // EXTENDED TESTS — Run weekly or before major releases
  // ============================================================

  test('@extended TC-AUTH-005: login page loads within 2 seconds', async ({ page }) => {
    const auth = new AuthFlow(page);

    const startTime = Date.now();
    await auth.loginPage.navigate();
    const loadTime = Date.now() - startTime;

    await expect(auth.loginPage.emailInput).toBeVisible();
    expect(loadTime).toBeLessThan(2000);
  });

  test('@extended TC-AUTH-006: admin user can access admin panel after login', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginAs('admin');

    await page.goto('/admin');
    await expect(page).toHaveURL(/.*\/admin/);
    await expect(page.getByTestId('admin-panel-header')).toBeVisible();
  });

  test('@extended TC-AUTH-007: standard user cannot access admin panel', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginAs('standard');

    await page.goto('/admin');
    // Should be redirected away or shown an access denied page
    await expect(page).not.toHaveURL(/.*\/admin/);
  });
});
```

---

## Step 7: Handle Test Data Properly

Hard-coding credentials in your code is a security risk and maintenance burden. Here's how to handle test data correctly.

### Option A: Environment Variables (Recommended for CI)

```bash
# .env.test (never commit this file — add to .gitignore)
BASE_URL=https://staging.myapp.com
TEST_USER_EMAIL=testuser@example.com
TEST_USER_PASSWORD=TestPass123!
ADMIN_USER_EMAIL=admin@example.com
ADMIN_USER_PASSWORD=AdminPass123!
```

Load them in your config:

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config({ path: '.env.test' });

export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL || 'https://staging.myapp.com',
  },
});
```

### Option B: Test Data File (for Non-Sensitive Data)

```json
// test-data/users.json
{
  "standard": {
    "name": "Test User",
    "email": "testuser@example.com",
    "role": "user"
  },
  "admin": {
    "name": "Admin User",
    "email": "admin@example.com",
    "role": "admin"
  }
}
```

### Option C: API-Created Test Users

For the most robust setup, create test users via the API before each test:

```javascript
// tests/helpers/api.helper.js
export async function createTestUser(request) {
  const response = await request.post('/api/test/users', {
    data: {
      email: `test-${Date.now()}@example.com`,
      password: 'TestPass123!',
      role: 'user',
    },
  });
  return response.json();
}

export async function deleteTestUser(request, userId) {
  await request.delete(`/api/test/users/${userId}`);
}
```

---

## Step 8: Run and Verify Your Test

### Run All Login Tests

```bash
npx playwright test tests/specs/auth/login.spec.js
```

### Run Only Smoke Tests

```bash
npx playwright test --grep @smoke
```

### Run in Headed Mode (Watch the Browser)

```bash
npx playwright test tests/specs/auth/login.spec.js --headed
```

### Run in Debug Mode (Step Through Line by Line)

```bash
npx playwright test tests/specs/auth/login.spec.js --debug
```

### View the HTML Report

```bash
npx playwright show-report
```

---

## Step 9: Add Tags and Descriptions

Tags (`@smoke`, `@regression`, `@extended`) are how you run subsets of your suite. Review the tags in the spec file above. Good tagging from the start saves hours later.

### Tagging Strategy Summary

| Tag | When to Run | Tests Included |
|-----|-------------|---------------|
| `@smoke` | Every PR, every deploy | Critical path only — "does it start?" |
| `@regression` | Nightly, before releases | All verified functionality |
| `@extended` | Weekly, pre-release | Performance, edge cases, cross-browser |
| `@module-auth` | When auth code changes | All auth tests specifically |

---

## Common Issues When Running Your First Test

| Error | Cause | Fix |
|-------|-------|-----|
| `Error: baseURL is not set` | Missing config | Set `baseURL` in `playwright.config.js` |
| `Timeout: waiting for locator` | Selector doesn't match | Use `--debug` to inspect the page, check data-testid values |
| `Error: Cannot find module '../../flows/auth.flow.js'` | Wrong import path | Check relative paths match your folder structure |
| Test passes but URL assertion fails | `waitForURL` needed | Add `await page.waitForURL(...)` after clicking submit |
| All tests fail on CI but pass locally | Different BASE_URL | Set `BASE_URL` environment variable in CI |

---

## Summary of Files Created

```
tests/
├── pages/
│   └── LoginPage.js          ← Knows all selectors for /login page
├── flows/
│   └── auth.flow.js          ← Orchestrates login journeys
└── specs/
    └── auth/
        └── login.spec.js     ← Describes WHAT should happen, asserts outcomes
```

You now have a complete, runnable, well-structured set of tests for authentication. These three files follow the architecture from the previous chapter and can grow with your application.

---

**Next:** [04 · Debugging Failing Tests →](./04-debug-failing-tests.md) — What to do when your tests fail (and they will).
