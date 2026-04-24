# Write Professional Bug Reports

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Coverage Map](./02-coverage-map.md) | [Next: Test Execution Checklist →](./04-test-execution-checklist.md)

---

## Introduction

A bug report is a communication document. Its job is to transfer information from your brain — where you just witnessed a failure — to a developer's brain, clearly enough that they can reproduce it, diagnose it, and fix it without talking to you.

Most bug reports fail at this job. Not because QA engineers are bad at finding bugs, but because writing clearly under time pressure is genuinely hard. You find a bug at 4:45 PM before a release, you're stressed, you quickly type "checkout doesn't work" with a screenshot and move on. The developer sees it the next morning, can't reproduce it, adds a comment, and the ticket dies in a swamp of "need more info."

AI assistance doesn't just make bug reports faster to write — it makes them more complete and consistent, even when you're rushed. Zen can take your raw observations and structure them into a professional report in under a minute.

---

## Anatomy of a Great Bug Report

Every great bug report contains these elements. No exceptions.

### The 10 Required Fields

| Field | Purpose | Bad Example | Good Example |
|-------|---------|-------------|--------------|
| **Title** | 1-line summary of what fails and where | "Checkout broken" | "Stripe payment fails with 422 when cart contains digital + physical items" |
| **Environment** | Where it was observed | "Staging" | "Staging (staging.myapp.com) — Chrome 121, macOS 14.2, logged in as customer@test.com" |
| **Severity** | Technical impact level | [missing] | **High** — prevents order completion |
| **Priority** | Business urgency | [missing] | **P1** — blocks release |
| **Steps to Reproduce** | Exact sequence to trigger the bug | "Go to checkout and try to pay" | Numbered steps with specific data (see below) |
| **Expected Result** | What should happen | "It should work" | "Payment processes successfully, user sees order confirmation, order created in DB" |
| **Actual Result** | What actually happened | "Error" | "Error toast: 'Payment failed. Please try again.' API returns 422 with body: {error: 'cart_type_mismatch'}" |
| **Attachments** | Evidence | [none] | Screenshot of error, HAR file with network request, browser console output |
| **Reproducibility** | How often it occurs | [missing] | "100% reproducible — confirmed 5 times across 2 accounts" |
| **Additional Context** | Other relevant info | [missing] | "Pure digital-only carts and pure physical carts work fine. Only mixed carts fail." |

---

## Severity vs. Priority: The Framework

This is one of the most commonly confused distinctions in QA. Severity and priority are different things and both matter.

### Severity — Technical Impact

| Level | Definition | Example |
|-------|-----------|---------|
| **Critical** | Application crash, data loss, security breach, complete workflow blockage | Payment flow crashes browser; user data exposed |
| **High** | Major feature broken, no workaround exists | Cannot complete checkout at all |
| **Medium** | Feature partially broken, workaround exists | Payment works via PayPal but fails via credit card |
| **Low** | Minor issue, cosmetic, workaround is simple | Error message is grammatically incorrect |

### Priority — Business Urgency

| Level | Definition | Example |
|-------|-----------|---------|
| **P1** | Must fix before release, blocking | Payment failure in production |
| **P2** | Must fix in this release | Major UX issue affecting most users |
| **P3** | Fix in next sprint | Non-critical feature malfunction |
| **P4** | Fix when time permits | Minor cosmetic issue |

### The Matrix

| | Critical Severity | High Severity | Medium Severity | Low Severity |
|--|------------------|--------------|----------------|--------------|
| **Core user journey** | P1 | P1 | P2 | P3 |
| **Important feature** | P1 | P2 | P2 | P3 |
| **Secondary feature** | P1 | P2 | P3 | P4 |
| **Nice-to-have** | P2 | P3 | P4 | P4 |

---

## Prompt Template: Generate a Bug Report from Raw Notes

Use this when you've found a bug and want to write a professional report without spending 20 minutes on it.

```
I found a bug and need to write a professional bug report. 
Here are my raw notes:

Raw observation: [describe what you saw in plain language]
Where I was testing: [feature, page, workflow]
What I was doing: [general steps]
What I expected: [expected behavior]
What happened instead: [actual behavior]
Error message (if any): [exact error text]
Evidence I have: [screenshot, console log, network request, etc.]
Environment: [browser, OS, test account, app version]
How often: [every time / sometimes / once]

Please write a complete, professional bug report with:
1. A clear, specific title (format: "[Feature] - [What fails] when [condition]")
2. Full environment details
3. Severity and Priority with justification
4. Numbered, specific steps to reproduce
5. Expected vs. actual result
6. Reproducibility statement
7. Notes on business impact
8. Suggested labels/tags

Application context:
[paste your project context or describe the app briefly]
```

---

## Three Complete Example Bug Reports

### Example 1: UI Bug

---

**Title:** `[Checkout] - Apply button missing from coupon field on iOS Safari 17`

**Environment:**
- URL: staging.shopcore.com
- Browser: Safari 17.2 on iOS 17.2.1 (iPhone 15 Pro)
- Test Account: testcustomer@example.com
- App Version: v2.4.1 (build 2401)
- Date Found: 2024-01-15

**Severity:** High
**Priority:** P2

**Severity Justification:** Core checkout flow feature is non-functional on iOS Safari, which represents approximately 28% of our mobile traffic (per analytics).

**Steps to Reproduce:**
1. Open staging.shopcore.com in Safari on an iOS device (tested iPhone 15 Pro, iOS 17.2.1)
2. Log in as `testcustomer@example.com` (password: `TestPass123!`)
3. Add any product to cart (tested with Product ID 1042)
4. Navigate to cart page
5. Click "Proceed to Checkout"
6. On the Cart Review step, locate the "Have a coupon code?" section
7. Click "Apply Coupon" link to expand the coupon input field
8. Observe the coupon input field

**Expected Result:**
Coupon input field expands showing a text input and an "Apply" button, allowing the user to enter and submit a coupon code.

**Actual Result:**
Coupon input field expands showing only the text input. The "Apply" button is completely absent. Pressing Enter/Return on the keyboard also does not submit the coupon. User has no way to apply a coupon code.

**Reproducibility:** 100% — Reproduced 6 times across 2 test accounts (testcustomer@example.com and testcustomer2@example.com). Not reproducible on Chrome iOS 121 or any desktop browser.

**Attachments:**
- `screenshot-ios-coupon-missing-button.png` — shows the coupon field with missing Apply button
- `screenshot-chrome-coupon-working.png` — shows expected state in Chrome for comparison

**Additional Context:**
- Issue appears to be Safari-specific. All other browsers (Chrome iOS, Chrome desktop, Firefox, Safari desktop) show the Apply button correctly.
- This bug does NOT prevent checkout — users can still pay without applying a coupon. However, affected users cannot receive promotional discounts.
- Related commit: `feat/coupon-ui-refresh` merged 2024-01-10 — may have introduced this regression.

**Suggested Labels:** `bug`, `ios`, `safari`, `checkout`, `regression`, `P2`

---

### Example 2: API Bug

---

**Title:** `[Orders API] - POST /api/v1/orders returns 500 instead of 422 when cart is empty`

**Environment:**
- API: staging-api.shopcore.com
- Tool: Postman v10.22
- Auth: Bearer token for testcustomer@example.com
- App Version: API v2.4.1

**Severity:** Medium
**Priority:** P2

**Severity Justification:** Incorrect HTTP status code causes client applications to mishandle the error. Returns 500 (server error) instead of 422 (validation error), which may trigger incorrect error handling in the mobile app (shows "Server Error" instead of "Your cart is empty").

**Request:**
```http
POST /api/v1/orders
Authorization: Bearer eyJhbGc...
Content-Type: application/json

{
  "cart_id": "empty-cart-abc123",
  "payment_token": "tok_visa",
  "shipping_address_id": "addr_55"
}
```

**Expected Response:**
```http
HTTP 422 Unprocessable Entity
{
  "error": "cart_empty",
  "message": "Cannot create an order with an empty cart."
}
```

**Actual Response:**
```http
HTTP 500 Internal Server Error
{
  "error": "internal_server_error",
  "message": "An unexpected error occurred."
}
```

**Steps to Reproduce:**
1. Create a test account or use testcustomer@example.com
2. Obtain auth token via `POST /api/v1/auth/login`
3. Create a new cart: `POST /api/v1/carts` → note the cart_id
4. Do NOT add any items to the cart
5. Attempt to create an order: `POST /api/v1/orders` with the empty cart_id
6. Observe the HTTP status code and response body

**Reproducibility:** 100% — Reproducible with any empty cart. Confirmed with 3 different cart IDs.

**Attachments:**
- `postman-collection-bug-repro.json` — Postman collection with all steps pre-configured

**Additional Context:**
- Non-empty carts work correctly
- If the cart does not exist at all (invalid cart_id), the API correctly returns 404
- Potential server-side null reference exception when iterating over empty cart items
- Impact on mobile app: the iOS app's error handler shows "Server Error" for 5xx responses and "Invalid request" for 4xx. Empty cart currently shows "Server Error" when it should show a user-friendly message.

**Suggested Labels:** `bug`, `api`, `orders`, `validation`, `http-status`, `P2`

---

### Example 3: Performance Bug

---

**Title:** `[Dashboard] - Metrics page takes 14+ seconds to load for accounts with 10,000+ orders`

**Environment:**
- URL: staging.shopcore.com/dashboard/metrics
- Browser: Chrome 121, macOS 14.2
- Test Account: bigaccount@loadtest.com (account with 10,247 orders seeded)
- Network: Office WiFi (100Mbps down measured via speedtest)
- App Version: v2.4.1

**Severity:** High
**Priority:** P2

**Severity Justification:** 14-second page load exceeds our defined performance SLO of <3 seconds for dashboard pages. Affected users are our high-value enterprise customers (large order history). This is not a blocking failure but significantly degrades the experience for our most important user segment.

**Steps to Reproduce:**
1. Log in as `bigaccount@loadtest.com` (password: `LoadTest123!`)
2. Navigate to `/dashboard/metrics`
3. Leave date range at default (Last 30 Days)
4. Start a timer when the page begins loading
5. Stop the timer when all 4 metrics (Revenue, Active Users, Churn, MRR) are fully displayed

**Expected Result:**
All metrics load within 3 seconds (as per our defined performance SLO).

**Actual Result:**
- Revenue metric loads in ~2 seconds
- Active Users loads in ~3 seconds
- Churn Rate loads in ~11 seconds
- MRR loads in ~14 seconds
- The page displays a loading spinner for 14 seconds before becoming fully usable

**Performance Data:**

| Metric | Load Time | API Endpoint | Response Time |
|--------|----------|--------------|---------------|
| Revenue | 2.1s | GET /api/metrics/revenue | 1.8s |
| Active Users | 3.0s | GET /api/metrics/users | 2.6s |
| Churn Rate | 11.2s | GET /api/metrics/churn | 10.9s |
| MRR | 14.1s | GET /api/metrics/mrr | 13.7s |

**Reproducibility:**
- 100% reproducible with bigaccount@loadtest.com (10,247 orders)
- NOT reproducible with smallaccount@loadtest.com (127 orders — loads in 1.2s)
- Performance degrades proportionally with order count. Threshold appears to be around 5,000 orders.

**Attachments:**
- `har-dashboard-load-bigaccount.har` — Full network trace showing all API calls and timing
- `screenshot-14sec-spinner.png` — Screenshot of loading state at 13 seconds
- `loom-performance-recording.mp4` — Screen recording of the full load experience

**Additional Context:**
- The churn and MRR calculations appear to be running full-table scans without date-range filtering at the query level. The API accepts a date_range parameter, but it may not be applied before aggregation.
- smallaccount@loadtest.com loads in 1.2s with 127 orders. bigaccount@loadtest.com takes 14s with 10,247 orders. Load time appears to be O(n) in order count — linear scaling indicates no database indexing or query optimization.
- Similar performance degradation was reported in a production support ticket (#SUP-2891) by enterprise customer AcmeCorp (est. 40,000+ orders).

**Suggested Labels:** `bug`, `performance`, `dashboard`, `database`, `enterprise`, `P2`

---

## Asking Zen to Estimate Business Impact

When you need to escalate a bug or justify its priority, use this prompt:

```
I found this bug and need to estimate its business impact for a P1 escalation:

Bug: [describe the bug]
Affected feature: [name]
Affected user percentage: [if known from analytics]
Revenue affected: [if known — e.g., "this is in the checkout flow which processes $X/day"]
Frequency: [how often does this occur?]
Workaround: [does one exist?]
Time to fix: [developer estimate if known]

Please help me:
1. Write a 2-paragraph business impact statement (non-technical, for a PM or executive)
2. Estimate the risk of NOT fixing this before release
3. Suggest how to phrase the escalation in our team's Slack channel
```

---

## Attaching and Describing Evidence

Evidence transforms a bug from "alleged" to "proven." Here's what to collect:

### Screenshot Best Practices
- Capture the full browser window (not just the element) so reviewers can see the URL and context
- Add annotations (arrows, circles, text) to highlight the specific issue
- Include before/after screenshots when possible
- Name screenshots descriptively: `checkout-ios-coupon-missing-button.png`, not `screenshot1.png`

### Video Recordings
- Keep recordings under 2 minutes — developers won't watch a 10-minute video
- Use Loom for screen + audio recording with easy sharing links
- Start from the beginning of the reproduction steps, not mid-flow

### Network/API Evidence
- Use Chrome DevTools → Network tab → Export as HAR
- In Postman: include the full request/response in your report
- Always include status codes, headers, and response body

### Log Files
- JavaScript console: Right-click → Save As, or copy the relevant lines
- If you have server log access: grep for the timestamp and request ID
- Include only the relevant section — not thousands of lines

### How to Ask Zen to Describe Your Evidence

```
I have a screenshot that shows this bug. Here's what the screenshot contains:

[Describe the screenshot in detail — what's visible, what's highlighted, 
what's missing that should be there]

Please write a 2-3 sentence description of this screenshot appropriate 
for inclusion in a bug report's "Attachments" section.
```

---

## The 2-Minute Sanity Check

Before submitting any bug report, run through this quick checklist:

- [ ] Could a developer reproduce this from my steps without asking me a single question?
- [ ] Have I included the exact test data (account, product, coupon code) I used?
- [ ] Have I stated what I expected AND what I got?
- [ ] Have I included at least one piece of visual evidence?
- [ ] Have I included the URL/endpoint/environment?
- [ ] Have I assigned severity AND priority?
- [ ] Have I checked if this bug already exists in the backlog?
- [ ] Have I tested whether the bug exists in production (not just staging)?

---

## What's Next

You can now find bugs and communicate them clearly. Next: make sure you're testing the right things in the right order before every release with AI-generated execution checklists.

**→ Continue to [04 — Build Test Execution Checklists](./04-test-execution-checklist.md)**
