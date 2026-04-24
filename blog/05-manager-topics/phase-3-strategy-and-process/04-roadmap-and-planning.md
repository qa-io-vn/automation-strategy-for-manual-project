> **Series:** From Senior Tester to Expert Test Manager — The AI Era Guide | **Phase 3: Strategy & Process** | [← Metrics for Managers](./03-metrics-for-managers.md) | [AI Tool Strategy →](./05-ai-tool-strategy.md)

# Quarterly Planning, OKRs, and the QA Roadmap

Most QA managers plan one sprint at a time. They're heads-down in execution, reacting to whatever the product team throws at them, and "strategic planning" is something they vaguely intend to do when things calm down.

Things never calm down.

The QA managers who get promoted to director and VP aren't better at testing than their peers — they're better at planning *above* the sprint level. They have a quarterly rhythm, a technology roadmap, a clear set of OKRs, and an answer ready when the CTO asks "where is QA headed in the next year?"

This guide gives you all of that, with Zen accelerating every step from OKR drafting to roadmap communication.

---

## The QA Quarterly Planning Ritual

Quarterly planning isn't a single meeting — it's a ritual that spans the last two weeks of one quarter and the first week of the next.

### Week -2 (Two Weeks Before Quarter End): Retrospective and Analysis

**What you do:**
- Review all Tier 2 and Tier 3 metrics from the current quarter
- Identify the top 3 quality wins and top 3 quality failures
- Gather input from your team: what's slowing us down, what's working?
- Pull engineering roadmap for next quarter: what new surfaces are being built?

```
Zen prompt — Quarterly Analysis Input:

"Analyze this quarter's QA performance data and help me prepare for quarterly planning:

METRICS SUMMARY: [paste your Tier 2 + Tier 3 metrics]
PRODUCTION INCIDENTS: [paste incidents with root cause notes]
TEAM FEEDBACK THEMES: [paste themes from team retros]
UPCOMING ENGINEERING ROADMAP: [paste or summarize next quarter's features]

Output:
1. Top 3 quality wins this quarter (with data)
2. Top 3 quality failures with root cause hypotheses
3. Gaps between current QA capability and next quarter's demands
4. 5 strategic questions for the QA team to answer this planning cycle"
```

### Week -1 (Last Week of Quarter): OKR Drafting and Roadmap Update

**What you do:**
- Draft OKRs for next quarter (use Zen prompt below)
- Update the technology roadmap based on current state and strategic direction
- Identify capacity needs: hiring, tooling, training

### Week +1 (First Week of New Quarter): Alignment and Communication

**What you do:**
- Align QA OKRs with engineering and product OKRs (are they compatible?)
- Communicate roadmap to your team: what's changing, what's the focus
- Confirm resource allocations with engineering manager

```
Zen prompt — Quarter Kickoff Communication:

"I need to communicate our Q[X] QA priorities to my team of [X] engineers.
Our OKRs for this quarter are: [paste OKRs].
Our key roadmap items are: [paste roadmap items].
Our main challenges from last quarter were: [paste challenges].

Write a quarterly kickoff message that:
1. Celebrates last quarter's wins
2. Honestly addresses what didn't work
3. Clearly explains Q[X] priorities
4. Gets the team energized about the AI initiatives we're launching
5. Invites questions and input

Tone: direct, optimistic, not corporate."
```

---

## Writing QA OKRs That Mean Something

OKRs (Objectives and Key Results) are powerful when done well and useless when they're just goals dressed up in framework language.

### The OKR Failure Pattern

Most bad QA OKRs look like this:

| Bad OKR Example | Why It's Bad |
|---|---|
| "Improve test coverage" | Not measurable, no target, no timeframe |
| "Reduce the number of bugs" | Wrong lever — fewer bugs found could mean less testing |
| "Implement AI in our QA process" | Output, not outcome. AI for what purpose? |
| "Increase automation by 20%" | Automation of what? Bad tests automated faster is still bad |
| "Be more strategic" | Unmeasurable platitude |

### Good QA OKRs: The Structure

**Objective:** Qualitative, ambitious, directional. Answers "what are we trying to achieve?"
**Key Results:** Quantitative, measurable, time-bound. Answers "how will we know we achieved it?"

### Good vs. Bad Examples

| Category | Bad OKR | Good OKR |
|---|---|---|
| **Quality** | "Reduce bugs" | "O: Achieve industry-leading defect detection before production. KR1: Defect Detection Rate > 92% by end of Q3. KR2: Zero Critical escapes to production. KR3: Mean time to detect production issues < 30 min" |
| **Automation** | "More automation" | "O: Build a reliable regression foundation that frees testers for exploratory work. KR1: Automation suite reliability > 96%. KR2: Automated regression runtime < 20 min. KR3: 30 new automation candidates delivered from new features" |
| **AI Adoption** | "Use AI more" | "O: Make AI assistance a core part of every QA workflow. KR1: 80% of test planning sessions use a Zen prompt. KR2: AI-generated tests account for 25% of new test coverage. KR3: Team saves estimated 15+ hours/sprint via AI tooling" |
| **Team** | "Improve team skills" | "O: Build the most technically capable QA team in the org. KR1: Every engineer completes AI testing certification. KR2: 2 engineers deliver internal tech talks. KR3: Team NPS > 8.5" |

```
Zen prompt — OKR Drafting from Strategy Goals:

"Help me write QA OKRs for next quarter. Here are our strategic goals and context:

COMPANY/ENGINEERING GOALS FOR NEXT QUARTER: [paste]
OUR BIGGEST QUALITY PROBLEMS RIGHT NOW: [paste]
OUR TEAM'S CURRENT CAPABILITIES: [paste]
WHAT WE'RE TRYING TO CHANGE OR IMPROVE: [paste]

Write 3 QA Objectives with 3 Key Results each that:
- Are ambitious but achievable for a team of [X]
- Connect quality outcomes to business outcomes (not just testing activity)
- Include at least one AI/automation-related OKR
- Have measurable KRs with specific numbers
- Avoid vanity metrics"
```

---

## Building the QA Technology Roadmap

Your technology roadmap answers: "what tools and capabilities are we building toward, and when?"

### Roadmap Horizons

| Horizon | Timeframe | Focus |
|---|---|---|
| **Now** | Current quarter | Active adoption, integration, training |
| **Next** | 1-2 quarters | Evaluating and piloting |
| **Later** | 3-4 quarters | Watching, researching |

### When to Adopt New Technology

Don't chase every new tool. Use these triggers to decide when adoption is warranted:

1. **Pain point trigger:** A specific, recurring, measurable problem exists
2. **Capability gap trigger:** There's something you *can't* do that you need to do
3. **Efficiency trigger:** A tool could save > 20% of time on a high-volume activity
4. **Competitive trigger:** Engineering teams have adopted something and QA is falling behind

```
Zen prompt — Technology Adoption Decision:

"We're considering adopting [tool name] for our QA team.
Context: we currently use [list current tools].
The problem we're trying to solve: [describe].

Evaluate this adoption decision:
1. How well does [tool] solve our specific problem?
2. What are the integration requirements with our current stack?
3. What is the learning curve and adoption timeline?
4. What are the risks (vendor, cost, dependency)?
5. What are the alternatives?
6. Recommendation: Adopt / Pilot / Watch / Skip — with rationale"
```

---

## AI Tool Evaluation Framework: 8 Criteria

When evaluating any AI tool for your QA team, score it across these 8 dimensions (1-5 each):

| Criterion | Description | Questions to Ask |
|---|---|---|
| **1. Accuracy** | Does it produce correct, useful output? | How often does AI-generated output require significant correction? |
| **2. Explainability** | Can you understand why it produced a result? | Can testers understand and validate AI decisions? |
| **3. Integration** | Does it fit your existing stack? | API access? CI/CD integration? Test management hooks? |
| **4. Cost** | Total cost including setup, training, and ongoing licensing | What's the cost per user/per run at our scale? |
| **5. Team Adoption** | Will your team actually use it? | How steep is the learning curve? Is there UI resistance? |
| **6. Maintenance** | How much ongoing effort does it require? | Who maintains it? What breaks when the product changes? |
| **7. Vendor Stability** | Is the vendor going to exist in 2 years? | Funding, customer base, product roadmap signals |
| **8. ROI** | Does the benefit outweigh the cost? | Measurable time savings, coverage improvements, defect improvements |

```
Zen prompt — AI Tool Evaluation Scorecard:

"Help me evaluate [tool name] for our QA team.

Here is what I know about the tool: [paste product description, pricing, capabilities]
Here is our current context: [team size, tech stack, current tools, main pain points]

Score this tool on these 8 criteria (1-5 with justification):
Accuracy, Explainability, Integration, Cost, Team Adoption, Maintenance, Vendor Stability, ROI

Total score out of 40. Recommendation: Adopt / Pilot / Watch / Skip.
Top 3 risks if we adopt. Top 3 reasons to adopt."
```

---

## Sprint Capacity Planning for QA

Quarterly planning sets direction. Sprint capacity planning makes it real.

### The Capacity Planning Formula

```
Available QA Hours per Sprint =
  (Headcount × Sprint Days × Hours/Day)
  − Meetings + ceremonies (estimate ~15% of time)
  − PTO and holidays
  − Unplanned work buffer (recommend 20% for mature teams)
```

**Example:**
- 3 QA engineers × 10 sprint days × 6 productive hours = 180 hours
- Minus ceremonies (15%): 27 hours → 153 hours
- Minus PTO/holidays: 12 hours → 141 hours
- Minus unplanned buffer (20%): 28 hours → **113 hours for planned QA work**

```
Zen prompt — Sprint Capacity Planning:

"Help me plan QA capacity for our upcoming sprint.

TEAM: [X] engineers. [List any PTO or ceremony time].
SPRINT DURATION: [X] days.
STORIES IN SCOPE: [paste story list with story point estimates].

Please:
1. Calculate available QA hours (use 6 productive hours/day, 15% ceremony overhead, 20% buffer)
2. Estimate test effort per story (Low=2h, Medium=4h, High=8h as baseline)
3. Flag stories that may not fit in the sprint with current capacity
4. Recommend which items to deprioritize if we're over capacity
5. Identify automation opportunities for this sprint's work"
```

---

## Risk-Adjusted Planning with AI Help

Standard capacity planning assumes everything goes to plan. Risk-adjusted planning adds a probability-weighted buffer for known risks.

### Risk Categories in Sprint Planning

| Risk Type | Probability | Buffer Impact |
|---|---|---|
| Unstable test environment | Medium | +10% to test execution time |
| New feature with ambiguous requirements | High | +25% for exploration + clarification |
| Cross-team dependency on shared service | Medium | +15% for coordination overhead |
| Legacy code being modified | High | +20% for expanded regression scope |
| First time testing on new platform/device | High | +30% for setup and unfamiliar behavior |

```
Zen prompt — Risk-Adjusted Sprint Plan:

"Here is our sprint QA plan: [paste plan].
Here are the known risks this sprint: [list risks with context].

Produce a risk-adjusted plan:
1. Original capacity and planned effort
2. Risk assessment for each story (Low/Medium/High)
3. Risk-adjusted effort estimate with buffer % applied
4. Total risk-adjusted capacity needed vs. available
5. If over capacity: recommended de-scope order with rationale
6. Contingency plan if a blocking risk materializes mid-sprint"
```

---

## Balancing Automation Investment vs. Manual Testing

One of the most consequential planning decisions a QA manager makes every quarter is how to allocate effort between automation and manual testing.

### The False Dichotomy

Automation vs. manual is the wrong frame. The right frame is: **which activities benefit most from which approach?**

| Activity | Best Approach | Why |
|---|---|---|
| Regression on stable features | Automation | High volume, repeatable, deterministic |
| Exploratory testing of new features | Manual | Requires judgment, creativity, intuition |
| API contract validation | Automation | Structured, high volume, exact comparison |
| UX/usability evaluation | Manual | Subjective, user-perspective required |
| Performance baseline testing | Automation | Needs repeatability across environments |
| Edge cases from customer reports | Manual first, then automate if recurring | Novel scenarios require exploration |
| Security testing | Both | Automation for known patterns; manual for logic flaws |

### The Quarterly Automation Investment Calculation

```
Automation ROI = (Manual Effort Avoided over 1 Year) - (Automation Build + Maintenance Cost)

Break-even point = Automation Cost / (Manual Test Time × Runs per Year)
```

If a test suite takes 8 hours to build and saves 2 hours of manual testing per sprint (26 sprints/year), break-even is 4 sprints, with 5+ years of compound savings.

---

## QA Roadmap on a Page

Every quarter, produce a single-page roadmap that any stakeholder can read in 60 seconds.

### Template

```
QA ROADMAP — Q[X] [YEAR]

OBJECTIVE: [One sentence describing our strategic direction]

NOW (This Quarter):
- [Initiative 1]: [What, why, who owns, success metric]
- [Initiative 2]: [What, why, who owns, success metric]
- [Initiative 3]: [What, why, who owns, success metric]

NEXT (Next 1-2 Quarters):
- [Tool/Capability we're evaluating]
- [Process improvement we're planning]

LATER (3-4 Quarters):
- [Technology we're watching]
- [Capability we're building toward]

KEY METRICS WE'RE MOVING:
- [Metric]: [Current] → [Target]
- [Metric]: [Current] → [Target]

DEPENDENCIES / ASKS:
- [What we need from engineering, product, leadership]
```

```
Zen prompt — Roadmap on a Page:

"Generate a one-page QA roadmap for Q[X] using this input:

STRATEGIC GOALS: [paste]
CURRENT INITIATIVES: [paste]
PLANNED FUTURE INITIATIVES: [paste]
KEY METRICS AND TARGETS: [paste]
DEPENDENCIES: [paste]

Format as a clean, executive-readable roadmap. Use the Now/Next/Later structure.
Keep the entire document under 400 words."
```

---

## Communicating Roadmap Changes to Stakeholders

Roadmaps change. A major production incident might reprioritize everything. A key engineer leaving shifts automation plans. Leadership adds a new strategic initiative.

**How to communicate roadmap changes without losing credibility:**

1. **Be proactive** — communicate before the change is visible, not after
2. **Explain the why** — "Here's what changed and why this is the right response"
3. **Show what's still true** — anchor to the things that haven't changed to reduce uncertainty
4. **Give new targets** — vague changes are more unsettling than specific new targets

```
Zen prompt — Roadmap Change Communication:

"I need to communicate a significant change to our Q[X] QA roadmap.

WHAT CHANGED: [describe the change]
WHY IT CHANGED: [describe the trigger — incident, new priority, capacity change]
WHAT'S NOW DEPRIORITIZED: [list]
WHAT'S NEW OR ACCELERATED: [list]
AUDIENCE: [describe who you're communicating to]

Write a clear, confident roadmap change communication that:
1. States the change upfront (no burying the lede)
2. Explains the reasoning without being defensive
3. Confirms what hasn't changed
4. Sets new expectations and timelines
5. Invites questions without creating panic"
```

---

## Checkpoint: Planning Rhythm Health Check

- [ ] **Quarterly planning ritual** — you have a defined process for quarterly planning, not just ad hoc sessions
- [ ] **OKRs written and aligned** — QA OKRs exist, are outcome-focused, and align with engineering OKRs
- [ ] **Technology roadmap current** — Now/Next/Later is up to date and shared
- [ ] **AI tool evaluation framework** — you have 8 criteria and a scoring process
- [ ] **Sprint capacity formula** — you're calculating available hours with overhead and buffer
- [ ] **Automation investment rationale** — you can articulate the ROI framework for automation decisions
- [ ] **Roadmap on a page** — exists and was shared with stakeholders this quarter
- [ ] **Change communication process** — you know how to communicate roadmap pivots confidently

> **The planning goal:** To operate one quarter ahead — never surprised by what's coming, always able to explain where you're going and why. Zen is your planning partner, not just your note-taker.

---

*Next: [AI Tool Strategy →](./05-ai-tool-strategy.md) — building and executing the AI tooling strategy for your QA organization.*
