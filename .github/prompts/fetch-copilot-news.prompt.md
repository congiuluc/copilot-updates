---
mode: agent
tools: ['web/fetch', 'edit'
description: 'Fetch GitHub Copilot changelog articles and create markdown files for a presentation'
---

# GitHub Copilot Changelog Fetcher

Fetch GitHub Copilot changelog articles from the GitHub Blog and create structured markdown files for PowerPoint conversion.

## Inputs

- **Start date**: ${input:startDate} (YYYY-MM-DD)
- **End date**: ${input:endDate} (YYYY-MM-DD)
- **Speaker notes language**: ${input:language} (default: italian)

## Step 1: Fetch article listings

Use `#fetch` to retrieve listing pages. URLs by year (no `&type=` filters):

- Current/latest year: `https://github.blog/changelog/?label=copilot`
- Specific year: `https://github.blog/changelog/{year}/?label=copilot`

Fetch all year pages that overlap the date range. Follow pagination links to collect every article.

Extract from each entry:
- **Title** and **URL** (`https://github.blog/changelog/YYYY-MM-DD-slug`)
- **Date** from the URL slug
- **Type** from badges: "New release"/"New" → `new-release`, "Improvement" → `improvement`, "Retired"/"Deprecation" → `deprecation`

Keep only articles within the date range (inclusive).

## Step 2: Fetch articles and save raw content

Use `#fetch` on each article URL. The tool returns **markdown-converted** content (not raw HTML). Extract:
- **Title**: first `# Heading` in the content
- **Hero image**: first markdown image `![...](URL)` found in the content. Ignore placeholder images like `[Image: Image]`. Empty string if none found.
- **Body text**: paragraph text from the article body

Save each as `output/raw/YYYY-MM-DD-slug.md`:

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

## Step 3: Generate summaries and classify

For each raw file, generate:

1. **Summary** (English, 3-5 sentences): what changed, who it affects, key benefit.
2. **Speaker notes** (${input:language}, 5-8 sentences): conversational explanation, practical impact, rollout dates, demo ideas.

**Language rule:** Title and summary are ALWAYS in English. Only speaker notes use ${input:language}.

Save final files organized by type:

```
output/new-releases/YYYY-MM-DD-slug.md
output/improvements/YYYY-MM-DD-slug.md
output/deprecations/YYYY-MM-DD-slug.md
```

## Final file format

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

English summary here (3-5 sentences).

<!--
speaker_notes:
Speaker notes in ${input:language} here (5-8 sentences).
-->
```

**Format rules:** Double-quoted YAML values. `---` separates image from summary (slide break). Speaker notes inside `<!-- speaker_notes: ... -->`. Omit `![hero]()` if no image. Filename matches the URL slug.
- If the type could not be determined from the listing page, infer it from the article content (e.g., if the article mentions deprecation or retirement, classify as `deprecation`)

### Step 7: Create index file

After all articles are processed, create an `output/index.md` file listing all articles grouped by type:

```markdown
# GitHub Copilot Updates: {startDate} - {endDate}

## 🚀 New Releases
- [Article Title](new-releases/YYYY-MM-DD-slug.md) - YYYY-MM-DD

## ✨ Improvements
- [Article Title](improvements/YYYY-MM-DD-slug.md) - YYYY-MM-DD

## ⚠️ Deprecations
- [Article Title](deprecations/YYYY-MM-DD-slug.md) - YYYY-MM-DD
```

## Important Notes

- Be thorough: process ALL articles in the date range, not just the first few
- Fetch all articles in a single listing request per year (no type filtering during fetch)
- Follow pagination links to collect all articles — do not stop at the first page
- Always save raw content to `output/raw/` before generating summaries or classifying by type
- If a type cannot be determined from the listing page labels, infer it from the article content
- Maintain consistent markdown formatting — the Python script depends on parsing this exact structure
- Download and inspect each article page individually to get accurate content
- The speaker notes language defaults to Italian if not specified
- ALL slide-visible text (titles, summaries) MUST always be in English — only speaker notes use the target language
