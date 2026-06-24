# openclaw-skills

A curated collection of [OpenClaw](https://openclaw.ai) skills maintained by the community.

Each skill in this repository follows the [OpenClaw skill format](https://docs.openclaw.ai/clawhub) — a `SKILL.md` file with YAML frontmatter plus optional supporting files — and is published to [ClawHub](https://clawhub.ai) for one-line installation.

## Skills

| Skill | Slug | Description | Version |
|-------|------|-------------|---------|
| [medium-blog-post-creator](./skills/medium-blog-post-creator/) | `medium-blog-post-creator` | Publish blog posts to Medium via GitHub Pages + URL import — no API token required. | 1.0.0 |

More skills will be added over time. See [CONTRIBUTING.md](./CONTRIBUTING.md) if you'd like to add one.

## Installation

Each skill is independently installable from ClawHub:

```bash
# Replace <slug> with the skill's slug from the table above.
openclaw skills install <owner>/<slug>

# Examples:
openclaw skills install @your-github-username/medium-blog-post-creator
```

You can also install directly from this repository by pointing OpenClaw at the skill folder:

```bash
openclaw skills install ./skills/medium-blog-post-creator
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
├── workflows/                 ← shared workflow docs (per-skill workflows live with their skill)
└── skills/
    └── <skill-slug>/
        ├── SKILL.md           ← required: skill procedure + frontmatter
        ├── README.md          ← per-skill documentation
        ├── CHANGELOG.md       ← per-skill changelog
        ├── examples/          ← optional: usage examples
        └── assets/            ← optional: images, diagrams

.github/workflows/                  ← auto-publish workflows (one per skill)
    └── skill-publish-<skill-slug>.yml
```

Every skill ships with its own GitHub Actions workflow at
`.github/workflows/skill-publish-<skill-slug>.yml` (at the repo root —
GitHub Actions does not discover workflows nested inside `skills/`).
Each workflow publishes that single skill to ClawHub when triggered
manually (`workflow_dispatch`) or via a tagged release.

## Publishing

This repository uses [ClawHub](https://clawhub.ai) as its distribution channel. Publishing is automated via GitHub Actions — see [CONTRIBUTING.md](./CONTRIBUTING.md#publishing) for details.

## License

MIT — see [LICENSE](./LICENSE).
