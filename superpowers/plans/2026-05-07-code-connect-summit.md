# Code Connect Summit 2026 — Conference Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-page dark-themed conference website for Code Connect Summit 2026, published via the existing Quarto + GitHub Pages pipeline.

**Architecture:** One `index.qmd` in `docs/code_connect_summit/` renders to `dist/code_connect_summit/index.html`. Raw HTML blocks inside the `.qmd` provide the dark immersive layout and vanilla JS speaker-expand interaction. The CI workflow is updated to detect and render `index.qmd` files in addition to the existing `<name>.md` pattern.

**Tech Stack:** Quarto (HTML format), vanilla CSS, vanilla JS, GitHub Actions

---

## File Map

| File | Action | Purpose |
|---|---|---|
| `docs/code_connect_summit/_quarto.yml` | Create | Quarto project config — output dir, HTML format, embed-resources |
| `docs/code_connect_summit/style.css` | Create | Dark theme CSS — all visual styles |
| `docs/code_connect_summit/index.qmd` | Create | Page content — nav, hero, about, schedule, speakers, RSVP, contact, footer |
| `.github/workflows/build-docs.yml` | Modify | Update render loop to detect `index.qmd` as well as `<name>.md` |

---

## Task 1: Create `_quarto.yml`

**Files:**
- Create: `docs/code_connect_summit/_quarto.yml`

- [ ] **Step 1: Create the directory and config file**

```bash
mkdir -p docs/code_connect_summit
```

Create `docs/code_connect_summit/_quarto.yml` with this exact content:

```yaml
project:
  output-dir: ../../dist/code_connect_summit

format:
  html:
    theme: none
    css: style.css
    toc: false
    embed-resources: true
    page-layout: full
```

`theme: none` prevents Quarto from injecting its default Bootstrap styles, which would conflict with the dark custom theme. `embed-resources: true` makes the output a single self-contained HTML file with no external dependencies.

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/_quarto.yml
git commit -m "feat(summit): scaffold quarto config for conference site"
```

---

## Task 2: Create `style.css`

**Files:**
- Create: `docs/code_connect_summit/style.css`

- [ ] **Step 1: Create the stylesheet**

Create `docs/code_connect_summit/style.css`:

```css
/* ================================================
   Code Connect Summit 2026 — Dark Theme
   ================================================ */

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* Quarto wraps content — reset its containers */
body, #quarto-content, main.content, .page-full {
  background: #0a0a1a !important;
  max-width: 100% !important;
  padding: 0 !important;
  margin: 0 !important;
  box-shadow: none !important;
  border-radius: 0 !important;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  color: #e2e8f0;
}

/* Hide Quarto title block — hero replaces it */
#title-block-header { display: none !important; }

/* ---- STICKY NAV ---- */
.ccs-nav {
  position: sticky; top: 0; z-index: 100;
  background: rgba(10, 10, 26, 0.92);
  backdrop-filter: blur(8px);
  border-bottom: 1px solid rgba(0, 229, 255, 0.15);
  padding: 14px 40px;
  display: flex; align-items: center; justify-content: space-between;
}
.ccs-nav-logo {
  color: #00e5ff; font-weight: 800; font-size: 15px; letter-spacing: 1px;
  text-decoration: none;
}
.ccs-nav-links { display: flex; gap: 28px; list-style: none; }
.ccs-nav-links a {
  color: #94a3b8; font-size: 13px; text-decoration: none;
  transition: color 0.15s;
}
.ccs-nav-links a:hover { color: #00e5ff; }
.ccs-nav-rsvp {
  background: linear-gradient(90deg, #7c3aed, #2563eb);
  color: #fff; border: none; border-radius: 5px;
  padding: 7px 18px; font-size: 13px; font-weight: 700;
  cursor: pointer; text-decoration: none; letter-spacing: 0.5px;
}

/* ---- HERO ---- */
.ccs-hero {
  background: linear-gradient(135deg, #0a0a1a 0%, #0d1b3e 45%, #1a0a2e 100%);
  padding: 90px 40px 70px;
  text-align: center;
  position: relative; overflow: hidden;
  border-bottom: 1px solid rgba(124, 58, 237, 0.25);
}
.ccs-hero::before {
  content: '';
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at 50% 0%, rgba(0, 229, 255, 0.07) 0%, transparent 65%);
  pointer-events: none;
}
.ccs-hero-label {
  font-size: 11px; letter-spacing: 5px; text-transform: uppercase;
  color: #7c3aed; margin-bottom: 18px;
}
.ccs-hero-title {
  font-size: 52px; font-weight: 800; line-height: 1.08; letter-spacing: -1.5px;
  background: linear-gradient(90deg, #00e5ff, #7c3aed);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 28px;
}
.ccs-hero-pills {
  display: flex; gap: 10px; justify-content: center; flex-wrap: wrap;
  margin-bottom: 24px;
}
.ccs-hero-pill {
  border: 1px solid rgba(0, 229, 255, 0.3); border-radius: 20px;
  padding: 5px 16px; font-size: 11px; letter-spacing: 1.5px;
  color: #00e5ff; text-transform: uppercase;
}
.ccs-hero-tagline {
  color: #94a3b8; font-size: 16px; max-width: 520px;
  margin: 0 auto 32px; line-height: 1.6;
}
.ccs-hero-meta {
  display: flex; gap: 36px; justify-content: center; flex-wrap: wrap;
  font-size: 14px; color: #cbd5e1; margin-bottom: 40px;
}
.ccs-hero-meta-item { display: flex; align-items: center; gap: 8px; }
.ccs-hero-meta-icon { color: #00e5ff; font-size: 16px; }
.ccs-hero-cta {
  display: inline-block;
  background: linear-gradient(90deg, #7c3aed, #2563eb);
  color: #fff; text-decoration: none; border-radius: 7px;
  padding: 14px 36px; font-size: 15px; font-weight: 700; letter-spacing: 0.5px;
  transition: opacity 0.15s;
}
.ccs-hero-cta:hover { opacity: 0.88; }

/* ---- SECTIONS ---- */
.ccs-section {
  max-width: 960px; margin: 0 auto; padding: 64px 40px;
}
.ccs-section-title {
  font-size: 11px; font-weight: 700; letter-spacing: 4px;
  text-transform: uppercase; color: #00e5ff;
  border-bottom: 1px solid rgba(0, 229, 255, 0.18);
  padding-bottom: 14px; margin-bottom: 32px;
}

/* ---- ABOUT ---- */
.ccs-about-text {
  color: #94a3b8; font-size: 16px; line-height: 1.75; max-width: 720px;
}

/* ---- SCHEDULE ---- */
.ccs-schedule-table {
  width: 100%; border-collapse: collapse; font-size: 14px;
}
.ccs-schedule-table th {
  text-align: left; color: #7c3aed; font-size: 10px;
  letter-spacing: 2.5px; text-transform: uppercase;
  padding: 8px 14px; border-bottom: 1px solid rgba(124, 58, 237, 0.3);
}
.ccs-schedule-table td {
  padding: 12px 14px; border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  color: #cbd5e1; vertical-align: top;
}
.ccs-schedule-table tr.ccs-row-break td { color: #475569; font-style: italic; }
.ccs-schedule-table tr.ccs-row-special td { background: rgba(124, 58, 237, 0.07); }
.ccs-time {
  white-space: nowrap; color: #00e5ff;
  font-family: 'SF Mono', 'Fira Code', monospace; font-size: 12px;
}
.ccs-talk-title { font-weight: 600; color: #e2e8f0; display: block; }
.ccs-talk-sub { color: #64748b; font-size: 12px; margin-top: 3px; display: block; }
.ccs-panel-questions {
  margin-top: 10px; padding-left: 0; list-style: none;
}
.ccs-panel-questions li {
  font-size: 12px; color: #64748b; padding: 3px 0 3px 14px; position: relative;
  line-height: 1.5;
}
.ccs-panel-questions li::before {
  content: counter(q) ".";
  counter-increment: q;
  position: absolute; left: 0; color: #7c3aed; font-size: 11px;
}
.ccs-panel-questions { counter-reset: q; }

/* ---- SPEAKERS ---- */
.ccs-speakers-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 14px;
}
.ccs-speaker-card {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(0, 229, 255, 0.1);
  border-radius: 9px; cursor: pointer;
  overflow: hidden; transition: border-color 0.2s;
}
.ccs-speaker-card:hover { border-color: rgba(0, 229, 255, 0.3); }
.ccs-speaker-card.open {
  grid-column: span 4;
  border-color: rgba(124, 58, 237, 0.5);
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.1), rgba(37, 99, 235, 0.07));
}
.ccs-speaker-front { padding: 18px; }
.ccs-speaker-avatar {
  width: 44px; height: 44px; border-radius: 50%;
  background: linear-gradient(135deg, #7c3aed, #2563eb);
  margin-bottom: 12px; overflow: hidden;
  display: flex; align-items: center; justify-content: center;
}
.ccs-speaker-avatar img { width: 100%; height: 100%; object-fit: cover; }
.ccs-speaker-name { font-weight: 700; font-size: 14px; color: #e2e8f0; margin-bottom: 4px; }
.ccs-speaker-talk { font-size: 12px; color: #7c3aed; line-height: 1.4; }

.ccs-speaker-detail {
  display: none;
  padding: 0 18px 18px;
  grid-template-columns: 140px 1fr;
  gap: 24px; align-items: start;
}
.ccs-speaker-card.open .ccs-speaker-detail { display: grid; }
.ccs-speaker-photo {
  width: 100%; aspect-ratio: 1; border-radius: 8px; overflow: hidden;
  background: linear-gradient(135deg, #1e3a5f, #1a0a2e);
  display: flex; align-items: center; justify-content: center;
  color: #475569; font-size: 11px; letter-spacing: 1px;
}
.ccs-speaker-photo img { width: 100%; height: 100%; object-fit: cover; }
.ccs-speaker-bio-name {
  font-size: 18px; font-weight: 700; color: #e2e8f0; margin-bottom: 4px;
}
.ccs-speaker-bio-talk {
  font-size: 12px; color: #7c3aed; margin-bottom: 12px;
}
.ccs-speaker-bio-text {
  font-size: 14px; color: #94a3b8; line-height: 1.65;
}

/* ---- RSVP ---- */
.ccs-rsvp-box {
  background: linear-gradient(135deg, rgba(124, 58, 237, 0.12), rgba(37, 99, 235, 0.12));
  border: 1px solid rgba(124, 58, 237, 0.35);
  border-radius: 14px; padding: 48px 40px; text-align: center;
}
.ccs-rsvp-box p { color: #94a3b8; font-size: 16px; margin-bottom: 28px; line-height: 1.6; }
.ccs-rsvp-btn {
  display: inline-block;
  background: linear-gradient(90deg, #7c3aed, #2563eb);
  color: #fff; text-decoration: none; border-radius: 7px;
  padding: 14px 36px; font-size: 15px; font-weight: 700;
  transition: opacity 0.15s;
}
.ccs-rsvp-btn:hover { opacity: 0.88; }

/* ---- CONTACT ---- */
.ccs-contact-text { color: #94a3b8; font-size: 15px; line-height: 1.8; }
.ccs-contact-text a { color: #00e5ff; text-decoration: none; }
.ccs-contact-text a:hover { text-decoration: underline; }

/* ---- FOOTER ---- */
.ccs-footer {
  text-align: center; padding: 36px 40px;
  border-top: 1px solid rgba(255, 255, 255, 0.06);
  color: #475569; font-size: 12px; letter-spacing: 0.5px;
}
.ccs-footer-logo { color: #7c3aed; font-weight: 700; margin-bottom: 6px; font-size: 14px; }
```

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/style.css
git commit -m "feat(summit): add dark theme CSS for conference site"
```

---

## Task 3: Create `index.qmd` — structure and nav/hero

**Files:**
- Create: `docs/code_connect_summit/index.qmd`

- [ ] **Step 1: Create the file with YAML front matter, nav, and hero**

Create `docs/code_connect_summit/index.qmd`:

````markdown
---
title: "Code Connect Summit 2026"
---

```{=html}
<!-- NAV -->
<nav class="ccs-nav">
  <a class="ccs-nav-logo" href="#">⬡ synthesis</a>
  <ul class="ccs-nav-links">
    <li><a href="#about">About</a></li>
    <li><a href="#schedule">Schedule</a></li>
    <li><a href="#speakers">Speakers</a></li>
    <li><a href="#contact">Contact</a></li>
  </ul>
  <a class="ccs-nav-rsvp" href="https://forms.office.com/Pages/ResponsePage.aspx?id=hdvPlCM9SUigZlza2WXM2AYv7RPSF49BsSdfvCGj1WdUQTJHV1Q1SUtVSjlDMVBZQTdOMTdUVlNWMS4u">RSVP Now</a>
</nav>

<!-- HERO -->
<div class="ccs-hero">
  <div class="ccs-hero-label">Conference · 2026</div>
  <h1 class="ccs-hero-title">Code Connect<br>Summit</h1>
  <div class="ccs-hero-pills">
    <span class="ccs-hero-pill">Presentations</span>
    <span class="ccs-hero-pill">Panel</span>
    <span class="ccs-hero-pill">Keynote</span>
    <span class="ccs-hero-pill">Workshop</span>
  </div>
  <p class="ccs-hero-tagline">An extraordinary showcase of innovation for the Synthesis tech team.</p>
  <div class="ccs-hero-meta">
    <span class="ccs-hero-meta-item"><span class="ccs-hero-meta-icon">⏰</span> June 5, 2026 &nbsp;·&nbsp; 09:00–16:30</span>
    <span class="ccs-hero-meta-item"><span class="ccs-hero-meta-icon">📍</span> Synthesis Office &nbsp;·&nbsp; JHB &amp; CPT</span>
    <span class="ccs-hero-meta-item"><span class="ccs-hero-meta-icon">🏛</span> Training Rooms</span>
  </div>
  <a class="ccs-hero-cta" href="https://forms.office.com/Pages/ResponsePage.aspx?id=hdvPlCM9SUigZlza2WXM2AYv7RPSF49BsSdfvCGj1WdUQTJHV1Q1SUtVSjlDMVBZQTdOMTdUVlNWMS4u">RSVP Now →</a>
</div>
```
````

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/index.qmd
git commit -m "feat(summit): add nav and hero sections"
```

---

## Task 4: Add About and Schedule sections to `index.qmd`

**Files:**
- Modify: `docs/code_connect_summit/index.qmd`

- [ ] **Step 1: Append About and Schedule HTML blocks**

Append the following to the end of `docs/code_connect_summit/index.qmd`:

````markdown

```{=html}
<!-- ABOUT -->
<section id="about" class="ccs-section">
  <div class="ccs-section-title">About</div>
  <p class="ccs-about-text">
    Code Connect Summit brings together the Synthesis tech team for a full day of knowledge sharing, debate, and hands-on learning.
    Featuring presentations, a keynote, a panel on the future of software engineering in the age of AI, and a hands-on workshop.
    Hosted simultaneously in Johannesburg and Cape Town.
  </p>
</section>

<!-- SCHEDULE -->
<section id="schedule" class="ccs-section">
  <div class="ccs-section-title">Schedule</div>
  <table class="ccs-schedule-table">
    <thead>
      <tr>
        <th>Time</th>
        <th>Session</th>
        <th>Presenter(s)</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td class="ccs-time">09:00–09:10</td>
        <td><span class="ccs-talk-title">Welcome</span></td>
        <td>Kieron &amp; Archie</td>
      </tr>
      <tr>
        <td class="ccs-time">09:10–09:40</td>
        <td><span class="ccs-talk-title">Keynote</span></td>
        <td>Tom</td>
      </tr>
      <tr>
        <td class="ccs-time">09:45–10:15</td>
        <td>
          <span class="ccs-talk-title">Streaming in a High-Pressure Environment</span>
          <span class="ccs-talk-sub">i.e. Fraud</span>
        </td>
        <td>Vincent &amp; Enoch</td>
      </tr>
      <tr>
        <td class="ccs-time">10:20–10:50</td>
        <td><span class="ccs-talk-title">LLM in Document Understanding</span></td>
        <td>Marcus &amp; Massimo</td>
      </tr>
      <tr class="ccs-row-break">
        <td class="ccs-time">10:50–11:10</td>
        <td colspan="2">☕ Break</td>
      </tr>
      <tr class="ccs-row-special">
        <td class="ccs-time">11:10–11:55</td>
        <td>
          <span class="ccs-talk-title">Panel: The Future of Software Engineering in the Age of AI</span>
          <span class="ccs-talk-sub">Facilitated by Kieron &amp; Archie &nbsp;·&nbsp; Panelists: Tom, Tjaard, Marais, MikeG</span>
          <ol class="ccs-panel-questions">
            <li>What is the future when the funding runs out for tools?</li>
            <li>Are we overestimating short-term impact and underestimating long-term disruption—or the opposite?</li>
            <li>Who is accountable when AI-generated systems fail in production?</li>
            <li>Are we heading toward more standardized architectures because AI performs better within constraints?</li>
            <li>Where should humans remain "in the loop" no matter how good AI gets?</li>
            <li>Are we moving toward fewer engineers with higher leverage, or fundamentally different roles altogether?</li>
          </ol>
        </td>
        <td></td>
      </tr>
      <tr>
        <td class="ccs-time">12:00–12:30</td>
        <td><span class="ccs-talk-title">Running Kubernetes at Home the Easy Way, Because Overcomplicating Things Is a Lifestyle Choice</span></td>
        <td>Rodney</td>
      </tr>
      <tr class="ccs-row-break">
        <td class="ccs-time">12:30–13:30</td>
        <td colspan="2">🍽 Lunch Break</td>
      </tr>
      <tr>
        <td class="ccs-time">13:30–14:00</td>
        <td><span class="ccs-talk-title">Building a Roblox Game with My Son — What Broke, What Scaled, What Didn't</span></td>
        <td>Tjaard</td>
      </tr>
      <tr>
        <td class="ccs-time">14:05–14:35</td>
        <td><span class="ccs-talk-title">Self-Healing Codebases</span></td>
        <td>Nick Driver</td>
      </tr>
      <tr class="ccs-row-special">
        <td class="ccs-time">14:40–15:40</td>
        <td><span class="ccs-talk-title">Workshop: Smart Trainer — Realtime AI Fitness Trainer</span></td>
        <td>Jason</td>
      </tr>
      <tr class="ccs-row-break">
        <td class="ccs-time">15:40–16:00</td>
        <td colspan="2">☕ Break</td>
      </tr>
      <tr>
        <td class="ccs-time">16:00–16:30</td>
        <td><span class="ccs-talk-title">Final Thank You &amp; Prizes</span></td>
        <td>Kieron &amp; Archie</td>
      </tr>
    </tbody>
  </table>
</section>
```
````

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/index.qmd
git commit -m "feat(summit): add about and schedule sections"
```

---

## Task 5: Add Speakers section to `index.qmd`

**Files:**
- Modify: `docs/code_connect_summit/index.qmd`

- [ ] **Step 1: Append Speakers HTML block and JS**

Append the following to the end of `docs/code_connect_summit/index.qmd`:

````markdown

```{=html}
<!-- SPEAKERS -->
<section id="speakers" class="ccs-section">
  <div class="ccs-section-title">Speakers</div>
  <div class="ccs-speakers-grid" id="speakersGrid">

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Tom</div>
        <div class="ccs-speaker-talk">Keynote</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Tom</div>
          <div class="ccs-speaker-bio-talk">Keynote</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Vincent &amp; Enoch</div>
        <div class="ccs-speaker-talk">Streaming in a High-Pressure Environment</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Vincent &amp; Enoch</div>
          <div class="ccs-speaker-bio-talk">Streaming in a High-Pressure Environment (i.e. Fraud)</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Marcus &amp; Massimo</div>
        <div class="ccs-speaker-talk">LLM in Document Understanding</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Marcus &amp; Massimo</div>
          <div class="ccs-speaker-bio-talk">LLM in Document Understanding</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Rodney</div>
        <div class="ccs-speaker-talk">Running Kubernetes at Home the Easy Way</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Rodney</div>
          <div class="ccs-speaker-bio-talk">Running Kubernetes at Home the Easy Way, Because Overcomplicating Things Is a Lifestyle Choice</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Tjaard</div>
        <div class="ccs-speaker-talk">Building a Roblox Game with My Son</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Tjaard</div>
          <div class="ccs-speaker-bio-talk">Building a Roblox Game with My Son — What Broke, What Scaled, What Didn't</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Nick Driver</div>
        <div class="ccs-speaker-talk">Self-Healing Codebases</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Nick Driver</div>
          <div class="ccs-speaker-bio-talk">Self-Healing Codebases</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

    <div class="ccs-speaker-card" onclick="expandSpeaker(this)">
      <div class="ccs-speaker-front">
        <div class="ccs-speaker-avatar"></div>
        <div class="ccs-speaker-name">Jason</div>
        <div class="ccs-speaker-talk">Workshop: Smart Trainer</div>
      </div>
      <div class="ccs-speaker-detail">
        <div class="ccs-speaker-photo">Photo coming soon</div>
        <div>
          <div class="ccs-speaker-bio-name">Jason</div>
          <div class="ccs-speaker-bio-talk">Workshop: Smart Trainer — Realtime AI Fitness Trainer</div>
          <div class="ccs-speaker-bio-text">Bio coming soon.</div>
        </div>
      </div>
    </div>

  </div>
</section>

<script>
function expandSpeaker(card) {
  var grid = document.getElementById('speakersGrid');
  var cards = grid.querySelectorAll('.ccs-speaker-card');
  var isOpen = card.classList.contains('open');
  cards.forEach(function(c) { c.classList.remove('open'); });
  if (!isOpen) { card.classList.add('open'); }
}
</script>
```
````

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/index.qmd
git commit -m "feat(summit): add speakers grid with expand-in-place interaction"
```

---

## Task 6: Add RSVP, Contact, and Footer sections to `index.qmd`

**Files:**
- Modify: `docs/code_connect_summit/index.qmd`

- [ ] **Step 1: Append RSVP, Contact, and Footer HTML blocks**

Append the following to the end of `docs/code_connect_summit/index.qmd`:

````markdown

```{=html}
<!-- RSVP -->
<section id="rsvp" class="ccs-section">
  <div class="ccs-section-title">RSVP</div>
  <div class="ccs-rsvp-box">
    <p>Secure your spot for Code Connect Summit 2026.<br>June 5th · Synthesis Office JHB &amp; CPT · 09:00–16:30</p>
    <a class="ccs-rsvp-btn" href="https://forms.office.com/Pages/ResponsePage.aspx?id=hdvPlCM9SUigZlza2WXM2AYv7RPSF49BsSdfvCGj1WdUQTJHV1Q1SUtVSjlDMVBZQTdOMTdUVlNWMS4u">Register Now →</a>
  </div>
</section>

<!-- CONTACT -->
<section id="contact" class="ccs-section">
  <div class="ccs-section-title">Contact</div>
  <div class="ccs-contact-text">
    Questions? Reach out to the hosts:<br>
    <a href="mailto:kieron@synthesis.co.za">kieron@synthesis.co.za</a>
  </div>
</section>

<!-- FOOTER -->
<footer class="ccs-footer">
  <div class="ccs-footer-logo">⬡ synthesis</div>
  Code Connect Summit 2026 &nbsp;·&nbsp; Synthesis Software Technologies
</footer>
```
````

- [ ] **Step 2: Commit**

```bash
git add docs/code_connect_summit/index.qmd
git commit -m "feat(summit): add RSVP, contact, and footer sections"
```

---

## Task 7: Update CI workflow to render `index.qmd`

**Files:**
- Modify: `.github/workflows/build-docs.yml`

The current render loop (line 33–35) only matches `<doc_name>.md`. The conference site uses `index.qmd`. Update the loop to detect `index.qmd` first.

- [ ] **Step 1: Update the render loop in `build-docs.yml`**

Find this block in `.github/workflows/build-docs.yml`:

```yaml
      - name: Render all documents
        run: |
          for doc_dir in docs/*/; do
            doc_name="$(basename "$doc_dir")"
            [ -f "$doc_dir/$doc_name.md" ] || continue
            echo "Rendering: $doc_name"
            (cd "$doc_dir" && quarto render "$doc_name.md")
          done
```

Replace with:

```yaml
      - name: Render all documents
        run: |
          for doc_dir in docs/*/; do
            doc_name="$(basename "$doc_dir")"
            if [ -f "$doc_dir/index.qmd" ]; then
              echo "Rendering: $doc_name (index.qmd)"
              (cd "$doc_dir" && quarto render "index.qmd")
            elif [ -f "$doc_dir/$doc_name.md" ]; then
              echo "Rendering: $doc_name"
              (cd "$doc_dir" && quarto render "$doc_name.md")
            fi
          done
```

- [ ] **Step 2: Commit**

```bash
git add .github/workflows/build-docs.yml
git commit -m "ci: render index.qmd files in addition to <name>.md"
```

---

## Task 8: Local build verification

**Files:** None modified.

- [ ] **Step 1: Build locally with Docker**

From the repo root:

```bash
./tools/build.sh code_connect_summit
```

Expected: script exits 0, output appears at `dist/code_connect_summit/index.html`.

If Docker is not available, install Quarto locally and run:

```bash
cd docs/code_connect_summit && quarto render index.qmd
```

- [ ] **Step 2: Open in browser and verify each section**

Open `dist/code_connect_summit/index.html` in a browser. Check:

- [ ] Sticky nav visible, links scroll to correct sections
- [ ] Hero: title gradient renders, pills visible, RSVP button links to form
- [ ] About: paragraph visible
- [ ] Schedule: all 13 rows present, break rows muted, panel and workshop rows highlighted, panel questions listed
- [ ] Speakers: 7 cards in grid; clicking each expands to full width; clicking again or clicking a different card collapses
- [ ] RSVP: button links to Microsoft Forms URL
- [ ] Contact: `kieron@synthesis.co.za` link present
- [ ] Footer: Synthesis logo and text visible

- [ ] **Step 3: Commit if any CSS fixups were needed**

```bash
git add docs/code_connect_summit/style.css
git commit -m "fix(summit): CSS adjustments from local build review"
```

(Skip this step if no changes were needed.)

---

## Adding Speaker Photos and Bios Later

When speaker content is ready, update each speaker card in `index.qmd`:

**Photo** — replace `<div class="ccs-speaker-photo">Photo coming soon</div>` with:
```html
<div class="ccs-speaker-photo"><img src="speakers/name.jpg" alt="Name"></div>
```
Place images in `docs/code_connect_summit/speakers/`. Quarto copies static assets alongside the output.

**Bio** — replace `<div class="ccs-speaker-bio-text">Bio coming soon.</div>` with actual bio text. No other changes needed.
