# Copilot Updates → PowerPoint

> A two-step workflow to fetch GitHub Copilot changelog articles and turn them into a polished presentation — powered by a VS Code Copilot agent and a Python script.

[![Built with GitHub Copilot](https://img.shields.io/badge/Built%20with-GitHub%20Copilot-8957e5?logo=githubcopilot)](https://github.com/features/copilot)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.11+](https://img.shields.io/badge/Python-3.11%2B-3776ab?logo=python&logoColor=white)](https://www.python.org)

---

## 🙏 Credits

Special thanks to [@congiuluc](https://github.com/congiuluc) for conceiving and inspiring this solution.

---

## ✨ Features

- 🤖 **Copilot agent prompt** fetches and classifies changelog articles automatically
- 📊 **Dark GitHub-themed slides** — widescreen 16:9 `.pptx` with section dividers, hero images, and speaker notes
- 🌍 **Multi-language support** — summaries and notes can be translated into any language
- 🚀 New Releases · ✨ Improvements · ⚠️ Deprecations — articles sorted by type

---

## 📋 Prerequisites

- [VS Code](https://code.visualstudio.com/) with [GitHub Copilot](https://github.com/features/copilot) (agent mode)
- [Python 3.11+](https://www.python.org)

## 🚀 Getting started

```bash
# Create a virtual environment
python -m venv .venv

# Activate it
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
# macOS / Linux
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

---

## 📖 Workflow

### Step 1 — Generate markdown files with the Copilot agent

Open VS Code Copilot Chat in **Agent mode** and run:

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

1. Fetch **all** article listings from `github.blog/changelog` filtered by `copilot` label
2. Fetch each individual article page and **save raw content** under `output/raw/`
3. Classify each article by type (new-release, improvement, deprecation)
4. Generate a summary and speaker notes (in your chosen language)
5. Save final structured markdown files under `output/` organized by type

<details>
<summary>📁 Output file structure</summary>

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

</details>

### Step 2 — Generate the PowerPoint

```bash
python create_pptx.py
```

Options:

```bash
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

> [!NOTE]
> When a non-English language is specified in Step 1, summaries and speaker notes are translated into that language. Article titles, product names, and technical terms always stay in English.

---

## 🗂️ Markdown file format

<details>
<summary>Each article markdown file follows this structure (produced by the Copilot agent)</summary>

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

</details>

---

## 🤖 Built with GitHub Copilot

This repository was created entirely with [GitHub Copilot](https://github.com/features/copilot).


## �📄 License

This project is licensed under the [MIT License](LICENSE).
