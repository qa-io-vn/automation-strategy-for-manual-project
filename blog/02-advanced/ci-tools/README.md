# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · CI/CD Integration Overview

> **Series Navigation**
> [← 05 · Regression Suite](../05-build-regression-suite.md) | **CI/CD Overview** | [GitHub Actions →](./01-github-actions.md)

---

## Purpose

A test suite that runs only on your laptop is a personal safety net. A test suite that runs automatically on every code change is a team safety net. That's the difference CI/CD integration makes.

This section covers the most widely used CI/CD platforms and shows you exactly how to integrate your Playwright suite with each one.

---

## What CI/CD Does for QA

When CI/CD is set up correctly:

- Every pull request triggers your **smoke suite** automatically (< 5 minutes)
- Every night, your **full regression suite** runs against staging
- Failed tests **block merges** or **notify the team** before bugs hit production
- Test reports and artifacts (traces, videos, screenshots) are **always available** for review
- No one has to remember to run tests manually before a release

```
Developer opens PR
        ↓
CI automatically runs @smoke suite (3-5 min)
        ↓
If smoke passes → PR can be reviewed and merged
If smoke fails  → PR is blocked; developer fixes before merging
        ↓
Merge to main
        ↓
Deployment to staging (handled by CD)
        ↓
Nightly schedule: Full @regression suite runs
        ↓
Results sent to Slack/email
        ↓
Artifacts (HTML report, traces) stored for 30 days
```

---

## CI/CD Tool Comparison

| Feature | GitHub Actions | GitLab CI | Jenkins | CircleCI | Azure DevOps |
|---------|---------------|-----------|---------|----------|-------------|
| **Host model** | Cloud (GitHub) | Cloud or Self-hosted | Self-hosted (primary) | Cloud | Cloud or Self-hosted |
| **Config format** | YAML (`.github/workflows/`) | YAML (`.gitlab-ci.yml`) | Groovy/YAML (`Jenkinsfile`) | YAML (`.circleci/config.yml`) | YAML (`azure-pipelines.yml`) |
| **Free tier** | 2000 min/month | 400 min/month | Free (self-hosted) | 6000 min/month | 1800 min/month |
| **Playwright support** | Excellent (official Docker image) | Excellent | Good (Docker) | Good (orbs) | Good |
| **Setup difficulty** | Easy | Easy–Medium | Hard | Medium | Medium |
| **Test reporting** | Via artifacts + 3rd party | Built-in HTML reports via Pages | Via HTML Publisher plugin | Via artifacts | Via Test Plans (native) |
| **Secrets management** | GitHub Secrets | CI/CD Variables | Jenkins Credentials | CircleCI Contexts | Variable Groups |
| **Matrix testing** | Native `matrix` strategy | Native `parallel` | Parallel stage | Native `parallelism` | Strategy matrix |
| **Scheduled runs** | `schedule` trigger | `rules: - if: $CI_PIPELINE_SOURCE` | Cron in job config | Scheduled workflows | Scheduled triggers |
| **Best for** | GitHub-hosted projects | GitLab-hosted or self-hosted | Enterprise with existing Jenkins | Teams wanting flexible cloud CI | Microsoft/Azure ecosystem |

---

## When to Choose Each Tool

### Choose GitHub Actions if:
- Your code is on GitHub (most common for open source + modern teams)
- You want the easiest setup with zero infrastructure to manage
- You want tight integration between PRs and test results
- Your team is small-to-medium and wants to get CI working today

### Choose GitLab CI if:
- Your code is on GitLab (self-hosted or gitlab.com)
- You want built-in CI/CD without leaving your Git platform
- You want free HTML report hosting via GitLab Pages
- You need granular pipeline stage control

### Choose Jenkins if:
- Your organization already has a Jenkins instance
- You need to integrate with non-Git artifact systems (Nexus, Artifactory)
- You need maximum customization and control
- You have DevOps engineers who maintain the CI infrastructure

### Choose CircleCI if:
- You want powerful parallelism and test splitting out of the box
- Your project uses multiple languages/environments
- You prefer a pipeline-as-code model with orbs (reusable config modules)
- Cost per minute is important and you want efficient caching

### Choose Azure DevOps if:
- Your organization uses the Microsoft ecosystem (Azure, Teams, Boards)
- You want native integration between test results and Azure Boards work items
- You need enterprise compliance features (audit logs, access control, etc.)
- Your team uses Azure Test Plans for manual test case management

---

## What All CI Setups Have in Common

Regardless of which tool you use, every CI integration for Playwright requires these components:

### 1. Node.js Setup
All tools need to install Node.js and your npm dependencies:

```bash
npm ci  # Always use 'ci' in pipelines, not 'npm install'
npx playwright install --with-deps  # Install browsers + OS dependencies
```

**Why `npm ci` instead of `npm install`?**
- `npm ci` installs exactly what's in `package-lock.json` — reproducible builds
- `npm install` may update packages, causing non-deterministic failures

### 2. Environment Variables for Secrets

Never put credentials in YAML files. Every CI tool has a secrets/variables mechanism:

```yaml
# Generic pattern — exact syntax varies by tool
env:
  BASE_URL: ${{ secrets.STAGING_URL }}
  TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
  TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
```

### 3. Artifact Upload for Test Results

Always save: HTML report, traces, screenshots, videos.

```yaml
# Generic pattern
- name: Upload test artifacts
  uses: upload-artifact-action
  with:
    path: |
      playwright-report/
      test-results/
```

### 4. Retry Configuration

CI environments can be slower/flakier than local machines. Configure retries:

```javascript
// playwright.config.js
retries: process.env.CI ? 2 : 0,
```

### 5. Headless Mode

CI has no display. Playwright runs headless by default, which is correct. Don't add `--headed` to CI commands.

---

## Recommended Secrets to Configure in Your CI Tool

Before setting up any CI workflow, prepare these secrets in your CI platform:

| Secret Name | Description | Example Value |
|-------------|-------------|---------------|
| `STAGING_BASE_URL` | Base URL for staging environment | `https://staging.myapp.com` |
| `TEST_USER_EMAIL` | Email for standard test user | `testuser@example.com` |
| `TEST_USER_PASSWORD` | Password for standard test user | (never shown) |
| `ADMIN_USER_EMAIL` | Email for admin test user | `admin@example.com` |
| `ADMIN_USER_PASSWORD` | Password for admin test user | (never shown) |
| `SLACK_WEBHOOK_URL` | Webhook for Slack failure notifications | (from Slack app settings) |
| `STRIPE_TEST_KEY` | Stripe test public key (if testing payments) | `pk_test_...` |

---

## Asking Zen to Generate CI Configurations

Zen can write CI configuration files for you. Use this prompt template:

```
I need to set up a CI/CD pipeline for my Playwright test suite.

**CI Tool:** [GitHub Actions / GitLab CI / Jenkins / CircleCI / Azure DevOps]

**My project setup:**
- Playwright version: [run: npx playwright --version]
- Node.js version: [run: node --version]
- Test directory: tests/specs
- Config file: playwright.config.js
- Tags used: @smoke, @regression, @extended, @module-*

**What I need the pipeline to do:**
1. On every Pull Request: run only @smoke tests on Chromium
2. On schedule (nightly): run @regression tests on Chromium, Firefox, Safari
3. Upload HTML report and traces as artifacts on every run
4. Send Slack notification on failure with link to the report
5. Cache node_modules and Playwright browsers for speed

**Secrets I have configured:**
- BASE_URL
- TEST_USER_EMAIL  
- TEST_USER_PASSWORD
- SLACK_WEBHOOK_URL

Please generate the complete YAML configuration for this pipeline.
```

Zen will generate a complete, working configuration. Then ask follow-up questions:

```
The workflow you generated works but takes 12 minutes for the smoke suite.
Can you add browser caching to bring this down to under 5 minutes?
```

---

## CI Pipeline Design Patterns

### Pattern 1: PR Guard (Fast Smoke)

```
PR opened/updated
      ↓
Run @smoke tests (1 browser, parallel)
      ↓
Pass: PR can proceed → code review
Fail: Block PR → developer fixes
```

**Target time: < 5 minutes**

### Pattern 2: Nightly Full Regression

```
Schedule: 2:00 AM every night
      ↓
Run @regression tests (3 browsers, parallel)
      ↓
Publish HTML report
      ↓
Pass: No action
Fail: Slack/email notification to QA team
```

**Target time: < 30 minutes**

### Pattern 3: Pre-Release Extended

```
Tag: v*.*.* (release tag pushed)
      ↓
Run @smoke + @regression + @extended (all browsers + mobile)
      ↓
Publish comprehensive HTML report
      ↓
Notify QA lead + engineering lead
      ↓
Block deployment if any test fails
```

**Target time: < 60 minutes**

---

## Choose Your CI Tool

| I use... | Go to... |
|----------|----------|
| GitHub | [01 · GitHub Actions →](./01-github-actions.md) |
| GitLab | [02 · GitLab CI →](./02-gitlab-ci.md) |
| Jenkins | [03 · Jenkins →](./03-jenkins.md) |
| CircleCI | [04 · CircleCI →](./04-circleci.md) |
| Azure DevOps | [05 · Azure DevOps →](./05-azure-devops.md) |

Each guide is standalone — go directly to the one that applies to your team.
