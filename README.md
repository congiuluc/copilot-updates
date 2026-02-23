# Copilot Updates → PowerPoint

A two-step workflow to fetch GitHub Copilot changelog articles and turn them into a presentation.

## Prerequisites

- **VS Code** with GitHub Copilot (agent mode)
- **Python 3.11+**

Install Python dependencies:

```bash
pip install -r requirements.txt
```

## Workflow

### Step 1: Generate markdown files with the Copilot agent

Open VS Code Copilot Chat in **Agent mode** and run the prompt:

```
/fetch-copilot-news
```

You'll be asked for:

| Input | Description | Example |
|---|---|---|
| `startDate` | Start of date range (YYYY-MM-DD) | `2025-01-01` |
| `endDate` | End of date range (YYYY-MM-DD) | `2025-12-31` |
| `language` | Language for summaries and speaker notes | `english` (default) |

The agent will:
1. Fetch **all** article listings from `github.blog/changelog` filtered by `copilot` label (no type filter)
2. Fetch each individual article page and **save raw content** locally under `output/raw/`
3. Classify each article by type (new-release, improvement, deprecation)
4. Generate a summary and speaker notes (in your chosen language)
5. Save final structured markdown files under `output/` organized by type

The resulting file structure:

```
output/
├── index.md                          # Table of contents
├── raw/                              # Raw fetched content (all articles)
│   ├── 2025-03-15-some-feature.md
│   ├── 2025-04-01-some-improvement.md
│   └── ...
├── new-releases/                     # Classified final files
│   ├── 2025-03-15-some-feature.md
│   └── ...
├── improvements/
│   ├── 2025-04-01-some-improvement.md
│   └── ...
└── deprecations/
    └── ...
```

### Step 2: Generate the PowerPoint

```bash
python create_pptx.py
```

Options:

```
python create_pptx.py --output-dir output/ --output my-presentation.pptx
python create_pptx.py --from-date 2025-01-01 --to-date 2025-12-31
```

| Flag | Default | Description |
|---|---|---|
| `--output-dir`, `-d` | `output/` | Directory with markdown files |
| `--output`, `-o` | auto-generated | Output `.pptx` filename |
| `--from-date` | auto-detected | Start date shown on title slide |
| `--to-date` | auto-detected | End date shown on title slide |

The script produces a widescreen (16:9) `.pptx` with dark GitHub-themed slides:

- **Title slide** with the date range
- **Section dividers** for New Releases 🚀, Improvements ✨, Deprecations ⚠️
- **Article title slide** with hero image
- **Summary slide** with speaker notes in the Notes pane

> When a non-English language is specified in Step 1, summaries and speaker notes are translated into that language. Article titles, product names, and technical terms always stay in English.

## Markdown file format

Each article markdown file follows this structure (produced by the Copilot agent):

```markdown
---
title: "Article Title"
date: "2025-03-15"
type: "new-release"
image_url: "https://github.blog/wp-content/uploads/..."
article_url: "https://github.blog/changelog/2025-03-15-slug"
---

# Article Title

![hero](https://github.blog/wp-content/uploads/...)

---

## Summary

Summary text here.

<!--
speaker_notes:
Speaker notes in the target language.
-->
```

## License

MIT
