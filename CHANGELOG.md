# Changelog (umbrella)

All notable changes to this repository are documented in this file.

This umbrella changelog tracks repo-wide changes (layout, workflows, docs).
Per-skill changes are tracked in each skill's own `CHANGELOG.md`.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
for the repository as a whole. Individual skills are versioned independently.

## [Unreleased]

### Added
- Initial repository skeleton: README, LICENSE, CONTRIBUTING, CODE_OF_CONDUCT, .gitignore
- Skill: `medium-blog-post-creator` v1.0.0

## [1.0.0] - 2026-06-24

### Added
- First public release.
- Per-skill GitHub Actions workflow at `.github/workflows/skill-publish-<skill-slug>.yml`
  that publishes each skill to ClawHub on `workflow_dispatch` or on a
  matching tag push.
- Initial skill: [`medium-blog-post-creator`](./skills/medium-blog-post-creator/) v1.0.0.
