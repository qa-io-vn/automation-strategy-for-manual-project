> **Series:** From Senior Tester to Expert Test Manager — The AI Era Guide | **Phase 1: Foundation** | [← 02 Skills Inventory](./02-skills-inventory.md) | [04 First 90 Days →](./04-first-90-days.md)

# Daily Practices: The Rhythm That Makes Everything Else Work

Great Test Managers aren't defined by what they do in a crisis. They're defined by what they do on a boring Tuesday. Their daily, weekly, and monthly rhythms are what make them effective when it matters — because they've already built the habits, the context, and the relationships that allow them to respond with clarity rather than scramble with confusion.

This guide gives you the exact operating system: morning rituals, daily Zen prompts, weekly practices, and monthly reviews that will transform you from reactive to strategic in 90 days or less.

---

## The Morning Ritual: 15 Minutes That Set the Tone

Before you open Slack. Before you check your email. Before anyone drags your attention somewhere else — spend 15 minutes anchoring your day.

### Minute 0–5: AI-Assisted Day Planning

Open Zen and run this prompt every morning. Customize it with your current context (sprint phase, ongoing issues, upcoming deadlines):

```
Zen, help me plan my day as a Test Manager. Here's my context:

- Sprint status: [Day X of Y, sprint goal is Z]
- Open escalations or blockers: [any?]
- Key meetings today: [list]
- Ongoing team situations: [e.g., "one engineer is struggling with a new framework"]
- My energy level: [High / Medium / Low]

Give me:
1. My single most important priority today
2. One person on my team I should deliberately check in with and why
3. One thing I should NOT spend time on today
4. A 2-sentence status update I could send to my manager if asked
```

This 5-minute exercise does something powerful: it forces you to articulate your context before reacting to other people's contexts. You stop your inbox from writing your priorities.

### Minute 5–10: Team Status Scan

Pull up your team's activity: CI dashboards, sprint board, Slack threads. You're not going to act on everything — you're pattern-matching. Ask:

- Is anything blocked that wasn't blocked yesterday?
- Is anyone unusually quiet or unusually behind?
- Are there any automated alerts firing that nobody has responded to?

Use Zen to contextualize what you see:

```
Zen, our CI has been showing a 15% increase in flaky test failures over the
last 3 days. The tests that are flaking are primarily in the checkout flow.
What are the most likely causes, and what's the fastest way to diagnose this?
```

### Minute 10–15: Priority Check and Intention Setting

Write down (or tell Zen) your single output for the day. Not a task list — one thing that, if accomplished, makes the day a success. Then write one intention about *how* you want to show up. Example:

- **Output:** Complete the first draft of the QA capacity proposal for next quarter
- **Intention:** Be patient in conversations — I'm behind on sleep and might snap; notice it before it happens

---

## 5 Daily Zen Prompts for the AI-Era Test Manager

These are your core power tools. Use them consistently and they become habits. Together, they cover the full manager operating system: people, priorities, communication, risk, and self-awareness.

### Prompt 1: Team Health Check

Use this at the start of each week or whenever you sense something is off:

```
Zen, I want to assess the health of my QA team. Here's what I've observed
this week:

- [Name 1]: [behavior/output observations]
- [Name 2]: [behavior/output observations]
- [Name 3]: [behavior/output observations]

Identify any patterns that suggest: (1) disengagement risk, (2) burnout
signals, (3) someone ready for a stretch opportunity. Be direct.
```

### Prompt 2: Priority Triage

When your inbox, Slack, and task board are competing for attention:

```
Zen, I have the following competing demands on my time today:

1. [Task A] — urgency: high, importance: medium
2. [Task B] — urgency: low, importance: high
3. [Task C] — urgency: high, importance: high (estimated 3 hrs)
4. [Task D] — someone else's urgent request, 30 min

My constraints: 4 hours of available deep work time, two fixed meetings.
Help me sequence these using the Eisenhower matrix, and suggest anything
I should defer or delegate entirely.
```

### Prompt 3: Communications Drafting

Any important message — stakeholder update, performance feedback, escalation notice, all-hands QA update — start with Zen:

```
Zen, draft a message for [audience: e.g., VP of Product] about [topic:
e.g., the decision to delay the mobile release by one sprint].

Tone: Direct but not alarming. Factual.
Key points to include:
- The specific risk that triggered the delay recommendation
- What we're doing to resolve it
- New target date and confidence level

What NOT to include: blame, technical jargon, apologies for doing our job.

Give me a first draft, then suggest what I might be missing.
```

### Prompt 4: Risk Identification

Run this every Thursday afternoon before your end-of-sprint activities:

```
Zen, it's end of sprint. Here's our current state:

- Sprint goal: [goal]
- Completed: [items done]
- Still in progress: [items + % complete]
- Known bugs: [count and severity]
- Automation coverage change this sprint: [+/- %]
- Upcoming release date: [date]

Identify the top 3 risks to quality for the next sprint or release. For
each risk, give me: likelihood (H/M/L), impact (H/M/L), and one mitigation
action I could take this week.
```

### Prompt 5: Decision Journaling

End each day with 2 minutes of decision logging. This builds self-awareness over time and creates a record you can reflect on:

```
Zen, I want to journal today's key decisions:

1. Decision I made today: [describe]
   - Information I had: [what you knew]
   - Information I didn't have: [what you didn't]
   - Outcome I expected: [what]

2. Decision I avoided or postponed: [describe]
   - Why: [honest reason]

Ask me 2 questions that challenge my assumptions about decision #1.
```

---

## Weekly Practices: The Foundation of Effective Management

### Monday: 1-on-1 Preparation (15 min per person)

Never go into a 1-on-1 unprepared. Use Zen to prep:

```
Zen, I have a 1-on-1 with [Name] tomorrow. Here's context about them:
- Role: [Senior QA Engineer]
- Last conversation topics: [automation backlog, they mentioned career growth]
- Recent work: [completed the login test suite, helped with API review]
- Current vibe: [seemed distracted in standup this week]

What are 3 good opening questions? What should I listen for?
What's one thing I could offer — recognition, challenge, support — that fits
where they are right now?
```

**One-on-one structure that works:**
1. Their agenda first (what's on their mind?)
2. Your observation (one specific positive, one growth area)
3. Forward-looking (what do they need this week to be successful?)
4. Career beat (one question per month about their longer-term growth)

### Tuesday/Wednesday: Deep Work Block

Block 90 minutes, mid-week, for strategic work: QA strategy, proposals, dashboard reviews, research on tools or techniques. This is not for email. Protect it.

Use Zen at the start of this block:

```
Zen, I have 90 minutes of deep work. My strategic goal this week is [goal].
Help me break this into a focused session plan with clear deliverables.
Suggest what I should produce by the end of this block.
```

### Thursday: Sprint Review Analysis

Before the sprint review meeting, run this:

```
Zen, help me prepare my section of the sprint review. Here's our QA data:

- Test cases executed: [N]
- Pass rate: [%]
- Bugs found pre-release: [N] by severity
- Bugs escaped to production: [N]
- Automation coverage: [%], delta from last sprint: [+/-]
- Test cycle time: [avg days from dev-done to QA sign-off]

Convert this into 3 talking points for non-technical stakeholders. Frame
everything in terms of risk and user impact, not QA process metrics.
```

### Friday: Self-Review (10 min)

The most neglected practice. Managers review their teams constantly but rarely review themselves. Every Friday:

```
Zen, here's my week as a manager:
- What I planned to do: [list]
- What I actually did: [list]
- Interactions I'm proud of: [note one]
- Interaction I'd do differently: [note one]
- Energy level by day: [Mon: H, Tue: M, Wed: L, Thu: H, Fri: M]

What patterns do you see? What should I do differently next week?
```

---

## Monthly Practices: The Strategic Layer

### QA Health Report Generation

Once a month, generate a QA health report for your leadership. Use Zen to synthesize:

```
Zen, I need to write the monthly QA health report. Here is the raw data
from this month:

[Paste: bug counts by severity, automation coverage trend, CI metrics,
team capacity, escaped defects, test debt backlog items]

Generate a report structured as:
1. Executive Summary (3 sentences max)
2. Quality Trend (better / stable / worse, with evidence)
3. Top 3 risks entering next month
4. Team highlights
5. Recommended actions

Write it for an audience that includes VPs who don't read QA reports carefully
but will skim for red flags.
```

### Team Growth Review

Once a month, step back from daily people management and look at each team member's trajectory:

```
Zen, I want to review the growth trajectory of my team. For each person,
I'll share what I've observed this month:

- [Name 1]: [observations: skills demonstrated, gaps, engagement level]
- [Name 2]: [observations]
- [Name 3]: [observations]

For each person, tell me: (1) what their development priority should be next
month, (2) whether I'm investing enough coaching time with them, (3) any
early warning signs I might be ignoring.
```

### Stakeholder Alignment

Once a month, deliberately check whether you and your key stakeholders are still aligned on priorities, expectations, and concerns:

```
Zen, I'm preparing for my monthly alignment meeting with [stakeholder/VP].
Last month we agreed on: [commitments].
Since then: [what happened — what delivered, what changed, what surprised].

Help me structure a 15-minute alignment conversation that: (1) acknowledges
what changed and why, (2) proposes any adjustments to our shared plan,
(3) surfaces one concern I have that needs their input.

What should I NOT say in this meeting?
```

---

## The Manager's Learning Hour

Block one hour per week — non-negotiable, calendar-protected — for your own development. Rotate focus monthly:

| Month | Focus | What to do in Learning Hour |
|---|---|---|
| 1 | AI tools exploration | Trial one AI testing tool, document findings |
| 2 | Leadership reading | Read one chapter of a management book, discuss with Zen |
| 3 | Peer benchmarking | Talk to a peer manager at another company |
| 4 | Retrospective | Review the past 90 days of your practice journal |
| Repeat | — | — |

**Zen prompt for Learning Hour:**

```
Zen, I have one hour to invest in my growth as a Test Manager. This month's
focus is [topic]. I know: [what you already know]. I'm weakest at: [honest gap].

Design a 1-hour self-directed learning session for me. Include: what to read
or watch (max 30 min), what to practice or create (30 min), and 3 questions
to journal on afterward.
```

---

## What NOT to Automate: Human Moments That Must Stay Human

AI can draft, summarize, analyze, and suggest. It cannot replace the moments that actually build a team.

**Never automate or shortcut these:**

- **The genuine check-in.** "How are you actually doing?" — this must come from you, in person or on video, with real attention.
- **Recognition in the moment.** When someone does something excellent, say it immediately, specifically, and in front of others when appropriate. Don't ask Zen to draft your recognition.
- **Difficult conversations.** You can prepare with Zen, but the conversation itself must be human. Do not read from a script.
- **Hiring final decisions.** AI can help you evaluate candidates, but the hiring decision carries your professional judgment and responsibility.
- **Conflict resolution.** When two team members are in conflict, they need to feel heard by a human — not handed a summary.
- **Celebrating failures.** When someone on your team takes a risk and it doesn't work out, how you respond to that shapes the psychological safety of your entire team. This cannot be delegated.

---

## Weekly Practice Tracker Template

Use this to hold yourself accountable. Review it every Friday with your Zen self-review prompt.

| Practice | Mon | Tue | Wed | Thu | Fri | Done This Week? |
|---|---|---|---|---|---|---|
| Morning ritual (15 min) | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Team health check prompt | ☐ | | | | | |
| 1-on-1 prep with Zen | ☐ | ☐ | ☐ | | | |
| Priority triage prompt | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Risk identification (Thursdays) | | | | ☐ | | |
| Decision journaling | ☐ | ☐ | ☐ | ☐ | ☐ | |
| Friday self-review | | | | | ☐ | |
| Deep work block (90 min) | | ☐ | ☐ | | | |
| Learning Hour | ☐ | | | | | |

**Monthly:**
- [ ] QA Health Report generated
- [ ] Team growth review completed
- [ ] Stakeholder alignment meeting held

---

## Checkpoint: Practice System Action Items

- [ ] Set your morning ritual for tomorrow — run the day planning prompt before opening Slack
- [ ] Add the 5 daily Zen prompts to a document you can access quickly
- [ ] Block your weekly deep work slot and Learning Hour in your calendar
- [ ] Run the Friday self-review today if it's end of week — don't wait
- [ ] Identify one practice from this list you *already do* and one you've been avoiding

The difference between a manager who burns out and one who thrives isn't intelligence or experience — it's rhythm. Build the rhythm before you need it.

---

*Next: [04 — The First 90 Days: Your New Manager Playbook](./04-first-90-days.md)*
