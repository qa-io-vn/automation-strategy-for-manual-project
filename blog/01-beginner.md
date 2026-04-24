# Part 1 — Beginner: Your First Week with AI-Assisted Testing

> **Series:** AI-Assisted QA — From Manual Tester to Automation Strategist  
> **Level:** Beginner  
> **Prev:** [Preconditions — What You Need Before Starting](./00-preconditions.md)  
> **Next:** [Part 2 — Advanced: Automating with AI](./02-advanced.md)

---

## What You'll Achieve This Week

By the end of this phase, you'll be able to:

- Generate structured test cases from requirements using AI
- Write professional bug reports in seconds
- Build a visual test coverage map
- Track all your testing work inside Zenflow

No coding required. This is 100% manual testing — just supercharged.

---

## Day 1: Generate Test Cases from Requirements

### The Old Way
You read requirements, stare at a blank spreadsheet, and start writing test cases from memory and experience. It takes hours.

### The AI Way

**Step 1:** Copy a requirement or user story.

**Step 2:** Send it to Zen with this prompt:

```
I'm testing the following requirement. Generate structured test cases covering 
happy path, edge cases, and negative scenarios.

Requirement:
[paste your requirement here]

Format each test case with:
- Title
- Preconditions
- Steps
- Expected Result
- Priority (P0/P1/P2)
- Type (Smoke/Regression/Extended)
```

**Step 3:** Review the output. Add, remove, or adjust based on your knowledge of the system.

**Step 4:** Ask Zen to fill gaps:

```
What scenarios did we miss? Consider permission-based access, 
concurrent users, and data validation edge cases.
```

> **Real result:** A requirement that used to take 3 hours to write test cases for now takes 30 minutes — with better coverage.

---

### Example: Login Feature

**Input requirement:**
> Users can log in with email and password. After 5 failed attempts, the account is locked for 30 minutes. Users can reset their password via email.

**Zen generates:**

| # | Title | Priority | Type |
|---|---|---|---|
| TC-001 | Login with valid credentials | P0 | Smoke |
| TC-002 | Login with invalid password | P1 | Regression |
| TC-003 | Login with non-existent email | P1 | Regression |
| TC-004 | Account locks after 5 failed attempts | P1 | Regression |
| TC-005 | Locked account cannot login within 30 min | P1 | Regression |
| TC-006 | Account unlocks after 30 minutes | P2 | Extended |
| TC-007 | Password reset email is sent | P1 | Regression |
| TC-008 | Password reset link expires after X minutes | P2 | Extended |
| TC-009 | Login with empty email field | P1 | Regression |
| TC-010 | Login with empty password field | P1 | Regression |
| TC-011 | SQL injection attempt in email field | P1 | Regression |
| TC-012 | Login with leading/trailing spaces in email | P2 | Extended |

12 test cases generated in under 60 seconds. You'd also review and add any business-specific scenarios you know about.

---

## Day 2: Map Your Coverage

Once you have test cases, ask Zen to help you understand your coverage posture.

### Coverage Gap Analysis Prompt

```
Here are my existing test cases for the [Module Name] module:
[paste your test cases]

And here are the requirements for that module:
[paste your requirements]

Identify:
1. Requirements that have NO test cases
2. Requirements that are only partially covered
3. Areas that seem over-tested (diminishing returns)
4. Suggested priority areas to add coverage
```

### Output You'll Get

Zen will give you a clear breakdown like:

- ✅ **Covered:** Login, Logout, Password Reset
- ⚠️ **Partial:** Account lockout (missing unlock scenario)
- ❌ **Not covered:** Session timeout, Remember me feature, SSO login
- 💡 **Over-tested:** Basic form validation (7 cases, could be 3)

This becomes your **living test coverage map** — something you update every sprint.

---

## Day 3: Write Bug Reports Like a Senior QA

### The Old Way
You find a bug, write a quick note: *"Submit button broken."* Developers ask follow-up questions. Back-and-forth takes days.

### The AI Way

When you find a bug, describe what happened to Zen:

```
I found a bug. When I click the Submit button on the checkout page after 
adding a promo code, nothing happens. The page doesn't show an error, 
the order isn't created, and the button doesn't disable or show a loading state.

Help me write a professional bug report.
App: [Your app name]
Environment: Staging
Browser: Chrome 124
```

**Zen generates:**

---

**Bug Title:** Submit button on checkout page is unresponsive after applying a promo code

**Environment:** Staging | Chrome 124 | macOS

**Severity:** P1 — High

**Steps to Reproduce:**
1. Log in as a registered user
2. Add any item to cart
3. Navigate to Checkout page
4. Enter a valid promo code and click "Apply"
5. Click the "Submit Order" button

**Actual Result:**  
The Submit button does not respond. No loading state, no error message, and no order is created in the system.

**Expected Result:**  
Clicking Submit should trigger order creation, show a loading state, then redirect to the order confirmation page.

**Impact:**  
Users cannot complete checkout after applying a promo code. This blocks the core purchase flow.

**Attachments:**  
[ screenshot / screen recording ]

---

That took 90 seconds instead of 15 minutes. And it's far more useful to your dev team.

---

## Day 4: Build Your Test Execution Checklist

Before each test cycle, use Zen to build a focused checklist:

```
I'm about to test the [Release Name / Sprint X] build.

Changed features this release:
- [Feature 1]
- [Feature 2]

Known risky areas:
- [Area 1]

Give me a structured test execution checklist covering:
1. Smoke tests to run first
2. Regression areas to focus on
3. New feature scenarios to validate
4. Regression risks from the changes
```

Zen will produce a ready-to-use checklist you can copy into your test tool or share with the team.

---

## Day 5: Organize It All in Zenflow

Everything you generate is only useful if it's tracked. Use Zenflow + Zen to:

- **Create tasks for bugs found** — ask Zen: *"Create a bug task for this issue in my [Project Name] project"*
- **Create tasks for missing test cases** — ask Zen: *"Create a task to write test cases for the SSO login feature"*
- **Track test execution** — ask Zen: *"Create a task for Sprint 12 regression execution with subtasks for each module"*

This keeps your work visible to the whole team — not just buried in your own spreadsheet.

---

## Weekly Habit: The AI-Assisted QA Ritual

Establish this rhythm every sprint:

| When | Action | AI Prompt |
|---|---|---|
| Sprint start | Generate test cases for new features | "Generate test cases for [user story]" |
| Sprint start | Review coverage gaps | "What's missing in our coverage for [module]?" |
| During sprint | Write bug reports | "Write a professional bug report for [description]" |
| Before release | Build execution checklist | "Build my regression checklist for this release" |
| After release | Summarize test results | "Summarize these test results and flag risks" |

---

## Common Mistakes at This Stage

- **Pasting AI output directly without review** — Always read and adjust. You know your system better than the AI.
- **Using vague prompts** — The more context you give, the better the output.
- **Not tracking generated work in Zenflow** — AI suggestions are only valuable if acted on and tracked.
- **Trying to do everything at once** — Start with one feature, one sprint. Build the habit first.

---

## Checkpoint: Are You Ready for Advanced?

Before moving on, you should be doing these consistently:

- [ ] Generating test cases from requirements with AI
- [ ] Writing bug reports with AI assistance
- [ ] Running coverage gap analysis before each sprint
- [ ] Tracking bugs and test tasks in Zenflow

→ **[Part 2: Advanced — Converting Manual Tests to Automation](./02-advanced.md)**

---

*Part of the series: **AI-Assisted QA — From Manual Tester to Automation Strategist***
