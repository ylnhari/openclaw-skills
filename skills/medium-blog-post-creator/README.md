# Medium Blog Post Creator

> **Publish blog posts to Medium via GitHub Pages + URL import — no API token required.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/ylnhari/openclaw-skills/blob/main/LICENSE)
[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-blueviolet)](https://docs.openclaw.ai/clawhub)

An [OpenClaw](https://openclaw.ai) skill that publishes a fully formatted blog
post to **Medium** — end-to-end, agent-driven, no API token, no scripts to copy.

## Why it exists

Medium stopped issuing new API integration tokens on **January 1, 2025**, so the
legacy Medium API tools are unusable for new users. This skill uses the only
stable path that remains: author a constrained HTML post, deploy it as a public
URL via GitHub Pages, and let Medium's own **URL-import** tool
(`medium.com/p/import`) fetch and convert it into blocks. It always stops at
**draft** for your review — it never auto-publishes.

## Install

From ClawHub (skills install by slug):

```bash
openclaw skills install medium-blog-post-creator
```

Or from a local checkout of this repository:

```bash
openclaw skills install ./skills/medium-blog-post-creator --as medium-blog-post-creator
```

## Prerequisites

- A **GitHub account** and a **Medium account**
- **`gh` CLI** installed and authenticated (`gh auth login`) — [install](https://cli.github.com)
- **`git`**
- The **OpenClaw `browser` tool**, signed into your Medium account

`gh` and `git` are declared in the skill's `requires.bins`, so OpenClaw can
detect them. The skill verifies everything at runtime and stops early if
anything is missing.

## Usage

Just ask your OpenClaw agent:

> "Write a blog post about local LLM inference optimization"

On the first run it asks a few setup questions and scaffolds your blog repo; on
later runs it remembers your setup and goes straight to writing. It pushes the
post to GitHub Pages, drives your browser through Medium's import, and leaves the
draft open for you to review and publish.

## How it's structured

This is a standard OpenClaw skill bundle — everything here installs to the user:

- [`SKILL.md`](./SKILL.md) — the full agent procedure (read this for the details)
- [`references/html-standards.md`](./references/html-standards.md) — the HTML subset Medium's importer accepts
- [`references/configuration.md`](./references/configuration.md) — persistent-config schemas and reset patterns
- [`assets/post-template.html`](./assets/post-template.html) — a ready-to-fill post template
- [`assets/meta.template.json`](./assets/meta.template.json) — the per-post metadata shape

## Notes

- **Stops at draft.** You click Publish when you're happy with it.
- **Posts are public on GitHub Pages** before import (Medium's importer needs a
  fetchable URL) — don't use it for content you don't want publicly visible.
- **Rate limit:** Medium caps publishing at ~2 stories / 24 hours; this skill
  drafts only, so it doesn't hit that — just don't auto-publish past the cap.

## License

MIT — see the umbrella repository's [LICENSE](https://github.com/ylnhari/openclaw-skills/blob/main/LICENSE).
