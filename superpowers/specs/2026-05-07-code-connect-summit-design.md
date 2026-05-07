# Code Connect Summit 2026 — Conference Website Design

**Date:** 2026-05-07  
**Status:** Approved

---

## Overview

A single-page conference website for **Code Connect Summit 2026**, an internal Synthesis tech team conference. Hosted at Synthesis Software Technologies offices in JHB and CPT on June 5, 2026, 09:00–16:30.

The site lives in this repo under `docs/code_connect_summit/` and is published via the existing GitHub Pages CI pipeline unchanged.

---

## Architecture

| Concern | Decision |
|---|---|
| Structure | Single `index.qmd` (one long-scroll page) |
| Repo location | `docs/code_connect_summit/` |
| Quarto config | `docs/code_connect_summit/_quarto.yml` |
| Stylesheet | `docs/code_connect_summit/style.css` |
| Output | `dist/code_connect_summit/index.html` (via existing CI) |
| Interactivity | Vanilla JS embedded in `index.qmd` raw HTML blocks |
| External deps | None — `embed-resources: true`, self-contained |

The existing `build-docs.yml` workflow iterates `docs/*/` and renders `<doc_name>.md` where `doc_name` matches the folder name. The conference site uses `index.qmd` (not `code_connect_summit.qmd`) because Quarto websites conventionally use `index` as the root file. The CI render loop must be updated to detect and render `index.qmd` when present, in addition to the existing `<name>.md` pattern.

---

## Visual Style

Matches the Code Connect Summit 2026 poster:

- **Background:** Deep dark (`#0a0a1a`, `#0d1b3e`, `#1a0a2e`)
- **Accent colours:** Neon cyan (`#00e5ff`), electric purple (`#7c3aed`), blue (`#2563eb`)
- **Typography:** System sans-serif stack; monospace for times
- **Gradients:** Cyan→purple on hero title; purple→blue on CTAs and highlights
- **Cards:** Low-opacity white fill, cyan border at 10–15% opacity

---

## Sections

### 1. Sticky Navigation
- Logo: Synthesis hexagon mark + wordmark (left)
- Links: About · Schedule · Speakers · Contact (anchor links)
- CTA: "RSVP Now" button (purple→blue gradient, right)
- Style: dark semi-transparent background, `backdrop-filter: blur`, cyan bottom border

### 2. Hero
- Conference label: "CONFERENCE · 2026" (small caps, purple)
- Title: "Code Connect Summit" (large, cyan→purple gradient text)
- Pill tags: Presentations · Panel · Keynote · Workshop
- Tagline: "An extraordinary showcase of innovation for the Synthesis tech team."
- Meta row: date/time · venue · room
- CTA button: "RSVP Now →" linking to RSVP form

### 3. About
Short paragraph describing the event — full-day knowledge sharing, keynote, panel, workshop, prizes. Hosted simultaneously JHB & CPT.

### 4. Schedule

Full agenda table with columns: **Time · Session · Presenter(s)**

| Time | Session | Presenter(s) |
|---|---|---|
| 09:00–09:10 | Welcome | Kieron & Archie |
| 09:10–09:40 | Keynote | Tom |
| 09:45–10:15 | Streaming in a High-Pressure Environment (i.e. Fraud) | Vincent & Enoch |
| 10:20–10:50 | LLM in Document Understanding | Marcus & Massimo |
| 10:50–11:10 | ☕ Break | — |
| 11:10–11:55 | **Panel:** The Future of Software Engineering in the Age of AI | Facilitators: Kieron & Archie; Panelists: Tom, Tjaard, Marais, MikeG |
| 12:00–12:30 | Running Kubernetes at Home the Easy Way, Because Overcomplicating Things Is a Lifestyle Choice | Rodney |
| 12:30–13:30 | 🍽 Lunch Break | — |
| 13:30–14:00 | Building a Roblox Game with My Son — What Broke, What Scaled, What Didn't | Tjaard |
| 14:05–14:35 | Self-Healing Codebases | Nick Driver |
| 14:40–15:40 | **Workshop:** Smart Trainer — Realtime AI Fitness Trainer | Jason |
| 15:40–16:00 | ☕ Break | — |
| 16:00–16:30 | Final Thank You & Prizes | Kieron & Archie |

Panel row includes the 6 discussion questions as a collapsed/inline sub-list:
1. What is the future when the funding runs out for tools?
2. Are we overestimating short-term impact and underestimating long-term disruption—or the opposite?
3. Who is accountable when AI-generated systems fail in production?
4. Are we heading toward more standardized architectures because AI performs better within constraints?
5. Where should humans remain "in the loop" no matter how good AI gets?
6. Are we moving toward fewer engineers with higher leverage, or fundamentally different roles altogether?

Visual treatment: breaks/lunch rows muted grey; panel/workshop rows accented purple.

### 5. Speakers

4-column grid of speaker cards. Each card shows: avatar placeholder, name, talk title.

**Interaction — expand in place:**
- Click a card → it expands to `grid-column: span 4` (full width)
- Expanded state shows: photo (placeholder until supplied), name, talk title, bio text
- Clicking again or clicking another card collapses the previous one
- Implemented with vanilla JS (`expandCard()` function)
- Bio and photo fields are placeholder until content is supplied; structure supports drop-in replacement

**Speakers:**
- Tom — Keynote
- Vincent & Enoch — Streaming in a High-Pressure Environment
- Marcus & Massimo — LLM in Document Understanding
- Rodney — Running Kubernetes at Home
- Tjaard — Building a Roblox Game with My Son
- Nick Driver — Self-Healing Codebases
- Jason — Workshop: Smart Trainer

### 6. RSVP

Gradient-bordered box with short copy and a prominent "Register Now →" button linking to:  
`https://forms.office.com/Pages/ResponsePage.aspx?id=hdvPlCM9SUigZlza2WXM2AYv7RPSF49BsSdfvCGj1WdUQTJHV1Q1SUtVSjlDMVBZQTdOMTdUVlNWMS4u`

### 7. Contact

Host contact details. Kieron's email: `kieron@synthesis.co.za`. Additional contacts TBD.

### 8. Footer

Synthesis hexagon mark, "Code Connect Summit 2026 · Synthesis Software Technologies".

---

## CI Change Required

The `build-docs.yml` render loop currently matches `<name>.md`:

```bash
[ -f "$doc_dir/$doc_name.md" ] || continue
quarto render "$doc_name.md"
```

Must be updated to also render `index.qmd` when present:

```bash
if [ -f "$doc_dir/index.qmd" ]; then
  quarto render "index.qmd"
elif [ -f "$doc_dir/$doc_name.md" ]; then
  quarto render "$doc_name.md"
else
  continue
fi
```

---

## Out of Scope

- Speaker photos (placeholder until supplied; structure ready for drop-in)
- Speaker bios (placeholder until supplied)
- Additional contact names beyond Kieron
- Registration backend (external Microsoft Forms handles this)
- Mobile-specific breakpoints (basic responsiveness via CSS grid `auto-fill`, full mobile polish deferred)
