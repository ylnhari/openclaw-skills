# Changelog (umbrella)

All notable changes to this repository are documented in this file.

This umbrella changelog tracks repo-wide changes (layout, workflows, docs) and
summarizes notable per-skill changes. Per-skill release history is tracked by
ClawHub releases (the `--changelog` passed at publish).

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
  templates to `assets/`. Removed the in-bundle `CHANGELOG.md` and trimmed the
  per-skill `README.md` to a short ClawHub listing page (the whole folder is
  published, so only what installers need stays).
- Frontmatter aligned with OpenClaw's strict allow-list: dropped the
  invalid `version`/`tags`/`prerequisites`; added `metadata.openclaw`
  (`emoji`, `requires.bins`), `homepage`, and `license`. Verified with the
  bundled `quick_validate.py`.

### Changed (cont.)
- Default repo-name suggestion is now `blog` (a normal project repo) instead of
  `<username>.github.io`. The user-site option still works; it's just no longer
  the default, since most people reserve their one user site for a homepage.

### Fixed
- Pre-publish audit corrections to `medium-blog-post-creator`: GitHub Pages
  URL resolution for user vs project sites, Pages-enable ordering (moved to
  Step 2, was effectively dead in Step 7), per-install config moved out of the
  installed bundle, clone-path handling, and the `openclaw skills install`
  command syntax (install by slug).
- Subsequent-run repo reuse: the per-install config now stores `last_repo`
  (owner/name/pages_url/branch) so later runs locate the existing blog repo
  without re-asking, instead of relying on the agent already being inside it.
- Scaffolding now writes a `.nojekyll` file so GitHub Pages serves the static
  HTML as-is (no Jekyll processing).

### Docs
- `CONTRIBUTING.md` corrected to OpenClaw's real conventions: strict
  frontmatter allow-list, lean `SKILL.md`-centric bundles (optional short
  README, no in-bundle CHANGELOG), config stored outside the bundle, and
  example (inactive) publish workflows.

## [1.0.0] - 2026-06-24

### Added
- First public release.
- Per-skill GitHub Actions workflow at `.github/workflows/skill-publish-<skill-slug>.yml`
  that publishes each skill to ClawHub on `workflow_dispatch` or on a
  matching tag push.
- Initial skill: [`medium-blog-post-creator`](./skills/medium-blog-post-creator/) v1.0.0.
