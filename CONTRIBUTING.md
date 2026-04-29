# Contributing

## Repository structure

Each document lives in its own folder under `docs/` alongside its own Quarto config and stylesheet:

```
docs/
  <doc-name>/
    <doc-name>.md       Source document
    _quarto.yml         Quarto config (formats, output path)
    style.css           Document-specific stylesheet
tools/
  build.sh              Build script (requires Docker)
dist/
  <doc-name>/           Generated artefacts — HTML and PDF (git-ignored)
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

## GitHub Pages

On every push to `main` that touches `docs/`, the [build-docs workflow](.github/workflows/build-docs.yml) renders all documents and deploys them to [GitHub Pages](https://synthesis-labs.github.io/techteam-standards/).

**One-time setup** (repo admin): Settings → Pages → Source → **GitHub Actions**.
