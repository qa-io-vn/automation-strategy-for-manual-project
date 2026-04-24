# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · CI/CD · CircleCI

> **Series Navigation**
> [← Jenkins](./03-jenkins.md) | **CircleCI** | [Azure DevOps →](./05-azure-devops.md)

---

## Purpose

This guide sets up a complete Playwright integration with CircleCI — including orb usage, PR vs. main branch workflows, test parallelism with splitting, artifact storage, Slack notifications, and caching. CircleCI's pipeline-as-code model is powerful and the parallelism features are among the best in class.

**Prerequisites:**
- A CircleCI account (circleci.com) with your repository connected
- Your code hosted on GitHub, GitLab, or Bitbucket
- Maintainer access to configure contexts and environment variables

---

## CircleCI Concepts for QA Engineers

| Concept | What It Means |
|---------|---------------|
| **Workflow** | A sequence of jobs (e.g., test → deploy) |
| **Job** | A unit of work that runs on one executor |
| **Step** | A command within a job |
| **Executor** | The machine/Docker container that runs the job |
| **Orb** | A reusable CircleCI config package (like npm packages for CI) |
| **Context** | A group of environment variables shared across projects |
| **Parallelism** | Running the same job on N simultaneous machines |
| **Test splitting** | CircleCI divides tests across parallel machines automatically |
| **Cache** | Saved directories to speed up future jobs |
| **Artifact** | Files saved for download after a job completes |

---

## Step 1: Configure Environment Variables

### Option A: Project-Level Variables

1. Go to **CircleCI Dashboard** → your project → **Project Settings** → **Environment Variables**
2. Add:

| Variable Name | Description |
|---------------|-------------|
| `STAGING_BASE_URL` | `https://staging.myapp.com` |
| `TEST_USER_EMAIL` | `testuser@example.com` |
| `TEST_USER_PASSWORD` | Your test password |
| `ADMIN_USER_EMAIL` | `admin@example.com` |
| `ADMIN_USER_PASSWORD` | Your admin password |
| `SLACK_WEBHOOK_URL` | Slack incoming webhook URL |

### Option B: Organization Context (Recommended for Teams)

Contexts allow sharing variables across multiple projects:

1. Go to **Organization Settings** → **Contexts** → **Create Context**
2. Name it `qa-credentials`
3. Add the same variables as above
4. Reference in config: `context: qa-credentials`

---

## Complete CircleCI Configuration

Create this file at `.circleci/config.yml`:

```yaml
# .circleci/config.yml
# Playwright Test Pipeline for CircleCI
# Supports: PR workflow (smoke), main branch workflow (regression), scheduled nightly, manual

version: 2.1

# ============================================================
# ORBS
# Orbs are pre-built, reusable config packages
# ============================================================
orbs:
  # Node.js orb for Node setup and caching
  node: circleci/node@5.2.0
  # Browser Tools orb (optional — provides browser installation helpers)
  browser-tools: circleci/browser-tools@1.4.8

# ============================================================
# EXECUTORS
# Define reusable machine/container configurations
# ============================================================
executors:
  playwright-executor:
    # Use Playwright's official Docker image with browsers pre-installed
    docker:
      - image: mcr.microsoft.com/playwright:v1.44.0-jammy
    # Allocate enough memory for browser processes
    resource_class: medium
    # Working directory for all commands
    working_directory: ~/project
    environment:
      # Tell Node.js where to find packages
      NODE_ENV: test
      CI: "true"

  playwright-executor-large:
    # Larger machine for parallel test splitting
    docker:
      - image: mcr.microsoft.com/playwright:v1.44.0-jammy
    resource_class: large
    working_directory: ~/project
    environment:
      CI: "true"

# ============================================================
# COMMANDS
# Reusable step sequences
# ============================================================
commands:
  # Install npm dependencies with caching
  install-dependencies:
    steps:
      - checkout
      - restore_cache:
          keys:
            # Cache key based on OS + lockfile hash
            - node-deps-v1-{{ arch }}-{{ checksum "package-lock.json" }}
            # Fallback to any previous cache if exact match not found
            - node-deps-v1-{{ arch }}-
      - run:
          name: Install npm dependencies
          command: npm ci
      - save_cache:
          key: node-deps-v1-{{ arch }}-{{ checksum "package-lock.json" }}
          paths:
            - node_modules

  # Upload test results and artifacts
  upload-results:
    parameters:
      suite-name:
        type: string
        default: "tests"
    steps:
      - store_artifacts:
          path: playwright-report
          destination: playwright-report-<< parameters.suite-name >>
      - store_artifacts:
          path: test-results
          destination: test-results-<< parameters.suite-name >>
      - store_test_results:
          # CircleCI parses JUnit XML and displays results in the Tests tab
          path: test-results

  # Send Slack notification
  notify-slack:
    parameters:
      status:
        type: enum
        enum: ["pass", "fail"]
      suite-name:
        type: string
        default: "Playwright Tests"
    steps:
      - run:
          name: Send Slack notification
          when: always
          command: |
            if [ "<< parameters.status >>" = "fail" ]; then
              EMOJI="🔴"
              STATUS_TEXT="FAILED"
            else
              EMOJI="✅"
              STATUS_TEXT="PASSED"
            fi

            curl -X POST "${SLACK_WEBHOOK_URL}" \
              -H 'Content-type: application/json' \
              --data "{
                \"text\": \"${EMOJI} << parameters.suite-name >> ${STATUS_TEXT}\",
                \"blocks\": [
                  {
                    \"type\": \"section\",
                    \"text\": {
                      \"type\": \"mrkdwn\",
                      \"text\": \"${EMOJI} *<< parameters.suite-name >> ${STATUS_TEXT}*\n*Project:* ${CIRCLE_PROJECT_REPONAME}\n*Branch:* \`${CIRCLE_BRANCH}\`\n*Build:* <${CIRCLE_BUILD_URL}|#${CIRCLE_BUILD_NUM}>\"
                    }
                  }
                ]
              }"

# ============================================================
# JOBS
# ============================================================
jobs:
  # ============================================================
  # JOB: Smoke Tests (PR Check — fast, Chromium only)
  # ============================================================
  smoke-tests:
    executor: playwright-executor
    steps:
      - install-dependencies
      - run:
          name: Run smoke tests (Chromium)
          command: |
            npx playwright test \
              --grep @smoke \
              --project=chromium \
              --reporter=html,junit,list
          environment:
            BASE_URL: $STAGING_BASE_URL
            TEST_USER_EMAIL: $TEST_USER_EMAIL
            TEST_USER_PASSWORD: $TEST_USER_PASSWORD
            PLAYWRIGHT_JUNIT_OUTPUT_NAME: test-results/junit.xml
      - upload-results:
          suite-name: smoke
      - run:
          name: Notify on failure
          when: on_fail
          command: |
            curl -X POST "${SLACK_WEBHOOK_URL}" \
              -H 'Content-type: application/json' \
              --data "{
                \"text\": \"🔴 *Smoke Tests FAILED on PR* — Branch: \`${CIRCLE_BRANCH}\` — <${CIRCLE_BUILD_URL}|View Build>\"
              }"

  # ============================================================
  # JOB: Regression Tests (Chromium — with test splitting)
  # ============================================================
  regression-chromium:
    executor: playwright-executor-large
    # Run 4 parallel machines — test splitting divides tests evenly
    parallelism: 4
    steps:
      - install-dependencies
      - run:
          name: Run regression tests — Chromium (shard << pipeline.parameters.test-shard >>)
          command: |
            # Get the list of test files
            TESTFILES=$(npx playwright test --list --grep @regression --project=chromium 2>/dev/null | grep "spec.js" | tr '\n' ' ')

            # Use CircleCI's test splitting to distribute files across parallel machines
            SPLIT_FILES=$(echo "$TESTFILES" | tr ' ' '\n' | circleci tests split --split-by=timings --timings-type=testname)

            # Run only this machine's assigned files
            npx playwright test $SPLIT_FILES \
              --grep @regression \
              --project=chromium \
              --reporter=html,junit,list \
              --output=test-results/chromium
          environment:
            BASE_URL: $STAGING_BASE_URL
            TEST_USER_EMAIL: $TEST_USER_EMAIL
            TEST_USER_PASSWORD: $TEST_USER_PASSWORD
            ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
            ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
            PLAYWRIGHT_JUNIT_OUTPUT_NAME: test-results/junit.xml
      - upload-results:
          suite-name: regression-chromium

  # ============================================================
  # JOB: Regression Tests — Firefox
  # ============================================================
  regression-firefox:
    executor: playwright-executor
    steps:
      - install-dependencies
      - run:
          name: Run regression tests — Firefox
          command: |
            npx playwright test \
              --grep @regression \
              --project=firefox \
              --reporter=junit,list \
              --output=test-results/firefox
          environment:
            BASE_URL: $STAGING_BASE_URL
            TEST_USER_EMAIL: $TEST_USER_EMAIL
            TEST_USER_PASSWORD: $TEST_USER_PASSWORD
            ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
            ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
            PLAYWRIGHT_JUNIT_OUTPUT_NAME: test-results/junit.xml
      - upload-results:
          suite-name: regression-firefox

  # ============================================================
  # JOB: Regression Tests — WebKit (Safari)
  # ============================================================
  regression-webkit:
    executor: playwright-executor
    steps:
      - install-dependencies
      - run:
          name: Run regression tests — WebKit
          command: |
            npx playwright test \
              --grep @regression \
              --project=webkit \
              --reporter=junit,list \
              --output=test-results/webkit
          environment:
            BASE_URL: $STAGING_BASE_URL
            TEST_USER_EMAIL: $TEST_USER_EMAIL
            TEST_USER_PASSWORD: $TEST_USER_PASSWORD
            ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
            ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
            PLAYWRIGHT_JUNIT_OUTPUT_NAME: test-results/junit.xml
      - upload-results:
          suite-name: regression-webkit

  # ============================================================
  # JOB: Nightly Notification (success/failure summary)
  # ============================================================
  nightly-notify:
    executor:
      name: node/default
      node-version: '20.14'
    steps:
      - run:
          name: Send nightly summary notification
          command: |
            curl -X POST "${SLACK_WEBHOOK_URL}" \
              -H 'Content-type: application/json' \
              --data "{
                \"text\": \"✅ *Nightly Regression COMPLETE* — Branch: \`${CIRCLE_BRANCH}\` — <${CIRCLE_BUILD_URL}|View Results>\"
              }"

# ============================================================
# WORKFLOWS
# Define when and how jobs run
# ============================================================
workflows:
  # ============================================================
  # WORKFLOW 1: PR Check — runs on every pull request
  # ============================================================
  pr-smoke-check:
    jobs:
      - smoke-tests:
          # Use the organization context for shared credentials
          context: qa-credentials
          # Only run on non-main branches (PRs)
          filters:
            branches:
              ignore:
                - main
                - develop

  # ============================================================
  # WORKFLOW 2: Full Regression — runs on merge to main
  # ============================================================
  main-regression:
    jobs:
      - smoke-tests:
          context: qa-credentials
          filters:
            branches:
              only:
                - main
      - regression-chromium:
          context: qa-credentials
          requires:
            - smoke-tests  # Only run regression if smoke passes
          filters:
            branches:
              only:
                - main
      - regression-firefox:
          context: qa-credentials
          requires:
            - smoke-tests
          filters:
            branches:
              only:
                - main
      - regression-webkit:
          context: qa-credentials
          requires:
            - smoke-tests
          filters:
            branches:
              only:
                - main

  # ============================================================
  # WORKFLOW 3: Nightly Scheduled Run
  # ============================================================
  nightly-regression:
    # Run every night at 2:00 AM UTC
    triggers:
      - schedule:
          cron: "0 2 * * *"
          filters:
            branches:
              only:
                - main
    jobs:
      - smoke-tests:
          context: qa-credentials
      - regression-chromium:
          context: qa-credentials
          requires:
            - smoke-tests
      - regression-firefox:
          context: qa-credentials
          requires:
            - smoke-tests
      - regression-webkit:
          context: qa-credentials
          requires:
            - smoke-tests
      - nightly-notify:
          context: qa-credentials
          requires:
            - regression-chromium
            - regression-firefox
            - regression-webkit
```

---

## Playwright Config for JUnit + CircleCI

CircleCI's **Tests** tab reads JUnit XML. Add the JUnit reporter and configure output:

```javascript
// playwright.config.js
reporter: process.env.CI
  ? [
      ['html', { open: 'never', outputFolder: 'playwright-report' }],
      [
        'junit',
        {
          outputFile: 'test-results/junit.xml',
          // Include test suite name (helps CircleCI group results)
          suiteName: 'Playwright Tests',
        },
      ],
      ['list'],
    ]
  : [['html', { open: 'on-failure' }], ['list']],
```

---

## How Test Splitting Works

CircleCI's test splitting is one of its most powerful features. When you set `parallelism: 4`, CircleCI spins up 4 identical machines. Test splitting distributes your test files across them intelligently:

```
Machine 1: login.spec.js, registration.spec.js  (estimated time: 45s)
Machine 2: checkout.spec.js                     (estimated time: 42s)
Machine 3: cart.spec.js, search.spec.js         (estimated time: 48s)
Machine 4: account.spec.js, admin.spec.js       (estimated time: 40s)
```

CircleCI uses historical timing data to balance the distribution. First run is random — subsequent runs improve.

**Simpler test splitting with Playwright sharding:**

```yaml
- run:
    name: Run tests with Playwright sharding
    command: |
      npx playwright test \
        --grep @regression \
        --project=chromium \
        --shard=${CIRCLE_NODE_INDEX}/${CIRCLE_NODE_TOTAL}
```

`CIRCLE_NODE_INDEX` (0-based) and `CIRCLE_NODE_TOTAL` are automatically injected by CircleCI when `parallelism` is set.

---

## Downloading Artifacts from CircleCI

After a job runs:
1. Click on the job in the CircleCI dashboard
2. Click the **Artifacts** tab
3. Find `playwright-report/index.html` and click to view in browser
4. Or download the full `playwright-report/` folder

For traces:
1. Find `test-results/` in artifacts
2. Download the `.zip` trace files
3. Run locally: `npx playwright show-trace trace.zip`

---

## Caching Strategy in Detail

The caching configuration in this guide uses a layered approach:

```yaml
restore_cache:
  keys:
    # 1st: Exact match — lockfile hasn't changed
    - node-deps-v1-{{ arch }}-{{ checksum "package-lock.json" }}
    # 2nd: Architecture match — lockfile changed, use closest cache
    - node-deps-v1-{{ arch }}-
```

**When to invalidate the cache:**
- Bump the cache key prefix (`v1` → `v2`) when:
  - node_modules is corrupted and builds fail mysteriously
  - You want to force a fresh install of all dependencies
  - A major Node.js or npm version change

---

## Asking Zen for CircleCI Help

### Generating the Configuration

```
I need to set up CircleCI for my Playwright test suite.

My setup:
- Playwright v1.44, Node.js 20
- Tests in tests/specs/ with tags @smoke, @regression, @extended
- playwright.config.js at root
- CircleCI connected to my GitHub repository

What I need:
1. PR workflow: smoke tests on Chromium only (fast)
2. Main branch workflow: full regression on 3 browsers after smoke passes
3. Nightly schedule: same as main branch workflow
4. Parallelism: 4 machines for regression-chromium with test splitting
5. Artifact upload for HTML report and traces
6. Slack notification on failure (have SLACK_WEBHOOK_URL in context)
7. Credentials in a CircleCI context called 'qa-credentials'

Please generate the complete .circleci/config.yml
```

### Optimizing Build Time

```
My CircleCI regression tests take 25 minutes. I want to get under 10 minutes.

Current setup:
- 45 @regression tests in 8 spec files
- Running on 1 machine (parallelism: 1)
- Using node:20 Docker image + installing Playwright from scratch each run
- No caching configured

What changes to .circleci/config.yml will have the biggest impact on build speed?
```

---

## Troubleshooting Common CircleCI Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| `Error: Executable doesn't exist` | Playwright browsers missing | Use `mcr.microsoft.com/playwright` image or add install step |
| Environment variable not found | `undefined` value in logs | Check context is attached to job; verify variable name spelling |
| Cache never restores | Cache key mismatch | Check `checksum` of `package-lock.json` matches; try fallback key |
| Tests pass locally, fail in CI | Timing/concurrency issues | Increase `retries` in `playwright.config.js` for CI |
| Test splitting uneven | Same machine runs most tests | Wait for 2nd run — timing data improves splits automatically |
| Artifacts not visible | Wrong artifact path | Verify path in `store_artifacts` matches actual output directory |
| Context variables not available | Context not attached | Add `context: qa-credentials` to the job in the workflow |
| `CIRCLE_NODE_INDEX` undefined | Parallelism not set | Add `parallelism: N` to the job definition |

---

## Summary

| Workflow | Trigger | Jobs | Parallelism |
|----------|---------|------|-------------|
| `pr-smoke-check` | Every non-main branch PR | smoke-tests | 1 |
| `main-regression` | Push to main | smoke → regression (3 browsers) | 4 for Chromium |
| `nightly-regression` | 2 AM UTC daily | smoke → regression → notify | 4 for Chromium |

CircleCI's combination of parallelism + test splitting + intelligent caching makes it one of the fastest CI options for large Playwright suites.

---

**Next:** [Azure DevOps →](./05-azure-devops.md) | [Back to CI/CD Overview →](./README.md)
