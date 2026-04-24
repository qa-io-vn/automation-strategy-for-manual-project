# Generate Test Cases with AI

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Beginner Overview](./README.md) | [Next: Coverage Map →](./02-coverage-map.md)

---

## Introduction

Writing test cases is foundational QA work. It's also, if we're honest, sometimes brutally tedious. Given a well-specified feature, you know the structure of what you need to produce — you just need to fill it out across dozens of scenarios, including the ones nobody told you about but that users will definitely try.

This is where Zen becomes genuinely transformative. Given good input, Zen can produce a comprehensive first draft of your test cases in minutes. Not a perfect draft — a starting point you refine with your domain knowledge. But that starting point eliminates the blank-page problem and surfaces scenarios you might have missed.

This guide covers every requirement format you'll encounter, how to get the best output, and how to iterate until your test cases are actually useful.

---

## Understanding What Makes a Good Test Case

Before prompting Zen, let's align on what a complete test case looks like. AI output is only as good as what you're able to evaluate.

### The Anatomy of a Complete Test Case

| Field | Description | Example |
|-------|-------------|---------|
| **ID** | Unique identifier | `TC-CHECKOUT-047` |
| **Title** | Short, descriptive name | "Apply expired coupon code at checkout" |
| **Module / Feature** | Which part of the app | Checkout → Coupon validation |
| **Priority** | P1 (critical) to P4 (low) | P2 |
| **Type** | Positive / Negative / Edge / Boundary | Negative |
| **Preconditions** | What must be true before testing | User logged in, cart has items, coupon exists in DB as expired |
| **Test Steps** | Numbered, specific actions | 1. Navigate to checkout, 2. Enter coupon "SUMMER20" in coupon field, 3. Click "Apply" |
| **Expected Result** | What should happen | Error message displays: "This coupon has expired." Discount is NOT applied. Cart total unchanged. |
| **Test Data** | Specific values to use | Coupon code: SUMMER20, Status: expired, Cart: 2 items, total $89.99 |
| **Notes** | Edge cases, known issues | Verify behavior is same for guest and logged-in users |

---

## Prompt Templates by Requirement Format

### Format 1: User Stories with Acceptance Criteria (AC)

This is the ideal format. If your team writes proper Gherkin-style AC, Zen can convert each AC directly into test cases.

**Input:**
```
User Story: As a logged-in customer, I want to apply a coupon code at checkout,
so that I can receive a discount on my order.

Acceptance Criteria:
- GIVEN a valid coupon code, WHEN I apply it, THEN the discount is applied and the total is updated
- GIVEN an expired coupon code, WHEN I apply it, THEN I see an error "This coupon has expired"
- GIVEN a coupon that doesn't exist, WHEN I apply it, THEN I see an error "Invalid coupon code"
- GIVEN a valid coupon, WHEN I apply it to an ineligible cart (minimum not met), 
  THEN I see "Order minimum of $X required"
- GIVEN a single-use coupon already used by this account, WHEN I apply it, 
  THEN I see "This coupon has already been used"
```

**Prompt:**
```
I have a user story with acceptance criteria. Please generate a comprehensive test case 
suite from this. For each AC, generate at minimum:
- The explicit happy/unhappy path described
- 2-3 boundary values or edge cases related to that scenario
- Any obvious test cases the AC implies but doesn't state

Format each test case with: ID, Title, Type (Positive/Negative/Edge), 
Preconditions, Test Steps (numbered), Expected Result, Test Data.

User Story and AC:
[paste story and AC]

Also, after generating the cases, list any ambiguities in the AC that I should 
clarify with the product team before testing.
```

**What to look for in the output:**
- Does it cover all AC? ✓
- Does it include negative cases for each positive AC? ✓
- Does it test boundary values (e.g., coupon with 0 uses remaining, coupon expiring today)? ✓
- Are preconditions specific enough to reproduce? ✓

---

### Format 2: Acceptance Criteria Only (No User Story)

Sometimes you get AC without the user story context. Add context yourself.

**Prompt:**
```
I have a set of acceptance criteria for a feature. I'll provide context about 
the application before you generate test cases.

Application context:
- E-commerce platform, React frontend, REST API backend
- Users can be: guests, registered customers, or admin users
- Orders can contain physical products and digital downloads
- Payment via Stripe (credit cards) and PayPal

Feature: Order history with filtering and export

Acceptance Criteria:
1. Customers can view a list of their past orders, sorted newest first
2. Customers can filter orders by: date range, status (pending/complete/refunded), and amount
3. Customers can export their order history as CSV
4. The CSV export includes: order ID, date, items, total, status
5. Admins can view order history for any customer
6. Guest users cannot access order history

Please generate:
1. Test cases covering all 6 AC (happy path + key negative cases)
2. Edge case test cases (empty states, large datasets, special characters in data)
3. Permission test cases (what each user role can/cannot do)
4. Any data integrity test cases (does the exported CSV match what's shown in UI?)
```

---

### Format 3: Prose Requirements (Informal Description)

A lot of real requirements come as informally written paragraphs. Handle them like this:

**Input example:**
```
"The new dashboard should show users their key metrics at a glance. 
We want to show total revenue, number of active users, churn rate this month, 
and MRR. There should be a date picker so they can see different time periods. 
The dashboard should update automatically. We're using Mixpanel data for this. 
Admin users should see company-wide metrics; regular users should only see 
their team's data."
```

**Prompt:**
```
I have informal prose requirements for a feature. Before generating test cases, 
I'd like you to:

Step 1: Extract all implied requirements as formal statements.
Step 2: Identify any ambiguities or missing information.
Step 3: Generate test cases based on what's clear, and mark cases that need 
        clarification before they can be executed.

Prose requirements:
[paste the text]

For the test cases, use this format:
- ID | Title | Type | Preconditions | Steps | Expected Result | Clarification Needed (yes/no)
```

---

### Format 4: API Specification (OpenAPI / Swagger)

When testing APIs, the spec IS the requirements. Here's how to generate API test cases:

**Prompt:**
```
Here is the OpenAPI specification for our POST /api/v1/orders endpoint.
Please generate a comprehensive test case suite for this endpoint covering:

1. Happy path — successful order creation
2. Required field validation — each required field missing individually
3. Field format validation — invalid formats (wrong types, too long, special chars)
4. Business rule validation — items out of stock, insufficient payment, invalid address
5. Authentication — missing token, expired token, token with wrong permissions
6. Idempotency — what happens if the same request is sent twice?
7. Boundary values — maximum items per order, minimum/maximum amounts

For each test case specify:
- HTTP Method and endpoint
- Request headers (especially Authorization)
- Request body (with specific test values)
- Expected HTTP status code
- Expected response body (key fields to verify)

[paste OpenAPI spec or relevant sections]
```

---

## Iterating and Refining AI Output

The first response from Zen is rarely the final answer. Here's how to iterate effectively:

### Iteration 1: Spot-Check for Coverage

After receiving the initial batch of test cases, run this follow-up:

```
Thank you. Now please review the test cases you just generated and:
1. What important scenarios are NOT covered that I should add?
2. Are there any test cases that are redundant or could be merged?
3. Which 5 test cases are highest priority (would catch the most critical bugs)?
4. Are there any cases where the expected result is ambiguous?
```

### Iteration 2: Add Your Domain Knowledge

You know things Zen doesn't. Feed that knowledge back in:

```
Based on our historical bugs and domain knowledge, I know:
1. Users frequently try to apply multiple coupon codes — our policy is one per order
2. We've had past bugs where discounts were applied twice in certain payment flows
3. Users with Japanese characters in their names have caused export failures before

Please add test cases specifically targeting these known risk areas.
```

### Iteration 3: Adjust for Your Environment

```
Our test environment has the following constraints:
- We only have 3 test coupon codes: SAVE10 (10% off), FLAT20 ($20 off), EXPIRED99 (expired)
- We have 2 test user accounts: testuser@example.com (regular) and admin@example.com (admin)
- We cannot test real payment processing — we use Stripe test cards only

Please update the test cases to use these specific test values and flag any 
cases that cannot be executed in our environment.
```

---

## Full Example: E-Commerce Checkout Flow

Here's a real-world example of what a well-structured Zen session produces.

**Initial Prompt:**
```
Generate test cases for the checkout flow of our e-commerce platform.

Context:
- React SPA frontend, Node.js REST API backend
- Stripe for credit card payments, PayPal as second option
- Checkout steps: Cart Review → Shipping → Payment → Confirmation
- Users can checkout as guest or logged-in
- Coupon codes supported (single-use and multi-use)
- Supports US shipping addresses only (currently)
- Cart can have 1–50 items; max cart value $10,000
- Digital products (no shipping required) and physical products can be mixed

Please generate test cases covering all stages of checkout. 
Organize them by section (Cart Review, Shipping, Payment, Confirmation).
Include: positive cases, negative cases, edge cases, and cross-browser concerns.
```

**Sample Output Structure (abbreviated):**

```
CART REVIEW
TC-CART-001 | Display cart items correctly | Positive
TC-CART-002 | Update item quantity in cart | Positive  
TC-CART-003 | Remove item from cart | Positive
TC-CART-004 | Apply valid coupon code | Positive
TC-CART-005 | Apply expired coupon code | Negative
TC-CART-006 | Apply coupon code with minimum order not met | Negative
TC-CART-007 | Cart with maximum 50 items | Edge
TC-CART-008 | Cart total at exactly $10,000 limit | Boundary
TC-CART-009 | Cart with mixed physical and digital items | Edge
TC-CART-010 | Empty cart — attempt to proceed | Edge

SHIPPING
TC-SHIP-001 | Enter valid US shipping address | Positive
TC-SHIP-002 | Select saved address (logged-in user) | Positive
TC-SHIP-003 | Enter non-US address | Negative
TC-SHIP-004 | Enter PO Box address | Edge (check business rule)
TC-SHIP-005 | Shipping options display correctly for cart weight | Positive
TC-SHIP-006 | Digital-only cart skips shipping step | Edge
TC-SHIP-007 | Address with special characters (é, ñ, etc.) | Edge

PAYMENT
TC-PAY-001 | Successful credit card payment (Stripe test card) | Positive
TC-PAY-002 | Declined credit card | Negative
TC-PAY-003 | PayPal payment flow | Positive
TC-PAY-004 | Invalid card number format | Negative
TC-PAY-005 | Expired credit card | Negative
TC-PAY-006 | CVV mismatch | Negative
TC-PAY-007 | 3D Secure authentication flow | Edge
TC-PAY-008 | Network timeout during payment | Edge
TC-PAY-009 | Browser back button during payment processing | Edge
TC-PAY-010 | Double-click Submit button | Edge (idempotency)

CONFIRMATION
TC-CONF-001 | Order confirmation email received | Positive
TC-CONF-002 | Order appears in order history | Positive
TC-CONF-003 | Inventory decremented after order | Positive
TC-CONF-004 | Coupon marked as used after order | Positive
TC-CONF-005 | Confirmation page shows correct order details | Positive
```

This gives you ~40 test cases as a starting point. With iteration, you'll typically end up with 60–80 comprehensive cases for a checkout flow.

---

## Full Example: SaaS Dashboard with Role-Based Access

**Prompt:**
```
Generate test cases for our SaaS analytics dashboard.

Context:
- Dashboard shows: monthly revenue, active users, churn rate, MRR trend chart
- User roles: Owner (all data), Admin (all data), Manager (team data only), Viewer (read-only, team data)
- Date picker: presets (last 7/30/90 days, this year) + custom date range
- Data refreshes every 15 minutes automatically; manual refresh button also available
- Data source: internal API pulling from PostgreSQL aggregates
- All 4 metrics show on all roles, but scope of data differs by role
- Empty state when no data exists for selected period

Please generate test cases covering:
1. Data display correctness (each metric for each role)
2. Date picker functionality
3. Role-based access control
4. Auto-refresh and manual refresh
5. Empty states and edge cases
6. Performance (large datasets)
```

**Zen output will include role-based cases like:**

```
RBAC TEST CASES
TC-RBAC-001 | Owner sees company-wide revenue | Positive
TC-RBAC-002 | Manager sees only their team's revenue | Positive
TC-RBAC-003 | Manager cannot see other teams' data | Negative
TC-RBAC-004 | Viewer role has no edit/export controls | Positive
TC-RBAC-005 | Session expires — redirect to login, not data exposure | Security/Edge

DATE PICKER
TC-DATE-001 | Select "Last 7 Days" preset | Positive
TC-DATE-002 | Custom range: start date = end date (same day) | Boundary
TC-DATE-003 | Custom range: start date after end date | Negative
TC-DATE-004 | Custom range spanning year boundary (Dec 31 – Jan 1) | Edge
TC-DATE-005 | Date range with no data | Empty state

REFRESH
TC-REF-001 | Auto-refresh triggers after 15 minutes | Positive
TC-REF-002 | Manual refresh shows loading state | Positive
TC-REF-003 | Data updates after refresh | Positive
TC-REF-004 | Manual refresh during auto-refresh interval | Edge
```

---

## Handling Ambiguous Requirements

Sometimes requirements are genuinely unclear. Instead of guessing, use Zen to surface the ambiguity explicitly.

**Prompt:**
```
I'm generating test cases for this requirement, but I'm not sure how to test it:

"The checkout should remember the user's preferences for next time."

This is ambiguous. Before generating test cases, please:
1. List every possible interpretation of "preferences" in a checkout context
2. For each interpretation, ask the clarifying question I should bring to the product team
3. Show me what the test cases would look like under 2-3 different interpretations

I'll get clarification and come back to finalize the test cases.
```

---

## Multi-Module Test Case Generation

For large features spanning multiple modules, use a hierarchical approach:

**Phase 1: Map the modules**
```
Our new onboarding flow spans 5 screens and involves 3 backend services.
Here are the modules:
1. Welcome screen (static, no logic)
2. Email verification
3. Profile setup (name, avatar, timezone)
4. Plan selection (free vs paid tiers)
5. Workspace creation

Please generate a TEST PLAN (not yet test cases) that:
- Defines the test scope for each module
- Identifies integration points between modules
- Lists the test types needed for each module
- Estimates the number of test cases per module
```

**Phase 2: Generate module by module**
```
Now let's focus on Module 2: Email Verification.

Context from the test plan: [paste relevant section]
Additional context: Verification emails are sent by SendGrid, 
links expire after 24 hours, links are single-use.

Generate comprehensive test cases for this module only.
```

---

## Quick Reference: Test Case Prompt Cheat Sheet

| Requirement Type | Key Prompt Phrase |
|-----------------|-------------------|
| User story with AC | "For each AC, generate the explicit path + 2-3 edge cases" |
| Prose requirements | "First extract formal requirements, then generate test cases" |
| API spec | "Cover: happy path, validation, auth, idempotency, boundaries" |
| Ambiguous requirements | "List interpretations before generating cases" |
| Large feature | "Generate a test plan first, then tackle each module" |
| After first draft | "What's missing? What's redundant? What are the top 5?" |
| Domain knowledge | "Based on this historical bug pattern, add cases targeting..." |
| Environment-specific | "Update to use these specific test values and credentials" |

---

## What's Next

You've generated your test cases. Now you need to understand how they fit into the bigger picture of your application's coverage. If you can't see where your coverage gaps are, you can't make strategic decisions about where to test next.

**→ Continue to [02 — Build a Test Coverage Map](./02-coverage-map.md)**
