# Preconditions: Setting Yourself Up for Success

> **Series:** AI-Assisted QA: From Manual Tester to Automation Strategist
> **Section:** 00 — Preconditions
> **Navigation:** [← Series Home](../README.md) | [Next Section: Beginner →](../01-beginner/README.md)

---

Before you write your first AI-assisted test case or generate your first automation script, you need a solid foundation. This section is the "mise en place" of AI-assisted QA — getting everything prepped, organized, and in the right place so the actual work flows smoothly.

Skipping this section is like trying to cook a complex dish without measuring your ingredients first. You *can* do it, but you'll spend more time fixing mistakes than making progress.

---

## Why Preconditions Matter

Zen (Zenflow's AI assistant) is only as good as the context you give it. A prompt like *"generate test cases for my app"* will give you something generic and nearly useless. A prompt like *"generate test cases for the checkout flow of our React e-commerce app, which uses Stripe for payments, supports guest and logged-in users, and has 3 known edge cases around coupon validation"* — that gives you something you can actually use on Monday morning.

The preconditions section teaches you how to **gather that context**, **understand your technical environment**, **configure Zenflow**, and **honestly assess where you're starting from**.

---

## What's in This Section

### [01 — What to Gather Before You Start](./01-what-to-gather.md)

**The artifact inventory sprint.**

Before Zen can help you test anything, you need to know what materials you're working with. This guide walks you through every type of document and resource you should collect: requirements docs, user stories, test cases, API specs, UI mockups, and more. 

Key topics:
- The complete QA artifact checklist
- How to handle projects with missing or outdated documentation
- Reverse-engineering requirements from a live application
- Tips for legacy projects where "the docs are in someone's head"
- How to organize your artifacts so Zen can use them effectively
- Prompts for extracting structure from messy, informal requirements

**Who needs this most:** QA engineers joining an existing project, testers taking over from someone who left, anyone working on legacy systems.

---

### [02 — Understand Your Tech Stack](./02-understand-your-stack.md)

**You don't need to be a developer — but you do need a map.**

Your testing approach changes significantly depending on whether you're testing a React SPA, a native iOS app, a REST API, or a desktop Electron app. This guide teaches you how to identify what you're working with, what it means for your testing strategy, and how to ask Zen to explain the technical parts you don't understand yet.

Key topics:
- Web vs. Mobile vs. API vs. Desktop: key differences for QA
- Reading a tech stack without being a developer
- Questions to ask your dev team (and how to phrase them)
- How your stack determines which CI/CD tools make sense
- Prompt templates for "explain this to me like I'm a QA engineer"
- Stack-to-tooling mapping table

**Who needs this most:** QA engineers working in cross-functional teams, testers moving into a new domain, anyone who's been handed a project and told "figure it out."

---

### [03 — Set Up Zenflow](./03-setup-zenflow.md)

**Your AI-assisted QA command center, configured properly.**

Zenflow isn't just a task tracker — it's the platform that connects Zen to your testing workflow. This step-by-step guide walks you through creating your project, connecting Zen via web, Slack, or Telegram, giving Zen the context it needs to be useful, and running your first meaningful prompts.

Key topics:
- Creating a project in Zenflow (with screenshots and annotations)
- Connecting Zen through different channels
- Building your project's memory file — the most important setup step
- First 5 prompts every QA engineer should run
- How to organize Zenflow tasks for a testing workflow
- Troubleshooting common setup issues

**Who needs this most:** Everyone — this is a required step before the rest of the series.

---

### [04 — Self-Assessment: Where Are You Starting From?](./04-self-assessment.md)

**Know thyself (as a QA engineer).**

Before diving into AI-assisted techniques, take 15 minutes to honestly assess your current skill level, pain points, and automation readiness. This isn't a test you pass or fail — it's a compass that tells you where to focus your energy and which parts of this series will give you the biggest return.

Key topics:
- The QA Skill Inventory (25-question self-assessment)
- Pain Point Inventory: what's eating most of your time?
- The Automation Readiness Score
- Recommended starting points based on your profile
- How to use your results to customize your learning path
- Tracking your progress over time

**Who needs this most:** Everyone, but especially those who feel overwhelmed by "where to start."

---

## Estimated Time Investment

| Guide | Reading Time | Hands-On Time | Total |
|-------|-------------|---------------|-------|
| 01 — What to Gather | 15 min | 1–3 hours | ~2–3 hours |
| 02 — Understand Your Stack | 20 min | 30 min | ~1 hour |
| 03 — Set Up Zenflow | 20 min | 45 min | ~1 hour |
| 04 — Self-Assessment | 10 min | 15 min | 25 min |
| **Total** | **~1 hour** | **~3–5 hours** | **~4–6 hours** |

This is a half-day investment. It will save you weeks of confusion and rework later. Do not skip it.

---

## Checklist: Preconditions Complete

Use this to verify you're ready to move on:

- [ ] I have collected all available project artifacts (or documented what's missing)
- [ ] I can describe my application's tech stack at a high level
- [ ] I understand how my stack affects my testing approach
- [ ] I have created a Zenflow project for my QA work
- [ ] I have connected Zen via at least one channel (web, Slack, or Telegram)
- [ ] I have created a project memory file that gives Zen context
- [ ] I have completed the self-assessment and know my starting profile
- [ ] I have run my first 5 prompts with Zen and gotten useful responses

---

## A Note on Mindset

This series isn't about replacing your QA judgment with AI. It's about **amplifying** what you already know. Zen will help you generate faster, think broader, and document better — but you're still the one who knows your users, understands the business risk, and decides what "good enough" actually means.

The best QA engineers using AI aren't the ones who prompt the most — they're the ones who bring the most context to the conversation.

Let's build that context now.

---

*Ready? Start with [01 — What to Gather](./01-what-to-gather.md).*
