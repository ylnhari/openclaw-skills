# Persistent configuration (sticky state)

After the first successful run, the skill remembers its setup so subsequent
invocations skip the scaffolding questions. State is stored in **two places**,
checked in this order on every invocation.

## 1. Per-install config (machine-local)

Lives at (first match wins):

1. `$MEDIUM_BLOG_CONFIG` (if the environment variable is set), or
2. the per-user config directory, **outside the installed skill folder**:
   - Linux/macOS: `${XDG_CONFIG_HOME:-~/.config}/medium-blog-post-creator/config.json`
   - Windows: `%APPDATA%\medium-blog-post-creator\config.json`

Keep this file **out of the skill's install directory.** OpenClaw installs
skills into the active workspace's `skills/` folder (or the shared managed
skills directory with `--global`), and `openclaw skills update` matches
installed bundles by hashing their files — a stray runtime file inside the
bundle can interfere with updates or be overwritten by them.

It is **never committed, never bundled, never published.** Holds personal
preferences that are stable across posts but not across machines:

```json
{
  "schema_version": 1,
  "default_working_dir": "~/Projects",
  "default_audience": "technical",
  "default_tone": "casual",
  "default_length": "medium",
  "default_branch": "main"
}
```

## 2. Per-repo marker (travels with the blog)

Lives at `<repo-local-path>/.medium-skill-config.json` and is committed to the
blog repository itself, so the blog's identity follows the repo — switch
machines, and a fresh clone brings the marker with you.

The file is **public** (it lives in a public GitHub repository), so it holds
only values that are safe to expose:

```json
{
  "schema_version": 1,
  "github_owner": "<github-username-or-org>",
  "github_repo": "<repo-name>",
  "pages_url": "https://<github-username-or-org>.github.io/<repo-name>/",
  "default_author": "<github-username-or-org>",
  "branch": "main"
}
```

`pages_url` is the resolved `<pages-base-url>` (see SKILL.md Step 2): the
domain root for a `<owner>.github.io` repo, or `https://<owner>.github.io/<repo>/`
for a project repo. The example above shows the project-site form.

## Resolution rules

- **Both files present:** proceed straight to Step 2 using the merged config.
  Do not re-ask any sticky questions.
- **Only the per-repo marker present (per-install missing):** ask the user for
  the per-install preferences, write the per-install config, then proceed.
- **Only the per-install config present (no per-repo marker):** ask the user
  for the GitHub owner + repo name, scaffold a fresh repo in Step 2, and write
  `.medium-skill-config.json` to it.
- **Neither present:** fall through to the original "Inputs collected from the
  user" flow.
- **Unknown `schema_version`:** if either file's `schema_version` is not `1`,
  warn the user, treat unrecognized fields as defaults, and continue.

## Per-invocation overrides

The user can override any sticky value for one invocation without touching the
config files. For example: *"Use `acme-corp/engineering-blog` for this post"*
is honored as a one-off; afterwards the config files are left unchanged.

## Resetting persistent state

| User says | Action |
|-----------|--------|
| "Forget my local config" / "Reset skill state" | Delete the per-install config file (`$MEDIUM_BLOG_CONFIG`, or the per-user config path). The per-repo marker is untouched. |
| "Forget this blog" / "Detach this repo from the skill" | Delete `<repo-local-path>/.medium-skill-config.json` and commit the removal. The per-install config is untouched. |
| "Forget everything" / "Start over" | Delete both files. The next invocation goes through the original first-run flow. |
| "Use a different blog for this post" | Override the GitHub owner + repo for this invocation only; do not touch the configs. |
