# CLAUDE.md

This repo is the SofaBuffers organization landing page — a single static `index.html` (plain HTML/CSS/JS, **no build step**) published via GitHub Pages from the `main` branch root.

## When the user says "update the page"

Read **[MAINTENANCE.md](MAINTENANCE.md)** and follow it. It is the playbook for keeping the page's facts (supported languages, repo links, format/spec details) in sync with the source repos, and lists every place each data point appears.

## Hard rules

- Plain HTML/CSS/JS only — never add a framework, bundler, or external dependency.
- Keep the two slogans verbatim (see MAINTENANCE.md §4).
- No Claude/AI attribution in committed files or commit messages.
- Merging to `main` deploys to production — only merge when the user explicitly confirms.
