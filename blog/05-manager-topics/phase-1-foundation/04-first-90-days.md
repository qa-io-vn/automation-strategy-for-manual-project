> **Series:** From Senior Tester to Expert Test Manager — The AI Era Guide | **Phase 1: Foundation** | [← 03 Daily Practices](./03-daily-practices.md) | [Phase 2: People Leadership →](../phase-2-people-leadership/)

# The First 90 Days: Your New Manager Playbook

The first 90 days as a Test Manager (or as a senior tester *preparing* for the role) will define how your team sees you for the next two years. Make bold moves too early and you'll build resentment before trust. Wait too long and you'll be seen as passive. The goal is not to impress anyone — it's to understand deeply, then act decisively.

This is your day-by-day, week-by-week playbook. It includes the exact Zen prompts to use at each phase and the honest advice most management books leave out.

---

## The Core Rule: Sequence Matters More Than Speed

Most new managers make one of two mistakes:

1. **Too fast:** Announce changes in week 2 based on gut instinct from week 1. Burns trust and builds resistance.
2. **Too slow:** Listen for 60 days but never act. The team loses confidence in your ability to lead.

The right sequence is: **observe → understand → hypothesize → act → measure**. This playbook maps that sequence to 90 days.

---

## Days 1–30: Listen, Map, and Build Context

Your only job in month one is to understand. Not to fix. Not to impress. Not to show your technical depth. **To understand.**

### What to Map in the First 30 Days

**The Team:**
- Who is on the team (names, roles, tenure, what they care about)
- Who are the informal leaders (the people others go to for advice)
- Team dynamics: alliances, tensions, collaboration patterns
- Current skills: automation capability, domain knowledge, AI tool adoption

**The Processes:**
- How does QA integrate with development? Shift-left or still mostly end-stage?
- What's the bug lifecycle? How are severity/priority decisions made?
- What are the release criteria? Who signs off?
- What automation exists? Coverage, health, maintenance burden?

**The Stakeholders:**
- Who are the 5–7 people whose opinion of QA shapes your team's budget and influence?
- What do they think QA does well? What do they think QA fails at?
- What do they actually care about — is it speed, stability, compliance, something else?

**The History:**
- What did the previous QA Manager do that worked well?
- What did they do that didn't work and why?
- What promises have been made to the team that haven't been kept?
- What are the sacred cows — processes nobody questions even if they're broken?

### AI-Assisted Stakeholder Mapping

After your first round of stakeholder conversations, use Zen to help you synthesize and strategize:

```
Zen, I've just completed my first round of stakeholder conversations as a
new Test Manager. Here's what I learned from each person:

- [Stakeholder 1, VP Engineering]: Cares most about release velocity.
  Frustrated that QA is "always the bottleneck." Thinks we have too many
  manual regression tests. Supportive of automation investment.

- [Stakeholder 2, Head of Product]: Wants better defect metrics reported
  in business terms. Doesn't understand the difference between a Sev-1 and
  Sev-2. Likes weekly status updates.

- [Stakeholder 3, Lead Developer]: Thinks QA should be embedded in
  feature teams, not a centralized team. Skeptical of current test
  coverage. Wants pairing opportunities.

Based on this, help me:
1. Identify who my most important relationship to invest in is and why
2. Identify the biggest misconception about QA I need to address
3. Suggest 2 low-cost, high-signal actions I could take in the next 2 weeks
   to build trust with each stakeholder
```

### Building Your Team Context File in Zen Memory

Zen is most powerful when it has context. In your first 30 days, build a team context document that you can reference in every future prompt. Store it in Zenflow memory or as a note you paste into Zen when needed:

```
Zen, I want to build a running context file about my team that I'll use
to improve the quality of our future conversations. Based on what I share,
help me structure this into a living document I can update monthly.

Team context — [Month 1]:
- Team size: 6 QA engineers (2 senior, 3 mid, 1 junior)
- Stack: React frontend, Java backend, AWS infrastructure
- Current automation: 340 Playwright E2E tests, ~60% passing reliably
- Biggest current pain: flaky tests in staging environment
- Team mood: cautiously optimistic — previous manager left abruptly
- Key person: [Name], 5 years on the team, knows all the tribal knowledge
- Upcoming pressure: major product launch in Q3

Save this. I'll update it as I learn more.
```

### What to Ask in Your First 1-on-1s

For each team member, run this prompt to prepare:

```
Zen, I'm meeting with [Name] for the first time as their new manager.
They are a [mid-level QA engineer] who has been on this team for [2 years].
I want this conversation to help me understand: what motivates them, where
they want to grow, and what they think is most broken about how QA works.

Give me 5 open questions that will generate honest answers, not polished
answers. Include at least one question that invites them to be critical.
```

**The 4 questions that never fail in first-30-days 1-on-1s:**
1. "What's something this team does really well that I should make sure not to break?"
2. "What's one thing you wish the previous QA Manager had done differently?"
3. "If you could change one thing about how QA works here, what would it be?"
4. "What do you need from me to do your best work?"

### What NOT to Do in Days 1–30

- Do not announce any process changes, even if they seem obvious
- Do not criticize what the previous manager did, even if invited to
- Do not commit to anything you can't deliver in 30 days
- Do not hold a team retrospective about problems — you don't have enough context yet
- Do not push your own automation framework preference

---

## Days 31–60: Identify Improvements and Draft Strategy

By day 30, you should have completed your stakeholder map, had 1-on-1s with every team member, attended all the recurring meetings, and built a clear picture of the landscape. Now you can start to think.

### Identify 2–3 High-Impact Improvements

Not 10. Not 5. Two or three. The discipline of constraint forces prioritization.

Use Zen to help you choose:

```
Zen, I've completed my first 30 days. Here's what I've learned:

Problems observed:
1. [Automation test suite has 40% flaky tests, slowing down CI by 35 min per run]
2. [No standard for how bugs are classified by severity — every dev/QA has a different view]
3. [QA is brought in after feature design is finalized — too late to influence testability]
4. [Team has no visibility into what product features are coming 2+ sprints ahead]
5. [One senior engineer is quietly disengaged and has been doing the same work for 18 months]

My constraints: team of 6, 2-week sprints, major launch in 5 months.

Help me rank these by impact-to-effort ratio and identify which 2–3 I should
focus on first. For each recommended focus, give me: the expected outcome,
the likely resistance, and one quick win I could achieve in 30 days.
```

### Draft Your First QA Strategy Proposal

This doesn't need to be a 20-page document. A focused 2-page strategy proposal, well-written and data-backed, carries more weight. Use Zen to draft it:

```
Zen, help me write a QA strategy proposal for the next quarter. Context:

Company: [B2B SaaS, ~200 employees, Series B]
Team: 6 QA engineers, mixed automation/manual capability
Product stage: Scaling an existing product, launching a new mobile app
Key stakeholders: VP Eng (velocity-focused), Head of Product (stability-focused)
Current state: [brief honest assessment]
Top 3 problems to solve: [from your earlier analysis]
Available resources: [current tooling, potential budget]

Structure the proposal as:
1. Current State (2 paragraphs — honest, not defensive)
2. Target State (what does "excellent QA" look like here in 6 months?)
3. Initiatives (3 prioritized, with owner, timeline, success metric)
4. Risks and Dependencies
5. Ask (what do I need from leadership to make this happen?)

Keep total length under 2 pages. Use clear business language.
```

### Getting Buy-In for Your Strategy

Before you present the strategy, pressure-test it:

```
Zen, I'm about to present my QA strategy proposal to the VP of Engineering
and Head of Product. Here is the proposal: [paste].

Play devil's advocate. What are the 3 strongest objections they might raise?
For each objection, give me a confident, data-backed response. Then tell me
what you think is the weakest section of this proposal and how to strengthen it.
```

---

## Days 61–90: Execute, Measure, and Build Your Dashboard

With buy-in secured (or at least directional approval), you move into execution mode. The key discipline here is *measurement from day one*. If you don't establish baselines now, you won't be able to show progress in month 6.

### Execution Priorities

**Quick wins first.** Find something your team can accomplish in 2 weeks that is visible to stakeholders. This builds momentum and demonstrates that you translate strategy into action.

Typical quick wins:
- Reducing flaky tests by 20% through one-time triage and disable/fix session
- Establishing a severity definition document everyone signs off on
- Adding QA to the sprint planning meeting for the next feature cycle
- Creating a weekly QA status update that stakeholders actually find useful

**Use Zen to track execution:**

```
Zen, here's my 90-day initiative status update:

Initiative 1: [Flaky test reduction]
- Goal: Reduce flaky test failure rate from 40% to <15%
- Status: [Week 4 of 8] — Current rate: 28%
- Blockers: [one engineer stuck on timeout issue in checkout tests]
- Next action: [pair programming session Thursday]

Initiative 2: [Severity definition standardization]
- Goal: Published, agreed severity guide used by all teams
- Status: [Draft written, waiting for review from dev lead]
- Blockers: [dev lead has been in crunch mode]
- Next action: [send async review request with 5-day deadline]

Based on this, are we on track? What risks do I need to flag now?
```

### Building Your QA Dashboard

By day 90, you should have a dashboard that tracks the metrics that matter. Don't try to measure everything — choose 5–7 metrics that tell the quality story for your team.

**Recommended core metrics:**

| Metric | What It Tells You | Target Trend |
|---|---|---|
| Escaped defect rate | How much gets through to users | Decreasing |
| Defects found pre-release by severity | Effectiveness of testing | Stable/increasing Sev-1/2 catch rate |
| Automation pass rate (non-flaky) | Health of your test suite | > 95% |
| Test cycle time | Speed of QA validation | Decreasing |
| Automation coverage % | Breadth of automated testing | Increasing at sustainable pace |
| Mean time to bug resolution | Dev-QA collaboration efficiency | Decreasing |

```
Zen, I'm setting up my QA metrics dashboard. I want to track 6 metrics
that best represent quality health for our team. Here are the candidates:
[list yours]. Help me: (1) prioritize to a final 6, (2) define what "good"
looks like as a target for each, (3) identify what tool or data source I
would use to collect each metric, (4) suggest how to visualize them for
a mixed technical/non-technical audience.
```

---

## 30/60/90 Milestone Checklist

| Milestone | Day 30 | Day 60 | Day 90 |
|---|---|---|---|
| **People** | Met 1-on-1 with every team member | Established regular 1-on-1 cadence | Completed first team performance pulse |
| | Mapped informal team dynamics | Identified growth priorities per person | Delivered one meaningful coaching moment |
| **Stakeholders** | Completed stakeholder mapping | Presented QA strategy | Received explicit buy-in from 2+ stakeholders |
| | Identified top 2 relationships to build | Delivered first stakeholder status update | Built one new cross-team relationship |
| **Process** | Documented current QA process as-is | Identified 2-3 high-impact improvements | First improvement initiative underway |
| | Understood release criteria and bug lifecycle | Published severity definition (if needed) | First quick win delivered and communicated |
| **Metrics** | Identified existing metrics (if any) | Defined target metrics and baselines | Dashboard live and reviewed by stakeholder |
| **AI / Tooling** | Audited current tools | Evaluated one new tool or process | Made at least one tooling recommendation |
| | Built team context file in Zen | Using Zen daily for management tasks | Team using at least one AI-assisted workflow |
| **Self** | Completed personal skills assessment | Identified top 2 learning priorities | Completed one deliberate skill-building activity |

---

## How to Fail Gracefully and Course-Correct

You will make mistakes in the first 90 days. Everyone does. What matters is how quickly you recognize them and what you do next.

**Signs you're off-track:**
- People are solving problems without you in the loop (they don't trust your judgment yet)
- Stakeholders are going around you to talk to your team directly
- Your team seems politely compliant but not engaged
- Your strategy proposal was received with silence, not questions

**When you recognize a mistake, use Zen to diagnose:**

```
Zen, I think I made a mistake in my first 60 days. Here's what happened:
[describe the situation]. The reaction from [team/stakeholder] was: [describe].

Help me: (1) understand what likely went wrong, (2) assess the damage honestly,
(3) identify a concrete recovery action I can take this week,
(4) think through what I should have done differently.

Don't soften this. I need honest feedback.
```

**The recovery formula:**
1. Acknowledge specifically (not generally) what went wrong
2. Ask what the person needs going forward
3. Change the behavior — not just apologize
4. Give it time; trust rebuilds slowly

---

## Specific Zen Prompts for Each Phase

### Phase 1 Prompts (Days 1–30)

```
Zen, I just finished my first week. I want to capture everything I've
learned before I forget it. I'll share my notes and you help me organize
them into: People Observations, Process Observations, Stakeholder Dynamics,
and Open Questions I still need to answer.
```

```
Zen, I'm three weeks in and I notice I'm feeling pressure to "do something."
Is this urge healthy or am I moving too fast? What questions should I be
asking myself before I take my first visible action?
```

### Phase 2 Prompts (Days 31–60)

```
Zen, based on everything I've shared with you about my team's context,
what do you think I'm missing or under-weighting in my analysis?
What blind spots should I be concerned about?
```

```
Zen, I want to propose shifting to left-testing — getting QA involved in
sprint planning and design reviews, not just implementation. This will
require changing developer habits and getting PM buy-in. Help me anticipate
resistance and design a pilot that makes it easy to say yes.
```

### Phase 3 Prompts (Days 61–90)

```
Zen, it's day 75. Here's where I am against my 90-day goals: [paste
milestone checklist with status]. What am I at risk of not completing?
What should I explicitly let go of or defer? What one thing, if I focused
on it now, would have the most impact?
```

```
Zen, I want to write a brief "state of QA" summary for my manager at the
90-day mark. It should cover: what I found, what I changed, what's in
progress, what I learned about myself. Help me make it honest, confident,
and forward-looking — not a list of achievements but a reflection on
the journey and what comes next.
```

---

## Checkpoint: First 90 Days Action Items

- [ ] Create your stakeholder map — minimum 5 people — in the first 2 weeks
- [ ] Complete 1-on-1s with every team member by day 15
- [ ] Build your team context file in Zen by day 20
- [ ] Choose your 2–3 improvement priorities by day 35
- [ ] Draft and present your QA strategy proposal by day 50
- [ ] Define your 6 core metrics and baselines by day 60
- [ ] Deliver at least one visible quick win by day 75
- [ ] Write your 90-day retrospective and share it with your manager

The first 90 days are not about being perfect. They're about demonstrating that you listen before you act, that you think before you speak, and that when you do act, you follow through. Do that, and you'll have earned the trust to lead effectively for the months that follow.

---

*Next: [Phase 2 — People Leadership](../phase-2-people-leadership/)*
