# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · 05: Building a Full Regression Suite

> **Series Navigation**
> [← 04 · Debugging](./04-debug-failing-tests.md) | **05 · Regression Suite** | [CI/CD Integration →](./ci-tools/README.md)

---

## Purpose

You have a few working tests. Now what? This guide shows you how to grow from 5 tests to a full, organized, maintainable regression suite — without creating a maintenance nightmare.

The most common mistake teams make is writing tests fast without a growth strategy. By sprint 6, they have 200 tests with no tags, no organization, unclear ownership, and no way to run just the tests that matter for a given change.

This guide gives you the strategy before you need it.

---

## The Growth Phases

| Phase | Test Count | Focus | Risk |
|-------|-----------|-------|------|
| **Bootstrap** | 0–15 | Critical paths only | Too few tests to catch real bugs |
| **Expansion** | 15–60 | Module coverage | Duplication, no structure |
| **Maturity** | 60–150 | Depth + edge cases | Slow suites, maintenance burden |
| **Scale** | 150+ | Full regression | Flakiness, test data complexity |

Each phase has different priorities. Know which phase you're in.

---

## Phase 1: Bootstrap (0–15 Tests)

**Goal:** Prove automation works and cover the most critical path.

**What to automate:**
- Login / Authentication (every app has this)
- The #1 revenue/value action (checkout, send message, publish post, etc.)
- Any flow with a history of production incidents

**What NOT to worry about yet:**
- Cross-browser testing
- Mobile testing
- Edge cases
- Performance

**Sample bootstrap suite:**

```
tests/specs/
├── auth/
│   └── login.spec.js          (2 tests: valid + invalid login)
├── checkout/
│   └── checkout.spec.js       (1 test: happy path checkout)
└── smoke.spec.js              (3 tests: app loads, nav works, key pages accessible)
```

---

## Phase 2: Expansion (15–60 Tests)

**Goal:** Cover every major module with at least smoke-level coverage.

**Organize by module, not by type:**

```
tests/specs/
├── auth/
│   ├── login.spec.js
│   ├── registration.spec.js
│   └── password-reset.spec.js
├── catalog/
│   ├── product-list.spec.js
│   ├── product-detail.spec.js
│   └── search.spec.js
├── cart/
│   ├── add-to-cart.spec.js
│   └── cart-management.spec.js
├── checkout/
│   ├── checkout-payment.spec.js
│   └── checkout-shipping.spec.js
├── account/
│   ├── profile.spec.js
│   └── order-history.spec.js
└── admin/
    ├── user-management.spec.js
    └── product-management.spec.js
```

**During this phase, establish your tagging system** (detailed below). Retrofitting tags onto 60 tests is painful.

---

## Phase 3: Maturity (60–150 Tests)

**Goal:** Add depth — error states, edge cases, role-based access, data variations.

Each module should have:
- Happy path test(s) — `@smoke`
- Common error states — `@regression`
- Boundary conditions — `@regression`
- Role-based behavior (if applicable) — `@regression`
- Performance/load time check — `@extended`

**Example for the cart module:**

| Test | Tag | Description |
|------|-----|-------------|
| Add single item to cart | `@smoke` | Core happy path |
| Add maximum quantity | `@regression` | Boundary: qty limit |
| Exceed stock limit | `@regression` | Error state |
| Cart persists on page reload | `@regression` | State management |
| Cart visible for guest user | `@regression` | Role-based |
| Cart shows correct totals | `@regression` | Calculation accuracy |
| Cart updates when item removed | `@regression` | State mutation |
| Cart empty state | `@regression` | Edge case |
| Add item from search results | `@extended` | Cross-feature |

---

## Phase 4: Scale (150+ Tests)

**Goal:** Maintain fast feedback loops while expanding coverage.

At this scale, you need:
- Parallel execution (configured in `playwright.config.js`)
- Test sharding across CI workers
- A flakiness monitoring process
- A test data factory or API-based setup
- Ownership assignments per module

---

## The Tagging Strategy

Tags control which tests run when. A good tagging system makes your suite flexible without being complicated.

### Core Tags

| Tag | Description | When to Run |
|-----|-------------|-------------|
| `@smoke` | Critical path tests, fast | Every PR, every deploy |
| `@regression` | Full feature verification | Nightly, pre-release |
| `@extended` | Deep edge cases, performance | Weekly, pre-major release |
| `@flaky` | Quarantined unstable tests | Never in main pipelines |
| `@wip` | Work-in-progress tests | Local only, never in CI |

### Module Tags

Add a module tag to every test. This lets you run only the tests affected by a change.

| Tag | Module |
|-----|--------|
| `@module-auth` | Authentication |
| `@module-cart` | Shopping cart |
| `@module-checkout` | Checkout flow |
| `@module-catalog` | Product catalog / search |
| `@module-account` | User account / profile |
| `@module-admin` | Admin panel |

### Environment Tags

Some tests only make sense in certain environments:

| Tag | Description |
|-----|-------------|
| `@prod-safe` | Safe to run against production (read-only) |
| `@staging-only` | Requires staging-specific test data |
| `@requires-stripe` | Needs Stripe test keys configured |

### How to Apply Tags

```javascript
// Apply to individual tests
test('@smoke @module-auth TC-AUTH-001: valid login → dashboard', async ({ page }) => {
  // test body
});

// Apply to all tests in a describe block
test.describe('@module-checkout', () => {
  test('@smoke checkout happy path', async ({ page }) => { ... });
  test('@regression checkout with expired card', async ({ page }) => { ... });
  test('@regression checkout with declined card', async ({ page }) => { ... });
});

// Apply to entire file using describe metadata
test.describe.configure({ mode: 'parallel' }); // run tests in this file in parallel

test.describe('@module-catalog @regression Product Search', () => {
  // All tests here are @module-catalog and @regression
  test('@smoke search by keyword', async ({ page }) => { ... });
  test('no results state', async ({ page }) => { ... });
  test('search with filters applied', async ({ page }) => { ... });
});
```

### Running Tagged Subsets

```bash
# Only smoke tests
npx playwright test --grep @smoke

# Smoke + regression, any module
npx playwright test --grep "@smoke|@regression"

# Only auth module regression tests
npx playwright test --grep "@regression.*@module-auth|@module-auth.*@regression"

# Exclude flaky tests from regression run
npx playwright test --grep @regression --grep-invert @flaky

# Only tests affected by cart changes
npx playwright test --grep @module-cart
```

---

## Playwright Configuration for Multiple Suites

Use projects in `playwright.config.js` to define suite configurations:

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/specs',
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : 2,

  reporter: [
    ['html', { open: 'never', outputFolder: 'playwright-report' }],
    ['json', { outputFile: 'test-results/results.json' }],
    ['list'],
  ],

  use: {
    baseURL: process.env.BASE_URL || 'https://staging.myapp.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },

  projects: [
    // Smoke suite — fast, chromium only
    {
      name: 'smoke-chromium',
      use: { ...devices['Desktop Chrome'] },
      grep: /@smoke/,
    },

    // Full regression — all three major browsers
    {
      name: 'regression-chromium',
      use: { ...devices['Desktop Chrome'] },
      grep: /@regression/,
    },
    {
      name: 'regression-firefox',
      use: { ...devices['Desktop Firefox'] },
      grep: /@regression/,
    },
    {
      name: 'regression-webkit',
      use: { ...devices['Desktop Safari'] },
      grep: /@regression/,
    },

    // Extended suite — chromium + mobile
    {
      name: 'extended-chromium',
      use: { ...devices['Desktop Chrome'] },
      grep: /@extended/,
    },
    {
      name: 'extended-mobile',
      use: { ...devices['Pixel 5'] },
      grep: /@extended/,
    },
  ],
});
```

### Running Specific Projects

```bash
# Run only the smoke suite
npx playwright test --project=smoke-chromium

# Run full regression across all browsers
npx playwright test --project=regression-chromium --project=regression-firefox --project=regression-webkit

# Run with a single project flag (for CI)
npx playwright test --project="smoke-chromium"
```

---

## Test Data Management Strategy

Test data is the #1 source of suite fragility at scale. Here's a strategy that grows with you.

### Level 1: Static JSON Files (1–15 tests)

Good enough for getting started:

```json
// test-data/users.json
{
  "standard": {
    "email": "testuser@example.com",
    "password": "TestPass123!",
    "name": "Test User"
  },
  "admin": {
    "email": "admin@example.com",
    "password": "AdminPass123!",
    "name": "Admin User"
  }
}
```

```json
// test-data/products.json
[
  { "id": "PROD-001", "name": "Test Widget", "price": 29.99, "sku": "TW-001" },
  { "id": "PROD-002", "name": "Sample Gadget", "price": 49.99, "sku": "SG-002" }
]
```

### Level 2: Environment Variables for Secrets (Always Required)

Never put passwords or tokens in files:

```bash
# .env.test (gitignored)
TEST_USER_EMAIL=testuser@example.com
TEST_USER_PASSWORD=TestPass123!
ADMIN_USER_EMAIL=admin@example.com
ADMIN_USER_PASSWORD=AdminPass123!
STRIPE_TEST_CARD=4242424242424242
BASE_URL=https://staging.myapp.com
```

Load with dotenv in `playwright.config.js`:

```javascript
import dotenv from 'dotenv';
dotenv.config({ path: '.env.test' });
```

### Level 3: Test Fixtures for Shared Setup (15–60 tests)

Playwright fixtures are reusable setup/teardown blocks:

```javascript
// tests/fixtures/auth.fixture.js
import { test as base } from '@playwright/test';
import { AuthFlow } from '../flows/auth.flow.js';

export const test = base.extend({
  // Automatically log in before each test that uses this fixture
  authenticatedPage: async ({ page }, use) => {
    const auth = new AuthFlow(page);
    await auth.loginWithValidUser();
    // Run the test with the logged-in page
    await use(page);
    // Optionally log out after (usually not needed — each test gets a fresh context)
  },

  // Admin-authenticated page
  adminPage: async ({ page }, use) => {
    const auth = new AuthFlow(page);
    await auth.loginAs('admin');
    await use(page);
  },
});

export { expect } from '@playwright/test';
```

```javascript
// tests/specs/account/profile.spec.js
import { test, expect } from '../../fixtures/auth.fixture.js';

// This test starts already logged in — no need to call login!
test('@regression @module-account user can update display name', async ({ authenticatedPage: page }) => {
  await page.goto('/account/profile');
  await page.getByTestId('display-name-input').fill('New Name');
  await page.getByTestId('save-profile-btn').click();
  await expect(page.getByTestId('success-toast')).toBeVisible();
});
```

### Level 4: API-Based Data Creation (60+ tests)

For large suites, create test data programmatically via API:

```javascript
// tests/helpers/data-factory.js
export class DataFactory {
  constructor(request) {
    this.request = request;
    this.created = { users: [], products: [], orders: [] };
  }

  async createUser(overrides = {}) {
    const timestamp = Date.now();
    const userData = {
      email: `test-${timestamp}@testmail.com`,
      password: 'TestPass123!',
      firstName: 'Test',
      lastName: 'User',
      role: 'user',
      ...overrides,
    };

    const response = await this.request.post('/api/test/users', { data: userData });
    const user = await response.json();
    this.created.users.push(user.id);
    return user;
  }

  async createProduct(overrides = {}) {
    const timestamp = Date.now();
    const productData = {
      name: `Test Product ${timestamp}`,
      price: 19.99,
      stock: 100,
      sku: `TEST-${timestamp}`,
      ...overrides,
    };

    const response = await this.request.post('/api/test/products', { data: productData });
    const product = await response.json();
    this.created.products.push(product.id);
    return product;
  }

  async cleanup() {
    // Delete all created data in reverse order
    for (const userId of this.created.users.reverse()) {
      await this.request.delete(`/api/test/users/${userId}`).catch(() => {});
    }
    for (const productId of this.created.products.reverse()) {
      await this.request.delete(`/api/test/products/${productId}`).catch(() => {});
    }
    this.created = { users: [], products: [], orders: [] };
  }
}
```

```javascript
// Using the data factory in a test
import { test, expect } from '@playwright/test';
import { DataFactory } from '../../helpers/data-factory.js';

test('@regression @module-admin admin can delete a user', async ({ page, request }) => {
  const factory = new DataFactory(request);

  // Create a user to delete
  const user = await factory.createUser({ firstName: 'ToDelete', role: 'user' });

  const auth = new AuthFlow(page);
  await auth.loginAs('admin');

  await page.goto(`/admin/users/${user.id}`);
  await page.getByTestId('delete-user-btn').click();
  await page.getByTestId('confirm-delete-btn').click();

  await expect(page.getByTestId('success-toast')).toContainText('User deleted');

  // Cleanup handled by factory — but delete already happened so this is a no-op
  await factory.cleanup();
});
```

---

## Keeping the Suite Healthy as It Grows

### Weekly Health Checks

Run this checklist every week:

```
□ Suite runs in < [your target] minutes on CI
  → If too slow: identify the 10 slowest tests, optimize or move to @extended
  
□ No new @flaky tests have appeared
  → Check CI failure logs for intermittent failures
  → Quarantine any discovered flaky tests
  
□ All test files have module tags
  → grep -r "^test\(" tests/specs | grep -v "@module-" will show untagged tests
  
□ Test count by tag is reasonable
  → @smoke: < 20 tests (should run in < 5 min)
  → @regression: < 100 tests (should run in < 20 min)
  → @extended: no limit (run less frequently)
  
□ No TODO comments left in test code
  → These become permanent fixtures

□ No hardcoded credentials in test files
  → grep -r "password" tests/specs should return nothing
```

### Suite Performance Targets

| Suite | Max Tests | Target Run Time | Max Workers |
|-------|-----------|----------------|-------------|
| Smoke | 20 | < 5 min | 4 |
| Regression (single browser) | 100 | < 20 min | 8 |
| Regression (3 browsers) | 100 × 3 | < 40 min | 16 |
| Extended | Unlimited | < 60 min | 16 |

### When Tests Start Slowing Down

```javascript
// Find the slowest tests by adding this to playwright.config.js
reporter: [
  ['html'],
  ['json', { outputFile: 'test-results/results.json' }],
],
```

Then analyze the JSON:

```bash
# Quick script to find slow tests
node -e "
const r = require('./test-results/results.json');
const sorted = r.suites
  .flatMap(s => s.specs)
  .sort((a, b) => b.tests[0]?.results[0]?.duration - a.tests[0]?.results[0]?.duration)
  .slice(0, 10);
sorted.forEach(s => console.log(s.tests[0]?.results[0]?.duration + 'ms', s.title));
"
```

---

## Asking Zen to Help Build Your Suite

### Generating Test Cases for a Module

```
I've just written the @smoke tests for my checkout module. Now I need to
expand to @regression tests. Here are the smoke tests I have:

[paste your existing smoke tests]

The checkout module has these features:
- Payment methods: credit card, PayPal, Apple Pay
- Guest checkout (no account required)
- Promo code input
- Address autocomplete
- Order confirmation page with order number
- Confirmation email sent after order

Please generate a list of @regression test cases I should add, in the format:
Test ID | Tag | Description | Priority (High/Medium/Low)

Focus on error states, boundary conditions, and edge cases I might miss.
```

### Getting Help With Test Data Strategy

```
My Playwright suite has grown to 45 tests. I'm having problems with test data:
- Tests sometimes fail because shared test data was modified by a previous test
- I can't run tests in parallel because they conflict over the same user account
- Some tests require specific data states that are hard to guarantee

My current setup:
- Staging environment with a shared database
- Tests use hardcoded user credentials from a .env file
- No API for creating test data

What's the best test data strategy for my situation? What would you recommend
I implement first to get the quickest improvement in test reliability?
```

---

## Suite Organization Reference

### Final Recommended Folder Structure (Mature Suite)

```
tests/
├── fixtures/
│   ├── auth.fixture.js          # Login fixtures
│   └── data.fixture.js          # Data factory fixtures
├── flows/
│   ├── auth.flow.js
│   ├── checkout.flow.js
│   ├── catalog.flow.js
│   └── account.flow.js
├── pages/
│   ├── LoginPage.js
│   ├── DashboardPage.js
│   ├── CheckoutPage.js
│   ├── CartPage.js
│   ├── ProductPage.js
│   ├── ProductListPage.js
│   ├── AccountPage.js
│   └── AdminPage.js
├── helpers/
│   ├── data-factory.js          # API-based data creation
│   ├── api.helper.js            # Direct API calls
│   └── date.helper.js           # Date/time utilities
├── specs/
│   ├── auth/
│   │   ├── login.spec.js
│   │   ├── registration.spec.js
│   │   └── password-reset.spec.js
│   ├── catalog/
│   │   ├── product-list.spec.js
│   │   ├── product-detail.spec.js
│   │   └── search.spec.js
│   ├── cart/
│   │   ├── add-to-cart.spec.js
│   │   └── cart-management.spec.js
│   ├── checkout/
│   │   ├── checkout-payment.spec.js
│   │   └── checkout-shipping.spec.js
│   ├── account/
│   │   └── profile.spec.js
│   └── admin/
│       ├── user-management.spec.js
│       └── product-management.spec.js
└── test-data/
    ├── products.json
    └── addresses.json
```

---

## Summary

| Growth Stage | Key Action |
|-------------|------------|
| Bootstrap (0–15) | Cover critical paths, establish folder structure |
| Expansion (15–60) | Add module tags, organize by feature |
| Maturity (60–150) | Add depth, introduce fixtures, monitor performance |
| Scale (150+) | Parallel execution, API data creation, ownership |

The most important rule: **never trade speed for maintainability.** A smaller, well-organized, fast suite beats a large, slow, disorganized one every time.

---

**Next:** [CI/CD Integration →](./ci-tools/README.md) — Connect your suite to your team's pipeline so tests run automatically on every code change.
