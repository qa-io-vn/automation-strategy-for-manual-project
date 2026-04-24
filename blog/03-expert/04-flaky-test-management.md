# AI-Assisted QA: From Manual Tester to Automation Strategist
## Expert Guide 04 — Flaky Test Management

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** Expert
> **← Prev:** [Suite Optimization](./03-suite-optimization.md) | **Next →** [Metrics & Reporting](./05-metrics-and-reporting.md)

---

## The Real Cost of Flaky Tests

A test that fails 10% of the time isn't failing 10% of the time — it's *lying* 10% of the time.

Flaky tests are worse than no tests because:

- They train your team to **ignore red builds** — the most dangerous habit in engineering
- They erode trust in the entire test suite ("oh, that's probably just flaky")
- They consume debugging time with no bug to show for it
- They mask real failures hiding in the noise
- They slow CI down through retries

The acceptable flaky rate for a production QA suite is **under 1%**. Most teams operate at 5–15% and call it "normal." It isn't.

This guide gives you a complete system: root cause taxonomy, trace-based diagnosis, quarantine policy, fix patterns, and how to use Zen to accelerate diagnosis.

---

## The Flaky Test Rate Baseline

Before you can manage flakiness, you need to measure it.

### Calculating Flaky Test Rate

```
Flaky Test Rate = (Tests that had at least 1 unexpected failure in N runs) / Total tests
```

A "flaky" test is one that:
- Failed at least once but also passed in the same code version
- Did not consistently fail across all runs (that's a broken test, not a flaky one)

### Tracking Flakiness in CI

**GitHub Actions — Extract Flaky Tests From Retry Data:**

```yaml
# playwright.config.ts: enable retries to detect flakes
retries: 2  # On CI only

# Tests that pass on retry are flaky — track them
```

```bash
# Parse Playwright JSON report for flaky tests (passed after retry)
cat test-results.json | jq -r '
  .suites[].suites[].specs[] |
  select(.tests[0].results | length > 1) |
  select(.tests[0].results[-1].status == "passed") |
  select(.tests[0].results[0].status == "failed") |
  .title
'
```

**Build a Flakiness Tracker:**

```typescript
// scripts/flakiness-tracker.ts
import * as fs from 'fs';

interface FlakyTestRecord {
  title: string;
  file: string;
  flakyCount: number;
  lastFlaky: string;
  rootCause?: string;
  quarantined: boolean;
}

const TRACKER_FILE = './flaky-tests.json';

function updateFlakinessTracker(reportFile: string) {
  const report = JSON.parse(fs.readFileSync(reportFile, 'utf-8'));
  const existing: Record<string, FlakyTestRecord> = fs.existsSync(TRACKER_FILE)
    ? JSON.parse(fs.readFileSync(TRACKER_FILE, 'utf-8'))
    : {};

  // Find tests that passed after retries (flaky)
  const newFlakes = report.suites
    .flatMap((s: any) => s.suites)
    .flatMap((s: any) => s.specs)
    .filter((spec: any) => {
      const results = spec.tests[0].results;
      return results.length > 1 &&
        results[0].status === 'failed' &&
        results[results.length - 1].status === 'passed';
    });

  for (const flake of newFlakes) {
    const key = `${flake.file}::${flake.title}`;
    existing[key] = {
      title: flake.title,
      file: flake.file,
      flakyCount: (existing[key]?.flakyCount ?? 0) + 1,
      lastFlaky: new Date().toISOString(),
      rootCause: existing[key]?.rootCause,
      quarantined: existing[key]?.quarantined ?? false,
    };
  }

  fs.writeFileSync(TRACKER_FILE, JSON.stringify(existing, null, 2));
  console.log(`Updated flakiness tracker. ${newFlakes.length} new flakes recorded.`);
}
```

---

## Root Cause Taxonomy

Every flaky test has a root cause in one of these categories. Understanding the category tells you exactly how to fix it.

### Category 1: Race Conditions

**What it looks like:** Test fails intermittently with "element not found" or assertion mismatches, usually after a click or navigation action.

**Root cause:** Test asserts before the application has finished updating. The assertion timing depends on CPU load, network speed, or JavaScript event loop timing — all of which vary.

**Diagnostic signals:**
- Failure rate increases under heavy CI load (multiple parallel workers)
- Test passes consistently when run in isolation
- Trace shows the element briefly not existing between a user action and assertion

**Fix patterns:**

```typescript
// BAD: Asserts immediately after click (race condition)
await page.click('[data-testid="submit-btn"]');
await expect(page.locator('.success-message')).toBeVisible();

// GOOD: Wait for the expected state change first
await page.click('[data-testid="submit-btn"]');
// Wait for the button to be disabled (the first state change after click)
await expect(page.locator('[data-testid="submit-btn"]')).toBeDisabled();
// Now assert the success message
await expect(page.locator('.success-message')).toBeVisible();

// BETTER: Use response interception to know when the API call completes
const responsePromise = page.waitForResponse(resp =>
  resp.url().includes('/api/submit') && resp.status() === 200
);
await page.click('[data-testid="submit-btn"]');
await responsePromise;
await expect(page.locator('.success-message')).toBeVisible();
```

---

### Category 2: Data Collision

**What it looks like:** Test fails with "element already exists," unique constraint violations, or unexpected data appearing in the UI.

**Root cause:** Multiple parallel tests share database records. Test A creates a record; Test B (running simultaneously) sees it and fails its "empty state" assertion, or both try to create the same record.

**Diagnostic signals:**
- Failures only occur when `workers > 1`
- Failures involve unique identifiers (email addresses, usernames, order numbers)
- Trace shows unexpected data in the UI that wasn't created by this test

**Fix patterns:**

```typescript
// BAD: All tests use the same email address
test('login with valid credentials', async ({ page }) => {
  await page.fill('[name="email"]', 'test@example.com');
  // If another test is concurrently modifying this account, this will flake
});

// GOOD: Per-test isolated data with unique identifiers
test('login with valid credentials', async ({ page, createUser }) => {
  const { email, password } = await createUser();
  // Each test gets its own unique user — no collision possible
  await page.fill('[name="email"]', email);
  await page.fill('[name="password"]', password);
});
```

```typescript
// Unique ID generation utility
export function uniqueEmail(prefix = 'test'): string {
  const timestamp = Date.now();
  const random = Math.random().toString(36).substring(2, 7);
  return `${prefix}+${timestamp}-${random}@example.com`;
}
```

---

### Category 3: Environment Issues

**What it looks like:** Tests fail in CI but pass locally. Or: tests pass Monday but fail Friday.

**Root cause:** External dependency variability — test environment restarts, third-party API rate limits, container resource constraints, network latency to external services.

**Diagnostic signals:**
- Failures cluster around certain times (CI queue congestion, scheduled restarts)
- Failures involve external service calls (email, SMS, payment sandbox, maps)
- Failures are "connection refused" or timeout errors, not assertion failures

**Fix patterns:**

```typescript
// PATTERN: Mock external services in tests
// Don't call real Stripe sandbox — mock the response
test('payment success flow', async ({ page }) => {
  // Intercept Stripe API call and return a canned response
  await page.route('**/api/payments/charge', async (route) => {
    await route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify({ id: 'pi_test_123', status: 'succeeded' }),
    });
  });
  
  await page.click('[data-testid="pay-now"]');
  await expect(page.locator('.payment-success')).toBeVisible();
});
```

```typescript
// PATTERN: Health check before test run
// global-setup.ts
async function waitForApplication(baseURL: string, maxRetries = 30) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(`${baseURL}/health`);
      if (response.ok) return;
    } catch {}
    await new Promise(r => setTimeout(r, 2000));
    console.log(`Waiting for application to be ready... (${i + 1}/${maxRetries})`);
  }
  throw new Error(`Application at ${baseURL} did not become ready`);
}
```

---

### Category 4: State Leakage

**What it looks like:** Tests pass in isolation but fail in sequence. Failure depends on the order tests ran previously.

**Root cause:** A test modifies shared state (database, cookies, local storage, application settings) and doesn't clean up. The next test inherits that state.

**Diagnostic signals:**
- Test passes when run alone (`npx playwright test specific.spec.ts`) but fails in the full suite
- Failure rate varies by which tests ran before it
- Trace shows unexpected pre-existing data or UI state

**Fix patterns:**

```typescript
// PATTERN: Explicit state cleanup in afterEach
test.afterEach(async ({ page, apiContext }) => {
  // Clean up any test data created during this test
  // Don't rely on the next test's setup to compensate
  await apiContext.delete('/api/test/cleanup-session');
  
  // Clear any localStorage/sessionStorage
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
});

// PATTERN: Validate starting state explicitly
test('empty cart shows zero items', async ({ page }) => {
  // Explicitly assert precondition — don't assume it
  await page.goto('/cart');
  // If cart isn't empty from a previous test, fail loudly
  const itemCount = await page.locator('[data-testid="cart-item"]').count();
  if (itemCount > 0) {
    throw new Error(`Precondition failed: Cart has ${itemCount} items. State leakage from previous test.`);
  }
  await expect(page.locator('[data-testid="empty-cart-message"]')).toBeVisible();
});
```

---

### Category 5: Animation and Timing Issues

**What it looks like:** Test fails with "element is obscured" or click targets the wrong element. Usually happens with elements that animate in/out.

**Root cause:** CSS animations, loading skeletons, or transition effects cause elements to be in an intermediate state that Playwright's interaction model rejects.

**Diagnostic signals:**
- Trace shows element exists but action fails
- Error messages include "intercepts pointer events" or "element is not stable"
- Issue disappears when animations are disabled

**Fix patterns:**

```typescript
// PATTERN: Disable animations globally in test environment
// playwright.config.ts
use: {
  // Reduce motion to eliminate animation timing issues
  reducedMotion: 'reduce',
}

// Or programmatically in tests
await page.emulateMedia({ reducedMotion: 'reduce' });
```

```css
/* In your app's CSS, honor prefers-reduced-motion */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

```typescript
// PATTERN: Wait for element stability before interaction
// Playwright's locator.click() already waits for stability, but for complex cases:
await page.locator('[data-testid="modal"]').waitFor({ state: 'visible' });
// Wait for any in-progress transitions to complete
await page.waitForFunction(() => {
  const modal = document.querySelector('[data-testid="modal"]');
  return modal && !modal.getAnimations().length;
});
await page.click('[data-testid="modal-confirm-btn"]');
```

---

## Using Playwright Traces to Diagnose Flakiness

Playwright traces are your most powerful diagnostic tool. Enable them on first retry:

```typescript
// playwright.config.ts
use: {
  trace: 'on-first-retry',
  video: 'on-first-retry',
  screenshot: 'only-on-failure',
}
```

### Trace Analysis Workflow

When a test fails and retries, you get a `.zip` trace file. Open it:

```bash
npx playwright show-trace test-results/test-name/trace.zip
```

**What to look for:**

| Timeline Signal | Likely Root Cause |
|-----------------|-------------------|
| Action fired → long gap → assertion → fail | Race condition |
| Network request pending at time of assertion | Race condition on API response |
| Element exists in DOM but assertion fails | Animation obscuring element |
| Unexpected data visible in screenshots | State leakage or data collision |
| Connection error in network tab | Environment issue |
| Test took 3× longer than usual | Resource contention (parallelism issue) |

### Reading the Network Timeline

In the trace viewer's Network tab:
1. Find the user action that should trigger data loading
2. Check if the relevant API call completed before your assertion
3. If the assertion ran before the API response arrived → race condition fix needed

---

## Using Zen to Diagnose Flaky Failures

### Prompt 1: Root Cause Classification

```
I have a Playwright test that is failing intermittently. Here's the test code and 
the failure information:

**Test code:**
[paste test code]

**Failure message:**
[paste error message]

**When it fails:**
- Fails in CI with 4 parallel workers
- Passes when run alone locally
- Fails approximately 15% of the time

**Trace observations:**
[describe what you see in the trace: timing, network, screenshots]

Based on the root cause taxonomy (Race Condition, Data Collision, Environment Issue, 
State Leakage, Animation Issue), what is the most likely root cause? 

Provide:
1. Root cause classification with confidence level
2. Specific evidence from the test code and failure info
3. The exact fix pattern to apply
4. How to verify the fix worked
```

### Prompt 2: Fix Pattern Generation

```
This Playwright test has a confirmed data collision flakiness issue. Two parallel 
workers are both trying to create users with the same email address.

Current test code:
[paste code]

Please rewrite this test to:
1. Use unique per-test data (using timestamp + random suffix pattern)
2. Clean up created data in test teardown
3. Keep the same test assertions
4. Add an explicit precondition check

Also suggest whether this test needs an API fixture or if inline data creation is sufficient.
```

### Prompt 3: Quarantine Decision

```
I have the following tests with their flaky rates over the last 30 days:

| Test | Flaky Rate | Root Cause (known?) | Last Failed |
|------|------------|---------------------|-------------|
[paste table]

Help me decide which tests should be quarantined vs. fixed immediately vs. monitored.

Quarantine criteria to apply:
- Flaky rate > 20% AND root cause unknown → quarantine
- Flaky rate > 5% AND affecting P0 coverage area → fix immediately
- Flaky rate < 5% AND root cause known → schedule fix

For each test, provide a recommendation and the reasoning.
```

---

## The Quarantine Policy

Quarantining doesn't mean ignoring. It means isolating the problem so it doesn't contaminate build signals.

### Quarantine Protocol

**Step 1: Tag the test**

```typescript
// Add a quarantine tag with tracking information
test.skip('checkout completes with credit card @quarantine', async ({ page }) => {
  // QUARANTINED: 2025-01-15
  // Flaky rate: 22% over last 30 days
  // Root cause: Suspected race condition on payment iframe load
  // Tracking: QA-447
  // Review date: 2025-02-15 (must fix or escalate)
  // ...test body unchanged...
});
```

**Step 2: Create a tracking issue**

```
Title: [QA Debt] Fix flaky test: checkout completes with credit card
Priority: P1 (this is a P0 coverage area)
Labels: qa-debt, flaky-test, quarantined

Description:
Test file: tests/checkout/payment.spec.ts
Flaky rate: 22% (last 30 days)
Coverage area: P0 — Checkout flow
Quarantine date: 2025-01-15

Suspected root cause: Race condition on Stripe payment iframe load
Next step: Reproduce in trace, apply iframe ready-state wait pattern
Deadline: 2025-02-15 — must fix or escalate to eng team
```

**Step 3: Run quarantined tests on a schedule**

```yaml
# .github/workflows/quarantine-check.yml
name: Quarantine Test Check
on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2am

jobs:
  quarantine:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install chromium
      # Run ONLY quarantined tests (remove the .skip programmatically)
      - run: |
          npx playwright test --grep "@quarantine" --retries=0
        continue-on-error: true  # Don't fail CI — just report
      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: quarantine-results
          path: test-results/
```

**Step 4: Review and graduate tests**

Every 2 weeks, review quarantined tests:
- Fixed and verified → remove `@quarantine` tag, un-skip test
- Still failing → update tracking issue, consider escalation
- Root cause found → move to active fix sprint
- Quarantined for 60+ days → escalate to engineering team as infrastructure issue

---

## Fix Patterns Quick Reference

| Root Cause | Quick Fix | Deep Fix |
|------------|-----------|----------|
| Race condition | Add response wait before assertion | Redesign to wait for semantic state, not timing |
| Data collision | Use unique per-test identifiers | Extract to proper fixture with cleanup |
| Environment issue | Mock external service | Create dedicated test infrastructure |
| State leakage | Add explicit cleanup in afterEach | Isolate test data completely with API seeding |
| Animation issue | Add `reducedMotion: 'reduce'` | Fix application to support reduced motion CSS |

---

## Flaky Rate as a Team Health Metric

Track flaky rate on a weekly cadence and treat it as a quality signal:

| Flaky Rate | Health Status | Action |
|------------|---------------|--------|
| < 1% | Healthy | Maintain |
| 1–3% | Acceptable | Monitor; fix as capacity allows |
| 3–7% | Degrading | Allocate sprint capacity for flaky fixes |
| 7–15% | Critical | Stop adding new tests; prioritize flaky fixes |
| > 15% | Crisis | Quarantine all flakes; dedicated fix sprint |

**Trend is more important than absolute number.** A suite at 3% trending upward needs immediate attention. A suite at 5% trending steadily downward over 3 months is a team executing well.

### Monthly Flakiness Retrospective Agenda (15 minutes)

1. **Current rate:** What's our flaky rate this month vs. last month?
2. **New flakes:** Which tests became flaky this month? Why?
3. **Fixed flakes:** Which quarantined tests were fixed? Celebrate.
4. **Persistent flakes:** Which tests have been quarantined 30+ days without resolution?
5. **Pattern analysis:** Do our flaky tests cluster in a particular area? What does that tell us about the code?

---

## Checkpoint

Your flaky test management system is in place when you can say yes to all of these:

- [ ] You have a measured flaky rate (even approximate)
- [ ] Every flaky test is tagged `@quarantine` or has a fix in progress
- [ ] Quarantined tests are excluded from the main build signal
- [ ] You have a tracking issue for each quarantined test with a deadline
- [ ] Quarantined tests run on a scheduled job (weekly) with results captured
- [ ] Your flaky rate is tracked over time (monthly is sufficient)

**Next:** [Metrics & Reporting →](./05-metrics-and-reporting.md) — You have a quality story to tell. Now learn how to tell it with data.
