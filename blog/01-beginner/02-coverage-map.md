# Build a Test Coverage Map

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Generate Test Cases](./01-generate-test-cases.md) | [Next: Bug Reports →](./03-bug-reports.md)

---

## Introduction

Ask most QA engineers "do you have good coverage?" and you'll get some version of "I think so" or "we cover most things." Ask them to show you exactly what's covered and what's not, and the conversation gets uncomfortable fast.

That's not a criticism — it's a structural problem. Test coverage is genuinely hard to see without a dedicated tool or a deliberately maintained artifact. And most teams don't have either.

A **test coverage map** solves this. It's a living document that shows every feature, every test dimension, and the state of coverage for each combination. It transforms "I think we have good coverage" into "we have 87% coverage of user-facing workflows, 100% coverage of payment flows, and zero coverage of the API error paths."

Building one with Zen takes a few hours the first time. Maintaining it takes 15 minutes a week.

---

## What Test Coverage Actually Means

"Coverage" is an overloaded word in QA. Let's define the dimensions you actually care about:

### Functional Coverage

Are all features and behaviors that the application is supposed to have actually tested?

- **Feature coverage**: Do you have at least one test case for each feature?
- **Scenario coverage**: Do you cover the happy path AND the most important negative paths for each feature?
- **Role coverage**: Do you test each feature for each user role that can access it?

### User Path Coverage

Do you test the complete journeys users actually take — not just individual screens in isolation?

- **Critical user journeys**: Sign up → onboard → use core feature → get value
- **Purchase flows**: Browse → select → configure → pay → confirm
- **Error recovery flows**: Encounter error → understand it → recover successfully

### API Coverage

Do you test the API independently from the UI?

- **Endpoint coverage**: Do you have at least one test for each documented endpoint?
- **Method coverage**: GET, POST, PUT, DELETE, PATCH — are all relevant methods tested?
- **Status code coverage**: Do you test for 200, 400, 401, 403, 404, 422, 500?

### Data Coverage

Do you test with a representative range of data?

- **Boundary values**: Minimum and maximum valid values
- **Invalid data**: Null, empty, too long, wrong type, special characters
- **Realistic data**: Data that mirrors real user input (not just "test123")

### Negative Coverage

This is where most teams have the biggest gaps. Every positive test case should have at least one corresponding negative case.

| Positive Case | Corresponding Negative Cases to Add |
|--------------|--------------------------------------|
| Login with valid credentials | Invalid password, locked account, unverified email |
| Create order with valid payment | Declined card, expired card, insufficient funds |
| Upload profile photo | File too large, wrong file type, corrupted file |
| Search with valid query | Empty search, very long query, SQL injection attempt |

---

## Building Your First Coverage Map

### Step 1: Define Your Coverage Dimensions

Start by listing every feature of your application and all relevant dimensions:

**Prompt:**
```
I need to build a test coverage map for my application.

Application: [name and description]
Key features:
- [Feature 1]
- [Feature 2]
- [Feature 3]
[continue]

User roles: [list roles]
Platforms/environments: [web, iOS, Android, etc.]

Please help me create a coverage map structure by:
1. Organizing the features into logical groups (modules)
2. Defining the test dimensions I should track for each feature 
   (e.g., functional, negative, boundary, API, permission)
3. Creating a table structure I can use to track coverage status

Don't fill in coverage status yet — just give me the structure.
```

### Step 2: Populate Your Existing Coverage

Now tell Zen what you currently have:

**Prompt:**
```
Here is the coverage map structure you created. Now help me populate it 
with my current test coverage.

Here are my existing test cases (exported from our test management tool):
[paste test case list or summary]

For each feature in the map:
1. Identify which test cases cover it
2. Mark the coverage status as: Full / Partial / Minimal / None
3. Identify the most critical gaps

Use this key:
✅ = Full coverage (happy path + key negatives + edge cases)
🟡 = Partial coverage (happy path only, or missing key scenarios)
🔴 = Minimal coverage (1–2 test cases, major gaps)
⬜ = No coverage
```

### Step 3: Generate a Gap Analysis

**Prompt:**
```
Based on the coverage map, please:
1. Rank the top 5 coverage gaps by risk (which uncovered areas are most likely to cause production issues?)
2. For each gap, estimate how many test cases I need to fill it
3. Suggest which gaps to address this sprint vs. next sprint
4. Are there any areas where I might be over-testing (coverage I could reduce)?
```

---

## Example Coverage Map: E-Commerce Platform

Here's a real coverage map for a mid-sized e-commerce application:

### Feature Coverage Table

| Module | Feature | Functional | Negative | API | Permission | Mobile | Status |
|--------|---------|-----------|---------|-----|-----------|--------|--------|
| **Auth** | Registration | ✅ | 🟡 | ✅ | ⬜ | 🔴 | Partial |
| **Auth** | Login | ✅ | ✅ | ✅ | ⬜ | 🟡 | Good |
| **Auth** | Password Reset | 🟡 | 🔴 | ⬜ | ⬜ | ⬜ | Poor |
| **Auth** | Session Management | 🔴 | 🔴 | ⬜ | ⬜ | ⬜ | Critical Gap |
| **Catalog** | Product Listing | ✅ | 🟡 | ✅ | ⬜ | ✅ | Good |
| **Catalog** | Product Search | ✅ | 🟡 | 🟡 | ⬜ | 🟡 | Partial |
| **Catalog** | Filters & Sort | 🟡 | 🔴 | ⬜ | ⬜ | 🔴 | Poor |
| **Catalog** | Product Detail | ✅ | 🟡 | ✅ | ⬜ | ✅ | Good |
| **Cart** | Add to Cart | ✅ | ✅ | ✅ | ⬜ | ✅ | Good |
| **Cart** | Update Quantity | ✅ | 🟡 | 🟡 | ⬜ | 🔴 | Partial |
| **Cart** | Remove Item | ✅ | 🟡 | 🟡 | ⬜ | 🟡 | Good |
| **Cart** | Coupon Codes | ✅ | ✅ | 🟡 | ⬜ | ⬜ | Partial |
| **Checkout** | Address Entry | ✅ | ✅ | ✅ | ⬜ | 🟡 | Good |
| **Checkout** | Payment (Card) | ✅ | ✅ | ✅ | ⬜ | 🟡 | Good |
| **Checkout** | Payment (PayPal) | 🟡 | 🔴 | ⬜ | ⬜ | ⬜ | Poor |
| **Checkout** | Order Confirmation | ✅ | 🟡 | 🟡 | ⬜ | 🟡 | Partial |
| **Orders** | Order History | ✅ | 🟡 | 🟡 | 🟡 | 🔴 | Partial |
| **Orders** | Order Detail | ✅ | 🟡 | 🟡 | 🟡 | ⬜ | Partial |
| **Orders** | Order Cancellation | 🔴 | 🔴 | ⬜ | ⬜ | ⬜ | Critical Gap |
| **Orders** | Refunds | ⬜ | ⬜ | ⬜ | ⬜ | ⬜ | Critical Gap |
| **Profile** | Edit Profile | ✅ | 🟡 | 🟡 | ⬜ | ⬜ | Partial |
| **Profile** | Address Book | 🟡 | 🔴 | ⬜ | ⬜ | ⬜ | Poor |
| **Profile** | Payment Methods | 🔴 | ⬜ | ⬜ | ⬜ | ⬜ | Critical Gap |
| **Admin** | User Management | 🟡 | 🔴 | ⬜ | 🔴 | ⬜ | Poor |
| **Admin** | Order Management | 🟡 | 🔴 | ⬜ | 🔴 | ⬜ | Poor |

### Coverage Summary by Module

| Module | Features Covered | Coverage Score | Priority |
|--------|-----------------|----------------|---------|
| Authentication | 2/4 features fully covered | 45% | High |
| Product Catalog | 2/4 features fully covered | 55% | Medium |
| Cart | 3/4 features fully covered | 65% | Medium |
| Checkout | 2/4 features fully covered | 50% | High |
| Orders | 0/4 features fully covered | 25% | Critical |
| User Profile | 0/3 features fully covered | 20% | Medium |
| Admin | 0/2 features fully covered | 15% | Medium |

### Gap Analysis Results

From this map, the gaps are immediately obvious:
1. **Critical:** Refunds have zero coverage — any refund bug goes straight to production
2. **Critical:** Order cancellation has minimal coverage — another revenue-impacting flow
3. **Critical:** Session management is barely tested — potential security exposure
4. **High:** PayPal payment flow is barely tested for an alternative payment method
5. **High:** Mobile coverage is poor across the board

---

## Prompt Templates for Coverage Audits

### Audit Existing Test Cases

```
I have [X] existing test cases for my [module/application].
Here's a summary of what they cover:

[paste test case titles or descriptions]

Please:
1. Identify which features appear to have no test cases
2. Identify which features appear to have only happy-path coverage
3. Calculate a rough coverage percentage by module
4. List the 5 most critical uncovered scenarios based on business risk

I'll use this to build a coverage map and prioritize new test case creation.
```

### Coverage Gap for a Specific Feature

```
I want to do a deep coverage audit on our [feature name].

Here are the test cases I currently have for this feature:
[list existing test cases]

Here is what the feature is supposed to do (requirements):
[paste requirements]

Please:
1. Identify what scenarios are covered
2. Identify what important scenarios are missing
3. Identify any test cases that test the same scenario (duplicates)
4. Generate the missing test cases for the top 5 gaps
```

### Risk-Based Coverage Prioritization

```
I have limited time before the release. I can only execute or add 
test cases for the highest-risk areas.

Here's my coverage map:
[paste map]

Here's recent production incident history:
[paste incident summary]

Here's what's changed in this release:
[paste release notes or changelog]

Please rank my test coverage priorities for this release:
1. What must be tested (no exceptions)
2. What should be tested if time permits
3. What can safely be deferred
4. Are there regression risks from the new changes that affect covered areas?
```

---

## Tracking Coverage Over Time

A one-time coverage map is useful. A coverage map you update every sprint becomes a strategic asset.

### The Weekly Coverage Update Workflow

Every Monday (or at the start of each sprint), spend 15 minutes updating your coverage map:

**Prompt:**
```
Here is my current coverage map: [paste map]

Since last update:
- New test cases added: [list titles]
- Features removed from scope: [list]
- New features added to scope: [list]
- Tests deprecated (no longer valid): [list]

Please:
1. Update the coverage map with these changes
2. Recalculate coverage scores
3. Tell me if any new gaps were introduced by the scope changes
4. Suggest the highest-priority addition for this sprint
```

### Coverage Trend Tracking

Keep a simple log in your Zenflow memory file:

```markdown
## Coverage History

| Date | Total Features | Fully Covered | Partially Covered | Not Covered | Overall % |
|------|---------------|---------------|-------------------|------------|-----------|
| 2024-01-08 | 25 | 8 | 11 | 6 | 54% |
| 2024-01-22 | 25 | 10 | 12 | 3 | 64% |
| 2024-02-05 | 27 | 11 | 13 | 3 | 65% |
| 2024-02-19 | 27 | 13 | 12 | 2 | 70% |
```

Sharing this trend in sprint reviews demonstrates your team's investment in quality improvement.

---

## Visualizing Coverage for Stakeholders

Sometimes you need to communicate coverage to people who won't read a table. Use Zen to help translate:

**Prompt:**
```
Here is my coverage map data. Please write a brief (3 paragraph) 
executive summary of our test coverage status, appropriate for sharing 
with a product manager or VP of Engineering.

The summary should:
1. State overall coverage as a simple percentage
2. Highlight the most critical gaps and their business risk
3. State what we're doing about it and the timeline
4. Avoid technical jargon

Coverage data:
[paste map]
```

---

## Coverage Map Maintenance Checklist

Keep your map accurate with this weekly routine:

- [ ] New test cases added this week? Update coverage map.
- [ ] Any features changed? Check if existing test cases are still valid.
- [ ] Any new features shipped? Add them to the map as uncovered.
- [ ] Any features deprecated? Remove from map.
- [ ] Any bugs found in previously "covered" areas? Downgrade coverage status.
- [ ] Share updated coverage score in the team channel.

---

## What's Next

Your coverage map shows you what you're testing. But when something goes wrong — and it will — you need to communicate that clearly and efficiently. Next up: writing bug reports that developers respect and prioritize.

**→ Continue to [03 — Write Professional Bug Reports](./03-bug-reports.md)**
