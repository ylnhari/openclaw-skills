# Changelog: medium-blog-post-creator

All notable changes to this skill are documented here. The format is based
on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the skill
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-06-24

### Added
- **Persistent configuration** so repeat invocations skip the setup
  questions. State is stored in two places:
  - Per-install: `<skill-install-dir>/config.local.json` (or
    `$MEDIUM_BLOG_CONFIG`) — per-machine preferences such as default
    working directory, audience, tone, length.
  - Per-repo: `.medium-skill-config.json` committed at the root of the
    user's blog repository — blog identity (GitHub owner, repo name,
    Pages URL, default author, branch).
- New **Step 0: Load or initialize persistent config** at the start of
  the workflow. Subsequent steps now use the merged config instead of
  re-asking setup questions.
- New example files:
  [`examples/config.example.json`](./examples/config.example.json) and
  [`examples/.medium-skill-config.example.json`](./examples/.medium-skill-config.example.json)
  documenting both config schemas.
- Reset patterns documented: "forget my local config", "forget this
  blog", "forget everything", and per-invocation overrides.

### Changed
- Step 2 (scaffold repo) now distinguishes between fresh setup and
  subsequent invocations: if the per-repo marker is present, the skill
  clones/updates the existing repo instead of creating a new one.
- Workflow is now 11 steps (Step 0 + the previous 10).
- `.gitignore` at the repo root now ignores `config.local.json` to
  prevent accidental commits if a contributor is also running the skill
  locally against this repo.

### Backward compatibility
- All v1.0.0 behavior is preserved. If no config files exist on the
  user's machine, the skill falls back to the original "Inputs
  collected from the user" flow.

## [1.0.0] - 2026-06-24

### Added
- Initial public release.
- Self-contained workflow: scaffolds a fresh static blog repository in the
  user's GitHub account from scratch — no external repos or scripts required.
- Constrained-HTML authoring standards aligned with Medium's URL importer.
- End-to-end agent-driven import via the OpenClaw `browser` tool.
- Auto-link back from repository metadata to the Medium draft URL.
- Stops at draft (no auto-publish) to respect Medium's 2025 API closure
  and 2026 publishing rate limits.
- Per-skill auto-publish workflow at
  `.github/workflows/skill-publish-medium-blog-post-creator.yml`
  (in the umbrella repository).
