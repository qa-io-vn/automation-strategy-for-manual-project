# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · CI/CD · GitHub Actions

> **Series Navigation**
> [← CI/CD Overview](./README.md) | **GitHub Actions** | [GitLab CI →](./02-gitlab-ci.md)

---

## Purpose

This guide sets up a complete Playwright CI/CD integration with GitHub Actions — including a fast PR smoke workflow, nightly regression pipeline, Slack notifications, artifact uploads, and cross-browser matrix testing.

**Prerequisites:**
- Your Playwright project is in a GitHub repository
- You have repository admin access (to configure secrets)
- Your `playwright.config.js` uses the tagging strategy from the previous chapter

---

## Repository Structure

Before we write any YAML, create the workflows directory:

```bash
mkdir -p .github/workflows
```

You'll create two workflow files:
- `.github/workflows/pr-smoke.yml` — fast check on every PR
- `.github/workflows/nightly-regression.yml` — full regression run

---

## Step 1: Configure GitHub Secrets

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these secrets:

| Secret Name | Description |
|-------------|-------------|
| `STAGING_BASE_URL` | `https://staging.myapp.com` |
| `TEST_USER_EMAIL` | `testuser@example.com` |
| `TEST_USER_PASSWORD` | Your test user's password |
| `ADMIN_USER_EMAIL` | `admin@example.com` |
| `ADMIN_USER_PASSWORD` | Your admin user's password |
| `SLACK_WEBHOOK_URL` | Your Slack incoming webhook URL |

**How to get a Slack webhook URL:**
1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Create a new app → "From scratch"
3. Enable "Incoming Webhooks"
4. Add webhook to your #qa-alerts channel
5. Copy the webhook URL

---

## Workflow 1: PR Smoke Suite

This workflow runs on every pull request — it's your first line of defense.

**Goal:** Fast (< 5 min), Chromium only, `@smoke` tests only, blocks merge on failure.

```yaml
# .github/workflows/pr-smoke.yml
name: PR Smoke Tests

on:
  pull_request:
    branches:
      - main
      - develop
      - 'release/**'
  # Also allow manual triggering
  workflow_dispatch:

# Cancel in-progress runs for the same PR (saves minutes when developer pushes rapidly)
concurrency:
  group: smoke-${{ github.ref }}
  cancel-in-progress: true

jobs:
  smoke-tests:
    name: Smoke Tests (Chromium)
    runs-on: ubuntu-latest
    timeout-minutes: 15

    steps:
      # 1. Check out the code
      - name: Checkout repository
        uses: actions/checkout@v4

      # 2. Set up Node.js with caching
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      # 3. Install npm dependencies
      # Using 'ci' ensures reproducible installs
      - name: Install dependencies
        run: npm ci

      # 4. Cache Playwright browsers
      # This avoids re-downloading ~300MB of browsers on every run
      - name: Cache Playwright browsers
        uses: actions/cache@v4
        id: playwright-cache
        with:
          path: ~/.cache/ms-playwright
          key: playwright-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            playwright-${{ runner.os }}-

      # 5. Install Playwright browsers (only if cache missed)
      - name: Install Playwright browsers
        if: steps.playwright-cache.outputs.cache-hit != 'true'
        run: npx playwright install --with-deps chromium

      # 6. Install browser dependencies even if browsers are cached
      # (OS-level dependencies may differ between cached and current runner)
      - name: Install browser OS dependencies
        if: steps.playwright-cache.outputs.cache-hit == 'true'
        run: npx playwright install-deps chromium

      # 7. Run the smoke suite
      - name: Run smoke tests
        run: npx playwright test --grep @smoke --project=chromium
        env:
          BASE_URL: ${{ secrets.STAGING_BASE_URL }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          CI: true

      # 8. Upload test report and artifacts (always — even on failure)
      - name: Upload test report
        uses: actions/upload-artifact@v4
        if: always()  # Upload even when tests fail
        with:
          name: smoke-report-${{ github.run_number }}
          path: |
            playwright-report/
            test-results/
          retention-days: 14

      # 9. Post Slack notification on failure
      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "🔴 *Smoke Tests FAILED* on PR #${{ github.event.pull_request.number }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "🔴 *Smoke Tests FAILED*\n*PR:* <${{ github.event.pull_request.html_url }}|#${{ github.event.pull_request.number }} — ${{ github.event.pull_request.title }}>\n*Author:* ${{ github.event.pull_request.user.login }}\n*Branch:* `${{ github.head_ref }}`"
                  }
                },
                {
                  "type": "actions",
                  "elements": [
                    {
                      "type": "button",
                      "text": { "type": "plain_text", "text": "View Report" },
                      "url": "https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                    }
                  ]
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

---

## Workflow 2: Nightly Regression Suite

This workflow runs every night and after merges to main. It covers all browsers and all `@regression` tests.

```yaml
# .github/workflows/nightly-regression.yml
name: Nightly Regression Suite

on:
  # Run every night at 2:00 AM UTC
  schedule:
    - cron: '0 2 * * *'

  # Also run when code merges to main
  push:
    branches:
      - main

  # Allow manual triggering with options
  workflow_dispatch:
    inputs:
      test-tag:
        description: 'Test tag to run (e.g., @regression, @smoke, @extended)'
        required: false
        default: '@regression'
      browser:
        description: 'Browser to test (chromium/firefox/webkit/all)'
        required: false
        default: 'all'

jobs:
  # ============================================================
  # JOB 1: Cross-Browser Matrix
  # ============================================================
  regression-matrix:
    name: Regression — ${{ matrix.browser }}
    runs-on: ubuntu-latest
    timeout-minutes: 45

    strategy:
      fail-fast: false  # Don't cancel other browsers if one fails
      matrix:
        browser: [chromium, firefox, webkit]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Cache Playwright browsers
        uses: actions/cache@v4
        id: playwright-cache
        with:
          path: ~/.cache/ms-playwright
          # Browser cache is keyed per browser to avoid cache invalidation
          key: playwright-${{ runner.os }}-${{ matrix.browser }}-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            playwright-${{ runner.os }}-${{ matrix.browser }}-

      - name: Install Playwright browsers
        if: steps.playwright-cache.outputs.cache-hit != 'true'
        run: npx playwright install --with-deps ${{ matrix.browser }}

      - name: Install browser OS dependencies
        if: steps.playwright-cache.outputs.cache-hit == 'true'
        run: npx playwright install-deps ${{ matrix.browser }}

      - name: Run regression tests
        run: |
          TAG="${{ github.event.inputs.test-tag || '@regression' }}"
          npx playwright test --grep "$TAG" --project=${{ matrix.browser }}
        env:
          BASE_URL: ${{ secrets.STAGING_BASE_URL }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          ADMIN_USER_EMAIL: ${{ secrets.ADMIN_USER_EMAIL }}
          ADMIN_USER_PASSWORD: ${{ secrets.ADMIN_USER_PASSWORD }}
          CI: true

      - name: Upload test artifacts
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: regression-${{ matrix.browser }}-${{ github.run_number }}
          path: |
            playwright-report/
            test-results/
          retention-days: 30

  # ============================================================
  # JOB 2: Summary & Notification (runs after all matrix jobs)
  # ============================================================
  notify:
    name: Notify Results
    runs-on: ubuntu-latest
    needs: [regression-matrix]
    if: always()  # Run even if matrix jobs failed

    steps:
      - name: Determine overall status
        id: status
        run: |
          if [ "${{ needs.regression-matrix.result }}" = "success" ]; then
            echo "status=success" >> $GITHUB_OUTPUT
            echo "emoji=✅" >> $GITHUB_OUTPUT
            echo "color=good" >> $GITHUB_OUTPUT
          else
            echo "status=failure" >> $GITHUB_OUTPUT
            echo "emoji=🔴" >> $GITHUB_OUTPUT
            echo "color=danger" >> $GITHUB_OUTPUT
          fi

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1.26.0
        with:
          payload: |
            {
              "text": "${{ steps.status.outputs.emoji }} *Nightly Regression: ${{ needs.regression-matrix.result == 'success' && 'PASSED' || 'FAILED' }}*",
              "blocks": [
                {
                  "type": "header",
                  "text": {
                    "type": "plain_text",
                    "text": "${{ steps.status.outputs.emoji }} Nightly Regression Suite"
                  }
                },
                {
                  "type": "section",
                  "fields": [
                    {
                      "type": "mrkdwn",
                      "text": "*Status:*\n${{ needs.regression-matrix.result == 'success' && '✅ All browsers passed' || '❌ One or more browsers failed' }}"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Branch:*\n`${{ github.ref_name }}`"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Run:*\n#${{ github.run_number }}"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Triggered by:*\n${{ github.event_name }}"
                    }
                  ]
                },
                {
                  "type": "actions",
                  "elements": [
                    {
                      "type": "button",
                      "text": { "type": "plain_text", "text": "View Run & Artifacts" },
                      "url": "https://github.com/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                    }
                  ]
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

---

## Optional: Mobile Testing Job

Add this job to your nightly workflow for mobile coverage:

```yaml
  mobile-tests:
    name: Regression — Mobile (${{ matrix.device }})
    runs-on: ubuntu-latest
    timeout-minutes: 30

    strategy:
      fail-fast: false
      matrix:
        device: ['Pixel 5', 'iPhone 13']

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci

      - name: Install Playwright (webkit for iPhone, chromium for Pixel)
        run: npx playwright install --with-deps

      - name: Run mobile smoke tests
        run: npx playwright test --grep @smoke --project="Mobile Chrome"
        env:
          BASE_URL: ${{ secrets.STAGING_BASE_URL }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          CI: true

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: mobile-${{ matrix.device }}-${{ github.run_number }}
          path: playwright-report/
          retention-days: 14
```

---

## Playwright Config Adjustments for GitHub Actions

Ensure your `playwright.config.js` handles CI properly:

```javascript
// playwright.config.js
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/specs',

  // Use 50% of available CPUs in CI (GitHub Actions runners have 2 CPUs)
  workers: process.env.CI ? 2 : undefined,

  // Retry failed tests in CI
  retries: process.env.CI ? 2 : 0,

  // Fail if a test is left as .only (prevents CI bypass tricks)
  forbidOnly: !!process.env.CI,

  reporter: process.env.CI
    ? [
        ['html', { open: 'never', outputFolder: 'playwright-report' }],
        ['github'],   // Annotates failed tests directly on the PR diff
        ['json', { outputFile: 'test-results/results.json' }],
      ]
    : [['html', { open: 'on-failure' }], ['list']],

  use: {
    baseURL: process.env.BASE_URL,
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'Mobile Chrome', use: { ...devices['Pixel 5'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
  ],
});
```

---

## Downloading and Viewing CI Artifacts

After a CI run (pass or fail):

1. Go to your GitHub repository → **Actions**
2. Click the failed/completed run
3. Scroll to **Artifacts** at the bottom of the run summary
4. Download `smoke-report-XXX.zip` or `regression-chromium-XXX.zip`
5. Extract the zip
6. Open `playwright-report/index.html` in your browser

**Or view traces directly:**
```bash
# Download the artifact zip, then:
cd playwright-report
npx playwright show-report .

# For a specific trace:
npx playwright show-trace test-results/auth-login-chromium/trace.zip
```

---

## How to Ask Zen to Help With This Workflow

### Debugging a Workflow Failure

```
My GitHub Actions workflow is failing. The YAML is below. The error is:

[paste the GitHub Actions log output]

The workflow is supposed to run Playwright smoke tests on every PR.
What's causing the failure and how do I fix it?

[paste your .github/workflows/pr-smoke.yml]
```

### Extending the Workflow

```
I have a working GitHub Actions workflow that runs Playwright smoke tests.
I want to add:
1. A matrix to also test on Firefox and WebKit for @regression tests
2. A step that fails the build if any test fails AND uploads the report before failing
3. A way to manually trigger the nightly workflow from the GitHub UI with custom parameters

Here's my current workflow:
[paste your YAML]

Please extend it to add these features.
```

### Optimizing Slow CI

```
My GitHub Actions Playwright workflow takes 18 minutes. I want to get it
under 8 minutes. Here's the workflow:

[paste your YAML]

Key details:
- 45 tests in the @regression suite
- Tests are in tests/specs/ across 6 subdirectories
- Running on ubuntu-latest
- No browser caching currently
- Not using parallelism

What are the highest-impact changes I can make to cut the run time in half?
```

---

## Troubleshooting Common GitHub Actions Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| `ENOENT: no such file or directory, open 'playwright.config.js'` | Playwright can't find config | Check `testDir` is relative to repo root, not to the workflow file |
| `Error: browserType.launch: Executable doesn't exist` | Browsers not installed | Add `npx playwright install --with-deps` step before test run |
| Tests timeout in CI (pass locally) | Slow CI runner | Increase `timeout-minutes`, add retries, check for missing waits |
| `secrets.STAGING_BASE_URL` is empty | Secret not set | Verify secret name matches exactly (case-sensitive) |
| Cache never hits | Lockfile changes frequently | Check `package-lock.json` is committed |
| `forbidOnly` error | A test.only was left in code | Find and remove `test.only` or `test.describe.only` |
| PR workflow runs on push to main | `on.push` also covers PRs | Use `branches` filter or separate conditions |

---

## Summary

| Workflow | Trigger | Tests | Browsers | Target Time |
|----------|---------|-------|----------|-------------|
| `pr-smoke.yml` | Every PR open/update | `@smoke` | Chromium | < 5 min |
| `nightly-regression.yml` | 2 AM UTC + merge to main | `@regression` | Chrome, Firefox, Safari | < 30 min |

With these two workflows, every PR is automatically checked against your most critical tests, and your full regression suite runs every night. Combined with artifact uploads and Slack notifications, your entire team gets visibility into test results without anyone lifting a finger.

---

**Next:** [GitLab CI →](./02-gitlab-ci.md) | [Back to CI/CD Overview →](./README.md)
