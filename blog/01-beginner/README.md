# Beginner: Start Generating Value Today

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 01 — Beginner
> **Navigation:** [← Preconditions](../00-preconditions/README.md) | [Next Section: Intermediate →](../02-intermediate/README.md)

---

## Introduction

You've done the prep work. Your context is organized, your workspace is configured, and you know where you're starting from. Now it's time to do something useful.

The beginner section is about **doing the core work of QA faster, better, and more consistently** with AI assistance. Not automation — not yet. These are the foundational activities every QA engineer does every day: writing test cases, mapping coverage, filing bug reports, building checklists, and tracking work.

You've probably done all of these before. The question is: how much time do they currently take? And how much of that time is genuine thinking versus mechanical work you could hand off?

By the end of this section, you should be generating test cases in minutes instead of hours, writing bug reports that developers actually respect, and having a clear picture of your test coverage at any given moment.

---

## What's in This Section

### [01 — Generate Test Cases with AI](./01-generate-test-cases.md)

**From blank page to comprehensive coverage in one session.**

Test case writing is the most time-consuming activity for most manual QA engineers. A feature that took a developer two weeks to build might need 50–100 test cases to cover properly. Writing those from scratch is slow, repetitive, and prone to missing edge cases.

Zen changes the game here. Given good requirements, it can generate a comprehensive initial set of test cases in minutes. Your job shifts from writing to reviewing, refining, and applying your domain knowledge.

Key topics:
- Prompt templates for every requirement format (user stories, AC, prose, informal notes)
- How to iterate and refine AI-generated test cases
- Multi-module test case generation strategies
- Full example: e-commerce checkout flow (50+ test cases)
- Full example: SaaS dashboard with role-based access
- Handling ambiguous requirements with follow-up prompts
- When to override AI suggestions with your own judgment

**Estimated time to complete:** 2–3 hours reading + hands-on practice

---

### [02 — Build a Test Coverage Map](./02-coverage-map.md)

**See exactly what you're testing and where the gaps are.**

"Do we have good coverage?" is one of the most common questions in QA — and one of the hardest to answer without a structured approach. A coverage map turns that vague question into a specific, visual, trackable answer.

With AI assistance, you can build and maintain a coverage map that shows every feature, every test scenario, and every gap — updated in real time as your application evolves.

Key topics:
- What test coverage actually means (hint: it's not just line coverage)
- Types of coverage: functional, user path, API, data, negative
- Building your first coverage map with Zen
- Gap analysis: what you're not testing and why that matters
- Coverage map table template with real examples
- Tracking coverage over time as features are added
- Coverage audit prompts for existing test suites

**Estimated time to complete:** 2 hours reading + hands-on practice

---

### [03 — Write Professional Bug Reports](./03-bug-reports.md)

**Bug reports that developers actually read — and fix.**

A bad bug report is a productivity killer for everyone. Developers can't reproduce it, PMs can't prioritize it, and you end up in a back-and-forth that takes days instead of hours. A great bug report is a gift — it clearly describes the problem, provides everything needed to reproduce it, and makes the fix obvious.

Most QA engineers know what a good bug report looks like. Fewer write them consistently, especially when under time pressure. Zen helps you produce professional-quality bug reports every time.

Key topics:
- Anatomy of a great bug report (every field, every time)
- Severity vs. priority — the framework developers actually use
- Prompt template for generating a bug report from raw notes
- 3 complete example bug reports: UI, API, and performance
- How to ask Zen to estimate business impact
- Describing and attaching evidence (screenshots, HAR files, logs)
- The "two-minute sanity check" before submitting

**Estimated time to complete:** 1.5 hours reading + practice with your own bugs

---

### [04 — Build Test Execution Checklists](./04-test-execution-checklist.md)

**Never forget what you were supposed to test.**

Pre-release panic is real. There's time pressure, there are questions flying in Slack, and you're trying to remember whether you tested that edge case in the payment flow. A well-built checklist eliminates that cognitive load.

Not all checklists are the same. A smoke test checklist for a hotfix is different from a full regression for a major release. This guide shows you how to build the right checklist for the right moment — and use Zenflow to track execution.

Key topics:
- Smoke vs. regression vs. sanity vs. exploratory: when to use each
- How to customize checklists by release type and risk level
- Prompt templates for generating each checklist type
- A full example: release checklist for a SaaS monthly release
- Using Zenflow tasks to track execution in real time
- Post-execution: the result summary prompt that saves you 30 minutes
- When to deviate from the checklist and how to document it

**Estimated time to complete:** 2 hours reading + hands-on practice

---

### [05 — Track All Your QA Work in Zenflow](./05-track-in-zenflow.md)

**Your QA work is only as visible as your tracking system.**

QA work has a visibility problem. Unlike development, where every commit is logged and every PR is reviewed, testing work often happens in the dark. A QA engineer can spend a full week doing invaluable work that no one else sees — and then get asked "but what have you actually been testing?"

Zenflow solves this. When your work is tracked in Zenflow, it's visible to your team, searchable by Zen, and auditable for retrospectives. This guide shows you how to use Zenflow for the full QA workflow.

Key topics:
- Creating and organizing bug tasks (with templates)
- Creating test execution tasks that track progress
- Creating improvement tasks from retrospective findings
- Setting up recurring automation reminders for QA rituals
- Reading and interpreting task progress as a manager would
- Generating a weekly status report with Zen
- How to use Zenflow to demonstrate your value to stakeholders

**Estimated time to complete:** 1.5 hours reading + full hands-on setup

---

## Estimated Time Investment

| Guide | Reading Time | Hands-On Time | Total |
|-------|-------------|---------------|-------|
| 01 — Generate Test Cases | 30 min | 2 hours | ~2.5 hours |
| 02 — Coverage Map | 30 min | 1.5 hours | ~2 hours |
| 03 — Bug Reports | 25 min | 1 hour | ~1.5 hours |
| 04 — Test Execution Checklists | 30 min | 1.5 hours | ~2 hours |
| 05 — Track in Zenflow | 25 min | 1 hour | ~1.5 hours |
| **Total** | **~2.5 hours** | **~7 hours** | **~9.5 hours** |

This is approximately one working day of focused effort. Many engineers spread it over a week, applying each technique as they go.

---

## What to Expect from the Beginner Section

By the time you complete this section:

- You will have **real test cases** for a feature in your actual project
- You will have **a working coverage map** you can update and share
- You will have **a template for writing bug reports** that produces professional output every time
- You will have **a pre-release checklist** ready for your next release
- You will have **a Zenflow project** actively tracking your work

These aren't exercises or examples — they're deliverables you'll use in your actual job.

---

## Beginner Section Checkpoint

At the end of this section, you should be able to:

- [ ] Generate 20+ test cases from a user story in under 30 minutes
- [ ] Identify coverage gaps in any feature by prompting Zen
- [ ] Write a complete bug report in under 5 minutes with AI assistance
- [ ] Build a pre-release smoke test checklist in under 15 minutes
- [ ] Have all your active QA work tracked and visible in Zenflow

---

## A Note on the Role of Your Judgment

Every guide in this section will give you prompts that generate output. Zen will produce test cases, coverage maps, and bug reports. But here's the thing: **Zen doesn't know your users the way you do**. It doesn't know about the weird edge case that comes from the migration 18 months ago. It doesn't know that users always try to do X even though the UI doesn't obviously support it.

Use AI output as a starting point and a structure. Add your domain knowledge. Remove what doesn't apply. Reorder by actual risk. The goal is not to replace your thinking — it's to eliminate the mechanical work so you can think more.

---

*Ready? Start with [01 — Generate Test Cases](./01-generate-test-cases.md).*
