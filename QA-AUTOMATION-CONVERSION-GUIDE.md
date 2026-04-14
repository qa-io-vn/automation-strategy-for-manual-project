# QA Automation Conversion Guide (Manual → Playwright Automation)
Owner: QA Automation Lead  
Tools: Playwright + JavaScript (POM + SOLID)

---

## 1. Purpose & Outcomes
This guide standardizes:
1) How to convert **manual regression test cases** into **automated tests**.
2) Coding rules/conventions to keep automation professional (POM + SOLID).
3) A scoring model to **prioritize** what to automate first when resources are limited.

The goal is to build a stable automated regression suite that can become a release gate.

---

## 2. Manual Test Case → Automation Test: Conversion Rules

### 2.1 Automation Eligibility Rules (Should we automate this case?)
Automate the manual test case if it meets most of these:
- High business value (P0/P1) or high risk area.
- Repeated regression every release/sprint.
- Stable requirements and stable UI/API behavior.
- Has deterministic expected results (not subjective/visual-only).
- Test data can be reliably created/seeded.

Avoid (or delay) automation when:
- The feature/UI changes every sprint (unstable).
- Expected result is subjective (visual layout, “looks good”).
- Depends on external services that are flaky and cannot be stubbed.
- Requires complex hardware, OTP/SMS, CAPTCHA, email approvals (unless test hooks exist).
- Requires too many steps and provides low additional risk coverage.

**Decision output:** `Automate now` / `Automate later` / `Manual only`

---

### 2.2 Conversion “Golden Workflow” (Step-by-step)
For every manual test case to convert, follow this flow:

**Step 1 — Normalize the manual test case**
Rewrite it into a strict structure:
- Title
- Module/Feature
- Preconditions
- Test data
- Steps (numbered)
- Expected results (mapped per step if needed)
- Priority: P0/P1/P2
- Type: Regression / Smoke / Sanity
- Notes: screenshots, links, known issues

**Step 2 — Choose the best automation level**
Prefer lower-level automation when possible:
- If expected result can be validated by API response or DB state → **API test** (preferred)
- If it validates critical end-user flow or UI integration → **UI E2E**
- If it only checks UI rendering → consider **manual** or limited snapshot (careful)

**Step 3 — Reduce UI steps using API setup**
Replace long UI setup steps with:
- API seeding (`createCustomer`, `createOrder`, etc.)
- Auth token reuse / login fixture
- Direct navigation to URL after seeding

**Step 4 — Identify stable selectors**
Rules:
- Use `data-testid` as primary locator strategy.
- If missing, create a ticket for dev: “Add data-testid for X”.
- Temporary fallback allowed: role/text locators (`getByRole`, `getByText`) if stable.
- Avoid: `nth-child`, long CSS chains, XPath (unless no choice).

**Step 5 — Define assertions (high signal only)**
Rules:
- Assert 1–3 most important outcomes, not everything.
- Prefer state-based asserts:
  - UI: key status text, URL, table row, toast, heading, visible element
  - API: status code + response fields + schema/contract if possible
- Avoid brittle assertions:
  - exact whitespace
  - dynamic timestamps unless normalized
  - pixel-perfect layout

**Step 6 — Implement using the 3-layer architecture**
- **Spec** (test file): scenario orchestration only
- **Flow**: business steps (create order, approve invoice)
- **Page Object**: locators + UI actions only

**Step 7 — Classify + tag**
Add tags:
- `@smoke` (small subset; PR/merge gate)
- `@regression` (release gate)
- `@module-<name>` (routing + ownership)

**Step 8 — Review checklist + merge**
- Passes locally
- Passes CI
- No sleeps
- Stable selectors
- Test data isolation
- Trace captured on failure

---

### 2.3 Manual-to-Automation Mapping Rules (What becomes what?)
Convert common manual patterns like this:

1) **“Login as user X” (precondition)**
- Convert to: `test.use({ storageState: ... })` or a `loginAs(role)` fixture
- Avoid repeating login UI in every test

2) **“Create entity A then B then C” (setup)**
- Convert to: API seeding helper (data builder + API client)
- UI only for the action being tested, not for setup

3) **“Verify multiple fields on a page”**
- Convert to: verify only fields that represent the requirement outcome
- Optional: one extra assert for critical field; avoid asserting every label

4) **“Same test with many inputs”**
- Convert to: data-driven tests (`test.describe` + loops) with careful limits
- Avoid producing 50 near-duplicate tests; prefer 5–10 high-value variants

5) **“Negative tests”**
- Convert to: verify error message and no state change
- API negative tests are usually more stable than UI negative tests

---

## 3. Automation Script Rules (Coding Conventions)

### 3.1 Architecture (must follow)
**Spec → Flow → Page**
- Spec file should not contain raw selectors (except rare cases).
- Flows can call multiple Pages and API helpers.
- Page Objects contain:
  - locators
  - UI actions (`clickSave`, `fillForm`, `open`)
  - simple UI assertions (optional), but prefer asserting in spec/flow.

**Anti-patterns**
- Mega Page Object containing 20 screens.
- Putting business logic in Page Objects.
- Tests directly manipulating locators everywhere (no reuse).

---

### 3.2 SOLID interpretation for test automation (practical)
- **Single Responsibility:**  
  Each test validates one scenario. Each Flow does one business task. Each Page handles one screen/area.
- **Open/Closed:**  
  Extend by adding new Flow methods, not rewriting existing ones for one test.
- **Liskov Substitution:**  
  If you have base pages, derived pages must keep same behavior.
- **Interface Segregation:**  
  Keep page APIs small (don’t expose every element).
- **Dependency Inversion:**  
  Tests depend on Flow APIs, not on DOM details.

---

### 3.3 Naming conventions
- Test title: `should <expected> when <condition>`
- File names: `<module>.<feature>.spec.js`
- Page class: `<Feature>Page`
- Flow: `<domain>.flow.js` exporting functions

---

### 3.4 Selector rules (hard rules)
Prefer, in order:
1) `page.getByTestId('...')`
2) `getByRole` with accessible name
3) `getByText` (only stable UI text)
4) CSS selectors only as last resort

Forbidden unless approved:
- `waitForTimeout`
- `nth-child` selectors
- brittle XPath

---

### 3.5 Wait rules (hard rules)
- Rely on Playwright auto-wait.
- Wait for:
  - network idle only when appropriate
  - a specific element state (`toBeVisible`, `toHaveText`, etc.)
  - stable API completion if needed

---

### 3.6 Assertions rules
- Keep assertions minimal & meaningful.
- Assert business outcome:
  - status transitions
  - created entity exists
  - totals correct
  - permissions enforced

---

### 3.7 Test data & isolation rules
- Each test must be runnable alone.
- Create data via API where possible.
- Use unique identifiers for created entities.
- Clean up if the system accumulates data badly (optional if environment resets nightly).

---

### 3.8 Tagging & suites
- `@smoke`: runs on every PR/merge (fast)
- `@regression`: runs nightly/release gate
- `@extended`: non-blocking or long-run suites
- `@module-*`: ownership and routing

---

### 3.9 Flaky test policy
- A test that fails intermittently must be:
  - fixed within 24–48 hours, or
  - quarantined (non-blocking) until fixed
- Retries are limited (e.g., 1) and must not be used to hide flakiness.

---

## 4. Prioritization Model: Which Manual Cases to Automate First?

### 4.1 Scoring overview
Compute two scores:
- **Business Priority Score (BPS)**: 1–5 (higher = automate sooner)
- **Automation Complexity Score (ACS)**: 1–5 (higher = harder)

Then compute:
- **Automation ROI Score = BPS / ACS** (higher = best candidates)

---

### 4.2 Business Priority Score (BPS) rubric (1–5)
Score each manual case:
1. Low impact, rarely used, edge case
2. Some usage but not critical
3. Medium impact, common path
4. High impact, frequently used, high defect history
5. Mission-critical (payment, core workflow), regulatory, high risk

Add +1 (cap at 5) if:
- historically buggy area
- high change frequency
- blocks release when broken

---

### 4.3 Automation Complexity Score (ACS) rubric (1–5)
1. Very easy:
   - stable UI
   - simple data
   - clear selectors
   - deterministic checks
2. Easy–medium:
   - some data setup
   - few pages
3. Medium:
   - multiple roles
   - multiple systems/pages
   - some async complexity
4. Hard:
   - complex setup
   - unstable UI
   - external dependencies
   - heavy permissions
5. Very hard:
   - CAPTCHA/OTP/email flows without hooks
   - highly unstable requirements
   - requires special environment, hardware, or third-party non-testable components

---

### 4.4 Automation Candidate Categories (use this to plan)
Use BPS and ACS to classify:

- **Category A (Do now):** BPS 4–5 AND ACS 1–3  
  => automate immediately, include in Regression Core
- **Category B (Do soon):** BPS 3–5 AND ACS 4  
  => automate after framework/data helpers improve
- **Category C (Defer):** BPS 1–2 OR ACS 5  
  => keep manual, or redesign product/test hooks first

---

### 4.5 Example scoring table
| Manual Test Case | BPS | ACS | ROI=BPS/ACS | Decision |
|---|---:|---:|---:|---|
| Create order + verify status | 5 | 2 | 2.5 | Do now |
| Export report PDF formatting | 3 | 4 | 0.75 | Do soon (maybe partial) |
| OTP login via SMS | 5 | 5 | 1.0 | Defer until test hook |
| Search customer + view details | 4 | 2 | 2.0 | Do now |

---

## 5. Conversion Checklist (Definition of Done)
A converted automated test is “Done” when:
- It uses Spec → Flow → Page structure
- Stable selectors (`data-testid` preferred)
- No hard sleeps
- Test data created reliably
- Assertions validate the requirement outcome
- Tagged properly (`@smoke` / `@regression` / module)
- Runs locally and in CI
- Trace/video configured on failure

---

## 6. Recommended Templates

### 6.1 Manual Test Case Intake Template
- Title:
- Module:
- Priority (P0/P1/P2):
- Type (Smoke/Regression/Extended):
- Preconditions:
- Test Data:
- Steps:
- Expected:
- Automation Level (API/UI/Hybrid):
- Complexity (1–5):
- Notes/Links:

### 6.2 Automation Implementation Template (AAA)
- Arrange: seed + login + navigate
- Act: perform action under test
- Assert: verify business outcome

---

## 7. Ownership & Review Rules
- The author owns fixing their test if it fails.
- Reviewers enforce conventions; no exceptions.
- Flaky tests are treated as bugs (priority).

---

## 8. Continuous Improvement
Monthly:
- Remove duplicates
- Refactor flows/pages
- Improve data seeding
- Reduce runtime
- Improve tags and suite composition