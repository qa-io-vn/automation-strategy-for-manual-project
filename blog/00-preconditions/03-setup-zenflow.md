# Set Up Zenflow: Your AI-Assisted QA Command Center

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 00 — Preconditions
> **Navigation:** [← Understand Your Stack](./02-understand-your-stack.md) | [Next: Self-Assessment →](./04-self-assessment.md)

---

## Introduction

Zenflow is the platform that turns Zen from a general-purpose AI into your dedicated QA assistant. It gives Zen memory, context, task management, and a persistent workspace that grows smarter the more you use it.

The setup process takes about 45 minutes if you do it right. Rushing this step is a false economy — a poorly configured workspace means you'll be re-explaining context to Zen on every conversation. A well-configured one means Zen knows your project, your conventions, and your priorities without you having to repeat yourself.

This guide walks you through every step, from account creation to your first productive conversation.

---

## Step 1: Create Your Zenflow Account

If you don't already have a Zenflow account, go to [zenflow.ai](https://zenflow.ai) and sign up. You can use Google, GitHub, or email/password.

**Recommended:** Use your work email if this is a professional project. This makes it easier to collaborate with teammates later.

---

## Step 2: Create a Project

Your Zenflow project is the container for everything related to one application or product. If you test multiple applications, create separate projects for each.

**To create a project:**

1. From the dashboard, click **"New Project"**
2. Give it a name that matches your application: e.g., `ShopCore QA`, `Payments Platform Testing`, `Mobile App v3 QA`
3. Add a brief description — this will help if teammates join later
4. Select a color and icon (optional, but helps with visual identification)
5. Click **"Create Project"**

**Naming conventions that work well:**
- `[AppName] QA` — for a single product
- `[AppName] [Platform] QA` — e.g., `ShopCore iOS QA` vs `ShopCore Web QA`
- `[TeamName] Testing` — if you're testing multiple services owned by one team

**What to avoid:**
- Generic names like "Testing" or "My Project" — unhelpful when you have multiple projects
- Names that won't make sense in 6 months

---

## Step 3: Connect Zen to Your Preferred Channel

Zen lives wherever you work. You can interact with it through the web UI, Slack, or Telegram. You can use multiple channels simultaneously.

### Option A: Web UI

The simplest option. Available immediately in your Zenflow project.

1. Open your project
2. Click **"Open Chat"** in the left sidebar
3. You're connected — start typing

**Best for:** Deep focus work sessions, complex prompts, when you need to share large amounts of context

### Option B: Slack Integration

Connect Zen to your Slack workspace so you can collaborate with your team.

1. In Zenflow, go to **Settings → Integrations → Slack**
2. Click **"Connect to Slack"**
3. Authorize Zenflow in the Slack OAuth flow
4. Choose which Slack workspace to connect
5. In Zenflow, click **"Add to Slack"** for your project
6. Zen will appear as a bot in your Slack workspace
7. You can DM Zen directly, or add it to a channel (e.g., `#qa-automation`)

**To use Zen in Slack:**
- DM the Zen bot directly for private conversations
- In a channel, mention `@Zen` followed by your message
- Zen will respond in the thread

**Best for:** Quick questions during the workday, team collaboration, getting QA input without leaving Slack

### Option C: Telegram Integration

1. In Zenflow, go to **Settings → Integrations → Telegram**
2. Click **"Connect to Telegram"**
3. Follow the bot link and click **"Start"**
4. Zenflow will verify the connection
5. Start messaging Zen in your Telegram chat

**Best for:** Working on the go, mobile-first workflows, personal use

---

## Step 4: Build Your Project Memory File

This is the single most important setup step. Zen has a memory system — files in your project's `memory/` directory that persist across conversations. Without a memory file, Zen starts fresh every time. With one, it knows your project before you type a word.

**How to create your memory file:**

1. In your Zenflow project, go to the **Files** section
2. Navigate to or create the `memory/` folder
3. Create a file called `_index.md`

This `_index.md` is Zen's entry point — it reads this file at the start of every conversation.

### Template: memory/_index.md

```markdown
# QA Project Memory: [Application Name]

Last updated: [date]

## Quick Reference

- **Application:** [Name of the app you're testing]
- **My Role:** QA Engineer (manual, transitioning to automation)
- **Project Phase:** [Alpha / Beta / Production / Maintenance]
- **Team Size:** [How many developers, PMs, designers]

## What I'm Testing

[2-3 sentences describing the application, its primary purpose, and its users]

## Tech Stack

- Frontend: [e.g., React SPA, deployed on Vercel]
- Backend: [e.g., Node.js/Express REST API]
- Database: [e.g., PostgreSQL]
- Mobile: [e.g., iOS native (Swift), Android native (Kotlin)]
- Key third-party services: [e.g., Stripe, Twilio, AWS S3]

## Test Environments

| Environment | URL | Purpose | Reset frequency |
|-------------|-----|---------|-----------------|
| Local | localhost:3000 | Dev testing | On demand |
| Staging | staging.myapp.com | Pre-release testing | Weekly |
| UAT | uat.myapp.com | Acceptance testing | Monthly |

## Current Sprint / Focus

[What features or areas are currently in scope for testing]

## Known Risk Areas

- [Area 1 and why it's risky]
- [Area 2 and why it's risky]
- [Area 3 and why it's risky]

## Testing Constraints

- [e.g., "Cannot run tests that send real emails — use test email addresses"]
- [e.g., "Staging database resets every Sunday at 2am"]
- [e.g., "Cannot test payment flows with real cards — use Stripe test cards only"]

## My Current Skills / Tools

- Currently using: [e.g., manual testing, Postman, basic SQL queries]
- Learning: [e.g., Playwright, API automation]
- Not yet using: [e.g., performance testing, security testing]

## Automation Strategy (evolving)

[Will update this as the series progresses]

## Related Files

- [Link to requirements doc]
- [Link to test case repository]
- [Link to bug tracker]
```

**Tips for a great memory file:**
- Keep it under 500 lines — Zen reads the whole thing at the start of every chat
- Update it weekly or when major things change
- Be honest about constraints and limitations
- Include context that would take you 5 minutes to explain to a new team member

### Creating Additional Memory Files

For larger projects, split context into separate memory files and reference them from `_index.md`:

```
memory/
  _index.md                  ← Zen reads this first
  application-overview.md    ← Detailed feature descriptions
  test-data.md               ← Test accounts, test cards, seeded data
  known-bugs.md              ← Active bugs that affect testing
  api-endpoints.md           ← Key API endpoints and their behavior
  release-notes/             ← Notes from past releases
```

In `_index.md`, reference these files:
```markdown
## Detailed Context Files

- [Application Overview](./application-overview.md) — full feature descriptions
- [Test Data](./test-data.md) — accounts, credentials, seeded data
- [Known Bugs](./known-bugs.md) — active bugs that may affect test results
```

---

## Step 5: Configure Your Zenflow Project Settings

Before diving into work, set a few project preferences:

### Labels / Tags

Create labels that match your testing categories. Suggested labels for QA projects:

- `bug` — defects to report
- `test-case` — test case creation tasks
- `automation` — automation tasks
- `coverage-gap` — identified gaps in coverage
- `regression` — regression testing tasks
- `blocked` — items waiting on something
- `sprint-N` — sprint-specific label

### Workflow Stages

Customize your task workflow to match your QA process. A good QA workflow:

```
To Do → In Progress → In Review → Done
```

Or for bug tracking:
```
To Do → Reproducing → Confirmed → Fixed → Verified → Closed
```

---

## Step 6: Your First 5 Prompts

These are the prompts every QA engineer should run when setting up a new Zenflow project. Run them in order — each one builds on the last.

### Prompt 1: Introduce Your Project

```
Hi Zen! I'm setting up a new QA project. Here's my project context:

[paste your memory/_index.md content]

Please confirm you've understood the context by:
1. Summarizing what application I'm testing in 2-3 sentences
2. Listing the 3 most important testing constraints I mentioned
3. Asking me 3 clarifying questions that would help you support me better
```

### Prompt 2: Generate a Testing Strategy Overview

```
Based on the project context you now have, please generate a high-level 
testing strategy overview for me. Include:

1. Recommended testing layers (unit, integration, E2E, API, performance)
2. Which layers are most critical for this type of application
3. Suggested test coverage priorities (what to test first, second, third)
4. A rough estimate of the test suite I should aim for (number of test cases per area)
5. Key risks I should address first

Keep this practical and actionable, not theoretical.
```

### Prompt 3: Audit Your Current Coverage

```
My current testing state:

- Test cases we have: [number or "none documented"]
- Areas covered: [list them]
- Areas NOT covered: [list them or "unknown"]
- Last time tests were run: [date]
- Current pass rate: [percentage or "unknown"]

What are the biggest gaps in my coverage? 
What should I address this week to most reduce our risk?
```

### Prompt 4: Set Up Your First Task

```
I want to create my first real QA task in Zenflow. 
Help me write a clear, actionable task for: 
[describe something you need to do this week — e.g., "write test cases for the new user registration flow"]

Please give me:
1. A clear task title
2. A description with acceptance criteria (when is this task "done"?)
3. Suggested subtasks to break the work down
4. Estimated effort (in hours) for someone at my skill level
```

### Prompt 5: Build Your Testing Vocabulary

```
I'll be working with you on QA tasks regularly. To communicate better, 
please create a quick-reference glossary of the 20 most important QA terms 
that relate to my specific application type ([web/mobile/API]).

For each term:
- Plain English definition
- Example specific to my application
- Why it matters for my testing

I want to be able to use these terms correctly when writing bug reports 
and communicating with developers.
```

---

## Step 7: Set Up a Recurring Check-In Automation

One of Zenflow's most powerful features for QA engineers is task automations — scheduled reminders that help you stay consistent.

**Setting up a weekly QA review automation:**

1. In your Zenflow project, go to **Automations**
2. Click **"New Automation"**
3. Configure:
   - **Name:** Weekly QA Coverage Review
   - **Schedule:** Every Monday at 9:00 AM
   - **Task created:** "Weekly QA check-in: review coverage, open bugs, and this week's testing priorities"
4. Save the automation

Now every Monday morning, Zenflow creates a task reminding you to:
- Review what was tested last week
- Check for new requirements or changes
- Update your coverage map
- Prioritize this week's testing work

You can also create automations for:
- Pre-release test execution reminders (trigger on release schedule)
- Monthly test case review (do test cases still reflect current behavior?)
- Weekly bug triage review

---

## Troubleshooting Common Setup Issues

### "Zen doesn't seem to remember context between conversations"

**Fix:** Make sure your memory file is saved in the `memory/` directory of your Zenflow project, and that it's named `_index.md`. Zen reads this automatically at the start of each conversation.

### "Zen gave me generic answers even after I provided context"

**Fix:** Your context prompt might be too long or unstructured. Try breaking your context into a numbered list and putting the most important constraints at the top. Also ensure you're asking specific questions, not open-ended ones.

### "The Slack bot isn't responding"

**Fix:** Check that the Zen bot has been invited to the channel (type `/invite @Zen`). If using a DM, ensure you're messaging the Zen app, not a user.

### "I created a task in Zenflow but Zen can't see it"

**Fix:** When asking Zen about tasks, reference the task ID or title explicitly: "Can you summarize task #47 — the login regression test suite?"

---

## Setup Checklist

Before moving on, confirm:

- [ ] Zenflow account created
- [ ] Project created with a clear name and description
- [ ] Zen connected via at least one channel (web / Slack / Telegram)
- [ ] `memory/_index.md` created with project context
- [ ] First 5 prompts completed
- [ ] Task labels configured
- [ ] At least one automation set up
- [ ] Zen is giving you useful, context-aware responses

---

## What's Next

Your command center is ready. Now take 15 minutes to honestly assess where you are as a QA engineer — so you know exactly where to focus your energy in the beginner section.

**→ Continue to [04 — Self-Assessment](./04-self-assessment.md)**
