# Keynote Ideas

A place to capture raw ideas, angles, and observations to weave into the talk.
Not structured yet — just things worth saying out loud.

---

## 1. Rethinking the OS for Agentic Development

**The idea:** macOS and Windows were designed for single-user, single-task computing. In the world of agentic software development, we're running many parallel workflows simultaneously — and our operating system isn't keeping up.

**The angle:** I've been experimenting with [Omarchy](https://omarchy.org) + Wayland + Hyprland as a development OS, and it's genuinely better for this style of work. Tiling window managers, keyboard-driven navigation, and a compositing pipeline built for multiple concurrent workspaces change how you move through context.

**A real example from my day:**
- Building this presentation
- Writing an Araxi strategy document
- Doing active development on a project — simulating a new architecture, writing a review document, distilling it into a deck, and building a reference implementation

All of these running in parallel, context-switching fluidly between them — plus 1:1s, meetings, and coffee.

That's not a hypothetical workload. That's Tuesday. And most operating systems treat it like an exceptional case rather than the norm.

**The point:** We need an OS optimised for *many concurrent agentic workflows*, not just one human in front of one window. The primitives matter — workspaces, compositing, input routing, notification management. The tools exist. We just haven't assembled them intentionally for this use case yet.

**Possible framing:** "The terminal was the right OS for the pre-GUI era. What's the right OS for the agentic era?"

---

## 2. Every Task Needs a Software Development Pipeline

**The idea:** The SDLC — source control, review, CI/CD, publishing — isn't just for code. Every knowledge work artifact should be run through the same pipeline. Documents, presentations, strategy papers, architecture reviews. All of it.

**The old way:** Draft a Word doc → save to SharePoint → attach to email → send for review → receive 14 conflicting tracked-changes versions → cry. This workflow was already painful for humans. For agents it's a dead end: they can't navigate document formats reliably, SharePoint is opaque to them, and email as a storage and collaboration primitive is essentially inaccessible.

**The better way:** Source text (e.g. [Typst](https://typst.app/) or Markdown) → build pipeline → rendered PDF/HTML → committed to a git repository → published to GitHub Pages. This is the same pipeline we already use for software, just applied to documents.

**Why agents love this:**
- Plain text source means agents can read, edit, and diff with the same tools they use for code
- Git prevents conflicts and overwrites — the same guarantees we rely on for source code apply here
- History is first-class: humans can follow along, review changes, and understand what changed and why without any special tooling
- Publishing is just a pipeline step — a PR merge can trigger a deploy to Pages, just like a software release

**The skill angle:** This is also where skills become powerful multipliers. A `typst-document` skill or `technical-presentation` skill encodes the entire pipeline approach once — the right source format, the build toolchain, the repo structure, the publish step — and any agent can pick it up and apply it consistently. You write the SDLC once; the skill carries it everywhere.

**Broader point:** The move here isn't "use better document tools." It's to recognise that *every artifact we produce is software* — it has source, it has a build, it has a release. Once you internalise that, the right tooling becomes obvious.

**Possible framing:** "If your document can't be diffed, it can't be collaborated on by agents. If it can't be diffed, it's legacy."

---

## 3. Agents Need Engineering Principles for Knowledge Work (The BA Pipeline)

**The idea:** Business analysis — reading source material, synthesising it, producing findings and recommendations — has always been knowledge work done in the dark. One person, one desktop, one Word doc that eventually gets emailed around. Agents are starting to do this work, and if we don't give them better underlying engineering principles, we just automate the opacity.

**The old BA workflow:**
- Analyst reads a pile of source documents on their laptop
- Synthesises findings privately, in their own mental model
- Produces a document or slide deck
- Distributes it — at which point the reasoning that produced it is completely invisible

No one can see the source material that was considered, what was excluded, or how the conclusions were reached. The artifact is the only thing that survives.

**The same problem, amplified with agents:** If an agent does this work inside a chat window and hands you a PDF, you've got the same opacity problem — worse, because there's no human whose judgment you can interrogate. The output exists but the pipeline is black box.

**The better model — a traceable BA pipeline in source control:**
```
source documents  →  analysis notes  →  findings  →  recommendations  →  published artifact
```
Each stage is a committed artifact. The links between them are explicit. You can see what sources informed what analysis, what analysis produced what findings. A human reviewer — or another agent — can audit any step. Agents can collaborate across stages without clobbering each other's work.

**Why this matters beyond BA:** This is the same principle as idea #2, applied to a workflow rather than just a document type. The insight generalises: *any multi-step knowledge work process should be a pipeline with committed intermediate artifacts*, not a private cognitive journey that surfaces only at the end. Source → analysis → findings → recommendations is just one instance of a broader pattern.

**The transparency angle:** Git history becomes an audit trail. "Why did we recommend X?" has a traceable answer: here's the source material we read, here's the analysis we derived, here's the finding that led to that recommendation. That's not just good for agents — it's good for organisations.

**Possible framing:** "We spent decades trying to make knowledge work collaborative. Agents give us the chance to make it *transparent*. But only if we treat it like engineering."

---

## 4. Encoding Architectural Constraints into Source Code (Effect Systems)

**The idea:** Agents write code that works — but they don't inherently respect your architecture. Effect systems let you encode architectural constraints directly into the type system, so violations are impossible rather than just discouraged.

**The problem:** An agent writing a new feature will reach across layer boundaries if the path of least resistance leads that way. A domain function calls the database directly. A logging side effect is buried three layers deep. It compiles. The tests pass. Nobody notices until two weeks later — because there was nothing structural to notice.

Three compounding problems:
1. Agents are **architecturally blind** — they write locally plausible code that violates global structure
2. **Side effects are invisible** — function signatures tell you inputs/outputs but not whether something writes to the DB, sends an email, or calls an external API
3. **Context windows miss architectural intent** — even large windows give agents a lossy view of a codebase's structure

**The effect systems answer:** Make effects part of the type. A function declares the effects it's permitted to use; the compiler enforces it. You can't accidentally do more than you declared.

```haskell
-- The full story, from the type alone:
processOrder
  :: (PaymentGateway :> es, EmailNotification :> es, OrderRepo :> es)
  => Order -> Eff es Result
```

**Why this matters for agents specifically:**
- **Architecture as guardrails:** Layer boundaries are enforced structurally. An agent writing at the API layer cannot reach the database — it's not in scope, the compiler rejects it. "You write the architecture once, in types. Every agent that ever touches this codebase is constrained by it, forever, for free."
- **Effects taint:** When an agent introduces a new side effect, it surfaces immediately in the type. You can't sneak a database write into a function that wasn't supposed to touch the database. Code review becomes effect review.
- **Types as compressed architecture:** You can describe your entire architecture to an agent in a few dozen type signatures — fits easily in any context window. Agents don't need to read the whole codebase; they need to read the types. "Types are a compression format for architectural intent."
- **Compiler as co-pilot:** The type signature is a machine-verified spec. The agent can't implement a function wrong without the compiler telling it. Each failed compilation is a guardrail working.

**Ecosystem options:** Haskell + `effectful` (strongest guarantees), Effect-TS for TypeScript (production-ready, accessible), ZIO for Scala, `effect` for Python.

**Full talk:** This one has its own 15-minute lightning talk — [Architecting Agentic Sandpits](https://drshade.github.io/talk_agentic_effect_systems/) — which goes deep on the Haskell implementation and the interpreter-as-layer-boundary insight.

**Possible framing:** "Stop trying to prompt agents into respecting your architecture. Make violating it structurally impossible."

---

## 5. Let Agents Build the Complexity So Humans (and Agents) Don't Have To

**The idea:** Agents can understand and implement sophisticated type systems and infrastructure abstractions better than most developers can maintain them day-to-day. Use that capability to build expressive, deeply type-safe frameworks once — then give both agents and humans a tiny, clean surface to work on top of.

**The example:** The `fence-occ-poc-simulator` project. The underlying architecture is genuinely hard — a distributed CQRS system on a commit log with causal dependency tracking (vector clocks), Optimistic Concurrency Control, deferred retry queues, Kafka Streams topologies. Lots of traps. Lots of protocol invariants that are easy to violate.

But adding a new domain entity to this system looks like this:

```kotlin
object InvoiceAggregate : Aggregate<
    InvoiceCommand.Create,
    InvoiceCommand.Mutate,
    InvoiceEvent.ForCreate,
    InvoiceEvent.ForMutate,
    InvoiceEvent,
    Invoice,
    InvoiceDeps,
> {
    override fun processCreate(cmd: InvoiceCommand.Create, deps: InvoiceDeps): InvoiceEvent.ForCreate = ...
    override fun processMutate(cmd: InvoiceCommand.Mutate, entity: Versioned<Invoice>, deps: InvoiceDeps): InvoiceEvent.ForMutate = ...
    override fun evolveCreate(event: InvoiceEvent.ForCreate): Versioned<Invoice> = ...
    override fun evolveMutate(entity: Versioned<Invoice>, event: InvoiceEvent.ForMutate): Versioned<Invoice>? = ...
}
```

Four pure functions. Domain types in, domain types out. The framework guarantees that `processCreate` is only called after the fence check passes, `processMutate` only after dep check + existence check + OCC version check. None of that protocol complexity is visible here. It cannot be violated from here because it isn't accessible from here.

**The DX principle:** The `Aggregate<CreateCmd, MutateCmd, CreateEvt, MutateEvt, Evt, Entity, DepView>` interface is not simple — it has seven type parameters. But it compresses the entire architecture into a shape that both agents and developers can implement correctly. The compiler guides you to a correct implementation. You cannot accidentally produce a `MutateEvt` from `processCreate` — the types won't allow it.

**What agents bring to this:** Building and maintaining this kind of interface is hard. Most teams wouldn't invest the upfront design effort, and they'd drift back to looser patterns over time. Agents can sustain that investment — they can work at the level of abstraction required, follow through on type-level design decisions that humans would shortcut, and keep the framework consistent as it grows. Once the framework exists, the agentic surface for adding new entities is small, well-defined, and type-checked. The agent implementing `InvoiceAggregate` doesn't need to understand Kafka or vector clocks — it just needs to implement four functions correctly.

**The inversion:** We usually think about agents as junior developers who need guardrails. This flips it: agents are the architects who build the guardrails. Agents design the framework; agents and humans alike use it.

**Possible framing:** "Don't simplify the problem. Build a type system so good that the complexity disappears at the surface. Agents are finally capable of building that kind of infrastructure."

---
