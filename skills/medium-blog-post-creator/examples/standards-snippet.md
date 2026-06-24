# HTML standards cheat sheet

Medium's URL importer converts only a constrained subset of HTML into
blocks cleanly. Anything outside this subset is stripped, mangled, or
rejected. Memorize this list — it's the entire contract.

## Allowed tags

| Tag | Notes |
|-----|-------|
| `<h1>` | Exactly one per post, identical to `<title>` |
| `<h2>` | Section headings (use these liberally) |
| `<h3>` | Subsections |
| `<p>` | Body paragraphs |
| `<strong>` | Bold |
| `<em>` | Italic |
| `<a href="...">` | Links |
| `<img src="..." alt="...">` | Absolute HTTPS URLs only |
| `<blockquote>` | Pull-quotes |
| `<pre><code>` | Code blocks; use `&lt;` and `&gt;` for `<` and `>` |
| `<ul>` / `<ol>` / `<li>` | Lists |
| `<hr>` | Section dividers — **max 2 per post** |

## Banned tags and patterns

| Banned | Why |
|--------|-----|
| `<script>` | Stripped; never executes in Medium |
| `<style>` | Stripped |
| `<iframe>` | Rejected |
| `<table>` | Rejected (Medium's importer doesn't render tables cleanly) |
| `<div>` | Stripped |
| `<span>` | Stripped |
| Inline `style="..."` | Stripped |
| `<h4>`, `<h5>`, `<h6>` | Use `<h3>` and bold text instead |

## Image rules

- **Absolute HTTPS only.** No relative paths.
- Recommended source: `https://images.unsplash.com/photo-{ID}?w=1200&q=80`
- Always provide `alt` text.

## Code block rules

Inside `<pre><code>`:

- Use `&lt;` and `&gt;` for angle brackets.
- The importer does not highlight syntax; rely on monospace formatting.

## YouTube embedding

Paste the bare URL in a `<p>` tag:

```html
<p>https://www.youtube.com/watch?v=XXXXXXXXXXX</p>
```

Medium's importer will convert it into an embed block automatically.

## Frontmatter for cover-image metadata

For posts to import with the correct cover image and published date, ensure
the `<head>` includes:

```html
<meta property="og:image" content="COVER_IMAGE_URL">
<meta property="article:published_time" content="YYYY-MM-DDTHH:MM:SSZ">
```

Both must use absolute HTTPS URLs / ISO-8601 dates.
