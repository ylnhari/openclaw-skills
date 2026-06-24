# Medium Blog Post Creator

> **Publish blog posts to Medium via GitHub Pages + URL import.**
> No Medium API token required.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/ylnhari/openclaw-skills/blob/main/LICENSE)
[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-blueviolet)](https://docs.openclaw.ai/clawhub)
[![ClawHub](https://img.shields.io/badge/ClawHub-install-blue)](https://clawhub.ai)

## What it does

This OpenClaw skill publishes a fully formatted blog post to **Medium** —
end-to-end, agent-driven, no API token, no scripts to copy.

It works around the **Medium API closure of January 2025** by using the
only stable import path that remains: author a constrained HTML post,
deploy it as a public URL via GitHub Pages, and let Medium's own
URL-import tool fetch and convert it into blocks.

## Why it exists

- **Medium stopped issuing new API tokens** on Jan 1, 2025. Every legacy
  Medium API tool (`medium-import`, `post-medium-draft`, Buffer, Zapier)
  is now unusable for new users.
- **The URL-import endpoint** at `https://medium.com/p/import` is the only
  official path that survives.
- **No existing OpenClaw skill** on ClawHub targets Medium publishing.
- **Other URL-import tools** (`medium-import-backdate` and similar) only
  handle a single sub-step. None combine authoring standards, GitHub Pages
  deployment, and browser-driven import.

## Install

From ClawHub:

```bash
openclaw skills install @<owner>/medium-blog-post-creator
```

Or directly from this repository:

```bash
openclaw skills install ./skills/medium-blog-post-creator
```

## Prerequisites

You'll need:

1. A **GitHub account** (to host the static blog repository)
2. A **Medium account** (the destination)
3. **`gh` CLI** installed and authenticated — [install](https://cli.github.com),
   then `gh auth login`
4. The **OpenClaw `browser` tool**, signed in to your Medium account
5. **Git** (any recent version)

The skill will check these for you and stop early if anything is missing.

## Usage

Just ask your OpenClaw agent:

> "Write a blog post about local LLM inference optimization"

The skill will then:

1. **Load persistent config** (see below) — skips all setup questions
   on subsequent invocations.
2. Confirm your prerequisites (`gh`, browser, Medium sign-in).
3. Scaffold or clone your static blog repository on GitHub.
4. Write the post HTML to a constrained subset Medium's importer accepts.
5. Push to GitHub and wait for Pages to deploy.
6. Drive your browser to `medium.com/p/import`, paste the URL, click Import.
7. Open the resulting draft for your review (never auto-publishes).
8. Link the Medium draft URL back into the repository's metadata.

You stay in control: the skill stops at draft, you click Publish when ready.

## Persistent configuration

The skill remembers its setup so you don't have to answer the same
questions every time. State lives in **two files**:

| File | Lives where | Holds what | Public? |
|------|-------------|-----------|---------|
| `config.local.json` | Next to the installed skill (or `$MEDIUM_BLOG_CONFIG`) | Per-machine preferences: default working dir, audience, tone, length, branch | No — never committed, never bundled |
| `.medium-skill-config.json` | At the root of your blog repository | Blog identity: GitHub owner, repo name, pages URL, default author | Yes — it lives in a public GitHub repo |

On every invocation the skill loads both and merges them. Once everything
is set up, future invocations go straight to creating the next post —
no questions asked unless you want to override something.

**Reset paths** (the skill handles these when you ask):

- *"Forget my local config"* → deletes `config.local.json`
- *"Forget this blog"* → deletes `.medium-skill-config.json` from the repo
- *"Forget everything"* → deletes both, next run is a fresh first-run

**One-off overrides** are honored without touching the configs. For
example: *"Use `acme-corp/engineering-blog` for this post"* works
without modifying your saved config.

See [`SKILL.md`](./SKILL.md) for the full resolution rules.

## HTML standards (why we have them)

Medium's URL importer is strict about what it converts cleanly into blocks.
Anything outside this subset is either stripped, mangled, or rejected:

**Allowed:** `h1`, `h2`, `h3`, `p`, `strong`, `em`, `a`, `img`, `blockquote`,
`pre`, `code`, `ul`, `ol`, `li`, `hr`

**Banned:** `<script>`, `<style>`, `<iframe>`, `<table>`, `<div>`, `<span>`,
inline `style="..."`, H4–H6

The skill enforces these automatically. See
[`examples/post-template.html`](./examples/post-template.html) for the
reference template and [`examples/standards-snippet.md`](./examples/standards-snippet.md)
for a compact rules reference.

## Rate limits

Medium caps publishing at **2 stories + 100 responses per rolling 24
hours** (as of early 2026). The skill stops at draft so this doesn't
block automated drafting — but please don't auto-publish more than
2 stories per day.

## Repository layout

```
skills/medium-blog-post-creator/
├── SKILL.md              ← agent reads this
├── README.md             ← you are here
├── CHANGELOG.md
├── examples/
│   ├── post-template.html
│   ├── meta.json
│   ├── standards-snippet.md
│   ├── config.example.json                  ← per-install config schema
│   └── .medium-skill-config.example.json    ← per-repo marker schema
└── assets/
    └── (cover image goes here)
```

## License

MIT — see the umbrella repository's [LICENSE](https://github.com/ylnhari/openclaw-skills/blob/main/LICENSE).
