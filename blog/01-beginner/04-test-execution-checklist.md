# Build Test Execution Checklists

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Bug Reports](./03-bug-reports.md) | [Next: Track in Zenflow →](./05-track-in-zenflow.md)

---

## Introduction

The hour before a release is chaotic. Messages are flying in, someone changed a requirement at 3 PM on deploy day, and you're trying to remember whether you covered that edge case in the payment flow. This is when bugs slip through — not because you can't find them, but because cognitive overload means you forget to look.

A pre-release checklist eliminates that cognitive load. It doesn't make you think less — it frees up your thinking for the things that actually require judgment, by handling the "did I remember to test X" question systematically.

Different release types need different checklists. This guide shows you how to build the right checklist for every situation and how to use Zenflow to track execution in real time.

---

## The Four Checklist Types

### 1. Smoke Test Checklist

**When to use:** First thing after any deployment, before deeper testing begins.

**Purpose:** Verify the application is alive and the most critical paths work. This is a "go / no-go" signal. If smoke tests fail, you stop and the deployment is rolled back — no point testing further.

**Scope:** 10–20 tests maximum. Must run in under 15 minutes.

**Characteristics:**
- Tests only the most critical, revenue-impacting flows
- Covers one happy path per major feature — no edge cases
- Can be run by any QA engineer regardless of feature knowledge
- Should be identical every time — no variation

**Example smoke tests (e-commerce):**
1. Application loads without errors
2. User can log in with valid credentials
3. Product listing page loads with products
4. User can add product to cart
5. Checkout flow can be initiated
6. Payment can be completed (with test card)
7. Order confirmation is displayed
8. Order appears in order history
9. User can log out
10. Application loads correctly on mobile (responsive)

---

### 2. Sanity Test Checklist

**When to use:** After a bug fix or a small targeted change.

**Purpose:** Verify that a specific fix works and didn't break adjacent functionality. More focused than smoke, but deeper than a single test.

**Scope:** 5–15 tests focused on the changed area and its neighbors.

**Characteristics:**
- Tests the specific area that was changed
- Tests related areas that could have been affected (regression check)
- Faster than full regression — not exhaustive
- Changes with each fix (it's specific to what changed)

**Example sanity tests for a coupon fix:**
1. Valid coupon applies correctly
2. Expired coupon shows correct error
3. Invalid coupon shows correct error
4. Coupon with minimum order not met shows correct error
5. Cart total updates correctly after coupon applied
6. Coupon removed on order cancellation
7. Payment still completes successfully after coupon applied (regression check)

---

### 3. Regression Test Checklist

**When to use:** Before a major release, after significant new features are merged.

**Purpose:** Ensure that existing functionality still works after new code has been added. Regression is about protecting what already worked — not about testing new things.

**Scope:** Comprehensive — covers all major features. May take hours or days.

**Characteristics:**
- Organized by module/feature area
- Includes positive and negative cases for each area
- Prioritized by risk (highest risk areas first)
- Can be distributed across multiple testers
- Tracks pass/fail per test case

---

### 4. Exploratory Testing Checklist

**When to use:** When testing a new feature, after a regression suite is complete, or when hunting for unexpected issues.

**Purpose:** Discover bugs that structured tests won't find. Exploratory testing is about applying your judgment and curiosity — not following a script. But a charter keeps you focused.

**Scope:** Time-boxed (30–90 minutes per session).

**Characteristics:**
- Defines the area to explore (not the exact steps)
- Sets a mission or question to answer
- Notes are taken during the session
- Findings are reported after the session

---

## Prompt Templates for Each Checklist Type

### Generate a Smoke Test Checklist

```
Please generate a smoke test checklist for our application.

Application: [name and brief description]
Tech stack: [frontend, backend]
Most critical user journeys (in order of business importance):
1. [Journey 1 — e.g., "User can register, log in, and access main feature"]
2. [Journey 2 — e.g., "User can complete a purchase"]
3. [Journey 3 — e.g., "Admin can manage users"]

Constraints:
- Must complete in 15 minutes or less
- Should have 15-20 test items maximum
- Each item should be a single, clear action with a pass/fail outcome

Format as a numbered checklist with:
- Test item description
- Expected outcome (one sentence)
- Estimated time (seconds)
```

### Generate a Sanity Test Checklist

```
I need a sanity test checklist for a specific change.

What changed: [describe the fix or feature that was implemented]
Affected area: [feature or module]
Adjacent areas that could be affected: [list related features]
Known regression risks: [anything the developer mentioned could be affected]

Please generate a sanity test checklist that:
1. Tests the specific change (3-5 tests)
2. Tests adjacent functionality that could have regressed (3-5 tests)
3. Includes one "circuit breaker" test — a critical path that absolutely must work

Keep it to 10-15 items maximum. Format as: Test Description | Pass/Fail | Notes
```

### Generate a Regression Test Checklist

```
I need a regression test checklist for our upcoming release.

Release: [version or sprint number]
What changed in this release: [list of new features and fixes]
What's new that might affect existing behavior: [specific risk areas]
Modules to include in regression:
- [Module 1]
- [Module 2]
- [Module 3]

Test accounts available: [list test accounts and their roles]
Time available for regression: [X hours/days]
Number of testers: [X]

Please generate a comprehensive regression test checklist organized by module.
For each module:
1. List the most important happy path tests (2-3 per feature)
2. List the most important negative tests (1-2 per feature)
3. Flag any area that is HIGH RISK based on what changed in this release

Also give me:
- A time estimate for the full regression
- How to split the work if I have multiple testers
- Which modules to test first based on risk
```

### Generate an Exploratory Testing Charter

```
I'm about to do exploratory testing on [feature/area]. 
Help me write a testing charter for a [30/60/90]-minute session.

Feature: [describe what it does]
New or recently changed: [yes/no, describe what's new]
Known risk areas: [any concerns the dev team mentioned]
User persona to embody: [who would use this? what would they try?]

Please write an exploratory testing charter with:
1. Mission statement: "Explore [area] to discover [type of issues]"
2. Focus areas: what specifically to explore
3. Boundaries: what's out of scope for this session
4. Techniques to use: (e.g., error guessing, boundary testing, state transition)
5. Oracle: how to know if something is a bug
6. Notes template: what to capture during the session
```

---

## Full Example: Monthly Release Checklist (SaaS)

Below is an AI-generated regression checklist for a mid-sized SaaS platform, customized for a monthly major release.

---

**Release:** ShopCore v2.5.0
**Release Date:** 2024-02-01
**Changes This Release:** New dashboard metrics page, checkout UI refresh, coupon engine rewrite

---

### CRITICAL — Must Pass (P1)

| # | Test | Expected Result | Pass/Fail | Tester | Notes |
|---|------|----------------|-----------|--------|-------|
| 1 | User can log in with valid credentials | Dashboard loads within 3s | | | |
| 2 | User can complete full purchase (Stripe test card) | Order confirmation shown, order in DB | | | |
| 3 | User can complete full purchase (PayPal) | Order confirmation shown, order in DB | | | |
| 4 | Admin can log in and access admin panel | Admin panel loads, all sections accessible | | | |
| 5 | Application does not crash on mobile Safari | All critical pages load without JS errors | | | |

### AUTHENTICATION (High Risk — no changes, but critical path)

| # | Test | Expected Result | Pass/Fail | Tester | Notes |
|---|------|----------------|-----------|--------|-------|
| 6 | Register new account | Account created, verification email sent | | | |
| 7 | Verify email via link | Account verified, user can log in | | | |
| 8 | Login with invalid password | Error shown, account NOT locked after 1 attempt | | | |
| 9 | Login fails 5 times — account locked | Account locked, lockout email sent | | | |
| 10 | Password reset flow | Reset email received, can set new password | | | |
| 11 | Social login (Google) | Successfully authenticated | | | |
| 12 | Log out | Session destroyed, can't access protected pages | | | |

### CHECKOUT (HIGH RISK — UI refresh this release)

| # | Test | Expected Result | Pass/Fail | Tester | Notes |
|---|------|----------------|-----------|--------|-------|
| 13 | Add physical product to cart | Cart shows item, quantity, price | | | |
| 14 | Add digital product to cart | Cart shows item, no shipping required | | | |
| 15 | Add mixed cart (physical + digital) | Cart shows both, shipping only for physical | | | |
| 16 | Update cart quantity | Cart total updates correctly | | | |
| 17 | Remove item from cart | Item removed, total recalculates | | | |
| 18 | Apply valid coupon (SAVE10) | 10% discount applied, total updated | | | |
| 19 | Apply expired coupon (EXPIRED99) | Error: "This coupon has expired" | | | |
| 20 | Checkout as guest user | Can complete purchase without account | | | |
| 21 | Checkout as logged-in user | Saved addresses available | | | |
| 22 | Enter new shipping address | Address saved, shipping calculated | | | |
| 23 | Select saved address | Address populates, shipping calculated | | | |
| 24 | Credit card payment — success (4242 4242 4242 4242) | Payment processed, order created | | | |
| 25 | Credit card payment — declined (4000 0000 0000 0002) | Declined message shown, cart intact | | | |
| 26 | Order confirmation email received | Email arrives within 2 minutes | | | |

### COUPON ENGINE (CRITICAL RISK — rewrite this release)

| # | Test | Expected Result | Pass/Fail | Tester | Notes |
|---|------|----------------|-----------|--------|-------|
| 27 | Percentage coupon (SAVE10 — 10%) | Correct % applied to subtotal | | | |
| 28 | Flat discount coupon (FLAT20 — $20 off) | $20 deducted from subtotal | | | |
| 29 | Coupon with minimum order ($50 minimum, cart is $30) | Error: "Minimum $50 required" | | | |
| 30 | Single-use coupon used twice by same account | Error on second use: "Already used" | | | |
| 31 | Coupon valid for specific product categories only | Discount applies only to eligible items | | | |
| 32 | Coupon with expiry date of today | Coupon accepted (not yet expired) | | | |
| 33 | Coupon with expiry date of yesterday | Error: "This coupon has expired" | | | |
| 34 | Multiple coupons attempted (two codes) | Error: "Only one coupon per order" | | | |

### NEW DASHBOARD (HIGH RISK — new feature this release)

| # | Test | Expected Result | Pass/Fail | Tester | Notes |
|---|------|----------------|-----------|--------|-------|
| 35 | Dashboard loads with all 4 metrics | Revenue, Users, Churn, MRR all display | | | |
| 36 | Metrics load in <3s (small account) | All metrics loaded before 3-second mark | | | |
| 37 | Date picker — Last 7 Days | Metrics update to correct period | | | |
| 38 | Date picker — Custom range | Metrics update to custom period | | | |
| 39 | Owner sees company-wide metrics | Company totals displayed | | | |
| 40 | Manager sees team metrics only | Only team data visible, no other teams | | | |

---

**Checklist Summary Table:**

| Section | Total Tests | Critical | High Risk | Time Estimate |
|---------|-------------|---------|-----------|---------------|
| Critical Path | 5 | 5 | — | 15 min |
| Authentication | 7 | 2 | 5 | 25 min |
| Checkout | 12 | 4 | 8 | 45 min |
| Coupon Engine | 8 | 8 | — | 30 min |
| New Dashboard | 6 | 2 | 4 | 20 min |
| **Total** | **38** | **21** | **17** | **~2.5 hours** |

---

## Tracking Checklist Execution in Zenflow

### Setting Up a Release Execution Task

1. In Zenflow, create a new task: `Release v2.5.0 — Regression Testing`
2. In the description, paste your checklist (or link to it)
3. As you complete each section, add a comment with the results
4. Use subtasks for each major section:
   - Subtask: Critical Path Tests
   - Subtask: Authentication Tests
   - Subtask: Checkout Tests
   - Subtask: Coupon Engine Tests
   - Subtask: Dashboard Tests

### Real-Time Status Updates

While testing, update the task status and add comments:

```
[10:30 AM] Critical Path Tests — COMPLETE
✅ All 5 tests passed. Deployment healthy.

[11:15 AM] Authentication Tests — COMPLETE
✅ 6 of 7 passed. 
⚠️ ISSUE: Social login (Google) failing in staging. 
   Filed as BUG-2891. Known env issue — OAuth callback URL not updated for new staging domain. 
   Checking with DevOps. Will retest after fix.
```

### What to Do After Execution

After completing a checklist, use Zen to generate a release sign-off summary:

```
I just completed regression testing for Release v2.5.0. Here are my results:

Total tests: 38
Passed: 35
Failed: 2
Blocked: 1

Failed tests:
1. TC-COUPON-031: Coupon for specific categories — incorrect behavior. 
   Bug filed: BUG-2892 (P2, fix in next release)
2. TC-DASH-040: Manager role shows other team data. 
   Bug filed: BUG-2893 (P1, MUST fix before release)

Blocked tests:
1. TC-AUTH-011: Social login (Google) — environment issue with OAuth callback.
   Not a code bug. DevOps to fix staging environment. Will retest.

Please generate:
1. A release test summary report (suitable for sharing with the PM and team lead)
2. A go/no-go recommendation based on these results
3. The list of conditions that must be met before this release can proceed
4. A message I can post in our #qa-results Slack channel
```

---

## Checklist Anti-Patterns to Avoid

| Anti-Pattern | What It Looks Like | Why It's a Problem |
|-------------|-------------------|-------------------|
| **The Everything List** | 500 items, never fully completed | No prioritization. Low-value tests crowd out high-value ones. |
| **Vague steps** | "Test checkout" | Can't be executed consistently by different testers |
| **Missing expected results** | "Click submit" (no expected outcome) | Pass/fail judgment is subjective |
| **No owner** | Checklist exists, nobody committed to run it | Items get skipped or forgotten |
| **Set and forget** | Same checklist used for 18 months unchanged | New features get no coverage; old tests become stale |
| **No time estimates** | Unlimited scope creep on testing time | Release gets delayed or tests get cut without visibility |

---

## What's Next

Your testing work is now structured, documented, and evidence-backed. The final step in the beginner section is making all of this work visible — to your team, your manager, and to Zen — by tracking it properly in Zenflow.

**→ Continue to [05 — Track All Your QA Work in Zenflow](./05-track-in-zenflow.md)**
