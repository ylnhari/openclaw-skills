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
> - The desired repository name (default suggestion: `blog`)
> - Whether they want a brand-new repo or to use an existing one

### Bad

> Clone `<some-absolute-local-path>` to `~/blog`.

The bad pattern ties the skill to one machine and one identity. The good
pattern makes the skill portable.

## Skill folder structure

```
skills/<your-skill-slug>/
├── SKILL.md              ← required (the only mandatory file)
├── README.md             ← optional: short listing page (ships to ClawHub)
├── references/           ← optional: docs the agent loads on demand
├── assets/               ← optional: templates / output resources
└── scripts/              ← optional: deterministic helpers
```

**The entire skill folder is published to ClawHub**, so keep it to what an end
user needs. `SKILL.md` is the only required file; OpenClaw's bundled skills are
typically just `SKILL.md` plus optional `references/`/`assets/`/`scripts/`. A
short `README.md` is **optional** — it ships in the bundle and ClawHub may
surface it on the listing page, so add one only if it helps installers. Keep
repo-meta out of the skill folder: no `CHANGELOG.md` (version history is the
ClawHub release changelog via `--changelog` plus the umbrella `CHANGELOG.md`),
and no duplicated `LICENSE`/`CONTRIBUTING` — those live at the repo root.

**Note:** per-skill publish workflows live at the repo root
(`.github/workflows/`), not inside the skill folder, because GitHub Actions
only discovers workflows there. They ship **inactive** with a `.yml.example`
suffix so nothing runs until you opt in — rename to `.yml` to activate.

### Persistent configuration (optional pattern)

Skills in this collection **may** maintain per-user runtime state so they
don't re-ask the same setup questions on every invocation. Two patterns
are supported:

- **Per-install config** stored **outside the installed skill bundle** —
  pointed to by an env var (e.g. `$MEDIUM_BLOG_CONFIG`) or in the per-user
  config directory (`~/.config/<skill>/`, `%APPDATA%\<skill>\`). Holds
  personal preferences that are stable across posts but not across
  machines. **Never bundled with the skill, never committed to the skill
  repo, never published to ClawHub.** Do not write it inside the install
  folder: `openclaw skills update` hashes bundle files to match versions,
  so a stray runtime file there can break updates.
- **Per-repo marker** at `<user-repo>/.<skill-slug>-config.json`,
  committed to the user's data repository. Holds values that are safe to
  expose publicly and that should travel with the data (e.g., GitHub
  owner, repo name, default author handle).

Document both files in the skill's `references/` folder (see
`medium-blog-post-creator/references/configuration.md`). The umbrella repo's
`.gitignore` blocks `config.local.json` to prevent accidental commits by
contributors who also run the skill locally against this repository.

**Skills that don't need persistent state should not introduce it.** Only
add config files when there's a clear UX win from skipping repeated
setup questions, and never store credentials, tokens, or any sensitive
data in either config file.

## SKILL.md frontmatter contract

The YAML frontmatter at the top of `SKILL.md` must declare:

| Field | Purpose |
|-------|---------|
| `name` | Skill slug, lowercase, hyphens only — must match the folder name |
| `description` | One-line, quoted summary (this is what ClawHub shows in search results); keep it short and trigger-focused |

Optional fields recognized by OpenClaw skills:

| Field | Purpose |
|-------|---------|
| `metadata` | Skill metadata; use `metadata.openclaw.requires.bins` to declare required CLIs (e.g. `["gh","git"]`) so OpenClaw can detect/prompt installs |
| `homepage` | URL to the skill's docs or source |
| `license` | SPDX license id (e.g. `MIT`) |
| `allowed-tools` | Restrict which tools the skill may use |
| `user-invocable` | Whether the user can invoke the skill directly |

Do **not** put `version` in frontmatter — version is supplied at publish
time (`clawhub skill publish --version …`) and tracked in `CHANGELOG.md`.
Discoverability keywords belong in the `description` and README, not a
`tags` field. List prerequisites in the skill body, and declare required
binaries via `metadata.openclaw.requires.bins`.

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

**Frontmatter is strictly allow-listed.** Validation only permits these keys:
`name`, `description`, `homepage`, `license`, `allowed-tools`,
`user-invocable`, `metadata`. Any other key (e.g. `version`, `tags`,
`prerequisites`) fails validation. `description` must be a non-empty string,
**must not contain `<` or `>`**, and is capped at 1024 characters. `name` must
be hyphen-case and ≤ 64 characters.

## Publishing

Each skill ships with an **example** publish workflow at
`.github/workflows/skill-publish-<your-skill-slug>.yml.example`. It is
**inactive** until you rename it to `.yml`. Once activated, it can be
triggered in two ways:

- **Manually** from the Actions tab (`workflow_dispatch`) — choose a
  version and whether to dry-run.
- **Automatically** when a tag matching
  `v-<your-skill-slug>-<version>` is pushed.

To publish a new version of an existing skill:

1. Decide the new version (passed via `--version` at publish) and record the
   notable changes in the umbrella `CHANGELOG.md`. Do not add a version to
   `SKILL.md` frontmatter — ClawHub tracks the version per release.
2. **Either** publish manually from a logged-in shell (simplest):

   ```bash
   npm i -g clawhub
   clawhub login
   clawhub skill publish ./skills/<skill-slug> \
     --slug <skill-slug> --name "<Human Name>" \
     --version <version> --changelog "<summary>"   # add --dry-run to preview
   ```

   **Or** use the workflow: rename `skill-publish-<skill-slug>.yml.example`
   to `.yml`, add the `CLAWHUB_TOKEN` secret, then Actions tab → "Publish
   <skill-slug> to ClawHub" → Run workflow with `dry_run: false`.
3. After verifying the publish succeeded, create a matching GitHub tag so
   future workflow runs can auto-trigger:
   `git tag v-<skill-slug>-<version> && git push --tags`

If you don't have write access to push tags or trigger workflows, ask a
maintainer in the PR — they can do it for you.

### First-time setup

For the workflow to actually publish (not just dry-run), the repository
needs one secret:

- `CLAWHUB_TOKEN` — generate at <https://clawhub.ai> → Settings → API
  tokens, then add it at
  <https://github.com/your-org/your-repo/settings/secrets/actions>.

### Adding a new skill

1. Create `skills/<your-skill-slug>/` with the standard layout (see
   "Skill folder structure" above).
2. Copy any existing skill's workflow file to a new path under
   `.github/workflows/` named for your skill. For example:

   ```bash
   # macOS / Linux / Git Bash on Windows
   cp .github/workflows/skill-publish-medium-blog-post-creator.yml.example \
      .github/workflows/skill-publish-<your-skill-slug>.yml.example
   ```

   ```powershell
   # PowerShell on Windows
   Copy-Item .github/workflows/skill-publish-medium-blog-post-creator.yml.example `
     .github/workflows/skill-publish-<your-skill-slug>.yml.example
   ```

3. Edit the copy: update the workflow `name:`, `SKILL_DIR`, `SLUG`,
   `SKILL_NAME`, and the `push.tags` pattern (it must match
   `v-<your-skill-slug>-*`).

## Versioning

- Each skill is versioned **independently**. Skill A can be at 2.3.1 while
  Skill B is still at 1.0.0.
- Patch (1.0.x): typo fixes, broken links, no procedure changes.
- Minor (1.x.0): new steps, new questions asked, backward-compatible additions.
- Major (x.0.0): breaking changes to frontmatter, renamed inputs, removed
  steps.

## Code of conduct

By participating, you agree to abide by [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md).
