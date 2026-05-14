# Contributing

## Repository structure

Three artifact types live side by side:

- **Documents** — long-form HTML written in Quarto under `docs/`.
- **Sites** — standalone Quarto pages or microsites under `sites/`.
- **Decks** — slide presentations written in [Slidev](https://sli.dev) under `decks/`.

```
docs/
  <doc-name>/
    <doc-name>.md       Source document
    _quarto.yml         Quarto config (formats, output path)
    style.css           Document-specific stylesheet
sites/
  <site-name>/
    index.qmd           Site entry point
    _quarto.yml         Quarto config (formats, output path)
    style.css           Site-specific stylesheet
    images/             Site-specific image assets
decks/
  <deck-name>/
    slides.md           Slidev source (markdown + per-slide YAML frontmatter)
    package.json        Slidev + Vue dependencies
    Makefile            install / dev / build / export targets
    layouts/            Custom layouts (cover, section, end)
    components/         Reusable Vue components
    public/             Static assets (logos, images)
tools/
  build.sh              Document build script (requires Docker)
dist/
  <doc-name>/           Generated document artefacts — HTML (git-ignored)
  sites/<site-name>/    Generated Quarto site artefacts (git-ignored, CI-only)
  decks/<deck-name>/    Generated deck SPA (git-ignored, CI-only)
```

## Generating documents locally

Documents are rendered to HTML and PDF using [Quarto](https://quarto.org) via Docker, so no local Quarto installation is required — only Docker.

Build all documents:
```bash
./tools/build.sh
```

Build a single document:
```bash
./tools/build.sh ai_maturity_field_guide
```

Output lands in `dist/<doc-name>/`:
- `dist/<doc-name>/<doc-name>.html` — self-contained HTML (no external dependencies)

### Requirements

- [Docker](https://docs.docker.com/get-docker/) — the build script pulls `ghcr.io/quarto-dev/quarto:latest` automatically on first run.

## Adding a new document

1. Create `docs/<name>/` with `<name>.md`, a `_quarto.yml` (copy from an existing doc and update `output-dir`), and a `style.css`.
2. Run `./tools/build.sh <name>` to test locally.
3. Push to `main` — the [build-docs workflow](.github/workflows/build-docs.yml) will render and publish automatically.

## Working with sites

Sites are standalone Quarto pages or microsites. Each site uses `index.qmd` as its entry point and writes to `dist/sites/<site-name>/`.

```bash
cd sites/<site-name>
quarto render index.qmd
```

## Working with decks

Decks are built with Slidev (Node.js). Requires Node 20+.

```bash
cd decks/<deck-name>
make install      # one-time, downloads ~280MB (playwright-chromium for PDF export)
make dev          # live preview at http://localhost:3030
make build        # static SPA in dist/
make export       # render to slides.pdf
```

Each deck ships with an `AGENTS.md` describing its layouts, components, and editing conventions.

## GitHub Pages

On every push to `main` that touches `docs/`, `sites/`, or `decks/`, the [build-docs workflow](.github/workflows/build-docs.yml) renders all documents, renders all sites, builds all decks, and deploys the combined site to [GitHub Pages](https://synthesis-labs.github.io/techteam-standards/). Sites publish at `/sites/<site-name>/`; decks publish at `/decks/<deck-name>/`.

**One-time setup** (repo admin): Settings → Pages → Source → **GitHub Actions**.
