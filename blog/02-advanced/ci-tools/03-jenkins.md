# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · CI/CD · Jenkins

> **Series Navigation**
> [← GitLab CI](./02-gitlab-ci.md) | **Jenkins** | [CircleCI →](./04-circleci.md)

---

## Purpose

This guide sets up a complete Playwright integration with Jenkins using a declarative pipeline (Jenkinsfile). It covers: a multi-stage pipeline for installation, testing, and reporting; parallel browser matrix testing; HTML report publishing via the HTML Publisher plugin; email notifications; and secure credential management using Jenkins Credentials.

**Prerequisites:**
- Jenkins 2.387+ with Pipeline support
- Jenkins plugins installed:
  - **HTML Publisher** (for Playwright HTML reports)
  - **Email Extension** (for email notifications)
  - **NodeJS** (for Node.js version management)
  - **Credentials Binding** (for secrets)
  - **Blue Ocean** (optional, for visual pipeline UI)
- Node.js configured in Jenkins → Manage Jenkins → Tools → NodeJS

---

## Jenkins Concepts for QA Engineers

Jenkins uses a `Jenkinsfile` (written in Groovy or declarative syntax) stored in your repository root. It defines the pipeline as code — no clicking through Jenkins UI to set up stages.

| Jenkins Concept | What It Means |
|-----------------|---------------|
| **Pipeline** | The entire workflow (install → test → report) |
| **Stage** | A named phase in the pipeline (e.g., "Install", "Test") |
| **Step** | A single command within a stage (e.g., `sh 'npm ci'`) |
| **Agent** | The machine/container where the pipeline runs |
| **Credentials** | Securely stored secrets (passwords, tokens, URLs) |
| **Artifacts** | Files saved after a build (reports, screenshots) |
| **Build** | A single run of the pipeline |

---

## Step 1: Install Required Jenkins Plugins

Go to **Jenkins** → **Manage Jenkins** → **Manage Plugins** → **Available plugins**

Install these plugins:
- `HTML Publisher Plugin`
- `NodeJS Plugin`
- `Email Extension Plugin`
- `Credentials Binding Plugin`
- `Pipeline: Stage View` (for the classic pipeline visualization)
- `Blue Ocean` (optional — much better visual interface)

---

## Step 2: Configure Node.js in Jenkins

1. Go to **Manage Jenkins** → **Global Tool Configuration**
2. Find **NodeJS** section → **Add NodeJS**
3. Set:
   - Name: `NodeJS-20`
   - Version: `NodeJS 20.x`
   - Global npm packages: (leave empty)
4. Save

---

## Step 3: Configure Jenkins Credentials

Go to **Manage Jenkins** → **Credentials** → **System** → **Global credentials** → **Add Credentials**

Add these credentials:

| Kind | ID | Description | Value |
|------|-----|-------------|-------|
| Secret text | `staging-base-url` | Staging environment URL | `https://staging.myapp.com` |
| Secret text | `test-user-email` | Test user email | `testuser@example.com` |
| Secret text | `test-user-password` | Test user password | Your test password |
| Secret text | `admin-user-email` | Admin user email | `admin@example.com` |
| Secret text | `admin-user-password` | Admin user password | Your admin password |
| Secret text | `slack-webhook-url` | Slack webhook URL | `https://hooks.slack.com/...` |

---

## Complete Jenkinsfile: Declarative Pipeline

Create this file at the root of your repository:

```groovy
// Jenkinsfile
// Playwright Test Pipeline — Declarative Syntax
// Supports: PR/branch builds (smoke), scheduled nightly runs (regression), manual runs

pipeline {
    // Run on any available agent
    // For Docker isolation, use: agent { docker { image 'mcr.microsoft.com/playwright:v1.44.0-jammy' } }
    agent any

    // Node.js tool configured in Manage Jenkins → Global Tool Configuration
    tools {
        nodejs 'NodeJS-20'
    }

    // Pipeline-level parameters (for manual triggering with options)
    parameters {
        choice(
            name: 'TEST_SUITE',
            choices: ['smoke', 'regression', 'extended'],
            description: 'Test suite to run'
        )
        choice(
            name: 'BROWSER',
            choices: ['chromium', 'firefox', 'webkit', 'all'],
            description: 'Browser to run tests on'
        )
    }

    // Environment variables — credentials loaded here
    environment {
        BASE_URL          = credentials('staging-base-url')
        TEST_USER_EMAIL   = credentials('test-user-email')
        TEST_USER_PASSWORD = credentials('test-user-password')
        ADMIN_USER_EMAIL  = credentials('admin-user-email')
        ADMIN_USER_PASSWORD = credentials('admin-user-password')
        SLACK_WEBHOOK_URL = credentials('slack-webhook-url')
        CI = 'true'
        // Playwright browser cache location (shared across builds on same agent)
        PLAYWRIGHT_BROWSERS_PATH = "${WORKSPACE}/../.playwright-browsers"
    }

    options {
        // Keep last 10 builds + their artifacts
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        // Timeout the entire pipeline at 60 minutes
        timeout(time: 60, unit: 'MINUTES')
        // Add timestamps to console output
        timestamps()
        // Don't allow concurrent builds on the same branch
        disableConcurrentBuilds()
    }

    triggers {
        // Run nightly at 2:00 AM (server time)
        cron('0 2 * * *')
    }

    stages {
        // ============================================================
        // STAGE 1: INSTALL
        // ============================================================
        stage('Install') {
            steps {
                echo "Installing dependencies for branch: ${env.BRANCH_NAME}"
                sh 'node --version'
                sh 'npm --version'

                // Install npm dependencies (reproducible install)
                sh 'npm ci'

                // Install Playwright browsers
                // --with-deps installs OS-level dependencies too
                sh 'npx playwright install --with-deps'
            }
        }

        // ============================================================
        // STAGE 2: SMOKE TESTS (Always runs — fast feedback)
        // ============================================================
        stage('Smoke Tests') {
            steps {
                echo 'Running smoke tests on Chromium...'
                sh 'npx playwright test --grep @smoke --project=chromium --reporter=html,list'
            }
            post {
                always {
                    // Archive smoke test results
                    archiveArtifacts artifacts: 'playwright-report/**/*', allowEmptyArchive: true

                    // Publish HTML report via HTML Publisher plugin
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Smoke Test Report',
                        reportTitles: "Smoke Tests — Build ${BUILD_NUMBER}"
                    ])
                }
            }
        }

        // ============================================================
        // STAGE 3: REGRESSION TESTS (Parallel browser matrix)
        // Runs on: scheduled nightly, manual trigger, main branch push
        // ============================================================
        stage('Regression Tests') {
            // Only run regression on scheduled builds, main branch, or manual with 'regression' selected
            when {
                anyOf {
                    triggeredBy 'TimerTrigger'
                    branch 'main'
                    expression { params.TEST_SUITE == 'regression' }
                    expression { params.TEST_SUITE == 'extended' }
                }
            }

            parallel {
                stage('Regression — Chromium') {
                    steps {
                        echo 'Running regression tests on Chromium...'
                        sh '''
                            npx playwright test \
                                --grep @regression \
                                --project=chromium \
                                --reporter=html,list \
                                --output=test-results/chromium
                        '''
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/chromium/**/*', allowEmptyArchive: true
                        }
                    }
                }

                stage('Regression — Firefox') {
                    steps {
                        echo 'Running regression tests on Firefox...'
                        sh '''
                            npx playwright test \
                                --grep @regression \
                                --project=firefox \
                                --reporter=list \
                                --output=test-results/firefox
                        '''
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/firefox/**/*', allowEmptyArchive: true
                        }
                    }
                }

                stage('Regression — WebKit') {
                    steps {
                        echo 'Running regression tests on WebKit...'
                        sh '''
                            npx playwright test \
                                --grep @regression \
                                --project=webkit \
                                --reporter=list \
                                --output=test-results/webkit
                        '''
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'test-results/webkit/**/*', allowEmptyArchive: true
                        }
                    }
                }
            }
        }

        // ============================================================
        // STAGE 4: EXTENDED TESTS (Weekly or pre-release only)
        // ============================================================
        stage('Extended Tests') {
            when {
                anyOf {
                    expression { params.TEST_SUITE == 'extended' }
                    // Tag-based trigger: only when a release tag is pushed
                    tag pattern: 'v\\d+\\.\\d+\\.\\d+', comparator: 'REGEXP'
                }
            }
            steps {
                echo 'Running extended tests...'
                sh '''
                    npx playwright test \
                        --grep @extended \
                        --project=chromium \
                        --reporter=html,list \
                        --output=test-results/extended
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'test-results/extended/**/*', allowEmptyArchive: true
                }
            }
        }

        // ============================================================
        // STAGE 5: PUBLISH REPORT (Consolidated after all test stages)
        // ============================================================
        stage('Publish Reports') {
            steps {
                echo 'Publishing consolidated test report...'
                publishHTML([
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'playwright-report',
                    reportFiles: 'index.html',
                    reportName: 'Playwright Test Report',
                    reportTitles: "All Tests — Build ${BUILD_NUMBER} — ${new Date().format('yyyy-MM-dd')}"
                ])
            }
        }
    }

    // ============================================================
    // POST-PIPELINE ACTIONS
    // ============================================================
    post {
        // Always archive traces and screenshots
        always {
            archiveArtifacts(
                artifacts: 'playwright-report/**/*,test-results/**/*.zip,test-results/**/*.png',
                allowEmptyArchive: true,
                fingerprint: true
            )
        }

        // Send email on failure
        failure {
            emailext(
                subject: "❌ Playwright Tests FAILED — ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>Playwright Test Suite Failed</h2>
                    <p><strong>Job:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build:</strong> #${env.BUILD_NUMBER}</p>
                    <p><strong>Branch:</strong> ${env.BRANCH_NAME ?: 'N/A'}</p>
                    <p><strong>URL:</strong> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                    <p><strong>Report:</strong> <a href="${env.BUILD_URL}Playwright_Test_Report/">View HTML Report</a></p>
                    <p><strong>Console:</strong> <a href="${env.BUILD_URL}console">View Console Output</a></p>
                """,
                mimeType: 'text/html',
                to: 'qa-team@yourcompany.com',
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )

            // Also notify Slack
            script {
                sh """
                    curl -X POST '${SLACK_WEBHOOK_URL}' \
                        -H 'Content-type: application/json' \
                        --data '{
                            "text": "🔴 Playwright Tests FAILED",
                            "blocks": [
                                {
                                    "type": "section",
                                    "text": {
                                        "type": "mrkdwn",
                                        "text": "🔴 *Playwright Tests FAILED*\\n*Job:* ${env.JOB_NAME}\\n*Build:* #${env.BUILD_NUMBER}\\n*Branch:* ${env.BRANCH_NAME ?: 'N/A'}\\n*Report:* <${env.BUILD_URL}Playwright_Test_Report/|View Report>"
                                    }
                                }
                            ]
                        }'
                """
            }
        }

        // Notify Slack on recovery (was failing, now passing)
        fixed {
            script {
                sh """
                    curl -X POST '${SLACK_WEBHOOK_URL}' \
                        -H 'Content-type: application/json' \
                        --data '{"text": "✅ Playwright Tests RECOVERED — ${env.JOB_NAME} #${env.BUILD_NUMBER}"}'
                """
            }
        }

        // Clean workspace after successful builds to save disk space
        success {
            cleanWs(
                cleanWhenSuccess: true,
                cleanWhenFailure: false,  // Keep workspace on failure for debugging
                notFailBuild: true
            )
        }
    }
}
```

---

## Step 4: Create the Jenkins Job

### Option A: Multibranch Pipeline (Recommended)

A multibranch pipeline automatically discovers all branches and PRs in your repository.

1. **Jenkins Dashboard** → **New Item**
2. Name: `playwright-tests` → Select **Multibranch Pipeline** → OK
3. Under **Branch Sources**, add your Git repository
4. Under **Build Configuration**, set **Script Path**: `Jenkinsfile`
5. Under **Scan Multibranch Pipeline Triggers**, enable periodic scanning
6. Save

Jenkins will scan your repository and create a job for each branch that contains a `Jenkinsfile`.

### Option B: Pipeline Job (Single Branch)

1. **Jenkins Dashboard** → **New Item**
2. Name: `playwright-nightly` → Select **Pipeline** → OK
3. Under **Build Triggers**, check **Build periodically**: `0 2 * * *`
4. Under **Pipeline**, set **Definition**: `Pipeline script from SCM`
5. Set your repository URL and credentials
6. Set **Script Path**: `Jenkinsfile`
7. Save

---

## Blue Ocean Pipeline Visualization

Blue Ocean provides a modern, visual pipeline interface. After installing:

1. Click **Open Blue Ocean** from the Jenkins dashboard
2. Navigate to your pipeline
3. Click on a build to see the visual stage view
4. Failed stages are highlighted in red — click to see the step-by-step log

**Blue Ocean tips for QA engineers:**
- The **Tests** tab shows a breakdown of passed/failed tests (requires JUnit output)
- Parallel stages show as a horizontal branching path — easy to see which browser failed
- Click any step to jump directly to its log output

---

## Adding JUnit Output for Jenkins Test Results

Jenkins can display test results natively with the JUnit plugin. Add the JUnit reporter:

```javascript
// playwright.config.js
reporter: process.env.CI
  ? [
      ['html', { open: 'never' }],
      ['junit', { outputFile: 'test-results/junit.xml' }],
      ['list'],
    ]
  : [['html', { open: 'on-failure' }], ['list']],
```

Add to your Jenkinsfile after the test stage:

```groovy
post {
    always {
        // Parse JUnit XML and display results in Jenkins UI
        junit allowEmptyResults: true, testResults: 'test-results/junit.xml'

        // Also publish the HTML report
        publishHTML([...])
    }
}
```

---

## Using Docker Agent for Isolation

For consistent, isolated builds, use the Playwright Docker image as your agent:

```groovy
pipeline {
    agent {
        docker {
            // Pre-installed with Node.js + all browsers
            image 'mcr.microsoft.com/playwright:v1.44.0-jammy'
            // Mount the workspace
            args '-v ${WORKSPACE}:/workspace -w /workspace'
        }
    }

    stages {
        stage('Install') {
            steps {
                // Browsers already installed — just install npm deps
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npx playwright test --grep @smoke --project=chromium'
            }
        }
    }
}
```

**Benefits of Docker agent:**
- Same environment every time — no "works on my Jenkins but not yours"
- No need to install Node.js or Playwright on the Jenkins agent
- Easy to upgrade by changing the image tag

---

## Asking Zen to Help With Jenkins

### Generating the Jenkinsfile

```
I need to create a Jenkinsfile for my Playwright test suite.

My setup:
- Jenkins 2.400 with Blue Ocean, HTML Publisher, NodeJS plugins
- Playwright v1.44, Node.js 20
- Tests in tests/specs/ with tags @smoke, @regression
- playwright.config.js at root
- Node.js configured in Jenkins Tools as "NodeJS-20"
- Credentials stored in Jenkins:
  - staging-base-url (secret text)
  - test-user-email (secret text)
  - test-user-password (secret text)
  - slack-webhook-url (secret text)

I need:
1. Declarative pipeline with stages: Install, Smoke Tests, Regression Tests (parallel browsers), Publish Report
2. Regression only on scheduled/main branch runs
3. HTML Publisher for the Playwright HTML report
4. Email + Slack notification on failure
5. Timeout of 45 minutes

Please generate the complete Jenkinsfile.
```

### Debugging a Jenkins Build

```
My Jenkins build is failing. Here's the error from the console output:

[paste Jenkins console log]

Here's my Jenkinsfile:
[paste Jenkinsfile]

The tests pass locally. What's causing the failure and how do I fix it?
```

---

## Troubleshooting Common Jenkins Issues

| Issue | Symptom | Fix |
|-------|---------|-----|
| `node: not found` | Node.js not available | Add `tools { nodejs 'NodeJS-20' }` to pipeline; configure in Global Tool Configuration |
| `Executable doesn't exist` (browser) | Playwright browsers missing | Add `sh 'npx playwright install --with-deps'` to Install stage |
| HTML report shows "Content Security Policy" error | CSP blocking report rendering | Go to Manage Jenkins → Script Console: run `System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")` |
| Credentials not injected | Wrong credential ID | Verify the ID in `credentials('your-id')` matches exactly what was saved in Jenkins |
| Parallel stages don't fail the build | `failFast` not set | Add `failFast: true` to the `parallel` block if you want one failure to stop all |
| `cleanWs()` fails | Plugin not installed | Install `Workspace Cleanup` plugin |
| Docker image not found | Runner can't pull image | Verify Docker is installed on the agent; check proxy settings |

---

## Summary

| Feature | Implementation |
|---------|---------------|
| Node.js version | Jenkins NodeJS Tool (`NodeJS-20`) |
| Secrets | Jenkins Credentials Store |
| Smoke tests | Stage: runs on every build |
| Regression tests | Stage: conditional (`when`) — scheduled or main branch |
| Cross-browser | `parallel` stages in the Regression stage |
| HTML report | HTML Publisher plugin |
| Email notification | Email Extension plugin |
| Slack notification | `curl` POST to webhook in `post { failure }` |
| Test results display | JUnit plugin (requires JUnit reporter) |
| Docker isolation | Optional: `agent { docker { image '...' } }` |

---

**Next:** [CircleCI →](./04-circleci.md) | [Back to CI/CD Overview →](./README.md)
