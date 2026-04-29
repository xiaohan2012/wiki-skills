---
name: wiki-read-arxiv
description: Use when adding an arxiv paper to the wiki. Fetches full LaTeX source for richer extraction than PDF, then runs the wiki-ingest pipeline.
---

# Wiki Read Arxiv

Fetch an arxiv paper's LaTeX source, read it fully, then ingest it into the wiki via the standard wiki-ingest pipeline.

## Pre-condition

Search for `SCHEMA.md` starting from the current directory and upward, or in common wiki locations (`~/wikis/`). If not found, tell the user to run `wiki-init` first.

Read `SCHEMA.md` to learn: wiki root path, page frontmatter format, cross-reference convention, log entry format, index category taxonomy.

## Process

### 1. Accept the URL

Accept any arxiv URL form:
- `https://arxiv.org/abs/2601.07372`
- `https://arxiv.org/pdf/2601.07372`
- `https://arxiv.org/src/2601.07372`

Extract the arxiv ID (e.g. `2601.07372`).

### 2. Download the LaTeX source

Normalize the URL to the source form: `https://arxiv.org/src/<arxiv_id>`

Check if `<wiki-root>/raw/<arxiv_id>.tar.gz` already exists. If so, skip the download.

Otherwise, download to `<wiki-root>/raw/<arxiv_id>.tar.gz`:

```bash
curl -L "https://arxiv.org/src/<arxiv_id>" -o "<wiki-root>/raw/<arxiv_id>.tar.gz"
```

### 3. Unpack

Unpack into `<wiki-root>/raw/<arxiv_id>/`:

```bash
mkdir -p "<wiki-root>/raw/<arxiv_id>"
tar -xzf "<wiki-root>/raw/<arxiv_id>.tar.gz" -C "<wiki-root>/raw/<arxiv_id>"
```

### 4. Locate the entrypoint

Find the main `.tex` file. Look for:
1. `main.tex`
2. The file containing `\documentclass`

### 5. Read the paper

Read the entrypoint `.tex` file. Follow all `\input{}` and `\include{}` directives recursively to read the full paper. Prefer LaTeX source over any compiled PDF — the source contains the full text without rendering artifacts.

### 6. Extract metadata from LaTeX source

Before handing off, extract from the LaTeX:
- **Title**: from `\title{}`
- **Authors**: from `\author{}`
- **Venue**: from `\booktitle{}`, `\journal{}`, or arxiv submission metadata
Fall back to `Unknown` for any field not found.

### 7. Hand off to wiki-ingest

Continue from **wiki-ingest step 3** onward (surface takeaways, ask user, write pages, update index/overview/log). Use the arxiv URL as the source reference and `<wiki-root>/raw/<arxiv_id>/` as the raw source location.

The slug should be derived from the paper title (not the arxiv ID), e.g. `attention-is-all-you-need`.
