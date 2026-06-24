# Contributing

Thanks for your interest in adding a skill to this collection! This document
walks through how to propose, structure, and publish a new skill.

## Ground rules

- One skill per folder under `skills/`. Folder name = skill slug (lowercase,
  hyphens only, max ~64 chars).
- Every skill **must** include a valid `SKILL.md` with YAML frontmatter.
- Keep skills **self-contained**. Do not reference external repos, private
  paths, or hard-coded local filesystem locations inside `SKILL.md`. Use the
  *ask-the-user* pattern (see below).
- Open a Pull Request even for trivial fixes — review is part of the process.

## The "ask the user" pattern

Skills in this repository must work for any user, on any machine, with any
GitHub or Medium handle. To achieve that, `SKILL.md` should **ask the user
explicit questions** at the start of a workflow rather than hard-coding paths,
repo names, or URLs.

### Good

> Ask the user for:
> - The GitHub username or org that will host the blog repository
> - The desired repository name (default suggestion: `<username>.github.io`)
> - Whether they want a brand-new repo or to use an existing one

### Bad

> Clone `<some-absolute-local-path>` to `~/blog`.

The bad pattern ties the skill to one machine and one identity. The good
pattern makes the skill portable.

## Skill folder structure

```
skills/<your-skill-slug>/
├── SKILL.md              ← required
├── README.md             ← required: human-readable docs
├── CHANGELOG.md          ← required: per-skill changelog
├── examples/             ← optional: example inputs/outputs
├── assets/               ← optional: images, screenshots
└── .github/workflows/    ← optional: per-skill CI (rarely needed)
```

## SKILL.md frontmatter contract

The YAML frontmatter at the top of `SKILL.md` must declare:

| Field | Purpose |
|-------|---------|
| `name` | Skill slug, lowercase, hyphens only |
| `description` | One-line summary, **max 160 bytes** (this is what ClawHub shows in search results) |

Recommended additional fields:

| Field | Purpose |
|-------|---------|
| `version` | Semver (e.g., `1.0.0`) — mirrors the `CHANGELOG.md` |
| `tags` | Comma-separated keywords for discoverability |
| `prerequisites` | Free-form list of accounts/tools the user needs |

## Local validation before publishing

```bash
# Install the ClawHub CLI once
npm i -g clawhub

# Validate a skill folder without uploading
clawhub skill publish ./skills/<your-skill-slug> \
  --slug <your-skill-slug> \
  --name "<Human Readable Name>" \
  --changelog "Initial release" \
  --dry-run
```

The dry-run reports missing files, malformed frontmatter, and oversized
descriptions.

## Publishing

This repository has a reusable GitHub Actions workflow that publishes every
immediate subfolder of `skills/` to ClawHub on tagged releases. To publish a
new skill:

1. Merge your skill PR into `main`.
2. Bump its `version` in `SKILL.md` and add an entry to its `CHANGELOG.md`.
3. Trigger the workflow manually from the Actions tab with
   `workflow_dispatch`. Set `dry_run: false` for a real publish.
4. After the publish succeeds, tag a release so future updates can be
   triggered automatically: `git tag v-<skill-slug>-<version> && git push --tags`
   (e.g., `v-medium-blog-post-creator-1.0.0`).

The workflow requires the `CLAWHUB_TOKEN` secret to be set at the
repository or org level. Generate the token at <https://clawhub.ai> →
Settings → API tokens, then add it under
<https://github.com/your-org/your-repo/settings/secrets/actions>.

If you don't have write access to push tags or trigger workflows, ask a
maintainer in the PR — they can do it for you.

## Versioning

- Each skill is versioned **independently**. Skill A can be at 2.3.1 while
  Skill B is still at 1.0.0.
- Patch (1.0.x): typo fixes, broken links, no procedure changes.
- Minor (1.x.0): new steps, new questions asked, backward-compatible additions.
- Major (x.0.0): breaking changes to frontmatter, renamed inputs, removed
  steps.

## Code of conduct

By participating, you agree to abide by [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).
