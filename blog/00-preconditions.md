# Preconditions: What You Need Before Bringing AI Into Your QA Workflow

> **Series:** AI-Assisted QA — From Manual Tester to Automation Strategist  
> **Level:** Preconditions (Start Here)  
> **Next:** [Part 1 — Beginner: Your First Week with AI-Assisted Testing](./01-beginner.md)

---

## Why Preconditions Matter

In testing, we always define preconditions before executing a test case. The same principle applies here. Before you bring an AI assistant like **Zen** into your QA workflow, you need to set the right foundation — otherwise you'll get generic outputs that don't match your project reality.

This post walks you through exactly what to prepare.

---

## What You Need to Have Ready

### 1. Your Project Documentation

Gather whatever docs describe what the system is supposed to do:

- Product requirement documents (PRDs)
- User stories or epics (Jira, Notion, Confluence, etc.)
- Functional specifications
- API documentation (Swagger, Postman, etc.)
- UI mockups or design files (Figma links)

> **Tip:** Even rough, informal docs work. The AI can help you clarify and structure them later.

---

### 2. Your Existing Manual Test Cases

Collect everything you already have, even if it's messy:

- Test cases in spreadsheets (Excel, Google Sheets)
- Test management tools (TestRail, Zephyr, Xray, Qase, etc.)
- Plain text or Confluence pages
- Informal checklists

For each test case, ideally you have:
- Title / scenario name
- Preconditions
- Steps to reproduce
- Expected results
- Priority (P0/P1/P2 or High/Medium/Low)

> **Don't have these yet?** That's fine — the Beginner guide will help you create them from scratch using AI.

---

### 3. Your Test Reports (Historical Results)

Past test reports are gold. They tell you:

- Which areas break the most (high-risk zones)
- Which tests are executed every release
- Where bugs have historically been found

Acceptable formats: screenshots, PDF reports, exported CSVs from your test tool, or even meeting notes from bug review sessions.

---

### 4. A Basic Understanding of Your Tech Stack

You don't need to be a developer. But knowing these things will help the AI give you better, more targeted output:

| Question | Example Answers |
|---|---|
| What type of app? | Web, Mobile (iOS/Android), API, Desktop |
| Frontend framework? | React, Vue, Angular, plain HTML |
| Backend language? | Node.js, Python, Java, .NET |
| How do users authenticate? | Email/password, SSO, OTP |
| Any 3rd party integrations? | Payment gateways, email services, maps |

---

### 5. Your Test Environment Details

Be aware of:

- **Environments available:** Dev, Staging, UAT, Production
- **Test data situation:** Is there a stable test account? Seed data? Reset scripts?
- **Access:** Do you have credentials to all necessary environments?

---

### 6. Your Current Pain Points

Be honest with yourself. Write down 3–5 things that slow you down most:

Examples:
- "I spend too much time writing test cases from scratch"
- "Regression takes 2 days every sprint"
- "I struggle to write clear bug reports quickly"
- "I have no visibility into what is and isn't covered"
- "I can't automate because I don't know how to code"

These pain points become your **AI use cases** — the specific things you'll tackle first.

---

## Set Up Your AI Assistant (Zen + Zenflow)

To follow this series, you'll work with **Zen** inside Zenflow. Here's how to get started:

1. **Access Zenflow** — via web UI, Slack, or Telegram
2. **Create your project** in Zenflow (or connect an existing one)
3. **Introduce your project to Zen** — paste a short description of your app and what you test
4. **Save key context** — Zen has memory, so the more context you give early, the better every future interaction becomes

> **First message to Zen:** *"I'm a QA tester on [App Name]. It's a [web/mobile/API] app that does [core purpose]. I have [X test cases] and test manually every sprint. My biggest pain is [your pain point]."*

---

## The Self-Assessment Checklist

Before moving to the Beginner guide, check off what you have:

- [ ] At least some requirements or user stories
- [ ] At least some existing test cases (even rough ones)
- [ ] Access to at least one test environment
- [ ] A basic understanding of what your app does
- [ ] 1–3 specific pain points you want to solve
- [ ] Zenflow/Zen access set up

You don't need everything to be perfect. **Having 50% of this list is enough to start.**

---

## What's Next

In the next post, you'll take what you've prepared here and start your first real AI-assisted testing tasks:

- Generating test cases from requirements
- Writing professional bug reports with AI help
- Building a quick test coverage map

→ **[Part 1: Beginner — Your First Week with AI-Assisted Testing](./01-beginner.md)**

---

*Part of the series: **AI-Assisted QA — From Manual Tester to Automation Strategist***
