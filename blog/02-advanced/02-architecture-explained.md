# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · 02: The Architecture — Spec → Flow → Page Object

> **Series Navigation**
> [← 01 · What to Automate](./01-what-to-automate.md) | **02 · Architecture** | [03 · First Test →](./03-convert-first-test.md)

---

## Purpose

This guide explains the **three-layer architecture** used in professional Playwright test suites. You don't need to be a developer to understand or apply it — you just need to understand *why* each layer exists and what problem it solves.

By the end, you'll understand:
- Why flat, unstructured tests are a maintenance trap
- What Spec, Flow, and Page Object files each do
- How SOLID principles apply to your test code (in plain English)
- The exact anti-patterns that cause test suites to collapse under their own weight

---

## The Analogy: A Restaurant Kitchen

Think of a restaurant kitchen. It has three distinct roles:

| Kitchen Role | Test Architecture Role | Responsibility |
|-------------|----------------------|----------------|
| **The Menu** (what customers order) | **Spec file** (.spec.js) | Describes *what* is being tested |
| **The Recipe** (step-by-step instructions) | **Flow file** (.flow.js) | Describes *how* a multi-step journey works |
| **The Station** (grill, prep, pastry) | **Page Object** (Page.js) | Knows every interaction with a specific page |

The chef (your spec) doesn't need to know how the grill (page object) works. The chef just says "grill the steak" and the grill station handles it. This separation is the entire point.

---

## Layer 1: The Spec File

**What it is:** The test file. This is the entry point. It contains `test()` blocks.

**What it knows about:** What you're testing and what the expected outcome is.

**What it does NOT know about:** How to interact with any specific UI element. It doesn't click buttons or fill forms directly.

**Naming:** `feature-name.spec.js` (e.g., `login.spec.js`, `checkout.spec.js`)

### A Good Spec File Reads Like a Requirement

```javascript
// tests/specs/auth/login.spec.js
import { test, expect } from '@playwright/test';
import { AuthFlow } from '../../flows/auth.flow.js';

test.describe('Login', () => {
  test('@smoke user can log in with valid credentials', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithValidUser();

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByTestId('welcome-message')).toBeVisible();
  });

  test('@regression user sees error with invalid password', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithInvalidPassword();

    await expect(page.getByTestId('login-error')).toHaveText(
      'Invalid email or password. Please try again.'
    );
  });

  test('@regression login form is pre-filled after failed attempt', async ({ page }) => {
    const auth = new AuthFlow(page);

    await auth.loginWithInvalidPassword();

    await expect(page.getByTestId('email-input')).toHaveValue('testuser@example.com');
  });
});
```

**Notice:** The spec file has zero knowledge of CSS selectors, URLs, or form filling mechanics. It just calls `auth.loginWithValidUser()` and asserts the outcome. It reads like plain English.

---

## Layer 2: The Flow File

**What it is:** A helper that orchestrates multi-step user journeys.

**What it knows about:** The sequence of actions needed to complete a user journey. It delegates each page interaction to the appropriate Page Object.

**What it does NOT know about:** Specific CSS selectors, element positions, or page-level implementation details.

**Naming:** `feature-name.flow.js` (e.g., `auth.flow.js`, `checkout.flow.js`)

### A Good Flow File is a Sequence of Page-Level Actions

```javascript
// tests/flows/auth.flow.js
import { LoginPage } from '../pages/LoginPage.js';

export class AuthFlow {
  constructor(page) {
    this.page = page;
    this.loginPage = new LoginPage(page);
  }

  async loginWithValidUser() {
    await this.loginPage.navigate();
    await this.loginPage.fillEmail('testuser@example.com');
    await this.loginPage.fillPassword('ValidPass123!');
    await this.loginPage.submit();
  }

  async loginWithInvalidPassword() {
    await this.loginPage.navigate();
    await this.loginPage.fillEmail('testuser@example.com');
    await this.loginPage.fillPassword('wrong-password');
    await this.loginPage.submit();
  }

  async loginAs(role) {
    const credentials = {
      admin: { email: 'admin@example.com', password: 'AdminPass123!' },
      user: { email: 'user@example.com', password: 'UserPass123!' },
      readonly: { email: 'readonly@example.com', password: 'ReadPass123!' },
    };

    const creds = credentials[role];
    await this.loginPage.navigate();
    await this.loginPage.fillEmail(creds.email);
    await this.loginPage.fillPassword(creds.password);
    await this.loginPage.submit();
  }
}
```

**Why flows are separate from specs:** A checkout flow might be used in dozens of tests — as setup, as verification, as part of a longer journey. Keeping it in a flow file means every test reuses the same logic. When checkout changes, you update one flow file, not dozens of specs.

---

## Layer 3: The Page Object

**What it is:** A JavaScript class that represents one page (or one logical section of a page) in your application.

**What it knows about:** Every interaction with that page — how to fill each input, where each button is, what locators to use.

**What it does NOT know about:** What test is using it, what the expected outcome is, or what happens on other pages.

**Naming:** `PageName.js` (e.g., `LoginPage.js`, `CheckoutPage.js`, `DashboardPage.js`)

### A Good Page Object Encapsulates Every Interaction

```javascript
// tests/pages/LoginPage.js
export class LoginPage {
  constructor(page) {
    this.page = page;

    // Define all locators in the constructor
    this.emailInput = page.getByTestId('email-input');
    this.passwordInput = page.getByTestId('password-input');
    this.submitButton = page.getByTestId('login-submit-btn');
    this.errorMessage = page.getByTestId('login-error');
    this.forgotPasswordLink = page.getByRole('link', { name: 'Forgot password?' });
    this.rememberMeCheckbox = page.getByLabel('Remember me');
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

  async submit() {
    await this.submitButton.click();
  }

  async login(email, password) {
    await this.fillEmail(email);
    await this.fillPassword(password);
    await this.submit();
  }

  async getErrorMessage() {
    return this.errorMessage.textContent();
  }

  async clickForgotPassword() {
    await this.forgotPasswordLink.click();
  }
}
```

---

## BAD vs. GOOD: The Same Test, Two Ways

This is the most important comparison in this entire guide. Both versions test the same thing. Only one is maintainable.

### BAD Version: The "Script Dump"

```javascript
// ❌ tests/login-bad.spec.js
// This is how most automation beginners write tests

import { test, expect } from '@playwright/test';

test('login test', async ({ page }) => {
  await page.goto('https://myapp.com/login');
  await page.fill('input[type="email"]', 'testuser@example.com');
  await page.fill('input[type="password"]', 'ValidPass123!');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('https://myapp.com/dashboard');
});

test('login with wrong password', async ({ page }) => {
  await page.goto('https://myapp.com/login');                           // duplicated
  await page.fill('input[type="email"]', 'testuser@example.com');      // duplicated
  await page.fill('input[type="password"]', 'wrong');                  // different
  await page.click('button[type="submit"]');                           // duplicated
  await expect(page.locator('.error-text')).toContainText('Invalid');  // fragile selector
});

test('dashboard shows after login', async ({ page }) => {
  await page.goto('https://myapp.com/login');                           // duplicated again
  await page.fill('input[type="email"]', 'testuser@example.com');      // duplicated again
  await page.fill('input[type="password"]', 'ValidPass123!');          // duplicated again
  await page.click('button[type="submit"]');                           // duplicated again
  await page.waitForURL('https://myapp.com/dashboard');
  await expect(page.locator('h1')).toContainText('Welcome');           // fragile selector
});
```

**What goes wrong with the BAD version:**

| Problem | Impact |
|---------|--------|
| URL hardcoded 6 times | When the URL changes, update 6 places (and miss one) |
| Selectors hardcoded in tests | When HTML changes, every test breaks |
| Login steps duplicated 3 times | When login changes, update 3 places |
| No clear test description | Can't tell what behavior is being verified |
| CSS class selectors (`.error-text`) | Breaks when developer renames class for styling |

### GOOD Version: The Layered Architecture

```javascript
// ✅ tests/specs/auth/login.spec.js

import { test, expect } from '@playwright/test';
import { AuthFlow } from '../../flows/auth.flow.js';

test.describe('Login', () => {
  let auth;

  test.beforeEach(async ({ page }) => {
    auth = new AuthFlow(page);
  });

  test('@smoke valid credentials → redirects to dashboard', async ({ page }) => {
    await auth.loginWithValidUser();
    await expect(page).toHaveURL('/dashboard');
  });

  test('@regression invalid password → shows error message', async ({ page }) => {
    await auth.loginWithInvalidPassword();
    await expect(page.getByTestId('login-error')).toContainText('Invalid email or password');
  });

  test('@regression welcome message visible after login', async ({ page }) => {
    await auth.loginWithValidUser();
    await expect(page.getByTestId('welcome-message')).toBeVisible();
  });
});
```

**What's different:**

| Improvement | Benefit |
|------------|---------|
| All login logic in `AuthFlow` | Change login once, affects all tests |
| All selectors in `LoginPage` | HTML changes → update one file |
| Tests read like requirements | A non-technical stakeholder can understand them |
| Tags (@smoke, @regression) | Run subsets of tests easily |
| `beforeEach` for setup | DRY — Don't Repeat Yourself |

---

## SOLID Principles in Plain English (For Testers)

SOLID is a set of software design principles. For test code, think of them as rules that prevent your suite from becoming unmaintainable.

### S — Single Responsibility Principle
**Plain English:** Each file/class should do exactly one thing.

- ❌ BAD: `LoginPage.js` also handles the forgot-password email flow
- ✅ GOOD: `LoginPage.js` handles login. `ForgotPasswordPage.js` handles forgot password.

**Test code example:**
```javascript
// ❌ Too much in one place
class LoginPage {
  async login() { ... }
  async checkInbox() { ... }         // Wrong! This is email, not login page.
  async verifyWelcomeMessage() { ... } // Wrong! This is dashboard, not login page.
}

// ✅ Each class has one responsibility
class LoginPage { async login() { ... } }
class EmailInboxPage { async checkInbox() { ... } }
class DashboardPage { async verifyWelcomeMessage() { ... } }
```

### O — Open/Closed Principle
**Plain English:** Add new behaviors without changing existing working code.

```javascript
// ✅ The AuthFlow class is open for extension (add loginAs(role))
// without modifying the existing loginWithValidUser() method
class AuthFlow {
  async loginWithValidUser() { ... }  // Don't touch this when adding new login variants
  async loginWithInvalidPassword() { ... }
  async loginAs(role) { ... }         // New behavior added, old methods untouched
}
```

### L — Liskov Substitution Principle
**Plain English (for testers):** If you have a `BasePage`, every page that inherits from it should work wherever `BasePage` is expected.

```javascript
class BasePage {
  constructor(page) {
    this.page = page;
  }

  async navigate(path) {
    await this.page.goto(path);
  }

  async getTitle() {
    return this.page.title();
  }
}

class LoginPage extends BasePage {
  constructor(page) {
    super(page);
    // LoginPage inherits navigate() and getTitle() — they work the same
  }
}
```

### I — Interface Segregation Principle
**Plain English:** Don't force a class to implement methods it doesn't need.

- ❌ BAD: A `ProductPage` that also has methods for checkout, user settings, and admin panels
- ✅ GOOD: `ProductPage`, `CheckoutPage`, `SettingsPage`, `AdminPage` — each with only their own methods

### D — Dependency Inversion Principle
**Plain English:** High-level code (spec) shouldn't depend directly on low-level code (selectors). Both should depend on abstractions (page objects and flows).

```
Spec file ──relies on──→ Flow file ──relies on──→ Page Object ──knows about──→ Selectors
```

If selectors change, only the Page Object needs to change. The Flow and Spec are unaffected.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                      SPEC FILE                          │
│  login.spec.js                                          │
│  • Describes WHAT to test                               │
│  • Contains assertions (expect)                         │
│  • Calls flows, never touches UI directly               │
└──────────────────────┬──────────────────────────────────┘
                       │ calls
                       ▼
┌─────────────────────────────────────────────────────────┐
│                      FLOW FILE                          │
│  auth.flow.js                                           │
│  • Orchestrates multi-step journeys                     │
│  • Knows WHICH pages are involved                       │
│  • Does NOT know about HTML/selectors                   │
└──────────────────────┬──────────────────────────────────┘
                       │ calls
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   PAGE OBJECT FILE                      │
│  LoginPage.js                                           │
│  • Knows ALL interactions with one page                 │
│  • Owns all selectors for that page                     │
│  • Returns simple values, never assertions              │
└──────────────────────┬──────────────────────────────────┘
                       │ interacts with
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   YOUR APPLICATION                      │
│  The actual browser, DOM, and UI                        │
└─────────────────────────────────────────────────────────┘
```

---

## Anti-Patterns to Avoid

These are the architectural mistakes that cause test suites to become unmaintainable.

### Anti-Pattern 1: Test Logic in Page Objects

```javascript
// ❌ Page objects should NOT contain assertions
class LoginPage {
  async verifyLoginSuccess() {
    await expect(this.page).toHaveURL('/dashboard'); // WRONG — this is test logic
  }
}
```

**Why it's wrong:** Page objects are about *doing* things, not *verifying* things. Assertions belong in spec files. If you put assertions in page objects, you can't reuse the page object for different test scenarios.

### Anti-Pattern 2: Sleeping Instead of Waiting

```javascript
// ❌ Never use fixed sleeps
await page.waitForTimeout(3000); // "Hope the page loads in 3 seconds"

// ✅ Always wait for a specific condition
await page.waitForURL('/dashboard');
await expect(page.getByTestId('dashboard-header')).toBeVisible();
```

**Why it's wrong:** Fixed sleeps make tests slow on fast machines and flaky on slow ones. Playwright's auto-waiting handles most cases automatically.

### Anti-Pattern 3: God Objects

```javascript
// ❌ One page object for the entire app
class AppPage {
  async login() { ... }
  async addToCart() { ... }
  async checkout() { ... }
  async viewProfile() { ... }
  async adminBanUser() { ... }
  // 200 more methods...
}
```

**Why it's wrong:** Impossible to maintain, impossible to find methods, breaks Single Responsibility. Split by page/feature.

### Anti-Pattern 4: Magic Strings Everywhere

```javascript
// ❌ Hardcoded strings spread across files
await page.goto('https://staging.myapp.com/login'); // hardcoded URL
await page.fill('#email-field-v2', user.email);     // CSS class that changes

// ✅ Centralized configuration + semantic selectors
await page.goto(process.env.BASE_URL + '/login');
await page.getByTestId('email-input').fill(user.email);
```

### Anti-Pattern 5: Tests That Depend on Each Other

```javascript
// ❌ Test 2 only works if Test 1 ran first
test('create user', async ({ page }) => { ... });
test('delete user', async ({ page }) => { /* relies on user from previous test */ });

// ✅ Each test is fully independent
test('create user', async ({ page }) => { ... });
test('delete user', async ({ page }) => {
  // Creates its own user first, then deletes it
  const userId = await api.createTestUser();
  await deleteUserFlow.run(page, userId);
  await api.cleanup(userId); // probably not needed if delete worked, but safe
});
```

### Anti-Pattern 6: Skipping the Flow Layer

```javascript
// ❌ Spec directly uses Page Object (acceptable for simple tests, 
//    but breaks down as tests get complex)
test('checkout', async ({ page }) => {
  const login = new LoginPage(page);
  await login.navigate();
  await login.fillEmail('user@test.com');
  await login.fillPassword('pass');
  await login.submit();

  const cart = new CartPage(page);
  await cart.addItem('product-123');
  // ... 50 more lines in the spec
});

// ✅ Spec uses flow, which coordinates page objects
test('checkout', async ({ page }) => {
  const checkout = new CheckoutFlow(page);
  await checkout.completePurchaseAs('standard_user');
  await expect(page.getByTestId('order-confirmation')).toBeVisible();
});
```

---

## When Is the Three-Layer Architecture Overkill?

For very small suites (< 10 tests), the three-layer approach can feel heavy. Here's the guidance:

| Suite Size | Recommended Architecture |
|-----------|--------------------------|
| 1–5 tests | Spec + Page Object (skip Flow layer) |
| 5–20 tests | Spec + Flow + Page Object |
| 20+ tests | Full three-layer + fixtures + helpers |

Even for small suites, **always use Page Objects** — they pay off the first time a selector changes.

---

## How to Ask Zen to Help With Architecture

### Ask Zen to Create a Page Object

```
I'm building a Page Object for the checkout page of an e-commerce app.
The page has:
- An order summary section (shows item name, quantity, price)
- A "Proceed to Payment" button (data-testid="checkout-pay-btn")
- A promo code input (data-testid="promo-code-input")
- An "Apply" button for the promo code (data-testid="promo-apply-btn")
- An error message if promo code is invalid (data-testid="promo-error")
- A total price display (data-testid="order-total")

Please write a CheckoutPage.js Page Object class following the pattern:
- Locators defined in constructor
- One method per action
- No assertions in the page object
- Methods return 'this' for chainability where appropriate
```

### Ask Zen to Review Your Architecture

```
Here is my current test file structure. Can you identify any architectural
issues and suggest improvements? Specifically check for:
- Are assertions in the right layer?
- Is there duplicated code that should be in flows?
- Are selectors stable enough?
- Does each file follow Single Responsibility?

[paste your code]
```

---

## Summary

| Layer | File | Knows About | Does NOT Know About |
|-------|------|-------------|---------------------|
| **Spec** | `login.spec.js` | What to test, expected outcomes | Selectors, URLs, page mechanics |
| **Flow** | `auth.flow.js` | User journey sequence, which pages | Selectors, HTML, CSS |
| **Page Object** | `LoginPage.js` | All selectors for one page | Which test is running, other pages |

The pattern pays off the moment something changes. And things always change.

---

**Next:** [03 · Convert Your First Test →](./03-convert-first-test.md) — Put this architecture into practice by converting a real manual test case into working Playwright code.
