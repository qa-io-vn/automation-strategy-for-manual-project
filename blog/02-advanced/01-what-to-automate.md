# AI-Assisted QA: From Manual Tester to Automation Strategist
## 02-Advanced · 01: What to Automate — The ROI Scoring Model

> **Series Navigation**
> [← Advanced Overview](./README.md) | **01 · What to Automate** | [02 · Architecture →](./02-architecture-explained.md)

---

## Purpose

Before writing a single line of Playwright code, you need to answer one question: **Is this test worth automating?**

The wrong answer costs weeks. Automating the wrong tests creates a suite that's expensive to maintain, slow to run, and brittle under change — which leads teams to abandon automation entirely.

This guide gives you a **data-driven scoring model** called **BPS × ACS** to make that decision objectively, every time.

---

## The Core Problem: Automation Instinct Is Wrong

Most new automation engineers automate what's *easy to automate*, not what's *worth automating*. This leads to:

- Suites full of happy-path tests that never catch real bugs
- No coverage on the flows that actually break
- High maintenance cost for low-value tests
- Stakeholders losing faith in automation ROI

The fix is a scoring system that forces you to evaluate every candidate test against objective criteria.

---

## The BPS × ACS Model

**BPS** = **Business Priority Score** — How important is this test to the business?
**ACS** = **Automation Compatibility Score** — How easy/stable is this to automate?

**ROI Score = BPS × ACS**

Higher is better. Tests that score above 60 are strong automation candidates. Tests below 30 are usually better left as manual tests.

---

## BPS: Business Priority Score (1–10)

The Business Priority Score answers: **If this broke in production and wasn't caught, how bad would it be?**

Score each test across five dimensions, then average them:

| Dimension | What to Ask | Score 1 | Score 5 | Score 10 |
|-----------|------------|---------|---------|----------|
| **Execution Frequency** | How often does QA run this manually? | Rarely (once a quarter) | Monthly | Every sprint or daily |
| **Business Impact** | What breaks if this fails in prod? | Minor UX issue | Features degrade | Revenue / data loss |
| **User Traffic** | How many users touch this flow? | Power users only | 10–30% of users | >50% of users |
| **Regression Risk** | Has this broken before due to changes? | Never | 1–2 times | Repeatedly |
| **Compliance / Legal** | Is this required for audit or regulation? | No | Nice to have | Mandatory |

**BPS Formula:**
```
BPS = (Frequency + Impact + Traffic + RegressionRisk + Compliance) / 5
```

---

## ACS: Automation Compatibility Score (1–10)

The Automation Compatibility Score answers: **Will this test be reliable and maintainable once automated?**

| Dimension | What to Ask | Score 1 | Score 5 | Score 10 |
|-----------|------------|---------|---------|----------|
| **UI Stability** | How often does this UI change? | Weekly redesigns | Quarterly updates | Stable for 6+ months |
| **Determinism** | Are inputs and outputs predictable? | Depends on live data | Partially controlled | Fully controlled |
| **Test Data Control** | Can you create/reset test data reliably? | No — uses production | Shared staging env | Isolated + seedable |
| **Element Selectability** | Do elements have stable IDs or test-ids? | No — text-based only | Mixed | All have `data-testid` |
| **Wait Complexity** | How many async operations are involved? | Many complex waits | Some waits | No significant waits |

**ACS Formula:**
```
ACS = (UIStability + Determinism + DataControl + Selectability + WaitComplexity) / 5
```

---

## ROI Score Interpretation

| ROI Score | Recommendation | Action |
|-----------|---------------|--------|
| **81–100** | **Automate immediately** | High-value, high-stability. Top priority. |
| **61–80** | **Automate this sprint** | Strong candidate. Schedule it. |
| **41–60** | **Automate with investment** | Worth it, but fix stability issues first (add test-ids, seed data) |
| **21–40** | **Manual for now** | Cost outweighs benefit today. Reassess in 3 months. |
| **1–20** | **Never automate** | Leave this to exploratory/manual testing |

---

## Example Scoring Table: 20 Real Test Cases

The following table shows how a mid-size e-commerce QA team scored their backlog using BPS × ACS.

| # | Test Case | BPS | ACS | ROI | Decision |
|---|-----------|-----|-----|-----|----------|
| 1 | Login with valid credentials | 9.2 | 9.0 | **82.8** | Automate immediately |
| 2 | Login with invalid password | 8.4 | 8.8 | **73.9** | Automate this sprint |
| 3 | Complete checkout (card payment) | 9.8 | 7.2 | **70.6** | Automate this sprint |
| 4 | Add item to cart | 8.6 | 9.2 | **79.1** | Automate immediately |
| 5 | Apply promo code (valid) | 7.4 | 7.8 | **57.7** | Automate with investment |
| 6 | Apply promo code (expired) | 6.8 | 7.6 | **51.7** | Automate with investment |
| 7 | User registration (new email) | 8.0 | 7.4 | **59.2** | Automate with investment |
| 8 | Forgot password flow | 7.6 | 5.8 | **44.1** | Automate with investment |
| 9 | Product search — exact match | 7.2 | 8.4 | **60.5** | Automate with investment |
| 10 | Product search — no results | 5.4 | 8.0 | **43.2** | Manual for now |
| 11 | Filter products by category | 6.6 | 7.0 | **46.2** | Automate with investment |
| 12 | Sort products by price | 5.8 | 7.4 | **42.9** | Manual for now |
| 13 | View order history | 7.0 | 8.6 | **60.2** | Automate with investment |
| 14 | Cancel order (within window) | 8.2 | 6.2 | **50.8** | Automate with investment |
| 15 | Review + rating submission | 4.6 | 5.4 | **24.8** | Manual for now |
| 16 | Live chat widget open | 3.2 | 3.0 | **9.6** | Never automate |
| 17 | Social share buttons | 2.4 | 2.8 | **6.7** | Never automate |
| 18 | Print invoice (PDF) | 4.0 | 2.6 | **10.4** | Never automate |
| 19 | Admin: bulk discount rule | 7.8 | 5.0 | **39.0** | Manual for now |
| 20 | Checkout with PayPal | 9.0 | 2.8 | **25.2** | Manual for now |

### Key Takeaways from the Table

- **Row 20 (PayPal checkout)** scores 9.0 on business priority but only 2.8 on ACS — PayPal's iframe-heavy UI is notoriously hard to automate reliably. The ROI score correctly flags it as "Manual for now."
- **Row 16 (Live chat)** is low priority AND hard to automate. Never touch this.
- **Rows 5–9** are worth automating, but only after investing in test data seeding and adding `data-testid` attributes.

---

## How to Use Zen to Score Test Cases

Zen can dramatically speed up your scoring process. Here's the workflow:

### Step 1: Export Your Test Cases as Text

Copy your test cases from TestRail, Jira, or a spreadsheet into a plain text list.

### Step 2: Prompt Zen

```
I'm a QA engineer using the BPS × ACS ROI model to decide which test cases to automate.

Here's how the model works:
- BPS (Business Priority Score, 1-10) = avg of: Execution Frequency, Business Impact, User Traffic, Regression Risk, Compliance
- ACS (Automation Compatibility Score, 1-10) = avg of: UI Stability, Determinism, Test Data Control, Element Selectability, Wait Complexity
- ROI = BPS × ACS

Here are my test cases:
1. User can log in with valid credentials
2. User sees error on invalid password
3. User can complete checkout with credit card
4. User can add item to cart
5. User can apply a promo code

Please score each one with estimated BPS, ACS, and ROI scores. For any test scoring below 40, explain why it's a poor automation candidate. This is for an e-commerce web application.
```

### Step 3: Refine with Context

After Zen's initial scores, give it more context about your specific app:

```
For test case #8 (Forgot password), the flow sends a real email. We don't have
a test email inbox set up yet. How does this affect the ACS score and what would
we need to do to make it automatable?
```

### Step 4: Build Your Prioritized Backlog

Ask Zen to output the results as a table you can paste into Confluence or Notion:

```
Can you format the scored test cases as a markdown table with columns:
Test Case | BPS | ACS | ROI | Sprint Recommendation | Blocker (if any)
```

---

## When to NEVER Automate

Some tests should **always** remain manual. No ROI score will change this.

### 1. Exploratory Testing
Exploratory testing is *defined* by its unscripted nature. An automated script cannot explore — it can only verify what was already anticipated. Automating exploratory testing defeats its purpose.

### 2. Usability and UX Judgment
Questions like "Is this button confusing?" or "Does the checkout flow feel natural?" require human judgment. No assertion can answer them.

### 3. Visually Complex Layouts
While visual regression tools exist, they're high-maintenance. If your team is just getting started, skip pixel-level visual testing. Spot-check manually.

### 4. Third-Party Auth Flows (OAuth, SSO)
Google Sign-In, GitHub OAuth, SAML SSO — these deliberately block automation. They use CAPTCHAs, bot detection, and iframe sandboxing. Automate the *result* (user lands on dashboard after login), not the third-party screens themselves.

### 5. One-Time Migrations
If a test case will only ever run once (a data migration, a one-time import), don't automate it. The automation cost exceeds the test execution cost before you even finish writing it.

### 6. Frequently Changing UI
If the UI team ships redesigns every two weeks, automated tests become a maintenance burden rather than a safety net. Wait until the design stabilizes.

### 7. Tests That Require Human Senses
- "Does the audio play correctly?"
- "Does the color match the brand guide?"
- "Does the animation feel smooth?"

These require eyes, ears, and aesthetic judgment. Automate the *functional* assertion (audio element exists, color value is a hex), not the sensory judgment.

---

## The 5 Most Common Beginner Mistakes

### Mistake #1: Automating What's Easy, Not What Matters

**What happens:** New automation engineers start with tests that are easy to write (simple forms, static pages) and never get to the tests that matter (checkout, payment, core user journeys).

**The fix:** Score your test cases first. Always start automation from the top of the ROI-sorted list, not the bottom.

### Mistake #2: One Giant Test Per Feature

**What happens:**
```javascript
test('Everything about checkout', async ({ page }) => {
  // 200 lines
  // Tests add to cart, apply coupon, enter address,
  // enter payment, submit, verify email, check order history...
});
```

**The fix:** One test = one behavior. If your test description contains "and," split it.

### Mistake #3: Automating Against Unstable Selectors

**What happens:**
```javascript
// This breaks when the developer reorders list items
await page.click('ul > li:nth-child(3) > a > span');
```

**The fix:** Work with developers to add `data-testid` attributes:
```html
<button data-testid="checkout-submit-btn">Place Order</button>
```
```javascript
await page.click('[data-testid="checkout-submit-btn"]');
```

### Mistake #4: Using Production Data

**What happens:** Tests run against production or a shared staging environment. Tests pass sometimes (when data is right) and fail sometimes (when another tester changed something). This is the #1 source of flakiness.

**The fix:** Invest in test data management before writing tests. Every test should create its own data and clean up after itself, or use a dedicated, seedable test environment.

### Mistake #5: No Reporting Strategy

**What happens:** 40 tests run in CI. Some fail. Nobody knows why. Reports are buried in terminal output. Stakeholders get `exit code: 1` emails and stop trusting automation.

**The fix:** Set up the HTML reporter from day one:
```javascript
// playwright.config.js
reporter: [['html', { open: 'never' }], ['list']],
```

Publish the report as a CI artifact. Link to it in your team Slack channel.

---

## Decision Matrix: Automate vs. Manual

Use this matrix when you're unsure:

```
                    BUSINESS IMPACT
                    Low          High
                 ┌────────────┬────────────┐
    High         │ Low         │ HIGH       │
    STABILITY    │ Priority    │ PRIORITY   │
                 │ Automate    │ AUTOMATE   │
                 │ (backlog)   │ NOW        │
                 ├────────────┼────────────┤
    Low          │ SKIP        │ Manual     │
    STABILITY    │ (never)     │ + Invest   │
                 │             │ in stability│
                 └────────────┴────────────┘
```

**Quadrant Guidance:**

- **High Stability + High Impact** → Drop everything and automate these first
- **High Stability + Low Impact** → Queue them — they're cheap to maintain, but don't rush
- **Low Stability + High Impact** → Manual test while you invest in stability improvements
- **Low Stability + Low Impact** → Walk away. These tests will cost more than they're worth.

---

## Building Your Automation Backlog

Follow this process at the start of each quarter:

1. **Export all manual test cases** (from TestRail, Jira, or wherever you store them)
2. **Score each test case** using BPS × ACS (use Zen to speed this up)
3. **Sort by ROI score** (descending)
4. **Flag blockers** for tests scoring 41–60 (what needs to be done to make them automatable?)
5. **Create tasks** for blockers (add data-testid, set up test data seeding)
6. **Assign top tests** to your sprint automation backlog
7. **Reassess monthly** — business priorities and UI stability change

### Sample Automation Backlog (After Scoring)

| Sprint | Test Case | ROI | Blocker | Owner |
|--------|-----------|-----|---------|-------|
| Sprint 1 | Login — valid credentials | 82.8 | None | @alice |
| Sprint 1 | Add item to cart | 79.1 | None | @alice |
| Sprint 1 | Login — invalid password | 73.9 | None | @bob |
| Sprint 2 | Complete checkout | 70.6 | Need card test tokens | @alice |
| Sprint 2 | User registration | 59.2 | Need email inbox | @bob |
| Sprint 3 | Apply promo code | 57.7 | Need promo seed data | @charlie |
| Backlog | Forgot password | 44.1 | Need email inbox | — |
| Backlog | Product search | 60.5 | Need search seed data | — |

---

## Summary

| Key Point | Action |
|-----------|--------|
| Not all tests should be automated | Score every candidate using BPS × ACS |
| Business value matters more than ease | Start from the top of the ROI list |
| Some tests should never be automated | Exploratory, UX, third-party auth, sensory |
| Flakiness often comes from bad choices | Fix data and selectors before writing tests |
| Zen can accelerate scoring | Use the prompt templates in this guide |

---

**Next:** [02 · Architecture Explained →](./02-architecture-explained.md) — Learn the Spec → Flow → Page Object pattern that makes your tests maintainable as the suite grows.
