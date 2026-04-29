---
title: "AI Maturity Levels"
subtitle: "A Field Guide"
---

::: {.doc-meta}
**VERSION** 1.0 · 2026 &nbsp;·&nbsp; **AUDIENCE** All staff &nbsp;·&nbsp; **OWNER** Technology Office
:::

This guide describes five maturity levels — L0 through L4 — that capture how individuals and teams adopt and integrate AI tools into their daily work. Where you sit on this scale is not a judgment; it's a starting point for understanding what's possible.

Each level is described twice: once in plain language (what it looks and feels like day-to-day) and once in technical terms (what it means for your toolchain, workflow, and engineering practice).

---

<div class="spectrum">
<div class="step s0"><div class="circle">0</div><div class="label">Resistant</div></div>
<div class="step s1"><div class="circle">1</div><div class="label">Experimental</div></div>
<div class="step s2"><div class="circle">2</div><div class="label">Selective</div></div>
<div class="step s3"><div class="circle">3</div><div class="label">Integrated</div></div>
<div class="step s4"><div class="circle">4</div><div class="label">Strategic</div></div>
</div>

---

::: {.level-card .level-0}

## Level 0 — Resistant

> *"I'd rather not touch it until we know it's safe."*

:::: {.columns}
::: {.column}
### Non-developer
- Won't use AI tools — worried about confidential information leaving the organisation
- Sales team avoids putting deal context, pricing, or customer account information into any AI tool
- Waits for explicit sign-off from leadership before touching anything AI-related
:::
::: {.column}
### Developer
- Declines AI coding tools on client projects citing IP risk
- All code reviews, tests, and documentation written entirely by hand
- Avoids AI-assisted tools even for internal tooling tasks
:::
::::

:::

::: {.level-card .level-1}

## Level 1 — Experimental

> *"I've tried it a few times, still figuring it out."*

:::: {.columns}
::: {.column}
### Non-developer
- Used an AI tool once or twice to draft a status update, proposal intro, or meeting summary
- Sales reps tried it for a capability statement — rewrote most of it before sending to the client
- Hasn't explored available AI features in productivity tools despite having access
:::
::: {.column}
### Developer
- Tried AI code completion for a few suggestions; accepted some, deleted most
- Pasted a stack trace into an AI assistant once to debug — found it useful but hasn't repeated it
- No consistent prompt pattern or workflow yet
:::
::::

:::

::: {.level-card .level-2}

## Level 2 — Selective

> *"I know exactly where it helps me and where it doesn't."*

:::: {.columns}
::: {.column}
### Non-developer
- Uses AI tools daily for client communications, retro notes, and report structuring
- Sales team drafts SOW summaries, RFP responses, and follow-up emails with AI — always reviews before sending
- Knows not to paste real customer data; uses anonymised context or sanitised examples
:::
::: {.column}
### Developer
- Uses AI tooling for unit test generation and boilerplate; manually reviews all suggestions
- Uses AI-assisted IDE tools for refactoring sessions but owns the final architectural decisions
- AI not yet wired into CI/CD or PR review gates on client delivery
:::
::::

:::

::: {.level-card .level-3}

## Level 3 — Integrated

> *"Removing AI from my workflow would noticeably slow me down."*

:::: {.columns}
::: {.column}
### Non-developer
- Runs standup synthesis, sprint reports, and governance forum prep entirely with AI assistance
- Sales team uses AI to research client context before meetings, generate tailored pitch decks, and draft commercial proposals end-to-end
- Can point to concrete time savings — e.g. proposal first drafts in 20 min vs half a day
:::
::: {.column}
### Developer
- Runs our internal AI-augmented development methodology end-to-end — applying agentic patterns at every SDLC phase with specialist sub-agents handling discrete tasks: requirements analysis, test generation, code review, security scanning, and documentation — each firing at the right stage in the pipeline
- The pipeline is intentional, not improvised — each agent handoff is defined, observable, and repeatable across sprints
- Tracks and reports measurable gains — sprint velocity, defect rates, review turnaround, and time-to-deploy are visibly better with AI in the loop, and you can prove it with numbers
:::
::::

:::

::: {.level-card .level-4}

## Level 4 — Strategic

> *"I'm not just using AI — I'm helping Synthesis use it better."*

:::: {.columns}
::: {.column}
### Non-developer
- Runs AI enablement sessions for other business units — not just uses tools, but transfers capability
- Sales team leads with AI-informed deal strategy: uses market intelligence, competitive analysis, and client profiling to walk into every engagement better prepared than the competition
- Owns documented playbooks (e.g. AI-assisted RFP workflow) that other teams can pick up and run
- Feeds insights into governance forums and client roundtables to embed AI standards org-wide
:::
::: {.column}
### Developer

**Seniors:**

- Own the AI toolchain standards for their squad — define which tools get used at which SDLC phase, enforce agentic design patterns in PR reviews, and set the quality bar for AI-generated output

**Principals:**

- Architect the multi-agent systems other squads build on — publishing reusable agent specifications and internal agentic design patterns to the Architecture Forum pattern library, ensuring every agent deployment has safety, observability, and escalation built in by design

**Both levels:**

- Drive adoption of the organisation's AI-augmented development methodology across delivery teams — not just using it themselves but embedding it as the Synthesis standard for AI-powered SDLC
- Establish internal agentic architecture standards that other squads implement on client engagements
:::
::::

:::

---

*This document is a living artefact. Update it as our methodology, tooling, and maturity evolve.*
