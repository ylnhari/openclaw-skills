# openclaw-skills

A curated collection of [OpenClaw](https://openclaw.ai) skills maintained by the community.

Each skill in this repository follows the [OpenClaw skill format](https://docs.openclaw.ai/clawhub) — a `SKILL.md` file with YAML frontmatter plus optional supporting files — and is published to [ClawHub](https://clawhub.ai) for one-line installation.

## Skills

| Skill | Slug | Description | Version |
|-------|------|-------------|---------|
| [medium-blog-post-creator](./skills/medium-blog-post-creator/) | `medium-blog-post-creator` | Publish blog posts to Medium via GitHub Pages + URL import — no API token required. | 1.1.0 |

More skills will be added over time. See [CONTRIBUTING.md](./CONTRIBUTING.md) if you'd like to add one.

## Installation

Each skill is independently installable from ClawHub. Skills install by
**slug** (slugs are unique on ClawHub):

```bash
# Install a skill published to ClawHub
openclaw skills install medium-blog-post-creator
```

You can also install directly from a local checkout by pointing OpenClaw
at the skill folder:

```bash
openclaw skills install ./skills/medium-blog-post-creator --as medium-blog-post-creator
```

## Repository layout

```
openclaw-skills/
├── README.md                  ← you are here
├── LICENSE                    ← MIT
├── CONTRIBUTING.md            ← how to add a skill
├── CODE_OF_CONDUCT.md         ← community standards
├── CHANGELOG.md               ← umbrella changelog
├── .gitignore
├── .github/workflows/         ← auto-publish workflows (one per skill)
│   └── skill-publish-<skill-slug>.yml
└── skills/
    └── <skill-slug>/         ← this whole folder is what publishes to ClawHub
        ├── SKILL.md           ← required: skill procedure + frontmatter
        ├── references/        ← optional: docs the agent loads on demand
        ├── assets/            ← optional: templates / output resources
        └── scripts/           ← optional: deterministic helpers
```

A published skill bundle is just `SKILL.md` plus its supporting
`references/`, `assets/`, and `scripts/` — the same shape as the skills that
ship with OpenClaw. Human-facing docs (this README, per-skill notes, version
history) live at the **repo root**, not inside the skill folder, so they don't
ship to every install.

Every skill ships with its own GitHub Actions workflow at
`.github/workflows/skill-publish-<skill-slug>.yml` (at the repo root —
GitHub Actions does not discover workflows nested inside `skills/`).
Each workflow publishes that single skill to ClawHub when triggered
manually (`workflow_dispatch`) or via a tagged release.

## Publishing

This repository uses [ClawHub](https://clawhub.ai) as its distribution channel. Publishing is automated via GitHub Actions — see [CONTRIBUTING.md](./CONTRIBUTING.md#publishing) for details.

## License

MIT — see [LICENSE](./LICENSE).
