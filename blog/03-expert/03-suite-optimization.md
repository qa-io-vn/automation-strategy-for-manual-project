# AI-Assisted QA: From Manual Tester to Automation Strategist
## Expert Guide 03 — Test Suite Performance Optimization

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** Expert
> **← Prev:** [Coverage Audit](./02-coverage-audit.md) | **Next →** [Flaky Test Management](./04-flaky-test-management.md)

---

## The 60-Minute Problem

A test suite that takes 60 minutes to run isn't just slow — it's broken. When feedback cycles are that long:

- Developers stop waiting for results and push anyway
- PRs accumulate multiple commits before any test signal
- Flaky failures get retried and ignored rather than investigated
- The suite becomes a post-merge check, not a pre-merge gate

The goal of suite optimization isn't speed for its own sake. It's restoring the test suite to its function as a **fast feedback mechanism**. Under 15 minutes is where test suites start earning their keep as true continuous integration tools.

This guide documents the journey from 60 minutes to 15 minutes with concrete, implementable techniques.

---

## Diagnosing Your Current Suite

Before optimizing, measure. Run a full suite with timing data:

```bash
# Generate a timeline report
npx playwright test --reporter=line,json > /dev/null
cat test-results.json | jq '.suites[].suites[].specs[] | {title: .title, duration: .tests[0].results[0].duration}' | sort -t: -k2 -rn | head -20
```

Build a runtime distribution:

```bash
# Group tests by runtime bucket
npx playwright test --reporter=json | jq -r '
  .suites[].suites[].specs[] |
  .tests[0].results[0].duration as $d |
  if $d > 30000 then "30s+"
  elif $d > 10000 then "10-30s"
  elif $d > 5000 then "5-10s"
  else "0-5s"
  end
' | sort | uniq -c
```

**Target distribution for a healthy suite:**

| Bucket | % of Tests |
|--------|------------|
| 0–5 seconds | >50% |
| 5–10 seconds | ~30% |
| 10–30 seconds | ~15% |
| 30+ seconds | <5% |

If more than 20% of your tests are over 30 seconds, that's your first optimization priority.

---

## Technique 1: Login State Sharing with `storageState`

The single highest-ROI optimization for most suites. Re-authenticating through the UI for every test is pure waste.

### The Problem

```typescript
// Anti-pattern: Every test does full UI login
test.beforeEach(async ({ page }) => {
  await page.goto('/login');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.click('[type="submit"]');
  await page.waitForURL('/dashboard');
  // 5-8 seconds, every single test
});
```

If you have 200 tests doing this: 200 × 6s = **20 minutes just on login**.

### The Solution: Global Setup + `storageState`

**Step 1:** Create a global setup file:

```typescript
// tests/global-setup.ts
import { chromium, FullConfig } from '@playwright/test';
import * as path from 'path';

async function globalSetup(config: FullConfig) {
  const { baseURL } = config.projects[0].use;
  const browser = await chromium.launch();
  const page = await browser.newPage();

  // Login once and save the authenticated state
  await page.goto(`${baseURL}/login`);
  await page.fill('[name="email"]', process.env.TEST_USER_EMAIL!);
  await page.fill('[name="password"]', process.env.TEST_USER_PASSWORD!);
  await page.click('[type="submit"]');
  await page.waitForURL('/dashboard');

  // Save state for standard user
  await page.context().storageState({
    path: path.join(__dirname, 'storage-states/user.json')
  });

  // Login as admin and save separately
  await page.goto(`${baseURL}/logout`);
  await page.goto(`${baseURL}/login`);
  await page.fill('[name="email"]', process.env.TEST_ADMIN_EMAIL!);
  await page.fill('[name="password"]', process.env.TEST_ADMIN_PASSWORD!);
  await page.click('[type="submit"]');
  await page.waitForURL('/admin/dashboard');

  await page.context().storageState({
    path: path.join(__dirname, 'storage-states/admin.json')
  });

  await browser.close();
}

export default globalSetup;
```

**Step 2:** Register and use in `playwright.config.ts`:

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';
import * as path from 'path';

export default defineConfig({
  globalSetup: require.resolve('./tests/global-setup'),
  
  projects: [
    // No authentication needed — used for auth tests themselves
    {
      name: 'unauthenticated',
      use: {
        ...devices['Desktop Chrome'],
      },
      testMatch: '**/auth/**/*.spec.ts',
    },
    
    // Standard user — reuses saved auth state
    {
      name: 'authenticated-user',
      use: {
        ...devices['Desktop Chrome'],
        storageState: path.join(__dirname, 'tests/storage-states/user.json'),
      },
      testIgnore: '**/auth/**/*.spec.ts',
    },
    
    // Admin user — separate auth state
    {
      name: 'authenticated-admin',
      use: {
        ...devices['Desktop Chrome'],
        storageState: path.join(__dirname, 'tests/storage-states/admin.json'),
      },
      testMatch: '**/admin/**/*.spec.ts',
    },
  ],
});
```

**Step 3:** Tests no longer need to log in:

```typescript
// tests/dashboard/overview.spec.ts
import { test, expect } from '@playwright/test';

// No login needed — state is loaded from storageState
test('dashboard shows user greeting', async ({ page }) => {
  await page.goto('/dashboard');
  await expect(page.locator('[data-testid="greeting"]')).toContainText('Welcome');
});
```

**Savings:** 200 tests × 6s login = 20 minutes → near zero (one login at suite start).

---

## Technique 2: API Seeding to Replace UI Setup

UI-driven test data setup is 5–10× slower than API calls. Every test that navigates through 3 screens to create precondition data is a performance problem.

### The Problem

```typescript
test('user can view order details', async ({ page }) => {
  // Setup: Create an order through the UI (60+ seconds of clicking)
  await page.goto('/products');
  await page.click('[data-testid="product-1"]');
  await page.click('[data-testid="add-to-cart"]');
  await page.goto('/cart');
  await page.click('[data-testid="checkout"]');
  await page.fill('[name="card"]', '4242424242424242');
  // ... 20 more steps ...
  
  // Now the actual test (10 seconds)
  await page.goto('/orders');
  await page.click('[data-testid="order-row"]');
  await expect(page.locator('[data-testid="order-status"]')).toHaveText('Confirmed');
});
```

### The Solution: API Fixtures for Data Setup

```typescript
// tests/fixtures/api-fixtures.ts
import { test as base, APIRequestContext } from '@playwright/test';

type TestFixtures = {
  apiContext: APIRequestContext;
  createOrder: (options?: { status?: string; items?: number }) => Promise<string>;
  createProduct: (options?: { name?: string; price?: number }) => Promise<string>;
};

export const test = base.extend<TestFixtures>({
  apiContext: async ({ playwright }, use) => {
    const context = await playwright.request.newContext({
      baseURL: process.env.API_BASE_URL,
      extraHTTPHeaders: {
        Authorization: `Bearer ${process.env.TEST_API_KEY}`,
      },
    });
    await use(context);
    await context.dispose();
  },

  createOrder: async ({ apiContext }, use) => {
    const createdOrders: string[] = [];
    
    const factory = async (options = {}) => {
      const response = await apiContext.post('/api/test/orders', {
        data: {
          status: options.status ?? 'confirmed',
          items: options.items ?? 1,
          userId: process.env.TEST_USER_ID,
        },
      });
      const { id } = await response.json();
      createdOrders.push(id);
      return id;
    };
    
    await use(factory);
    
    // Cleanup after test
    for (const id of createdOrders) {
      await apiContext.delete(`/api/test/orders/${id}`).catch(() => {});
    }
  },
  
  createProduct: async ({ apiContext }, use) => {
    // Similar pattern...
    const factory = async (options = {}) => {
      const response = await apiContext.post('/api/test/products', {
        data: { name: options.name ?? 'Test Product', price: options.price ?? 9.99 }
      });
      const { id } = await response.json();
      return id;
    };
    await use(factory);
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/orders/order-details.spec.ts
import { test, expect } from '../fixtures/api-fixtures';

test('user can view order details', async ({ page, createOrder }) => {
  // Setup: 200ms API call instead of 60 seconds of UI navigation
  const orderId = await createOrder({ status: 'confirmed' });
  
  // The actual test (10 seconds of meaningful validation)
  await page.goto(`/orders/${orderId}`);
  await expect(page.locator('[data-testid="order-status"]')).toHaveText('Confirmed');
  await expect(page.locator('[data-testid="order-id"]')).toContainText(orderId);
});
```

**Savings:** 70s test → 12s test. Across 50 data-setup-heavy tests: **48+ minutes saved**.

### Test API Endpoint Pattern

Your backend needs test-only endpoints for this. Document and protect them carefully:

```typescript
// backend: routes/test-helpers.ts (only mounted in test/staging environments)
if (process.env.NODE_ENV !== 'production') {
  router.post('/api/test/orders', testAuthMiddleware, async (req, res) => {
    const order = await OrderService.createTestOrder(req.body);
    res.json({ id: order.id });
  });
  
  router.delete('/api/test/orders/:id', testAuthMiddleware, async (req, res) => {
    await OrderService.deleteTestOrder(req.params.id);
    res.json({ success: true });
  });
}
```

---

## Technique 3: Parallel Test Execution

### The Risk Model for Parallelism

Parallelism isn't free. Before enabling it, understand the failure modes:

| Risk | Cause | Mitigation |
|------|-------|------------|
| Data collision | Tests sharing database records | Per-test data isolation (API seeding) |
| Port conflicts | Tests starting local servers | Use unique ports per worker |
| State leakage | Shared browser context | Use `test.use({ storageState: ... })` scoped to project |
| Resource exhaustion | Too many workers on CI | Cap workers based on available CPU |
| Flaky failures | Race conditions in app code | Fix the app, not the test |

### Playwright Parallel Configuration

```typescript
// playwright.config.ts
export default defineConfig({
  // Global: run test files in parallel
  fullyParallel: true,
  
  // Number of worker processes
  // Local dev: use 50% of CPUs to keep machine usable
  workers: process.env.CI ? 4 : Math.floor(require('os').cpus().length / 2),
  
  // Retry failed tests in CI (not locally — failures should be investigated)
  retries: process.env.CI ? 2 : 0,
  
  // Timeout per test
  timeout: 30 * 1000,
  
  // Timeout for expect() assertions
  expect: {
    timeout: 5000,
  },
});
```

### Sharding for Large Suites Across Multiple CI Machines

```typescript
// playwright.config.ts — sharding config
export default defineConfig({
  // No workers change needed — sharding splits files across machines
  // Each shard runs in a separate CI job
});
```

**GitHub Actions Sharding Setup:**

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on: [push, pull_request]

jobs:
  playwright:
    name: 'Playwright Tests (Shard ${{ matrix.shardIndex }}/${{ matrix.shardTotal }})'
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          
      - name: Install dependencies
        run: npm ci
        
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
        
      - name: Run Playwright tests (shard ${{ matrix.shardIndex }}/${{ matrix.shardTotal }})
        run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
        env:
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          TEST_API_KEY: ${{ secrets.TEST_API_KEY }}
          
      - name: Upload blob report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: blob-report-${{ matrix.shardIndex }}
          path: blob-report/
          retention-days: 1
          
  merge-reports:
    name: 'Merge Reports'
    needs: [playwright]
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      
      - name: Download blob reports
        uses: actions/download-artifact@v4
        with:
          path: all-blob-reports
          pattern: blob-report-*
          merge-multiple: true
          
      - name: Merge reports
        run: npx playwright merge-reports --reporter html ./all-blob-reports
        
      - name: Upload HTML report
        uses: actions/upload-artifact@v4
        with:
          name: html-report--attempt-${{ github.run_attempt }}
          path: playwright-report/
          retention-days: 14
```

**Impact:** With 4 shards on a 60-minute suite, theoretical time drops to ~18 minutes (accounting for setup overhead).

---

## Technique 4: Removing Redundant Tests

From your coverage audit (Guide 02), you identified duplicate tests. Here's how to safely remove them:

### The Safe Deletion Process

1. **Tag the candidate for deletion** — don't delete immediately:

```typescript
test.skip('login button is visible on home page', async ({ page }) => {
  // MARKED FOR DELETION: Covered by 'successful login flow' test
  // Review date: 2025-02-01. Delete if no objections.
});
```

2. **Run one sprint with it skipped** — verify nothing breaks and no one misses it.

3. **Delete and commit** with a clear message:

```bash
git commit -m "test: remove redundant login visibility test

Covered by e2e/auth/login.spec.ts 'successful login flow'.
Skipped for sprint 24, no regressions. Safe to remove.
Saves ~8s per CI run."
```

4. **Update your RTM** — remove the test link if applicable.

---

## Technique 5: Optimizing Slow Assertions

### Replace `waitForTimeout` With Condition Waits

```typescript
// Slow: Hard-coded delay
await page.waitForTimeout(3000);
await expect(page.locator('.toast')).toBeVisible();

// Fast: Wait for the specific condition
await expect(page.locator('.toast')).toBeVisible({ timeout: 5000 });
// Playwright will proceed as soon as it's visible — often much faster than 3000ms
```

### Use Specific Locators Over Broad Ones

```typescript
// Slow: Page-level text search (scans entire DOM)
await expect(page.getByText('Order confirmed')).toBeVisible();

// Fast: Scoped to expected location
await expect(page.locator('[data-testid="order-confirmation-banner"]')).toBeVisible();
```

### Batch Network Idle Waits

```typescript
// Slow: Waiting for network idle after every navigation step
await page.goto('/products');
await page.waitForLoadState('networkidle');  // Can be 5+ seconds
await page.click('[data-testid="category-filter"]');
await page.waitForLoadState('networkidle');  // Another wait

// Better: Wait for specific UI state instead of network idle
await page.goto('/products');
await expect(page.locator('[data-testid="product-grid"]')).toBeVisible();
await page.click('[data-testid="category-filter"]');
await expect(page.locator('[data-testid="product-grid"]')).not.toBeEmpty();
```

### Parallelize Independent Expects

```typescript
// Sequential: 3 × assertion timeout
await expect(page.locator('h1')).toHaveText('Dashboard');
await expect(page.locator('[data-testid="user-name"]')).toBeVisible();
await expect(page.locator('[data-testid="metrics-card"]')).toBeVisible();

// Parallel: Max of the assertion timeouts
await Promise.all([
  expect(page.locator('h1')).toHaveText('Dashboard'),
  expect(page.locator('[data-testid="user-name"]')).toBeVisible(),
  expect(page.locator('[data-testid="metrics-card"]')).toBeVisible(),
]);
```

---

## Technique 6: Test Data Management at Scale

As suites grow, test data management becomes a performance and reliability problem.

### The Data Isolation Hierarchy

```
Level 1: Shared static data (read-only)
  → Seed once, never modified by tests
  → Reference data: countries, categories, plan tiers
  → Fastest; no cleanup needed

Level 2: Per-suite data (created in globalSetup)
  → Created once for the test run
  → Test users, standard products
  → Deleted in globalTeardown

Level 3: Per-test data (created in test setup via API)
  → Created fresh for each test
  → Orders, user-generated content
  → Deleted in test fixture teardown

Level 4: Per-test snapshot (database snapshot/restore)
  → Heaviest option; use only for complex state scenarios
  → Restore DB to known state before test
```

### Database Seeding Script

```typescript
// scripts/seed-test-db.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Level 1: Static reference data
  await prisma.category.createMany({
    data: [
      { id: 'cat-electronics', name: 'Electronics', slug: 'electronics' },
      { id: 'cat-clothing', name: 'Clothing', slug: 'clothing' },
    ],
    skipDuplicates: true,
  });

  // Level 2: Standard test users (used across tests)
  await prisma.user.upsert({
    where: { email: process.env.TEST_USER_EMAIL },
    create: {
      email: process.env.TEST_USER_EMAIL!,
      role: 'user',
      emailVerified: new Date(),
    },
    update: {},
  });

  await prisma.user.upsert({
    where: { email: process.env.TEST_ADMIN_EMAIL },
    create: {
      email: process.env.TEST_ADMIN_EMAIL!,
      role: 'admin',
      emailVerified: new Date(),
    },
    update: {},
  });

  console.log('Test database seeded successfully');
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

```bash
# Run before the test suite in CI
npx tsx scripts/seed-test-db.ts
npx playwright test
```

---

## The 60 → 15 Minute Transformation Roadmap

Here's the realistic timeline and impact of applying each technique:

| Technique | Effort | Impact | When to Apply |
|-----------|--------|--------|---------------|
| storageState login sharing | 4 hours | −15 to −25 min | First |
| Remove obvious duplicates | 2 hours | −5 to −10 min | Second |
| API seeding for data setup | 1–2 days | −10 to −20 min | Third |
| Parallelize (4 workers) | 2 hours | −20 to −35 min | Fourth |
| Optimize slow assertions | Ongoing | −3 to −8 min | Ongoing |
| Sharding (CI only) | 4 hours | −30 to −45 min (wall time) | When parallelism isn't enough |

**Realistic scenario:**
```
Starting: 60 minutes, 1 worker, full UI login every test

After storageState:       45 minutes  (−15 min)
After removing 30 dupes:  38 minutes  (−7 min)
After API seeding:        22 minutes  (−16 min)
After 4 workers (local):  12 minutes  (−10 min wall time)
After sharding (4 CI):    ~8 minutes  (wall time, 4 machines)
```

---

## Full Optimized `playwright.config.ts`

```typescript
import { defineConfig, devices } from '@playwright/test';
import * as path from 'path';
import * as os from 'os';

export default defineConfig({
  testDir: './tests',
  
  // Run test files in parallel
  fullyParallel: true,
  
  // Fail fast in CI; continue locally for debugging
  forbidOnly: !!process.env.CI,
  
  // Retries for CI (not local — debug failures immediately)
  retries: process.env.CI ? 2 : 0,
  
  // Worker count: 4 on CI, 50% of cores locally
  workers: process.env.CI ? 4 : Math.floor(os.cpus().length / 2),
  
  // Reporter: concise locally, detailed on CI
  reporter: process.env.CI
    ? [['github'], ['blob'], ['html', { open: 'never' }]]
    : [['list'], ['html', { open: 'on-failure' }]],
  
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    
    // Capture traces only on failure (not every run — too slow)
    trace: 'on-first-retry',
    
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Video only on retry (investigate flaky failures)
    video: 'on-first-retry',
    
    // Conservative action timeout
    actionTimeout: 10_000,
    
    // Navigation timeout
    navigationTimeout: 30_000,
  },
  
  // Global setup: authenticate users, seed database
  globalSetup: require.resolve('./tests/global-setup'),
  globalTeardown: require.resolve('./tests/global-teardown'),
  
  projects: [
    // Tests that don't need auth (login page, registration, etc.)
    {
      name: 'unauthenticated',
      use: { ...devices['Desktop Chrome'] },
      testMatch: ['**/auth/**/*.spec.ts', '**/public/**/*.spec.ts'],
    },
    
    // Standard user flows
    {
      name: 'user',
      use: {
        ...devices['Desktop Chrome'],
        storageState: path.join(__dirname, 'tests/.auth/user.json'),
      },
      testIgnore: ['**/auth/**/*.spec.ts', '**/admin/**/*.spec.ts'],
    },
    
    // Admin flows
    {
      name: 'admin',
      use: {
        ...devices['Desktop Chrome'],
        storageState: path.join(__dirname, 'tests/.auth/admin.json'),
      },
      testMatch: '**/admin/**/*.spec.ts',
    },
    
    // Mobile viewport (critical flows only)
    {
      name: 'mobile-chrome',
      use: {
        ...devices['Pixel 7'],
        storageState: path.join(__dirname, 'tests/.auth/user.json'),
      },
      testMatch: '**/critical/**/*.spec.ts',
    },
  ],
  
  // Ensure test server is running
  webServer: process.env.CI ? undefined : {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: true,
    timeout: 60_000,
  },
});
```

---

## Checkpoint

Your suite is now fast, parallel, and data-isolated. Verify:

- [ ] Suite runtime is under 20 minutes (target: under 15)
- [ ] Login is handled via `storageState` — no UI login in test bodies
- [ ] Test data is seeded via API, not UI flows
- [ ] CI runs tests in parallel (at least 4 workers or 4 shards)
- [ ] No `waitForTimeout` calls remain in the test suite
- [ ] Duplicate tests from your coverage audit have been removed

**Next:** [Flaky Test Management →](./04-flaky-test-management.md) — A fast suite full of flaky tests is still a broken suite. Let's fix that.
