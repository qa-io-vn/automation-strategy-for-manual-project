# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · CI/CD · GitLab CI

> **Series Navigation**
> [← GitHub Actions](./01-github-actions.md) | **GitLab CI** | [Jenkins →](./03-jenkins.md)

---

## Purpose

This guide sets up a complete Playwright integration with GitLab CI — including merge request pipelines, nightly scheduled regression, artifact storage, GitLab Pages HTML report publishing, parallelism, and environment variable management.

**Prerequisites:**
- Your Playwright project is in a GitLab repository (gitlab.com or self-hosted)
- You have Maintainer access (to configure CI/CD variables)
- GitLab Runner is available (gitlab.com provides shared runners; self-hosted GitLab requires your own)

---

## How GitLab CI Works

GitLab CI is configured through a single file at the root of your repository: `.gitlab-ci.yml`. Every push to any branch triggers the pipeline (unless you add conditions).

Pipeline structure:
```
.gitlab-ci.yml
     ↓
Defines stages (e.g., install → test → report)
     ↓
Each stage has one or more jobs
     ↓
Jobs run on GitLab Runners (shared or your own)
     ↓
Artifacts are saved and GitLab Pages can host HTML reports
```

---

## Step 1: Configure CI/CD Variables

Go to your GitLab project → **Settings** → **CI/CD** → **Variables** → **Add variable**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `STAGING_BASE_URL` | Variable | `https://staging.myapp.com` |
| `TEST_USER_EMAIL` | Variable | `testuser@example.com` |
| `TEST_USER_PASSWORD` | Masked | Your test user's password |
| `ADMIN_USER_EMAIL` | Variable | `admin@example.com` |
| `ADMIN_USER_PASSWORD` | Masked | Your admin user's password |
| `SLACK_WEBHOOK_URL` | Masked | Slack incoming webhook URL |

**Variable flags:**
- **Masked**: Value is hidden in job logs (use for passwords, tokens)
- **Protected**: Only available on protected branches (use for production secrets)
- **Expand variable reference**: Allows `$OTHER_VARIABLE` syntax in value

---

## Complete .gitlab-ci.yml

This single file handles both merge request pipelines and nightly scheduled runs:

```yaml
# .gitlab-ci.yml
# Playwright CI for GitLab
# Supports: MR pipelines (smoke), Scheduled pipelines (regression), Manual runs

# ============================================================
# GLOBAL CONFIGURATION
# ============================================================

# Use the official Playwright Docker image
# This includes Node.js + all browsers pre-installed
# Tag matches Playwright version in package.json
image: mcr.microsoft.com/playwright:v1.44.0-jammy

# Define pipeline stages in order
stages:
  - install
  - test
  - report
  - notify

# Global variables available in all jobs
variables:
  # Tell npm to use the project's node_modules cache
  npm_config_cache: "$CI_PROJECT_DIR/.npm"
  # Playwright cache location
  PLAYWRIGHT_BROWSERS_PATH: "$CI_PROJECT_DIR/pw-browsers"
  # Disable Playwright browser download (already in Docker image)
  PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD: "1"

# Global cache configuration
# Cached between jobs on the same runner
cache:
  key:
    files:
      - package-lock.json
  paths:
    - .npm/
    - node_modules/
  policy: pull-push

# ============================================================
# STAGE 1: INSTALL
# ============================================================

install-dependencies:
  stage: install
  script:
    - npm ci --cache .npm --prefer-offline
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour
  # Only run once; other jobs download from artifacts
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_PIPELINE_SOURCE == "web"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# ============================================================
# STAGE 2: TEST — SMOKE (Merge Request Pipeline)
# ============================================================

smoke-tests:
  stage: test
  needs:
    - install-dependencies
  script:
    - npx playwright test --grep @smoke --project=chromium
  variables:
    BASE_URL: $STAGING_BASE_URL
    TEST_USER_EMAIL: $TEST_USER_EMAIL
    TEST_USER_PASSWORD: $TEST_USER_PASSWORD
  artifacts:
    when: always  # Save artifacts even on failure
    paths:
      - playwright-report/
      - test-results/
    expire_in: 14 days
    reports:
      # GitLab will display this in the MR — shows pass/fail counts
      junit: test-results/junit.xml
  coverage: '/Lines\s*:\s*\d+\.\d+/'
  rules:
    # Only run on merge requests, not on direct pushes to main
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ============================================================
# STAGE 2: TEST — REGRESSION (Nightly + Main Branch)
# ============================================================

# Regression on Chromium
regression-chromium:
  stage: test
  needs:
    - install-dependencies
  script:
    - npx playwright test --grep @regression --project=chromium
  variables:
    BASE_URL: $STAGING_BASE_URL
    TEST_USER_EMAIL: $TEST_USER_EMAIL
    TEST_USER_PASSWORD: $TEST_USER_PASSWORD
    ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
    ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 30 days
    reports:
      junit: test-results/junit.xml
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "web"

# Regression on Firefox — parallel job
regression-firefox:
  stage: test
  needs:
    - install-dependencies
  script:
    - npx playwright test --grep @regression --project=firefox
  variables:
    BASE_URL: $STAGING_BASE_URL
    TEST_USER_EMAIL: $TEST_USER_EMAIL
    TEST_USER_PASSWORD: $TEST_USER_PASSWORD
    ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
    ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 30 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_PIPELINE_SOURCE == "web"

# Regression on WebKit (Safari) — parallel job
regression-webkit:
  stage: test
  needs:
    - install-dependencies
  script:
    - npx playwright test --grep @regression --project=webkit
  variables:
    BASE_URL: $STAGING_BASE_URL
    TEST_USER_EMAIL: $TEST_USER_EMAIL
    TEST_USER_PASSWORD: $TEST_USER_PASSWORD
    ADMIN_USER_EMAIL: $ADMIN_USER_EMAIL
    ADMIN_USER_PASSWORD: $ADMIN_USER_PASSWORD
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 30 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_PIPELINE_SOURCE == "web"

# ============================================================
# STAGE 3: REPORT — Publish to GitLab Pages
# ============================================================

# GitLab Pages allows you to host the HTML report at:
# https://<namespace>.gitlab.io/<project>/
pages:
  stage: report
  needs:
    - job: regression-chromium
      artifacts: true
  script:
    # GitLab Pages serves from the 'public/' directory
    - mkdir -p public
    - cp -r playwright-report/* public/
    # Add timestamp and pipeline info to the report
    - echo "<p>Pipeline #$CI_PIPELINE_ID | $CI_COMMIT_SHORT_SHA | $(date)</p>" >> public/index.html
  artifacts:
    paths:
      - public/
    expire_in: 30 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# ============================================================
# STAGE 4: NOTIFY — Slack on Failure
# ============================================================

notify-failure:
  stage: notify
  image: alpine:latest
  needs:
    - job: regression-chromium
      optional: true
    - job: regression-firefox
      optional: true
    - job: regression-webkit
      optional: true
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl -X POST "$SLACK_WEBHOOK_URL" \
        -H 'Content-type: application/json' \
        --data "{
          \"text\": \"🔴 *Regression Suite FAILED* — Pipeline #$CI_PIPELINE_ID\",
          \"blocks\": [
            {
              \"type\": \"section\",
              \"text\": {
                \"type\": \"mrkdwn\",
                \"text\": \"🔴 *Nightly Regression Failed*\n*Project:* $CI_PROJECT_PATH\n*Branch:* \`$CI_COMMIT_BRANCH\`\n*Commit:* \`$CI_COMMIT_SHORT_SHA\`\n*Pipeline:* <$CI_PIPELINE_URL|#$CI_PIPELINE_ID>\"
              }
            },
            {
              \"type\": \"actions\",
              \"elements\": [
                {
                  \"type\": \"button\",
                  \"text\": { \"type\": \"plain_text\", \"text\": \"View Pipeline\" },
                  \"url\": \"$CI_PIPELINE_URL\"
                },
                {
                  \"type\": \"button\",
                  \"text\": { \"type\": \"plain_text\", \"text\": \"View Report\" },
                  \"url\": \"https://$CI_PROJECT_NAMESPACE.gitlab.io/$CI_PROJECT_NAME/\"
                }
              ]
            }
          ]
        }"
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
      when: on_failure
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: on_failure

notify-success:
  stage: notify
  image: alpine:latest
  needs:
    - job: regression-chromium
      optional: true
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl -X POST "$SLACK_WEBHOOK_URL" \
        -H 'Content-type: application/json' \
        --data "{
          \"text\": \"✅ *Nightly Regression PASSED* — Pipeline #$CI_PIPELINE_ID\",
          \"blocks\": [
            {
              \"type\": \"section\",
              \"text\": {
                \"type\": \"mrkdwn\",
                \"text\": \"✅ *Nightly Regression Passed*\n*Branch:* \`$CI_COMMIT_BRANCH\`\n*Pipeline:* <$CI_PIPELINE_URL|#$CI_PIPELINE_ID>\n*Report:* <https://$CI_PROJECT_NAMESPACE.gitlab.io/$CI_PROJECT_NAME/|View HTML Report>\"
              }
            }
          ]
        }"
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
      when: on_success
```

---

## Step 2: Configure the Nightly Schedule

1. Go to your GitLab project → **Build** → **Pipeline schedules**
2. Click **New schedule**
3. Fill in:
   - **Description:** `Nightly Regression Suite`
   - **Interval Pattern:** `0 2 * * *` (2:00 AM UTC daily)
   - **Target branch:** `main`
   - **Variables:** Add any schedule-specific overrides if needed
4. Save

---

## Playwright Config for JUnit Reports

GitLab's MR integration reads JUnit XML to display test results inline. Add the JUnit reporter:

```javascript
// playwright.config.js
reporter: process.env.CI
  ? [
      ['html', { open: 'never', outputFolder: 'playwright-report' }],
      ['junit', { outputFile: 'test-results/junit.xml' }],
      ['list'],
    ]
  : [['html', { open: 'on-failure' }], ['list']],
```

With JUnit output, GitLab will display a **Tests** tab on each pipeline showing pass/fail/skip counts, and will highlight failed tests directly on merge request pages.

---

## Using Parallel Jobs to Speed Up Tests

For large suites, split tests across multiple parallel jobs using `parallel`:

```yaml
# Split regression tests across 4 parallel jobs
regression-parallel:
  stage: test
  needs:
    - install-dependencies
  parallel: 4
  script:
    # CI_NODE_INDEX is 1-based (1, 2, 3, 4)
    # CI_NODE_TOTAL is total parallel jobs (4)
    - npx playwright test --grep @regression --project=chromium --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  variables:
    BASE_URL: $STAGING_BASE_URL
    TEST_USER_EMAIL: $TEST_USER_EMAIL
    TEST_USER_PASSWORD: $TEST_USER_PASSWORD
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 30 days
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
```

**How sharding works:**
- `--shard=1/4` runs the first quarter of tests
- `--shard=2/4` runs the second quarter
- Playwright distributes tests evenly based on file count

---

## Merging Reports from Parallel Jobs

When using sharding, each job generates its own report. Merge them in a post-test job:

```yaml
merge-reports:
  stage: report
  needs:
    - job: regression-parallel
      artifacts: true
  script:
    # Merge all blob reports from parallel jobs into one HTML report
    - npx playwright merge-reports --reporter=html ./test-results/blob-reports/
  artifacts:
    paths:
      - playwright-report/
    expire_in: 30 days
```

To enable blob reports for merging, add to `playwright.config.js`:

```javascript
reporter: process.env.CI
  ? [
      ['blob'],  // Generates blob files that can be merged
      ['list'],
    ]
  : [['html', { open: 'on-failure' }], ['list']],
```

---

## GitLab-Specific Tips

### Viewing Artifacts

After a pipeline runs:
1. Go to your pipeline → click the job
2. On the right sidebar, find **Job artifacts**
3. Download or browse directly in the GitLab UI

### Viewing the HTML Report via GitLab Pages

After the `pages` job runs successfully:
- Report is available at: `https://<namespace>.gitlab.io/<project-name>/`
- Navigate to **Deploy** → **Pages** in your project to find the URL

### Using Docker Images with Pre-Installed Browsers

The Playwright Docker image (`mcr.microsoft.com/playwright`) is pre-installed with all browsers. This eliminates the browser install step in your pipeline:

```yaml
# No need for 'npx playwright install' when using this image
image: mcr.microsoft.com/playwright:v1.44.0-jammy

# Match the image version to your Playwright package version
# Check available tags: https://hub.docker.com/_/microsoft-playwright
```

### Self-Hosted Runner Configuration

If using a self-hosted GitLab Runner with Docker executor:

```toml
# /etc/gitlab-runner/config.toml
[[runners]]
  name = "playwright-runner"
  url = "https://gitlab.com/"
  executor = "docker"
  [runners.docker]
    image = "mcr.microsoft.com/playwright:v1.44.0-jammy"
    privileged = false
    volumes = ["/cache"]
    shm_size = 0
```

---

## Asking Zen for GitLab CI Help

### Generating the Configuration

```
I need to set up GitLab CI for my Playwright test suite.

My setup:
- GitLab.com hosted repository
- Playwright v1.44, Node.js 20
- Tests in tests/specs/ with tags @smoke, @regression, @extended
- playwright.config.js at root
- I want: MR pipeline (smoke on Chromium), nightly schedule (regression on 3 browsers),
  HTML report published via GitLab Pages, Slack notification on failure

Secrets configured as GitLab CI/CD variables:
- STAGING_BASE_URL, TEST_USER_EMAIL, TEST_USER_PASSWORD, SLACK_WEBHOOK_URL

Please generate the complete .gitlab-ci.yml
```

### Debugging a Failed Pipeline

```
My GitLab CI pipeline is failing at the test stage. Here's the job log:

[paste the full GitLab CI job log]

And here's my .gitlab-ci.yml:
[paste your YAML]

The tests pass locally. What's causing the CI failure?
```

---

## Troubleshooting

| Issue | Symptom | Fix |
|-------|---------|-----|
| `Error: browserType.launch: Executable doesn't exist` | Using wrong Docker image or didn't install browsers | Use `mcr.microsoft.com/playwright` image or add `npx playwright install` |
| `npm ci` fails with cache errors | Cache corrupted | Clear GitLab Runner cache: Settings → CI/CD → Clear Runner caches |
| Pages job fails: `public/ not found` | Playwright report directory doesn't match | Ensure `playwright-report/` exists and the `cp` command is correct |
| Variables not available in job | Variable added after pipeline started | Re-run the pipeline; variables load at start |
| Tests fail with `ECONNREFUSED` | Staging URL not reachable from runner | Check `STAGING_BASE_URL` value; test runner network access |
| Parallel jobs conflict on test data | Multiple jobs modify same data | Use unique test data per shard (e.g., user email includes `$CI_NODE_INDEX`) |

---

## Summary

| Pipeline | Trigger | Tests | Browsers |
|----------|---------|-------|----------|
| MR Pipeline | Every merge request | `@smoke` | Chromium |
| Nightly | Schedule (2 AM UTC) | `@regression` | Chrome, Firefox, Safari |
| Push to main | Commit merged | `@regression` | Chromium |
| Manual | `web` trigger | Configurable | Configurable |

---

**Next:** [Jenkins →](./03-jenkins.md) | [Back to CI/CD Overview →](./README.md)
