# Changelog: medium-blog-post-creator

All notable changes to this skill are documented here. The format is based
on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the skill
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
