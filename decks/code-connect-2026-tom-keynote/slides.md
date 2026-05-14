---
theme: default
title: Claude + Git + Skills is all you need
info: |
  Code Connect Summit 2026 — Tom Wells
  Or how to become an Engineering Manager for great good
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
mdc: true
fonts:
  sans: Inter
  mono: JetBrains Mono
layout: cover
subtitle: Or how to become an Engineering Manager for great good
author: "Tom Wells"
date: "TBD"
eyebrow: "CODE CONNECT SUMMIT · 2026"
organisation: "Synthesis"
logo: /synthesis-logo.png
---

<style>
:root {
  --deck-navy: #0D1120;
  --deck-navy-deep: #162440;
  --deck-blue-accent: #4D7EF7;
  --deck-blue-pale: #A5B4FC;
  --deck-body-ink: #1A1F2E;
  --deck-muted: #6B7280;
  --deck-border-cool: #E2E8F0;
  --deck-tip-bg: #EEF2FF;
  --deck-tip-border: #4D7EF7;
  --deck-tip-title: #1D3476;
  --deck-warn-bg: #FFF5F0;
  --deck-warn-border: #C2400C;
}

.slidev-layout {
  font-family: Inter, "Helvetica Neue", system-ui, sans-serif;
  color: var(--deck-body-ink);
}

.slidev-layout h1 {
  color: var(--deck-navy);
  font-weight: 600;
  letter-spacing: -0.01em;
}

.slidev-layout h2 {
  color: var(--deck-navy);
  font-weight: 600;
}

.slidev-layout.default h1::after {
  content: "";
  display: block;
  width: 56px;
  height: 3px;
  background: var(--deck-blue-accent);
  margin-top: 0.4em;
  margin-bottom: 0.2em;
}

.slidev-layout code {
  font-family: "JetBrains Mono", "Fira Code", monospace;
}

.slidev-layout pre {
  font-size: 0.85em;
  border-radius: 6px;
}

.slidev-layout table {
  font-size: 0.95em;
}

.slidev-layout th {
  color: var(--deck-navy);
  font-weight: 600;
  text-align: left;
  border-bottom: 2px solid var(--deck-border-cool);
  padding: 0.5em 0.8em;
}

.slidev-layout td {
  border-bottom: 1px solid var(--deck-border-cool);
  padding: 0.5em 0.8em;
}

.text-accent { color: var(--deck-blue-accent); }
.text-muted  { color: var(--deck-muted); }
.text-navy   { color: var(--deck-navy); }
</style>

---
layout: center
class: text-center
---

<PullQuote>
A short, memorable statement that lives <em>alone</em> on the slide.
</PullQuote>

---

# Three things this talk shows

<v-clicks>

1. First point — what's at stake.
2. Second point — the mechanism.
3. Third point — the payoff.

</v-clicks>

<div class="mt-12 text-muted text-sm">
A one-line setup for what's coming.
</div>

---
layout: section
number: "01"
title: First section title
subtitle: Optional one-line explainer.
---

---

# A content slide with code

```kotlin
fun greet(name: String) = "Hello, $name"
```

<div class="mt-6">
Body paragraph below the code. Keep code blocks under ~12 lines.
</div>

---

<Principle title="Why this matters">
Use <code>Principle</code> for key takeaways and design rules. Blue left-border card.
</Principle>

---

<Caveat>
Use <code>Caveat</code> for trade-offs, warnings, or "but watch out for" notes. Rust left-border card.
</Caveat>

---
layout: end
title: Thank you
eyebrow: QUESTIONS WELCOME
speaker: "Tom Wells"
organisation: "Synthesis"
email: "tom@synthesis.co.za"
footer: "Code Connect Summit · TBD"
---
