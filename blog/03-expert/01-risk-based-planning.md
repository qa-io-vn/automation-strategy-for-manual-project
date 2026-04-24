# AI-Assisted QA: From Manual Tester to Automation Strategist
## Expert Guide 01 — Risk-Based Test Planning

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** Expert
> **← Prev:** [Expert Section Overview](./README.md) | **Next →** [Coverage Audit](./02-coverage-audit.md)

---

## The Problem With "Test Everything"

Most QA teams have an implicit strategy: test as much as possible, prioritize based on gut feel, and negotiate scope cuts under release pressure. This approach breaks down at scale because *everything feels equally important* until something catastrophic slips through.

Risk-based test planning replaces gut feel with a **structured, evidence-backed model** that tells you — and your stakeholders — exactly why you're testing what you're testing, and why you're not testing something else.

When you can answer "why aren't we testing this?" with data instead of silence, you've made the leap from technician to strategist.

---

## The Four Dimensions of Risk

Every feature or system component carries risk along four orthogonal dimensions. Scoring each one gives you a composite risk profile that drives test depth decisions.

### Dimension 1: Business Impact

*If this breaks, how bad is it for the business?*

| Score | Label | Definition | Example |
|-------|-------|------------|---------|
| 4 | Critical | Revenue loss, legal exposure, or complete service unavailability | Payment processing, user authentication |
| 3 | High | Significant user frustration, SLA breach, or major feature failure | Search, order history, notifications |
| 2 | Medium | Degraded experience for a subset of users | PDF export, advanced filters |
| 1 | Low | Cosmetic or minor functional issue | Tooltip text, icon alignment |

### Dimension 2: Change Frequency

*How often does this area change?*

| Score | Label | Definition |
|-------|-------|------------|
| 4 | Very High | Changed in almost every sprint; active development area |
| 3 | High | Changed monthly; feature iteration ongoing |
| 2 | Medium | Changed quarterly; mostly stable |
| 1 | Low | Rarely touched; legacy/stable component |

> **Why it matters:** A stable, high-impact area may need less frequent regression than a rapidly-changing medium-impact area. Risk isn't static.

### Dimension 3: Defect History

*What does the bug record tell us?*

| Score | Label | Definition |
|-------|-------|------------|
| 4 | Very Buggy | 3+ production incidents or 5+ bugs in last 6 months |
| 3 | Buggy | 1–2 production incidents or 3–4 bugs in last 6 months |
| 2 | Some Issues | 1–2 bugs in last 6 months, no production incidents |
| 1 | Clean | No bugs or incidents in last 6 months |

> **Data source:** Pull this from your bug tracker. If you're using Jira, a simple JQL query gives you this: `project = PROD AND type = Bug AND created >= -180d ORDER BY component`.

### Dimension 4: Technical Complexity

*How hard is this to get right?*

| Score | Label | Definition |
|-------|-------|------------|
| 4 | Very Complex | Async workflows, third-party integrations, complex business rules, race conditions |
| 3 | Complex | Multi-step flows, conditional logic, cross-service dependencies |
| 2 | Moderate | Standard CRUD with some business logic |
| 1 | Simple | Display-only, static content, simple validation |

---

## The Risk Scoring Model

**Composite Risk Score = (Business Impact × 2) + Change Frequency + Defect History + Complexity**

The weighting doubles Business Impact because a low-traffic, buggy, complex feature that doesn't affect revenue is still less critical than a high-revenue, stable, simple one.

**Risk Tiers:**

| Score | Tier | Test Depth |
|-------|------|------------|
| 14–16 | P0 — Critical | Full automated coverage + exploratory testing every release |
| 10–13 | P1 — High | Comprehensive automated coverage + targeted exploratory |
| 6–9 | P2 — Medium | Core path automation + manual exploratory quarterly |
| 2–5 | P3 — Low | Happy path only; manual ad-hoc as needed |

---

## Worked Example 1: E-Commerce Platform

**Components being evaluated:** Checkout, Product Search, Wishlist, Order History, Admin Dashboard

### Scoring Matrix

| Component | Business Impact (×2) | Change Freq | Defect History | Complexity | **Total** | **Tier** |
|-----------|----------------------|-------------|----------------|------------|-----------|----------|
| Checkout | 4 (8) | 3 | 4 | 4 | **19** | P0 |
| Product Search | 3 (6) | 4 | 3 | 3 | **16** | P0 |
| Order History | 3 (6) | 2 | 2 | 2 | **12** | P1 |
| Wishlist | 2 (4) | 3 | 1 | 2 | **10** | P1 |
| Admin Dashboard | 2 (4) | 2 | 1 | 3 | **10** | P1 |

### What This Tells You

- **Checkout** scores 19 — the highest possible. Every change to this component should trigger a full regression. This is where your deepest Playwright suite should live.
- **Product Search** is equally critical despite being "just" search — high change frequency and defect history pull it to P0.
- **Wishlist** is P1 despite being low-business-impact because it changes frequently. Don't neglect it, but don't over-invest either.
- **Admin Dashboard** can tolerate lighter coverage if it's stable; redirect those testing hours to Checkout.

---

## Worked Example 2: B2B SaaS Platform

**Components:** User Onboarding, Role-Based Access Control (RBAC), Billing/Subscriptions, Report Generation, API Integrations, Help Center

### Scoring Matrix

| Component | Business Impact (×2) | Change Freq | Defect History | Complexity | **Total** | **Tier** |
|-----------|----------------------|-------------|----------------|------------|-----------|----------|
| Billing/Subscriptions | 4 (8) | 2 | 3 | 4 | **17** | P0 |
| RBAC | 4 (8) | 2 | 2 | 4 | **16** | P0 |
| API Integrations | 3 (6) | 4 | 4 | 4 | **18** | P0 |
| User Onboarding | 3 (6) | 4 | 2 | 3 | **15** | P0 |
| Report Generation | 2 (4) | 3 | 3 | 3 | **13** | P1 |
| Help Center | 1 (2) | 1 | 1 | 1 | **5** | P3 |

### Insights

- **API Integrations** tops the chart at 18 — third-party dependencies create compounding risk. Every integration point needs contract tests and failure mode automation.
- **RBAC** at 16 reflects a legal/compliance risk premium — a permission bug is a security incident, not just a bug.
- **Help Center** at P3 is fine with annual manual walkthroughs. Don't write Playwright tests for static content.

---

## Worked Example 3: Fintech Application

**Components:** Transaction Processing, Fraud Detection, Account Statements, KYC/Verification, Mobile Push Notifications, Settings/Preferences

### Scoring Matrix

| Component | Business Impact (×2) | Change Freq | Defect History | Complexity | **Total** | **Tier** |
|-----------|----------------------|-------------|----------------|------------|-----------|----------|
| Transaction Processing | 4 (8) | 3 | 3 | 4 | **18** | P0 |
| Fraud Detection | 4 (8) | 4 | 2 | 4 | **18** | P0 |
| KYC/Verification | 4 (8) | 2 | 3 | 4 | **17** | P0 |
| Account Statements | 3 (6) | 2 | 2 | 3 | **13** | P1 |
| Push Notifications | 2 (4) | 3 | 3 | 2 | **12** | P1 |
| Settings/Preferences | 1 (2) | 2 | 1 | 1 | **6** | P2 |

### Fintech-Specific Notes

In fintech, regulatory and legal exposure elevates Business Impact scores across the board. A bug in KYC that allows non-compliant users through is a regulatory incident — it's not a P2 regardless of what the technical complexity score says. When domain knowledge contradicts your model, **override the score and document the reason**.

This is an important principle: the risk model is a *decision support tool*, not a decision-making oracle.

---

## Prompting Zen for Risk Analysis

Zen can help you build and maintain your risk model — especially when pulling context from multiple sources.

### Prompt 1: Initial Risk Scoring From Documentation

```
You are a QA risk analyst. I'm going to describe the components of our [product type] 
application. For each component, score it on four dimensions using this scale:

Business Impact: 4=Critical revenue/legal, 3=High user impact, 2=Medium, 1=Low/cosmetic
Change Frequency: 4=Every sprint, 3=Monthly, 2=Quarterly, 1=Rarely
Defect History: 4=3+ incidents/5+ bugs in 6mo, 3=1-2 incidents, 2=1-2 bugs, 1=Clean
Technical Complexity: 4=Async/integrations/complex rules, 3=Multi-step flows, 2=CRUD, 1=Simple

Calculate: Composite Score = (Business Impact × 2) + Change Freq + Defect History + Complexity

Assign risk tier:
- 14+: P0 Critical
- 10-13: P1 High
- 6-9: P2 Medium
- 2-5: P3 Low

Here are our components:
[paste your component list with brief descriptions]

Also flag any components where domain context (regulatory, security) might override the 
mathematical score. Explain your reasoning for each score.
```

### Prompt 2: Updating Risk Model After a Production Incident

```
We had a production incident last [timeframe] affecting [component]. 
Here's what happened: [incident description]

Given this incident:
1. How should I update the Defect History score for [component]?
2. Which adjacent components should have their risk score reviewed as a result?
3. What test gaps does this incident expose in our current coverage model?
4. Draft a one-paragraph "Risk Model Update" note I can add to our internal wiki.
```

### Prompt 3: Quarterly Risk Model Review

```
It's been 3 months since we last updated our risk model. Here's our current model:
[paste existing risk matrix]

Here are the changes in our product over the last quarter:
- New features shipped: [list]
- Components that received the most bug fixes: [list]
- Components that were refactored: [list]
- New third-party integrations added: [list]

Please:
1. Identify which risk scores likely need updating and why
2. Flag any components that may have moved between risk tiers
3. Identify any new components that need to be added to the model
4. Suggest any components that might now deserve a lower risk score (and why)
```

### Prompt 4: Coverage Gap Analysis Against Risk Model

```
Here's our risk model with current test coverage status:

| Component | Risk Tier | Automated Tests | Manual Scripts | Coverage Estimate |
|-----------|-----------|-----------------|----------------|-------------------|
[paste your data]

Identify:
1. Any P0 components with less than 80% estimated coverage — these are critical gaps
2. Any P3 components with extensive automation — this is over-investment to redirect
3. Suggest a prioritized backlog of coverage improvements
4. Estimate the risk reduction from addressing the top 3 gaps
```

---

## Presenting Risk Findings to Stakeholders

A risk matrix sitting in a Confluence page nobody reads is worthless. The goal is to make your risk model *actionable* in planning conversations.

### The Risk Briefing One-Pager

Use this template to present risk findings in sprint planning or release planning:

```
## Quality Risk Briefing — [Sprint/Release Name]

**High-Risk Changes This Cycle**
- [Component A] (P0): Changed in 3 PRs. Full regression planned. ETA: 4 hours.
- [Component B] (P1): New feature added to [area]. Targeted test expansion needed.

**Risk Items Requiring Decision**
- [Component C]: Refactored without test updates. Risk score increased from P2 → P1.
  Recommendation: Add 2-day test debt story to next sprint before next release.

**Accepted Risk**  
- [Component D] (P3): Not covered by automation. Manual check scheduled. Accepted.

**Current Risk Posture: MEDIUM**
[Brief narrative: 2-3 sentences on overall confidence]
```

### What NOT to Say in Stakeholder Meetings

| Don't Say | Say Instead |
|-----------|-------------|
| "We haven't tested X" | "X carries P2 risk. Here's our mitigation plan." |
| "We can't ship this safely" | "The current risk posture for this release is HIGH. Here's what would move it to MEDIUM." |
| "Testing takes time" | "Full regression on P0 components is 4 hours. Here's the tradeoff if we skip it." |
| "I'm not comfortable with this" | "This release has 2 untested P0 changes. My recommendation is [X]." |

The language shift from *feelings* to *data + recommendation* is the core of expert QA communication.

---

## Adjusting Test Depth by Risk Tier

### P0 — Critical Components

- **Automated:** Full happy path + all documented edge cases + all documented error states
- **Exploratory:** Minimum 2 hours session per release cycle
- **Data coverage:** Multiple data profiles (e.g., free user, paid user, admin, edge-case data)
- **Cross-browser:** Yes — at least Chrome + Safari (or whatever your analytics shows as top 2)
- **Mobile:** Yes, if usage data supports it
- **Regression trigger:** Every PR that touches this component

### P1 — High Risk Components

- **Automated:** Full happy path + critical edge cases
- **Exploratory:** 1 hour session when major changes land
- **Data coverage:** 2 data profiles minimum
- **Cross-browser:** Chrome only for automation; periodic manual cross-browser
- **Regression trigger:** PRs touching this component + weekly scheduled run

### P2 — Medium Risk Components

- **Automated:** Happy path only
- **Exploratory:** Quarterly, or when a bug is reported
- **Data coverage:** Standard test user
- **Cross-browser:** Not required in automation
- **Regression trigger:** Weekly scheduled run

### P3 — Low Risk Components

- **Automated:** None required (or a single smoke test if it's a user-facing entry point)
- **Exploratory:** Ad-hoc, triggered by reported issues only
- **Regression trigger:** Monthly scheduled run or manual

---

## Maintaining a Living Risk Model

The worst risk model is a static one. Build these habits:

1. **Post-incident review:** After every production bug, ask "would our risk model have predicted this?" If not, update it.

2. **Pre-sprint check-in:** 5 minutes at sprint planning to identify if any planned changes touch P0/P1 components.

3. **Quarterly full review:** Reassess all scores with fresh defect data. Use Zen Prompt 3 above to structure this.

4. **New feature intake:** Every new feature gets risk-scored before the sprint it's built. QA should be in design conversations, not just acceptance conversations.

---

## Checkpoint

You've completed the risk-based planning guide. Before moving on, make sure you can answer:

- [ ] Can you score any component in your product using the four-dimension model?
- [ ] Do you have a current risk matrix for your product (even a rough draft)?
- [ ] Can you explain your test depth decisions using risk tier language to a non-technical stakeholder?
- [ ] Have you used Zen to draft or refine any part of your risk model?

**Next:** [Coverage Audit →](./02-coverage-audit.md) — Now that you know what to prioritize, let's find out if you're actually testing the right things.
