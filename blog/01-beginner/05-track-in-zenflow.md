# Track All Your QA Work in Zenflow

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Test Execution Checklists](./04-test-execution-checklist.md) | [Next Section: Intermediate →](../02-intermediate/README.md)

---

## Introduction

QA work has a visibility problem that most QA engineers underestimate until it costs them something.

A developer's work is visible by default: every commit is logged, every PR is reviewed, every deployment is recorded. A QA engineer can spend a full week testing complex scenarios, writing detailed bug reports, and preventing multiple production incidents — and if none of that is tracked, it looks like they did nothing.

This isn't about self-promotion. It's about feedback loops. When your work is visible:
- Your team knows which features are cleared for release
- Product managers can adjust scope based on testing status
- You can answer "what did you test?" from memory rather than reverse-engineering your week
- Zen can refer to your task history to give you better context-aware help
- Retrospectives have data instead of impressions

Zenflow is your tracking system. This guide shows you how to use it for every category of QA work.

---

## The Five Task Types Every QA Engineer Needs

### 1. Bug Tasks

Every bug you file should be a Zenflow task. This gives you:
- A permanent record of all bugs with status tracking
- The ability to query your bug history with Zen
- Visibility to product managers and developers without a separate tool
- A trail for retrospectives ("we filed 23 bugs in this sprint, 18 were P1/P2")

**Bug Task Template:**

```
Title: [BUG] [Module] - [What fails] when [condition]

Description:
**Environment:** [URL, browser, OS, app version, test account]
**Severity:** [Critical / High / Medium / Low]
**Priority:** [P1 / P2 / P3 / P4]
**Reproducibility:** [100% / Intermittent (X/10) / Unable to reproduce after initial]

**Steps to Reproduce:**
1. 
2. 
3. 

**Expected Result:**
[What should happen]

**Actual Result:**
[What actually happened]

**Evidence:**
- [screenshot link]
- [video link]
- [network trace / API response]

**Additional Context:**
[Impact, related bugs, developer notes]
```

**Prompt to generate a bug task from raw notes:**

```
Convert these raw bug notes into a properly formatted Zenflow bug task:

Raw notes: [paste your raw observations]
Application context: [brief description]
Environment: [where you were testing]

Use the bug task template format. Fill in all fields based on the information I provided.
For any field I didn't explicitly mention, use your best judgment based on context 
and flag it with [NEEDS CONFIRMATION].
```

**Bug task workflow stages:**

```
New → Triaged → In Progress (dev fixing) → Fixed → Verifying → Closed
```

For each stage transition:
- **New → Triaged:** PM or tech lead reviewed and confirmed it's a real bug
- **Triaged → In Progress:** Developer picked it up
- **In Progress → Fixed:** Developer says fix is deployed to staging
- **Fixed → Verifying:** QA is actively retesting the fix
- **Verifying → Closed:** QA confirmed the fix works (or the bug didn't qualify as a bug)

---

### 2. Test Execution Tasks

For every planned test session — whether a smoke test, regression, or exploratory session — create a task. This makes your testing work visible and auditable.

**Test Execution Task Template:**

```
Title: [TEST RUN] [Type] - [Feature/Release] - [Date]
Example: [TEST RUN] Regression - v2.5.0 Release - 2024-02-01

Description:
**Test Type:** [Smoke / Sanity / Regression / Exploratory]
**Scope:** [What features/modules are being tested]
**Release/Build:** [Version being tested]
**Environment:** [Staging / UAT / Production]
**Testers:** [Who is running this]
**Scheduled:** [Date and time]
**Estimated Duration:** [X hours]

**Checklist:**
[paste or link to your test checklist]

**Acceptance Criteria (when is this task Done?):**
- All checklist items executed
- Results documented for each item
- All P1/P2 bugs filed as separate tasks
- Go/No-Go recommendation submitted to PM

**Progress Updates:**
[Updated during execution — add comments as sections are completed]

**Final Results:**
Total: X | Passed: X | Failed: X | Blocked: X
Go/No-Go: [GO / NO-GO — with reason]
```

**Setting up subtasks for large test runs:**

For a full regression, create subtasks by module:

```
Parent task: [TEST RUN] Regression — v2.5.0
  ↳ Subtask: Authentication module
  ↳ Subtask: Checkout and payment
  ↳ Subtask: Coupon engine (HIGH RISK)
  ↳ Subtask: New dashboard feature
  ↳ Subtask: Mobile web (iOS Safari, Chrome iOS)
```

Each subtask can be assigned to a different tester and tracked independently.

---

### 3. Test Case Creation Tasks

When you're building out test coverage for a feature, track it as a task. This is often invisible work that takes significant time.

**Test Case Task Template:**

```
Title: [TEST CASES] Write test cases for [Feature Name]

Description:
**Feature:** [Name and brief description]
**Requirements link:** [Link to PRD or user stories]
**Target coverage:** [What scenarios to cover]
**Estimated cases:** [Rough number]
**Priority:** [When does this need to be done?]

**Test Case Modules to Cover:**
- [ ] Happy path scenarios
- [ ] Negative / error scenarios
- [ ] Edge cases and boundary values
- [ ] API-level test cases (if applicable)
- [ ] Permission/role-based test cases

**Definition of Done:**
- [ ] All modules listed above have at least one test case
- [ ] Each test case has complete preconditions, steps, and expected results
- [ ] Test cases reviewed by at least one developer for technical accuracy
- [ ] Cases loaded into test management tool OR linked from this task
- [ ] Coverage map updated to reflect new coverage
```

---

### 4. Coverage Improvement Tasks

When your coverage audit reveals a gap that needs addressing, create a dedicated task. This prevents coverage improvements from being forgotten in favor of urgent bugs.

**Coverage Improvement Task Template:**

```
Title: [COVERAGE] Add test coverage for [Area] - [Gap type]
Example: [COVERAGE] Add API test coverage for /orders endpoint error paths

Description:
**Gap identified:** [Describe what's not covered and why it matters]
**Risk if uncovered:** [What bugs could slip through without this coverage?]
**Estimated effort:** [Number of test cases needed, time to write them]
**Dependencies:** [What needs to be in place before these tests can be written?]

**Why This Matters:**
[Business case — what user impact or revenue is at risk?]

**Acceptance Criteria:**
- [ ] [X] test cases written for [scenario type]
- [ ] Tests executed and all pass (or failures filed as bugs)
- [ ] Coverage map updated
```

---

### 5. Process Improvement Tasks

As you discover process problems — things that slow you down, cause mistakes, or create inconsistency — capture them as tasks. This is how QA engineers continuously improve their own workflow.

**Examples:**
- `[PROCESS] Set up pre-commit hook to run smoke tests automatically`
- `[PROCESS] Create standardized bug report template for the team`
- `[PROCESS] Document test data setup procedure for new team members`
- `[PROCESS] Review and update stale test cases from Q3 (>6 months old)`

---

## Setting Up Recurring Automations

The most effective QA teams have rhythms — regular practices that happen whether or not anyone explicitly asks for them. Zenflow automations enforce these rhythms automatically.

### Automation 1: Weekly Coverage Review

**When:** Every Monday, 9:00 AM

**Task created:**
```
Title: Weekly QA Coverage Review — [auto week]
Description: 
- Review coverage map: are there new features without coverage?
- Review open bugs: any P1/P2 issues languishing?
- Review last week's test results: any patterns in failures?
- Update memory/_index.md with any significant changes
- Ask Zen: "What should I prioritize testing this week based on my project context?"
```

### Automation 2: Pre-Release Smoke Test Reminder

**When:** Every Tuesday and Thursday, 2:00 PM (or timed to your release cycle)

**Task created:**
```
Title: Smoke Test — [auto date]
Description:
Run the standard smoke test checklist before EOD.
Checklist: [link to smoke test checklist]
If anything fails: file a bug task immediately and notify the dev team.
Update this task with pass/fail results before closing.
```

### Automation 3: Monthly Test Case Audit

**When:** First Monday of each month, 10:00 AM

**Task created:**
```
Title: Monthly Test Case Audit — [auto month]
Description:
Review all test cases for staleness:
- Have any features changed that make existing test cases invalid?
- Are there new features added last month with no test cases?
- Are there deprecated features with test cases that should be removed?

Use Zen prompt: "Review my coverage map and test case list. What's stale, 
what's missing, and what should I prioritize this month?"
```

### Automation 4: Sprint Retrospective Prep

**When:** Last Friday of each sprint (typically biweekly)

**Task created:**
```
Title: Sprint QA Retrospective Prep — [auto sprint]
Description:
Prepare QA metrics for the retrospective:
- Total bugs filed this sprint
- Bugs by severity/priority breakdown
- Tests run vs. tests planned
- Coverage improvements made
- What slowed down testing this sprint?
- What improved?

Use Zen prompt: "Summarize my QA work this sprint based on the tasks in Zenflow.
Generate a retrospective summary I can share with the team."
```

### Setting Up Automations in Zenflow

1. Go to your project → **Automations** → **New Automation**
2. Set the trigger: **Schedule** → choose time and days
3. Set the action: **Create Task** → paste your task template
4. Toggle the automation to **Active**
5. Confirm the first task is created at the scheduled time

---

## Reading and Interpreting Task Progress

### Your QA Dashboard at a Glance

In Zenflow, use these filters to get instant status views:

| View | Filter | Tells You |
|------|--------|-----------|
| Active bugs | Label: `bug`, Status: not `Closed` | Open issues you're responsible for |
| P1 bugs | Label: `bug`, Label: `P1` | Critical issues requiring immediate attention |
| Pending regressions | Label: `test-run`, Status: `To Do` | Upcoming test sessions |
| Coverage gaps | Label: `coverage-gap` | Areas needing more test cases |
| Blocked items | Label: `blocked` | Items waiting on someone else |

### Generating a Status Report with Zen

At any point, ask Zen to summarize your work:

```
Please generate a weekly QA status report based on my Zenflow tasks.

This week's context:
- Sprint: [sprint number/name]
- Release target: [release name/date]
- Team: [who's on the team this sprint]

Here's a summary of my Zenflow activity this week:
- Bugs filed: [list titles and priorities]
- Test runs completed: [list test sessions with results]
- Coverage work done: [what was added]
- Blockers: [what's blocking testing]

Format this as:
1. Executive Summary (2-3 sentences, non-technical)
2. Testing Progress (what was tested, results)
3. Issues Found (bug count by priority, key issues)
4. Coverage Status (% change if tracked)
5. Risks and Blockers
6. Next Week's Plan
```

---

## Demonstrating Your Value to Stakeholders

One of the most practical uses of Zenflow tracking is answering the question every QA engineer faces eventually: "What exactly have you been doing?"

With good Zenflow tracking, you can pull data like:

- "This sprint, I filed 17 bugs. 4 were P1 severity — all fixed before release."
- "We increased test coverage of the checkout module from 45% to 78% this quarter."
- "I ran 3 regression test sessions totaling 127 test cases. Pass rate: 91%."
- "3 of the 5 production incidents this quarter could have been caught with the coverage we added in February."

**Generating a quarterly QA impact report with Zen:**

```
I need to generate a quarterly QA impact report showing my team's value.

Data from this quarter:
- Bugs filed: [count] across [sprints/months]
- Bugs by severity: Critical: X, High: X, Medium: X, Low: X
- Test cases added: [count]
- Coverage improvement: from X% to Y%
- Test runs completed: [count]
- Production incidents this quarter: [count] — estimated X% preventable with our coverage
- Process improvements implemented: [list]

Please write a 1-page QA Impact Report that:
1. Leads with business value (prevented incidents, blocked releases of broken code)
2. Shows concrete metrics with trend
3. Describes qualitative improvements (processes, knowledge base)
4. Frames testing as investment, not overhead
5. Includes a forward-looking section: what we're building toward next quarter

Tone: professional but direct. Audience: Engineering Manager and VP Product.
```

---

## Zenflow Task Hygiene

A few rules to keep your workspace useful instead of cluttered:

### Daily (2 minutes)
- Update any in-progress task status
- Add a comment to any task where there's a new development
- Close any tasks that are genuinely done

### Weekly (10 minutes)
- Close resolved bugs that developers have marked fixed and you've verified
- Archive completed test run tasks
- Review your automations — did the tasks get created? Were they completed?

### Monthly (20 minutes)
- Review all open tasks. Archive or close anything older than 60 days with no activity.
- Update your memory file with any changes to the project context
- Update your coverage map in the memory file

---

## Beginner Section Checkpoint

Congratulations — you've completed the Beginner section. Let's verify:

- [ ] I have generated at least 20 test cases for a real feature in my project
- [ ] I have a working test coverage map showing at least 10 features
- [ ] I have written at least 3 bug reports using the AI-assisted template
- [ ] I have a pre-release smoke test checklist ready for my next release
- [ ] I have set up at least 2 recurring automations in Zenflow
- [ ] All my current bugs, test runs, and coverage work are tracked in Zenflow
- [ ] I can generate a status report in under 5 minutes using Zen

---

## What's Next

You've built the foundation. Your testing work is structured, documented, visible, and AI-assisted. In the Intermediate section, you'll go deeper: API testing without being a developer, exploratory testing with AI guidance, basic automation script writing, and building a performance testing baseline.

**→ Continue to the [Intermediate Section](../02-intermediate/README.md)**

---

*Didn't find what you needed in this section? Ask Zen directly:*

```
I just finished the Beginner section of the AI-Assisted QA series.
My specific situation is: [describe your role, your application, your current challenges].
What should I focus on next, and which Intermediate topics are most relevant to me?
```
