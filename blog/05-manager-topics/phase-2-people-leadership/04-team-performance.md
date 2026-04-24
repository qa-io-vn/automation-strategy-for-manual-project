> **Series:** From Senior Tester to Expert Test Manager — The AI Era Guide | **Phase 2: People Leadership** | [← 03 Coaching and Growth](./03-coaching-and-growth.md) | [Phase 5: Organizational Impact →](../phase-5-organizational-impact/)

# Managing Team Performance: Standards, Conversations, and Accountability

Performance management is where most managers flinch. The good intentions are there. The follow-through isn't. Either they avoid hard conversations until a situation has festered for months, or they address performance problems clumsily and damage the relationship in the process.

This guide is practical. It covers what to measure, how to set expectations clearly in the AI era, how to use Zen to improve feedback quality, and how to handle underperformance and the more challenging retention of your best people.

---

## Setting Clear Performance Expectations in the AI Era

You cannot hold people accountable to expectations they don't know they have. Before you assess performance, you must articulate performance — specifically, in writing, with measurable indicators.

In 2025, performance expectations for QA engineers have three dimensions:

### Dimension 1: Output (What they produce)

- Test coverage completeness (does their work produce quality coverage?)
- Bug catch rate (pre-release vs. post-release defects attributable to their features)
- Automation contribution (scripts written, maintained, coverage added)
- Cycle time (how quickly do they move from feature-ready to test-complete?)

### Dimension 2: Process (How they work)

- Communication quality (do they surface risks proactively?)
- Collaboration effectiveness (are dev, PM, and design partners satisfied?)
- Documentation discipline (are test plans, decisions, and coverage gaps documented?)
- Reliability (do they meet commitments? Do they ask for help before missing them?)

### Dimension 3: AI Adoption (How they evolve)

- Are they actively using AI tools in their workflow?
- Are they reviewing AI output critically?
- Are they contributing to the team's shared prompt library or AI practices?
- Are they building their AI fluency over time?

**Zen prompt to draft expectations for a role:**

```
Zen, help me write clear performance expectations for a Senior QA Engineer
on my team. I want to define what "meeting expectations," "exceeding
expectations," and "below expectations" looks like across three dimensions:
Output, Process, and AI Adoption.

Our team context: [paste team context].
The engineer's specific focus: [e.g., API testing + performance test suite].

For each dimension, write 3–5 observable indicators. These should be
behavioral and observable — not personality traits. Include examples
of what each performance level looks like in practice.
```

---

## Performance Review Framework for QA Teams

**Review cadence that works:**

| Review Type | Frequency | Duration | Purpose |
|---|---|---|---|
| Weekly check-in | Weekly (in 1-on-1) | 5 min | Pulse check, current blockers |
| Monthly growth conversation | Monthly | 30 min | Progress vs. IDP, what's working |
| Formal mid-year review | Twice/year | 60 min | Holistic assessment, calibration |
| Annual review | Yearly | 90 min | Compensation, promotion, year retrospective |

**The mid-year and annual review structure:**

1. **Self-assessment first.** Ask the engineer to rate themselves across the three dimensions *before* you share your view. This reveals how aligned your perceptions are.
2. **Your assessment.** Based on observed behaviors and outcomes — not personality or likability.
3. **Calibration discussion.** Where you see it the same. Where you see it differently. Why.
4. **Forward-looking.** What changes in the next period? What support do they need?
5. **Development commitment.** One specific commitment from you and one from them.

**Zen prompt for review preparation:**

```
Zen, I'm preparing for a mid-year performance review with a QA Engineer.
Here is my raw evidence collected over the last 6 months:

Output observations: [paste]
Process observations: [paste]
AI adoption observations: [paste]
Notable incidents (positive and negative): [paste]
Their self-assessment: [paste if you have it]

Help me: (1) identify themes and patterns in this evidence,
(2) distinguish between observations and interpretations,
(3) draft an assessment that is evidence-based, specific, and actionable,
(4) anticipate how they might react and how I should handle each scenario.
```

---

## Using Zen to Gather 360-Degree Feedback Patterns

360 feedback is valuable but time-consuming if done manually. Use Zen to synthesize feedback from multiple sources:

```
Zen, I've collected informal feedback about a team member from 4 colleagues.
Here are their responses to my question "What is [name]'s biggest strength and
one area where they could have more impact?" (anonymized):

Feedback 1: [paste]
Feedback 2: [paste]
Feedback 3: [paste]
Feedback 4: [paste]

Synthesize this into: (1) clear themes that appear across multiple sources,
(2) potential contradictions and what might explain them,
(3) what I should raise in the performance conversation vs. what I should hold
(too thin, potentially biased, or not actionable), (4) the 2–3 most important
observations that should shape my assessment.
```

**Important guardrail:** Never share specific feedback that would identify the source. Synthesize themes, not quotes.

---

## The Underperformance Conversation: When, What to Say, How to Document

The most common mistake: waiting too long. By the time a manager formally addresses underperformance, the problem has usually been visible for 3–6 months. The team has noticed. Morale has started to erode. And the engineer is confused because nobody told them.

**When to have the conversation:**
- Two or more consecutive sprints of missed commitments without a clear external cause
- Feedback from teammates or stakeholders that a pattern is forming
- You've noticed a gap and raised it informally but nothing has changed

**The conversation structure:**

*Opening:* "I want to talk about something I've been observing. I want to be direct because I think you deserve that."

*Observation (not interpretation):* "Over the last three sprints, I've noticed [specific behavior or pattern]. For example, [specific instance]."

*Impact:* "The impact of this is [specific consequence for the team or product]."

*Question:* "I want to understand — is there something going on that's contributing to this that I don't know about?"

[Listen fully before responding]

*Expectation:* "Going forward, here's what I need to see: [specific, observable change]. I want to support you in making that change. Here's how I can help."

*Next check-in:* "Let's plan to revisit this in [2–4 weeks] to see how things are tracking."

**What to document after the conversation:**

```
Date: [date]
Participants: Manager, Engineer
Topic: Performance concern

Observations discussed:
- [specific behavior/pattern #1]
- [specific behavior/pattern #2]

Impact stated:
- [team/product impact]

Engineer's response:
- [what they said — their perspective, any context they provided]

Expectations agreed:
- [specific change expected]
- [timeline]

Manager support offered:
- [what you committed to]

Follow-up date: [date]
```

Keep this in a private document. If the situation escalates to an HR process, you need a clear timeline.

---

## PIP Process: When Appropriate and How to Run It Humanely

A Performance Improvement Plan (PIP) is a formal, structured 30–90 day process with documented goals, support, and consequences. It is not a punishment — or at least, it shouldn't be.

**When a PIP is appropriate:**
- Multiple documented informal conversations haven't produced sustainable change
- The performance gap is significant enough to affect the team or product
- HR has been consulted and agrees it's the right path

**When a PIP is NOT appropriate:**
- As a first response to a performance issue
- As a backdoor to termination that you've already decided on (this is unethical and often legally risky)
- For a skills gap that could be addressed with training or reassignment

**A humane PIP has:**
1. **Clear, measurable goals** — "Improve automation coverage of API tests from 40% to 70% within 60 days"
2. **Weekly check-ins with documentation**
3. **Named support** — a mentor, pair programming sessions, specific training
4. **Honest framing** — "This is a real risk to your continued employment here. I want to help you succeed."
5. **Defined success** — what does completing the PIP mean? What happens next?

**Use Zen to draft PIP goals:**

```
Zen, I'm designing a PIP for a QA Engineer whose performance has been below
expectations for 3 months. The specific gaps are:

1. [Gap 1 — e.g., consistently incomplete test coverage on assigned features]
2. [Gap 2 — e.g., not communicating blockers until after deadlines are missed]
3. [Gap 3 — e.g., automation scripts have a high defect rate after merge]

Help me write: (1) 3 measurable improvement goals tied to each gap,
(2) weekly check-in criteria for each goal, (3) what success looks like at
30 and 60 days, (4) support commitments from my side. Keep it concrete and
non-punitive in tone.
```

---

## High-Performer Retention: What Top QA Engineers Want

Your best people are always optionable. They know they are. If you're not thinking about what they want, someone else is.

Research and experience consistently show that top performers in technical roles leave for four reasons, usually in this order:

1. **Manager relationship** — They don't feel seen, developed, or trusted
2. **Growth stagnation** — They've hit a ceiling and don't see the next level
3. **Mission disconnect** — They don't believe in what they're building
4. **Compensation** — Usually a tipping point, rarely the primary driver

**What top QA engineers in the AI era specifically want:**

- **Autonomy over their workflow** — including which AI tools they use and how
- **Access to modern tooling** — being blocked from using AI tools is a morale killer
- **Stretch into technical leadership** — architecture decisions, tool evaluations, mentoring
- **Visibility to stakeholders** — credit for their impact, not just their manager's summary
- **Growth that matches the pace of AI** — they want to stay ahead of the curve, not fall behind it

**Retention conversation (run quarterly for your top 2):**

```
Zen, I'm preparing a quarterly "stay conversation" with my strongest QA
engineer. Context: they've been on the team 2.5 years, have been high-performing,
and I know they're starting to be recruited externally.

Their motivations I know about: [technical growth, AI tooling, ownership]
What I can offer: [what's genuinely available — be honest]
What I can't offer: [constraints — be honest here too]

Help me structure a 30-minute conversation that:
(1) opens with genuine curiosity about where they want to go (not "are you
happy here?"), (2) explores what would make the next 12 months excellent
for them, (3) makes concrete commitments based on what I can actually deliver,
(4) is honest about limitations so I don't over-promise.
```

---

## Avoiding Performance Review Bias with AI-Assisted Calibration

Managers bring bias to performance reviews. Common patterns in QA teams:

- **Recency bias:** The last sprint's performance overshadows 5 months of data
- **Halo effect:** One outstanding project colors the entire assessment
- **Affinity bias:** Rating people you get along with better than objective evidence supports
- **Attribution bias:** Attributing team successes to senior members and failures to juniors

**Use Zen to audit your own assessments:**

```
Zen, I've written performance assessments for 5 team members. I want you to
check my work for common bias patterns.

For each assessment, here's what I wrote: [paste anonymized summaries]

Specifically look for: (1) evidence-based vs. adjective-based language,
(2) whether recent events seem to dominate the assessment,
(3) whether I'm harder/easier on different people without clear evidence-based
reason, (4) whether my strongest-relationship reports consistently have
higher ratings than my observations alone would support.

Be direct. I want my calibration to be as fair as possible.
```

---

## Team Morale Indicators and Early Warning Signs

Morale problems don't announce themselves. They whisper before they shout. Watch for:

**Early warning signs:**
- Quieter standups — people stop bringing up problems voluntarily
- Decreasing quality of retrospective contributions — "nothing to improve" becomes the norm
- Increase in "that's not my job" responses — ownership shrinks
- Slower response times on Slack — people are disengaged from the team channel
- Subtle increase in PTO — could be healthy, could be job searching

**Team morale Zen check:**

```
Zen, I want to assess the morale health of my team. Here's what I've
observed over the last 2 weeks:

- Standup dynamics: [observations]
- Retrospective quality: [observations]
- 1-on-1 content: [general themes, no names]
- Communication patterns: [observations]
- Any events that might have affected morale: [e.g., failed release, team conflict, reorg news]

Identify: (1) early warning signs I might be dismissing, (2) the most
likely root cause if there is a morale issue, (3) one low-cost, immediate
action I could take this week to improve the team's sense of safety and
belonging.
```

---

## Handling Team Dynamics When Someone Leaves

When a team member leaves — voluntarily or not — the team recalibrates. How you manage the transition determines whether it strengthens or damages the team.

**When someone leaves voluntarily:**
1. Be transparent about the timeline (within what the departing person has consented to share)
2. Have a private conversation with each remaining team member to check their read on the situation
3. Redistribute work quickly and visibly — don't let uncertainty fester
4. Acknowledge the loss genuinely: "We'll miss [X]. Here's how we're going to handle the transition."

**When someone is let go:**
- You usually cannot share the reason. Acknowledge this directly: "I can't share details, but I want to be available if you have concerns about your own standing."
- The team will be watching how you treat the departing person. Dignity matters.
- Don't rush to fill the gap with overwork on the remaining team — that's how you lose the next person.

---

## Building Psychological Safety: Bugs AND Problems Surface Early

A team with psychological safety tells you when something is wrong before it becomes a crisis. A team without it hides problems until they explode.

The most powerful thing you can do: **respond well the first time someone surfaces a risk or problem.**

If an engineer tells you "I think we're not ready to release" and you override them without genuine consideration — that signal will not come again. They'll learn to keep it to themselves.

**Building signals:**
- "Good catch — I'm glad you raised that." (specifically, when they do)
- "I disagree with your recommendation, and here's why. But I want you to keep raising these." (even in disagreement, validate the behavior)
- When you're wrong: "You were right, I was wrong. Next time, push harder."
- Model it yourself: bring your own uncertainties and mistakes to the team

**Zen prompt for psychological safety audit:**

```
Zen, I want to assess whether my team has psychological safety — specifically
the kind where engineers surface quality risks without being asked and push
back when they think a release isn't ready.

Here's evidence from the last 2 months: [describe incidents, conversations,
patterns where risks were or weren't surfaced proactively].

What does this evidence suggest about the psychological safety level of my team?
What's one specific thing I've done that might have reduced safety?
What's one specific action I could take this sprint to actively improve it?
```

---

## Checkpoint: Team Performance Action Items

- [ ] Write clear performance expectations for each role on your team — across all 3 dimensions
- [ ] Schedule a "stay conversation" with your top 2 performers this quarter
- [ ] Review your last performance assessments with Zen for bias
- [ ] Identify any team member you've been informally concerned about — schedule a direct conversation this week
- [ ] Run the team morale check prompt today and act on one finding
- [ ] Establish documentation practices for performance conversations — even informal ones

Great performance management is not about catching people failing. It's about creating conditions where success is inevitable — and addressing drift quickly, kindly, and specifically when it happens.

---

*Next: [Phase 5 — Organizational Impact: QA as Company-Wide Infrastructure](../phase-5-organizational-impact/)*
