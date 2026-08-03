# OpenClaw Skills project instructions

This repository distributes public, installable OpenClaw skill bundles. Everything under `skills/<slug>/` may ship to end users through ClawHub.

- Keep each skill self-contained, portable, and free of private paths, account names, credentials, machine-local configuration, and unpublished material.
- Preserve the required skill layout and valid allow-listed frontmatter.
- Treat runtime configuration as external to the installed bundle; never commit secrets or user-specific state.
- Validate any changed skill and review its public documentation, references, assets, and release metadata as one distributable unit.
- Publishing is user-gated: do not activate workflows, publish to ClawHub, create or push release tags, commit, or push unless explicitly requested.
- Keep clone-required policy in this committed `AGENTS.md` and the public bundle docs. Optional user-level shared policy may add stricter local rules, but the repository never depends on a private shared core.
