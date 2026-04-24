# AI-Assisted QA: From Manual Tester to Automation Strategist
## Expert Guide 02 — The Quarterly Coverage Audit

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** Expert
> **← Prev:** [Risk-Based Planning](./01-risk-based-planning.md) | **Next →** [Suite Optimization](./03-suite-optimization.md)

---

## Why Coverage Audits Fail (And How to Fix That)

Most QA teams track coverage as a single number: "We have 847 automated tests." This number is almost meaningless. It doesn't tell you:

- Which *requirements* are covered
- Which *risk areas* are covered
- How much of that coverage is redundant
- Which critical flows have *no* coverage at all

A coverage audit isn't about counting tests. It's about answering three questions:

1. **What are we testing that we don't need to?** (Over-testing = wasted CI time and maintenance burden)
2. **What aren't we testing that we should be?** (Under-testing = escaped defects)
3. **How does our coverage map to actual business risk?** (Coverage quality = risk-aligned testing)

Run this audit quarterly. It takes about 4 hours. Done consistently, it's the single highest-leverage QA activity you can schedule.

---

## Phase 1: Inventory Everything

Before you can audit, you need a complete map of what exists.

### 1A: Automated Test Inventory

For each automated test, capture:

| Field | Source |
|-------|--------|
| Test name | Playwright test file |
| Test file path | File system |
| Feature/component | Directory structure or tag |
| Risk tier | Your risk model from Guide 01 |
| Last modified | `git log --follow -1 --format="%ai" <file>` |
| Average runtime | CI run data |
| Flaky rate (last 30 days) | CI flakiness report |
| Linked requirement | Manual tag or mapping |

**Bash command to generate a base inventory:**

```bash
# List all Playwright test files with their last modification time
find ./tests -name "*.spec.ts" -exec sh -c '
  file="$1"
  modified=$(git log --follow -1 --format="%ai" "$file" 2>/dev/null || echo "unknown")
  echo "$file | $modified"
' _ {} \;
```

**Generate test count per directory:**

```bash
find ./tests -name "*.spec.ts" | \
  sed 's|/[^/]*$||' | \
  sort | uniq -c | sort -rn
```

### 1B: Manual Test Inventory

Pull your manual test cases from wherever they live (TestRail, Confluence, spreadsheets). For each:

| Field | Notes |
|-------|-------|
| Test case ID | From your test management tool |
| Title | Brief description |
| Feature/component | Category |
| Last executed | Date |
| Result of last run | Pass/Fail/Not Run |
| Automation candidate? | Your assessment |

If you don't have formal manual test cases, create a list of "manual test rituals" — things you (or your team) habitually check before releases that aren't written down anywhere.

> **Zen Prompt:** "I have a QA team that does the following informal pre-release checks: [list]. Help me convert these into formal manual test cases with titles, preconditions, steps, and expected results."

### 1C: Requirements Inventory

This is the hardest part because requirements are often scattered or implicit. Sources to mine:

- Product specs and PRDs (Confluence, Notion, Google Docs)
- Accepted user stories from your sprint backlog (Jira, Linear)
- Bug reports — each bug implies a requirement that wasn't met
- API contracts and OpenAPI specs
- Regulatory requirements (if applicable)

**Goal:** Produce a flat list of user-facing behaviors the system is supposed to support. You don't need every edge case — focus on the *capabilities*.

Example excerpt:
```
REQ-001: User can register with email and password
REQ-002: User can log in with valid credentials
REQ-003: User receives email verification after registration
REQ-004: User can reset password via email
REQ-005: User can update profile information
REQ-006: User can delete account (with 30-day grace period)
...
```

---

## Phase 2: Build the Traceability Matrix

The Requirements Traceability Matrix (RTM) links requirements to tests. It's the core artifact of your coverage audit.

### RTM Structure

| Req ID | Requirement | Risk Tier | Automated Tests | Manual Tests | Coverage Status |
|--------|-------------|-----------|-----------------|--------------|-----------------|
| REQ-001 | User registration | P0 | auth/register.spec.ts (3 tests) | TC-014, TC-015 | ✅ Good |
| REQ-002 | User login | P0 | auth/login.spec.ts (5 tests) | TC-016 | ✅ Good |
| REQ-003 | Email verification | P1 | None | TC-017 | ⚠️ Manual Only |
| REQ-004 | Password reset | P1 | auth/reset.spec.ts (2 tests) | None | ⚠️ Limited |
| REQ-005 | Profile update | P2 | profile.spec.ts (1 test) | None | ✅ Acceptable |
| REQ-006 | Account deletion | P1 | None | None | 🔴 Dark Area |

**Coverage Status Definitions:**

| Status | Meaning |
|--------|---------|
| ✅ Good | Automated coverage proportional to risk tier |
| ⚠️ Manual Only | No automation; acceptable for P2/P3, gap for P0/P1 |
| ⚠️ Limited | Automation exists but doesn't cover key scenarios |
| 🔴 Dark Area | No coverage of any kind |
| 🔵 Over-Tested | More coverage than risk tier justifies |

---

## Phase 3: Detect Over-Testing

Over-testing is the budget you can reclaim. It hides in plain sight:

### Signal 1: Duplicate Test Intent

Tests that verify the same behavior from different angles without adding new risk coverage:

```typescript
// These three tests are functionally redundant:
test('user can click the login button', async ({ page }) => { ... })
test('login button navigates to dashboard', async ({ page }) => { ... })
test('after login, user sees dashboard', async ({ page }) => { ... })

// One test should cover this:
test('successful login redirects to dashboard', async ({ page }) => { ... })
```

**Zen Prompt to find duplicates:**

```
Here is a list of my automated test names:
[paste test names]

Identify:
1. Groups of tests that appear to be testing the same behavior
2. Tests where the test name suggests redundancy (similar intent)
3. Suggested consolidations with a merged test name

Format as a table: Test Group | Tests to Merge | Suggested Replacement Name
```

### Signal 2: P3 Components With Heavy Automation

If your risk model shows P3 (Low Risk) but your test inventory shows 20+ tests, you're over-investing. This happens when:
- A feature was once high-risk but has since stabilized
- Someone wrote tests for a feature they were excited about, not for risk
- Tests were migrated from a previous tool without re-evaluation

### Signal 3: Setup-Heavy Tests With Low Validation

Tests that take 30+ seconds to execute but only assert one trivial condition are prime optimization targets — or deletion candidates.

```typescript
// Anti-pattern: 45 seconds of setup to check a page title
test('settings page title', async ({ page }) => {
  await loginAsAdmin(page);           // 8 seconds
  await navigateToSettings(page);     // 5 seconds  
  await page.waitForLoadState();      // 3 seconds
  // ...30 seconds of other setup...
  
  await expect(page.locator('h1')).toHaveText('Settings'); // The only assertion
});
```

### Signal 4: Tests That Never Fail

A test that has passed 100% of the time for 6+ months with no code changes in the area is either:
- A valuable regression guard (keep it)
- Testing something so stable it's noise (evaluate for removal)

Pull your CI pass rate data and flag tests with 100% pass rate over 6 months. Review each one for whether it would actually catch a regression.

---

## Phase 4: Detect Under-Testing (Dark Areas)

Dark areas are the coverage gaps that escaped defects fall through.

### Method 1: RTM Gap Analysis

From your traceability matrix, every row with `🔴 Dark Area` is a gap. Rank them by risk tier — P0/P1 dark areas are sprint-worthy backlog items.

### Method 2: Bug Origin Analysis

Pull all bugs from the last 6 months. For each one, find the answer to: *"Would any of our current tests have caught this before release?"*

If the answer is "no," you've found a coverage gap. Categorize these:

```
Bug Analysis — Q3 2024
======================
Total bugs: 47
Bugs caught by existing automation: 12 (26%)
Bugs caught by manual testing: 18 (38%)
Bugs reached production (escaped): 17 (36%)

Of escaped bugs:
- 8 were in areas with NO automation
- 6 were edge cases not covered by existing tests
- 3 were in P0 components — critical gap

Top uncovered areas:
1. Payment confirmation email flow
2. Concurrent session handling  
3. Account merge edge cases
```

### Method 3: Code Coverage as a Signal (Not a Goal)

Istanbul/V8 code coverage isn't the right primary metric for E2E tests, but it can surface blind spots:

```bash
# Run Playwright with V8 coverage
npx playwright test --reporter=html
# Then check which modules have 0% E2E touch
```

Use code coverage to *identify candidate areas for exploratory testing*, not as a benchmark for success.

### Method 4: Feature Changelog Cross-Reference

Pull your git commit history and product changelog for the last quarter. Map each feature shipped to coverage status:

```bash
# Features shipped in last quarter (from git tags or CHANGELOG)
git log --oneline v2.3.0..v2.6.0 --merges | grep -i "feat\|feature\|add"
```

For each shipped feature: does it have corresponding tests added in the same timeframe? If not, it's a gap.

---

## Coverage Metrics to Track

### Metrics Dashboard — Quarterly Snapshot

| Metric | Formula | Target | Red Flag |
|--------|---------|--------|----------|
| **Requirement Coverage %** | Requirements with ≥1 test / Total requirements | >90% | <70% |
| **P0 Automation Coverage %** | P0 reqs with automation / Total P0 reqs | >95% | <80% |
| **P1 Automation Coverage %** | P1 reqs with automation / Total P1 reqs | >80% | <60% |
| **Dark Area Count** | Requirements with 0 tests | <5 | >15 |
| **Over-Test Ratio** | P3 tests / Total tests | <10% | >25% |
| **Orphaned Tests** | Tests with no linked requirement | <5% | >20% |
| **Stale Test Rate** | Tests not modified in 12+ months in active areas | <15% | >30% |

### Tracking Over Time

Track these quarterly and graph the trend. A rising Dark Area Count is more alarming than a static one, even if the absolute number is low.

---

## Zen-Assisted Audit Prompts

### Prompt 1: RTM Construction From User Stories

```
I'm building a Requirements Traceability Matrix for our QA coverage audit.

Here are our accepted user stories from the last 2 quarters:
[paste user stories]

For each user story:
1. Extract a clear, testable requirement statement (REQ-XXX format)
2. Identify the primary user action and expected outcome
3. Suggest the risk tier (P0/P1/P2/P3) based on this heuristic:
   - P0: Core revenue flow, authentication, data integrity
   - P1: Key user journeys, notifications, account management
   - P2: Secondary features, filters, sorting, exports
   - P3: Cosmetic, help content, informational pages

Output as a table: REQ ID | Requirement | Risk Tier | Rationale
```

### Prompt 2: Duplicate Test Detection

```
Here are the names of all automated tests in our suite:
[paste test names, one per line]

Group these tests by:
1. Likely duplicate intent (tests that appear to cover the same scenario)
2. Overlapping coverage (tests where one likely subsumes another)
3. Redundant setup-to-assertion ratio (tests that sound like they do heavy setup for minimal verification)

For each group, suggest whether to: merge, delete, or keep as-is, and explain why.
```

### Prompt 3: Dark Area Prioritization

```
Here are the dark areas (untested requirements) from our coverage audit:

[paste list of untested requirements with risk tiers]

Given our risk model:
- P0 dark areas need immediate automation
- P1 dark areas should be addressed within this quarter
- P2 dark areas need at least manual test coverage
- P3 dark areas can remain uncovered with documented rationale

Please:
1. Rank these by urgency (combining risk tier with estimated test complexity)
2. Estimate the test effort for each (in story points or hours)
3. Draft a prioritized backlog of automation stories
4. Flag any that might be faster to address with exploratory testing sessions
   instead of automation
```

### Prompt 4: Coverage Trend Analysis

```
Here's our coverage audit data for the last 4 quarters:

Q1: [paste metrics]
Q2: [paste metrics]
Q3: [paste metrics]
Q4: [paste metrics]

Analyze:
1. Which metrics are trending in the wrong direction?
2. Which improvements are most significant?
3. What do the trends tell us about our team's priorities?
4. What should be our top 3 focus areas next quarter?

Format as a QA Health Trend Report I can share with engineering leadership.
```

---

## Turning Audit Findings Into a Sprint Backlog

A coverage audit without action is just documentation. Here's how to convert findings into work.

### The Audit-to-Backlog Pipeline

**Step 1: Categorize findings**

| Finding Type | Action | Priority |
|-------------|--------|----------|
| P0 dark area | Write automation | Must-do this sprint |
| P1 dark area | Write automation or structured manual | This quarter |
| P1 manual-only | Convert to automation | This quarter |
| Duplicate tests | Merge or delete | Nice-to-have |
| Orphaned tests | Link to requirement or delete | Nice-to-have |
| Stale tests in active areas | Review and update | This sprint |

**Step 2: Write actionable stories**

Bad story: *"Improve coverage of account deletion"*

Good story:
```
[QA] Automate account deletion flow — 3 scenarios
Acceptance Criteria:
- AC1: Test covers standard account deletion (30-day grace period warning shown)
- AC2: Test covers attempted login during grace period (account locked message)
- AC3: Test covers re-registration with deleted email after grace period expires
Risk Tier: P1
Estimated effort: 4 hours
Linked requirement: REQ-006
```

**Step 3: Balance coverage work with feature work**

A sustainable ratio: **20% of QA sprint capacity** goes to coverage improvement stories. If your audit reveals a critical P0 dark area, escalate it to the engineering sprint — it's a quality debt item that belongs on the engineering backlog, not just the QA backlog.

**Step 4: Set quarterly OKRs from audit output**

```
QA Coverage OKRs — Q1 2025
===========================
Objective: Close critical coverage gaps identified in Q4 audit

KR1: Bring P0 automation coverage from 78% to 95%
KR2: Eliminate all 6 identified P0/P1 dark areas
KR3: Reduce orphaned test count from 47 to <10
KR4: Remove or merge 30 identified duplicate tests (saving ~8min CI runtime)
```

---

## The Audit Calendar

| Month | Activity | Duration |
|-------|----------|----------|
| January (Q4 → Q1) | Full audit | 4 hours |
| February | Act on Q1 backlog | Ongoing |
| March | Mid-quarter check (metrics only) | 30 minutes |
| April (Q1 → Q2) | Full audit | 4 hours |
| ... | ... | ... |

**Pro tip:** Schedule the full audit as a recurring calendar block 2 weeks before your quarterly planning. That way, audit findings directly inform your team's quarterly priorities while they're still being set.

---

## Checkpoint

Before moving to Suite Optimization, make sure you have:

- [ ] A complete test inventory (automated + manual)
- [ ] A requirements list (even a rough one) mapped to your product
- [ ] At least a partial RTM — even 20 requirements is a starting point
- [ ] At least 3 dark areas identified
- [ ] At least 3 over-tested areas identified
- [ ] A prioritized backlog item for the most critical gap

**Next:** [Suite Optimization →](./03-suite-optimization.md) — You know what to test. Now let's make sure your suite doesn't take an hour to run.
