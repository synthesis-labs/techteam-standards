# Claude + Git + Skills is all you need — Agent Guidelines

Working notes for anyone (human or AI) editing this deck. Read before making changes.

---

## What this deck is

A talk titled **Claude + Git + Skills is all you need** by Tom Wells for Code Connect Summit 2026.

[Add a 2-3 sentence summary of the deck's argument or arc here.]

---

## File structure

| File | Purpose |
|---|---|
| `slides.md` | Deck source. Per-slide frontmatter + markdown. Global style block at top. |
| `package.json` | Slidev + dependencies. |
| `Makefile` | `make install` / `make dev` / `make build` / `make export`. |
| `layouts/cover.vue` | Navy cover layout. Reads `title`, `subtitle`, `eyebrow`, `author`, `date`, `logo`, `organisation` from frontmatter. |
| `layouts/section.vue` | Section divider. Reads `number`, `title`, `subtitle`. |
| `layouts/end.vue` | Closing slide. Reads `title`, `eyebrow`, `speaker`, `organisation`, `email`, `footer`. |
| `components/Principle.vue` | Blue left-border card. `<Principle title="...">body</Principle>`. |
| `components/Caveat.vue` | Rust left-border card. `<Caveat>body</Caveat>` or `<Caveat label="...">`. |
| `components/PullQuote.vue` | Italic quote with blue rule. Use `<em>...</em>` for accent emphasis. |
| `components/BeforeAfter.vue` | Two-column code comparison. Use `<template #before>` / `<template #after>` slots (see syntax notes below). |
| `public/` | Static assets. Served at the root URL — a file at `public/foo.png` is reached at `/foo.png`. |

Build with `make dev` (live preview at http://localhost:3030).
Export PDF with `make export` (requires `playwright-chromium`, in devDependencies).

---

## Visual identity

| Token | Hex | Use |
|---|---|---|
| `--deck-navy` | `#0D1120` | Section dividers |
| `--deck-navy-deep` | `#162440` | Cover, end |
| `--deck-blue-accent` | `#4D7EF7` | Rules, accents, "after" labels |
| `--deck-blue-pale` | `#A5B4FC` | Eyebrows, subtitles on dark backgrounds |
| `--deck-body-ink` | `#1A1F2E` | Body text on light slides |
| `--deck-muted` | `#6B7280` | Captions, "before" labels |

Defined in the global `<style>` block at the top of `slides.md`.

---

## Editing conventions

**Small corrections** (a word, a number, an emphasis): edit `slides.md` directly. Run `make dev` and visually verify.

**Structural changes** (adding/removing slides, reordering, new component, new layout): write a short plan in `specs/` first, get it reviewed, then apply. Run `make dev` and walk every slide.

**After any edit:** check the live preview; check that `make build` succeeds (this catches type errors in custom Vue components that hot-reload sometimes masks).

---

## Voice and style

[Speaker should fill in target audience and tone here.]

**Do:**
- Short, declarative sentences. Slides hold one idea.
- Code examples must fit one screen without scrolling.

**Avoid:**
- Bullet walls. If a slide has more than 4 bullets, break it.
- Code blocks longer than 12 lines on a single slide.

---

## Verified facts

| Claim | Source |
|---|---|
| | |
