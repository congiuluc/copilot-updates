---
mode: agent
tools: ['web/fetch', 'edit']
description: 'Fetch GitHub Copilot changelog articles and create structured markdown files for PowerPoint conversion'
---

# GitHub Copilot Changelog Fetcher

Fetch GitHub Copilot changelog articles from the GitHub Blog and produce structured markdown files ready for PowerPoint conversion via `create_pptx.py`.

## Inputs

- **Start date**: ${input:startDate} (YYYY-MM-DD)
- **End date**: ${input:endDate} (YYYY-MM-DD)
- **Language**: ${input:language} (default: english)

## Rules (apply throughout all steps)

- Process **ALL** articles in the date range — never stop early or skip items.
- Follow **all pagination links** on listing pages; do not stop at the first page.
- **Language handling**: When `${input:language}` is `english` (default), write everything in English. When a different language is specified, translate **summaries** and **speaker notes** into that language. **Do NOT translate**: article titles (keep the original English title in YAML `title`, `# Heading`, and slide references), product/feature names (e.g. "Copilot", "GitHub Actions", "Gemini 3.1 Pro"), proper nouns, technical terms, or any text that originates from the source article and would lose meaning if translated.
- YAML values must be **double-quoted**. Filenames must match the URL slug.
- The Python script parses this exact structure — do not alter headings, separators, or comment format.

---

## Step 1 — Fetch article listings

Use `#fetch` to retrieve listing pages (no `&type=` filter):

| Scope | URL |
|---|---|
| Current/latest year | `https://github.blog/changelog/?label=copilot` |
| Specific year | `https://github.blog/changelog/{year}/?label=copilot` |

Fetch every year page that overlaps the date range. Follow all pagination links.

Extract per entry:

| Field | Source |
|---|---|
| **Title** | Entry heading |
| **URL** | `https://github.blog/changelog/YYYY-MM-DD-slug` |
| **Date** | From the URL slug |
| **Type** | Badge label → `new-release` (New release / New), `improvement`, `deprecation` (Retired / Deprecation) |

Keep only articles whose date falls within [startDate, endDate] inclusive.

## Step 2 — Fetch each article & save raw content

Use `#fetch` on each article URL (returns markdown-converted content). Extract:

- **Title**: first `# Heading`
- **Hero image**: first `![...](URL)` — skip placeholders like `[Image: Image]`; empty string if none
- **Body text**: remaining paragraph text

Save to `output/raw/YYYY-MM-DD-slug.md`:

```markdown
---
title: "Article Title"
date: "YYYY-MM-DD"
type: "new-release|improvement|deprecation"
image_url: "https://..."
article_url: "https://github.blog/changelog/YYYY-MM-DD-slug"
---

# Article Title

## Raw Content

Plain text body here.
```

## Step 3 — Generate summaries, classify & save

For each raw file, generate:

1. **Summary** (3–5 sentences): what changed, who it affects, key benefit.
2. **Speaker notes** (5–8 sentences): conversational tone — practical impact, rollout dates, demo ideas.

Both must be written in `${input:language}`. Keep article titles, product names, feature names, and technical terms in their original English form — translate only the explanatory prose around them.

If the type could not be determined from the listing badges, infer it from the article content (e.g. mentions of retirement → `deprecation`).

Save final files organized by type:

```
output/new-releases/YYYY-MM-DD-slug.md
output/improvements/YYYY-MM-DD-slug.md
output/deprecations/YYYY-MM-DD-slug.md
```

### Output file format

```markdown
---
title: "Article Title"
date: "YYYY-MM-DD"
type: "new-release|improvement|deprecation"
image_url: "https://..."
article_url: "https://github.blog/changelog/YYYY-MM-DD-slug"
---

# Article Title

![hero](image_url_here)

---

## Summary

Summary in ${input:language} here (3–5 sentences).

<!--
speaker_notes:
Speaker notes in ${input:language} here (5–8 sentences).
-->
```

> Omit `![hero]()` line entirely when no image is available.

## Step 4 — Create index file

After all articles are processed, create `output/index.md`:

```markdown
# GitHub Copilot Updates: {startDate} - {endDate}

## 🚀 New Releases
- [Article Title](new-releases/YYYY-MM-DD-slug.md) - YYYY-MM-DD

## ✨ Improvements
- [Article Title](improvements/YYYY-MM-DD-slug.md) - YYYY-MM-DD

## ⚠️ Deprecations
- [Article Title](deprecations/YYYY-MM-DD-slug.md) - YYYY-MM-DD
```
