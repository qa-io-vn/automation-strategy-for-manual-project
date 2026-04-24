# What to Gather Before You Start

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 00 — Preconditions
> **Navigation:** [← Section Overview](./README.md) | [Next: Understand Your Stack →](./02-understand-your-stack.md)

---

## Introduction

One of the biggest mistakes QA engineers make when starting with AI-assisted testing is diving straight into prompting without any preparation. They type something like *"help me test the login page"* and wonder why Zen's response feels generic and unhelpful.

Here's the truth: **Zen is not psychic**. It doesn't know your application, your business rules, your user base, or your risk tolerance. But give it rich, structured context — and it becomes a force multiplier.

This guide is your artifact collection sprint. Before you write a single prompt or create a single test case, spend time gathering everything that describes what you're testing, why it matters, and how it's supposed to behave. You'll use these materials throughout the entire series.

---

## The QA Artifact Hierarchy

Not all documents are created equal. Some will have a massive impact on your AI-assisted testing; others are nice-to-have. Here's how to prioritize:

| Priority | Artifact Type | Impact | Where to Find It |
|----------|--------------|--------|-----------------|
| **Critical** | Requirements / PRDs | Defines what should exist | Confluence, Notion, Google Docs |
| **Critical** | User Stories / Acceptance Criteria | Defines what "done" looks like | Jira, Linear, Trello, Asana |
| **Critical** | Bug history / known issues | Shows where problems cluster | Jira, GitHub Issues, Sentry |
| **High** | API documentation | Enables API-level testing | Swagger/OpenAPI, Postman collections |
| **High** | Existing test cases | Reveals current coverage | TestRail, Zephyr, spreadsheets |
| **High** | Test reports | Shows pass/fail trends | TestRail, CI pipelines, spreadsheets |
| **Medium** | UI mockups / wireframes | Clarifies expected UI behavior | Figma, Zeplin, InVision |
| **Medium** | Data dictionaries / DB schemas | Reveals field rules and constraints | Confluence, GitHub, DB tools |
| **Medium** | User flow diagrams | Shows expected navigation paths | Miro, Lucidchart, Figma |
| **Low** | Architecture diagrams | Helps understand integrations | Confluence, wiki |
| **Low** | Runbooks / SOPs | Shows operational expectations | Confluence, Notion |

---

## Gathering Each Artifact Type

### Requirements Documents and PRDs

Product Requirements Documents (PRDs) are gold. They describe the *why* behind features — the business context, the user problem being solved, and the constraints the solution must respect. This context is critical for AI-assisted test case generation because it helps Zen understand the boundaries of acceptable behavior.

**What to look for in a PRD:**
- Feature description and business justification
- User personas affected
- Functional requirements (what the system must do)
- Non-functional requirements (performance, security, accessibility)
- Out-of-scope items (equally important — you don't want to test things that were deliberately excluded)
- Success metrics

**How to extract this into a Zen-friendly format:**

```
Prompt: I have a PRD for our new checkout feature. Here are the key sections:

[paste PRD content]

Please help me extract:
1. A numbered list of all functional requirements
2. A list of non-functional requirements
3. Any explicit exclusions or out-of-scope items
4. 3-5 potential risk areas based on this feature

Format this as a structured QA intake document.
```

**If your PRD is messy or informal:**
Many teams write requirements as Slack messages, email threads, or informal Google Docs notes. That's okay. Gather whatever you have and use Zen to help structure it:

```
Prompt: These are my raw requirements notes for the user profile feature.
They're informal and incomplete. Please:
1. Extract all implied requirements as formal "The system shall..." statements
2. Flag any contradictions or ambiguities you spot
3. List the questions I should ask the product team before testing

[paste raw notes]
```

---

### User Stories and Acceptance Criteria

User stories are the QA engineer's best friend. A well-written user story with acceptance criteria (AC) is essentially a test specification waiting to be formatted. 

**The standard format:**
```
As a [user type],
I want to [perform action],
So that [I achieve goal].

Acceptance Criteria:
- GIVEN [precondition], WHEN [action], THEN [expected result]
- GIVEN [precondition], WHEN [action], THEN [expected result]
```

**What to collect:**
- Export all stories for the feature or sprint you're testing
- Include story status (ready for dev, in progress, done)
- Capture any clarifying comments from the dev team on the story
- Note stories that have been split or re-scoped mid-sprint

**Common problems with user stories and how to handle them:**

| Problem | Symptom | Zen Prompt Fix |
|---------|---------|----------------|
| Missing AC | Story has no "then" clauses | "Generate AC for this story based on the description and common UX patterns" |
| Vague AC | "The system should be fast" | "Make this AC measurable and specific. What threshold counts as 'fast'?" |
| Contradictory stories | Two stories describe different behavior for the same scenario | "These two stories seem to conflict. What questions should I raise with the product team?" |
| Stale stories | Story was written 6 months ago and the app has changed | "Given that the app now does X, identify which parts of this story's AC may be outdated" |

---

### Existing Test Cases

If there are existing test cases — even in a spreadsheet — gather them. They represent accumulated knowledge about the application, even if they're messy.

**What to look for:**
- Test case ID, title, and description
- Preconditions and test data
- Step-by-step actions
- Expected results
- Pass/fail status and last execution date
- Any notes about flaky or environment-dependent tests

**Turning existing test cases into AI fuel:**

```
Prompt: I have 47 existing manual test cases for our authentication module. 
Here are the first 10 as a sample:

[paste test cases]

Please:
1. Identify gaps — what important scenarios are NOT covered?
2. Identify duplicates or near-duplicates
3. Suggest which cases are good candidates for automation
4. Flag any cases where the expected result is ambiguous
```

---

### API Documentation

If your application has any backend services, API documentation is essential. It tells you:
- What endpoints exist
- What inputs they accept (and reject)
- What responses to expect
- Authentication/authorization requirements
- Rate limits and pagination

**Where to find it:**
- Swagger UI (usually at `/api/docs`, `/swagger`, or `/api-docs`)
- Postman collections (ask your dev team to export)
- Confluence/Notion pages written by backend engineers
- GraphQL playground at `/graphql`
- ReadMe.io or similar API documentation platforms

**Organizing API docs for Zen:**

```
Prompt: Here is the OpenAPI spec for our /orders endpoint (truncated for key sections):

[paste relevant spec sections]

Please generate test cases covering:
1. Happy path — successful order creation
2. Validation errors — missing required fields
3. Edge cases — boundary values for quantity, price
4. Authentication failures — missing/invalid token
5. Business rule violations — ordering out-of-stock items

For each test case, include: endpoint, method, request body, expected status code, expected response shape.
```

---

### Test Reports and Bug History

Historical data tells you where problems have lived before. A feature that had 15 bugs filed against it last quarter is higher risk than a feature with no bug history.

**What to collect:**
- Last 3–6 months of bug reports for your target features
- Test execution reports (what was run, what passed, what failed)
- Production incidents related to your features
- Customer-reported issues from support channels

**Mining bug history with Zen:**

```
Prompt: Here are the last 3 months of bugs filed against our payment module:

[paste bug summaries]

Please:
1. Categorize bugs by type (UI, logic, performance, data, integration)
2. Identify the 3 highest-risk areas based on frequency and severity
3. Suggest what test cases I should add to prevent regression of the top 5 bugs
4. Are there any patterns that suggest a systemic issue?
```

---

## Handling Missing Documentation

The reality of QA in most organizations: documentation is incomplete, outdated, or nonexistent. Here's how to handle the most common scenarios.

### Scenario 1: No Requirements at All

The feature exists. People use it. But no one wrote down what it's supposed to do.

**Your move — explore and document:**

1. **Use the application** — walk through every path you can find
2. **Talk to users** — ask customer success or support what customers expect
3. **Talk to developers** — a 15-minute call can replace 2 hours of guessing
4. **Check Git commit messages** — "Add X feature for Y reason" is documentation
5. **Review analytics** — what paths do real users actually take?

**Zen prompt for reverse-engineering requirements:**

```
Prompt: I'm testing a feature with no written requirements. 
Based on my exploration, here's how it currently behaves:

[describe current behavior step by step]

Please:
1. Convert this into formal requirements ("The system shall...")
2. Identify any behaviors that seem unintentional or inconsistent
3. List questions I should ask the team to clarify intent
4. Generate exploratory test ideas to discover behavior I might have missed
```

---

### Scenario 2: Legacy Project, No Documentation

You've inherited a 10-year-old application. The original developers are gone. There are test cases from 2015 that reference UI elements that no longer exist.

**The legacy documentation survival kit:**

1. **Code archaeology** — ask a developer to walk you through the key modules at a high level
2. **Database exploration** — schema tells you a lot about business rules (field lengths, foreign keys, constraints)
3. **Log analysis** — application logs show what operations are actually being performed
4. **Interview longtime users** — they know the edge cases better than anyone
5. **Sentry / error tracking** — existing error data is documentation of failure modes

```
Prompt: I'm testing a legacy application with no documentation. 
Here's what I've been able to learn from talking to the team and exploring the app:

Application: [name and rough description]
Key modules: [list them]
Known business rules I've discovered: [list them]
Known issues I've been warned about: [list them]
Tech stack: [what I know]

Please help me create a rough test strategy starting from this baseline.
What should I prioritize testing first? What questions are most critical to answer?
```

---

### Scenario 3: Outdated Documentation

The docs exist but they're 18 months out of date. Features have been added, others changed.

**Approach:**

1. Collect the outdated docs anyway — they're a starting point
2. Do a quick "delta walk" — use the app and compare to docs
3. Flag discrepancies and schedule time with the team to resolve them
4. Mark your working documents with "as of [date]" to track what you've verified

```
Prompt: I have requirements documentation from 18 months ago.
The application has changed significantly. Here are the old requirements:

[paste old requirements]

And here's what the app actually does now based on my testing:

[paste your observations]

Please:
1. Mark which old requirements appear to still be valid
2. Mark which appear to have changed
3. Identify what's in the current app that isn't in the old requirements
4. Generate a list of clarifying questions for my next team meeting
```

---

## Organizing Your Artifacts

Once you've gathered everything, organize it so Zen can reference it efficiently. Create a simple folder structure:

```
/qa-artifacts/
  /requirements/
    - prd-checkout-v2.md
    - user-stories-sprint-47.md
  /test-cases/
    - existing-test-cases-auth.csv
    - existing-test-cases-checkout.xlsx
  /api-docs/
    - openapi-spec.yaml
    - postman-collection.json
  /bug-history/
    - bugs-q1-2024.csv
    - production-incidents.md
  /ui-mockups/
    - (links to Figma)
  /project-context.md  ← Your master context file for Zen
```

### Building Your Master Context File

The `project-context.md` file is the most important artifact you'll create. It's the single document you paste at the start of any Zen conversation to bring it up to speed instantly.

**Template:**

```markdown
# Project Context: [Application Name]

## What This Application Does
[2-3 sentence description]

## Primary Users
[Who uses it, how often, what they're trying to accomplish]

## Tech Stack
[Frontend, backend, database, key third-party services]

## Key Features (for QA scope)
- [Feature 1]: [brief description]
- [Feature 2]: [brief description]

## Known Risk Areas
- [Area with historical issues]

## Testing Constraints
- [What we can't automate, what environments exist, any timing restrictions]

## Current Test Coverage
[Brief summary of what's tested and how]

## Out of Scope
[What we deliberately don't test and why]
```

---

## The Artifact Checklist

Before moving to the next section, confirm what you have:

| Artifact | Have It | Partial | Missing |
|----------|---------|---------|---------|
| Product requirements / PRD | ☐ | ☐ | ☐ |
| User stories with AC | ☐ | ☐ | ☐ |
| Existing test cases | ☐ | ☐ | ☐ |
| Bug history (3–6 months) | ☐ | ☐ | ☐ |
| API documentation | ☐ | ☐ | ☐ |
| UI mockups or designs | ☐ | ☐ | ☐ |
| Test execution reports | ☐ | ☐ | ☐ |
| Production incident history | ☐ | ☐ | ☐ |
| Project context file (created by you) | ☐ | ☐ | ☐ |

For anything in the "Missing" column, either schedule time to gather it this week, or document why it doesn't exist and how you'll compensate.

---

## What's Next

Now that you have your artifacts, you need to understand the technical environment you're testing in. Even if you're not a developer, knowing your tech stack will dramatically improve your ability to plan tests, communicate with engineers, and choose the right tools.

**→ Continue to [02 — Understand Your Tech Stack](./02-understand-your-stack.md)**
