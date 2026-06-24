# Changelog (umbrella)

All notable changes to this repository are documented in this file.

This umbrella changelog tracks repo-wide changes (layout, workflows, docs).
Per-skill changes are tracked in each skill's own `CHANGELOG.md`.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
for the repository as a whole. Individual skills are versioned independently.

## [Unreleased]

### Added
- Skill: `medium-blog-post-creator` — persistent configuration so repeat
  invocations skip the setup questions (per-install config + per-repo marker).

### Changed
- **Restructured the `medium-blog-post-creator` bundle to match how real
  ClawHub/OpenClaw skills ship**: now just `SKILL.md` + `references/` +
  `assets/`. Moved the HTML rules to `references/html-standards.md`, the
  config schemas/resolution to `references/configuration.md`, and the
  templates to `assets/`. Removed the in-bundle `README.md` and `CHANGELOG.md`
  (no published OpenClaw skill ships those; the whole folder is published).
- Frontmatter aligned with OpenClaw's strict allow-list: dropped the
  invalid `version`/`tags`/`prerequisites`; added `metadata.openclaw`
  (`emoji`, `requires.bins`), `homepage`, and `license`. Verified with the
  bundled `quick_validate.py`.

### Fixed
- Pre-publish audit corrections to `medium-blog-post-creator`: GitHub Pages
  URL resolution for user vs project sites, Pages-enable ordering (moved to
  Step 2, was effectively dead in Step 7), per-install config moved out of the
  installed bundle, clone-path handling, and the `openclaw skills install`
  command syntax (install by slug).

### Docs
- `CONTRIBUTING.md` corrected to OpenClaw's real conventions: strict
  frontmatter allow-list, `SKILL.md`-only bundles (no per-skill
  README/CHANGELOG), and config stored outside the bundle.

## [1.0.0] - 2026-06-24

### Added
- First public release.
- Per-skill GitHub Actions workflow at `.github/workflows/skill-publish-<skill-slug>.yml`
  that publishes each skill to ClawHub on `workflow_dispatch` or on a
  matching tag push.
- Initial skill: [`medium-blog-post-creator`](./skills/medium-blog-post-creator/) v1.0.0.
