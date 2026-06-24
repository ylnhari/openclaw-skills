---
name: "medium-blog-post-creator"
description: "Publish blog posts to Medium via GitHub Pages + URL import — no API token required."
version: "1.0.0"
tags: "medium, blog, publishing, github-pages, static-site, automation"
prerequisites: "GitHub account, Medium account, gh CLI authenticated, OpenClaw browser tool"
---

# Medium Blog Post Creator

Publish a fully formatted blog post to **Medium** without needing an API token.

Medium stopped issuing new integration tokens on **January 1, 2025**. This skill
works around that closure by using the **only remaining stable import path**:
author a constrained HTML post, deploy it as a public URL, and let Medium's
own URL-import tool fetch and convert it into blocks.

The skill is **end-to-end agent-driven** — no external repos, no local paths
to remember, no scripts to copy. It asks the user for the inputs it needs,
scaffolds a fresh static blog repository in their own GitHub account,
deploys each post to GitHub Pages, and walks the browser through Medium's
import flow until the draft is sitting in the user's draft list ready for
review.

## Why this exists

- **Medium API is closed** for new integrations (since Jan 2025).
- **Most existing tools** (`medium-import`, `post-medium-draft`, etc.) rely
  on legacy tokens and are unusable for new users.
- **The URL import endpoint** at `https://medium.com/p/import` is the only
  official path that survives the closure.
- **No OpenClaw skill** on ClawHub currently targets Medium publishing.
- **Other URL-import tools** (e.g., `medium-import-backdate`) only handle a
  single sub-step (backdating HTML). None combine authoring standards,
  GitHub Pages deployment, and browser-driven import.

## When to use this skill

Trigger this skill when the user says any of:

- "Write a blog post about X" / "Create a Medium post about X"
- "Publish a draft to Medium about X" / "Blog about X"
- "I want to post this to Medium"
- "Make a Medium draft for X"
- "Cross-post this to Medium"

Also trigger if the user provides content (Markdown, outline, or notes) and
mentions Medium.

## What this skill does NOT do

- **No auto-publish on Medium.** The skill always stops at **draft** for
  human review. (Auto-publishing is intentionally out of scope to avoid
  triggering Medium's rate limits or anti-spam filters.)
- **No Medium API token required.** The skill uses the public URL-import
  endpoint instead.
- **No pre-existing blog repository needed.** The skill scaffolds one in
  the user's GitHub account from scratch.
- **No OS assumptions.** Paths are derived from the user's inputs and
  current working directory; the same procedure works on Windows,
  macOS, and Linux.

## Prerequisites (collected from the user)

Before starting, the skill must verify that the user has the prerequisites
available. Ask the user to confirm or provide each one.

### Required accounts

| Account | Why |
|---------|-----|
| **GitHub** | To host the static blog repository and serve it via GitHub Pages |
| **Medium** | The destination for the imported draft |

### Required tools

| Tool | Why |
|------|-----|
| **GitHub CLI (`gh`)** | To create the repo, authenticate, and push from the agent's shell |
| **OpenClaw `browser` tool** | To drive Medium's URL-import UI |
| **Git** | Standard `git` operations on the new repo |

### Required authentication

- `gh auth status` must show an authenticated GitHub user. If not, ask the
  user to run `gh auth login` in a terminal and retry.
- The OpenClaw browser tool must be logged into Medium under the user's
  account. If not, the user must sign in via the browser tool before the
  import step.

If any prerequisite is missing, stop and tell the user what to install or
sign in to. Do not proceed with placeholders.

## Inputs collected from the user (in one message, before any work begins)

Ask the user for the following in a single message. If they don't answer
some, use the defaults noted below — but always **state your assumptions
explicitly** before proceeding.

1. **Topic / title** (required) — what the post is about.
2. **Target audience** — `technical`, `general`, or `beginner`. Default:
   `technical`.
3. **Tone** — `casual`, `formal`, or `tutorial`. Default: `casual`.
4. **Approximate length** — `short` (~500 words), `medium` (~1000 words),
   or `long` (~2000+ words). Default: `medium`.
5. **Key sections or bullet points** — what the post must cover.
6. **GitHub username or org** where the blog repository will live
   (required). Example: `octocat`.
7. **Repository name** for the blog. Default suggestion:
   `<username>.github.io` (a user site, simplest setup) or
   `blog` (a project site under a dedicated repo). Default:
   `<username>.github.io`.
8. **Local working directory** where the new repository should be cloned.
   Default: the user's current working directory.
9. **Tags** for the post on Medium (2–5 short keywords). The skill will
   ask later if the user wants to revise them in the Medium UI.
10. **Optional: cover image URL.** If the user doesn't supply one, the
    skill picks a relevant free-license image from Unsplash.

If the user gave a brief prompt (e.g., "blog about vLLM"), proceed with
sensible defaults and **clearly state the assumptions you made**.

## Workflow

The workflow has 10 steps. Each step has a clear completion check before
moving on. **Do not skip steps.**

### Step 1 — Confirm prerequisites

Verify in the agent's shell:

- `gh --version` — must succeed.
- `gh auth status` — must show an authenticated user.
- `git --version` — must succeed.
- OpenClaw `browser` tool is reachable.

If any check fails, stop and report the exact missing item. Do not proceed.

### Step 2 — Scaffold the blog repository

The skill scaffolds a **minimal static site** in the user's chosen
repository. The site is a single static page that lists posts, with each
post living in its own folder. No frameworks, no build step — only
plain HTML, CSS, and a JSON index. This keeps the skill free of toolchain
dependencies and works the same on every machine.

Throughout the rest of this workflow, `<repo-local-path>` refers to the
local filesystem path of the cloned repository (typically
`<local-working-dir>/<repo-name>`).

1. In the agent's shell, run:

   ```bash
   gh repo create <github-username-or-org>/<repo-name> \
     --public \
     --description "Static blog for Medium cross-posting" \
     --clone \
     --add-readme
   ```

2. If the repository already exists, fall back to:

   ```bash
   gh repo clone <github-username-or-org>/<repo-name> \
     "<local-working-dir>/<repo-name>"
   ```

3. Confirm the clone succeeded and `<repo-local-path>` exists.

### Step 3 — Create the post HTML file

Write the post to:

```
<repo-local-path>/posts/<YYYY-MM-DD>-<slug>/index.html
```

Where `<slug>` is derived from the post title: lowercase, hyphens only,
max 60 characters. Example: `running-llms-locally`.

**HTML standards (mandatory — Medium's importer rejects or mangles anything
outside this subset):**

- Exactly **one `<h1>`**, at the very top, identical to the `<title>`.
- Only these tags are allowed:
  `h1, h2, h3, p, strong, em, a, img, blockquote, pre, code, ul, ol, li, hr`.
- **Banned tags and patterns:** `<script>`, `<style>`, `<iframe>`, `<table>`,
  `<div>`, `<span>`, inline `style="..."`, H4–H6.
- All `<img src>` must be **absolute HTTPS URLs**. No relative paths.
  Recommended source: `https://images.unsplash.com/photo-{ID}?w=1200&q=80`.
- Inside `<pre><code>` blocks, use HTML entities `&lt;` and `&gt;` for
  `<` and `>`.
- YouTube: paste the bare URL in a `<p>` tag. Medium's importer will
  convert it into an embed block.
- Maximum of 2 `<hr>` per post.

For the full reference HTML template, see
[`examples/post-template.html`](./examples/post-template.html) in this
skill's folder.

The post body should include:

- An opening hook paragraph.
- 3–6 sections with `<h2>` headings.
- Code examples where relevant (inside `<pre><code>`).
- At least one image (Unsplash URL relevant to the topic).
- Bold/italic emphasis where appropriate.
- A `<blockquote>` for a key insight or quote.
- A closing paragraph with a summary or call to action.

**No hallucinated facts. No filler. No LLM-tells like "in conclusion" or
"as we have seen".**

### Step 4 — Create the metadata sidecar

Write a JSON file at:

```
<repo-local-path>/posts/<YYYY-MM-DD>-<slug>/meta.json
```

Shape:

```json
{
  "title": "Full post title",
  "date": "YYYY-MM-DD",
  "author": "<github-username-or-org>",
  "description": "1–2 sentence summary, max 200 chars",
  "tags": ["tag1", "tag2", "tag3"],
  "status": "draft",
  "medium_url": null,
  "cover_image": "https://images.unsplash.com/photo-...?w=1200&q=80"
}
```

The `medium_url` is updated to the live Medium draft URL after Step 8
so the post is traceable end-to-end.

### Step 5 — Update the post index

The repository keeps a top-level `posts/index.json` listing every post
(newest first). After creating the new post:

1. Read `posts/index.json`. If the file doesn't exist, create it as `[]`.
2. Prepend the new entry to the array. **Do not re-sort existing entries.**
3. Write it back.

Entry shape:

```json
{
  "slug": "YYYY-MM-DD-slug",
  "title": "Full post title",
  "date": "YYYY-MM-DD",
  "description": "1–2 sentence summary, max 200 chars",
  "tags": ["tag1", "tag2"],
  "cover_image": "https://images.unsplash.com/photo-...?w=1200&q=80",
  "status": "draft",
  "medium_url": null
}
```

### Step 6 — Commit and push

In the agent's shell:

```bash
cd "<repo-local-path>"
git add posts/<YYYY-MM-DD>-<slug>/ posts/index.json
git commit -m "feat(posts): add \"<POST TITLE>\" [<YYYY-MM-DD>]"
git push origin main
```

If push fails with an auth error, ask the user to run `gh auth login`
and retry. Do not retry more than twice — if it still fails after the
second attempt, report the error and stop.

### Step 7 — Enable GitHub Pages and wait for deploy

Determine whether this is the **first post** the skill is publishing to
this repository by checking whether `posts/index.json` exists at the
repo root and is a non-empty array. If it does not exist or is empty,
this is the first post and Pages may need to be enabled manually.

**If this is the first post:**

1. If the repository name is `<username>.github.io`, GitHub Pages is
   automatically enabled — skip to the polling step below.
2. For any other repository name, the user must enable Pages in the
   repository settings (Settings → Pages → Source: `Deploy from a
   branch` → Branch: `main` / `/ (root)`). Direct the user to the
   settings page: `https://github.com/<username-or-org>/<repo-name>/settings/pages`.
3. Wait for the user to confirm Pages is enabled before continuing.

**If this is not the first post:** Pages is already enabled. Skip
straight to polling.

**Poll for deploy (cross-platform):**

The skill explicitly supports Windows, macOS, and Linux. Use the
PowerShell variant on Windows (PowerShell 7+) and the `curl` variant
on macOS/Linux. Do not assume `curl` is the `curl.exe` binary on
Windows.

```powershell
# PowerShell 7+ (Windows, macOS, Linux)
# Poll the post URL until it returns HTTP 200, max 3 minutes.
$url = "https://<username-or-org>.github.io/<repo-name>/posts/<YYYY-MM-DD>-<slug>/"
$deadline = (Get-Date).AddMinutes(3)
while ((Get-Date) -lt $deadline) {
  try {
    $code = (Invoke-WebRequest -Uri $url -UseBasicParsing -MaximumRedirection 5 -ErrorAction Stop).StatusCode
    if ($code -eq 200) { exit 0 }
  } catch {
    $code = 404
  }
  if ($code -eq 200) { exit 0 }
  Start-Sleep -Seconds 15
}
exit 1
```

```bash
# macOS / Linux (curl from system or curl.exe on Windows)
# Poll the post URL until it returns HTTP 200, max 3 minutes.
deadline=$(( $(date +%s) + 180 ))
url="https://<username-or-org>.github.io/<repo-name>/posts/<YYYY-MM-DD>-<slug>/"
while [ "$(date +%s)" -lt "$deadline" ]; do
  code=$(curl -fsS -o /dev/null -w "%{http_code}" "$url" 2>/dev/null || echo 000)
  if [ "$code" = "200" ]; then exit 0; fi
  sleep 15
done
exit 1
```

If the poll succeeds (returns `200`), proceed to Step 8. If it times
out after 3 minutes (still `404`), direct the user to check the
repository's Actions deploy status at
`https://github.com/<username-or-org>/<repo-name>/actions` and report
back.

**Do not proceed to Step 8 until the URL returns 200.**

### Step 8 — Import to Medium via the browser

Use the OpenClaw `browser` tool to drive Medium's URL import.

1. Navigate to `https://medium.com/p/import`. Wait for the page to
   fully load.
2. Find the URL input field. Paste:
   `https://<username-or-org>.github.io/<repo-name>/posts/<YYYY-MM-DD>-<slug>/`
   (include the trailing slash — Medium's importer is sensitive to it).
3. Click the Import button.
4. Wait for Medium's confirmation message. Look for text containing
   "Imported the story" or equivalent.
5. Click the "See your story" / "Open draft" link. This opens the
   draft editor at `https://medium.com/p/<draft-id>/edit`.
6. **Do not click Publish.** Leave the post as a draft for the user's
   review.
7. Copy the draft editor URL.

**Tag handling:** Medium may present a tag input field. If the user
supplied tags in the inputs, attempt to add each tag. If Medium's
UI rejects a tag (for example, a tag that doesn't exist as a topic
yet), skip it silently and continue. **Do not loop more than twice
on tag UI failures** — if a tag won't add, skip it and move on.

### Step 9 — Update metadata with the Medium URL

After the import succeeds, write the Medium draft URL back into the
repository so the post is traceable:

1. Update `posts/<YYYY-MM-DD>-<slug>/meta.json` — set `medium_url` to
   the draft editor URL.
2. Update `posts/index.json` — set `medium_url` for the same post.
3. Commit and push:

   ```bash
   git add posts/<YYYY-MM-DD>-<slug>/meta.json posts/index.json
   git commit -m "chore(posts): link medium draft for \"<POST TITLE>\""
   git push origin main
   ```

### Step 10 — Report to the user

Send the user a final summary in this shape:

```
✅ Post created: "<TITLE>"
📄 GitHub Pages: https://<username-or-org>.github.io/<repo-name>/posts/<YYYY-MM-DD>-<slug>/
📝 Medium draft: <draft-editor-url>

The draft is ready for your review. Open the Medium link, make any edits,
and click Publish when you're happy with it.
```

If anything failed at any step, report the exact step, the exact
error, and the next action the user should take. Do not invent
success messages.

## Error recovery

| Problem | Fix |
|---------|-----|
| `gh` not installed | Ask the user to install it: https://cli.github.com |
| `gh auth status` shows unauthenticated | Ask the user to run `gh auth login` |
| Repository creation fails (already exists) | Fall back to cloning the existing repo |
| Git push fails with auth error | Ask the user to run `gh auth login`, then retry (max 2 attempts) |
| GitHub Pages returns 404 | Wait 60s, retry. If still 404 after 3 min, ask user to check the Actions tab |
| Medium import shows empty content | The HTML must be static — verify it loads in a browser first by visiting the GitHub Pages URL |
| Medium import fails silently | Verify the GitHub Pages URL returns HTTP 200 and includes the trailing slash |
| `posts/index.json` merge conflict | `git pull --rebase origin main`, then re-add changes |
| Browser times out on Medium | Close the tab, open a fresh one, retry the navigate step |
| Tag won't add in Medium UI | Skip that tag, continue. Do not loop. |

## Security and privacy notes

- **Posts are public on GitHub Pages** before being imported to Medium.
  This is by design — Medium's importer requires a publicly fetchable
  URL. Do not use this skill for posts that contain secrets or
  information the user does not want published in a public repository.
- **No credentials are stored by the skill.** Authentication is delegated
  to `gh auth login` and the user's existing browser session.
- **The skill does not call any third-party API** beyond the user's own
  GitHub repository and Medium's public import endpoint.

## Rate limits to be aware of

Medium enforces a daily publishing limit (as of early 2026: **2 stories
+ 100 responses per rolling 24 hours**). This skill stops at draft, so
the limit does not block automated drafting — but **the user should not
auto-publish more than 2 stories per day** from this skill's output.

## License

MIT. This skill is part of the [openclaw-skills](https://github.com/ylnhari/openclaw-skills)
collection. See the umbrella repository's [LICENSE](https://github.com/ylnhari/openclaw-skills/blob/main/LICENSE)
for the full text.
