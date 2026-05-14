---
name: wiki-request
description: Queue a paper or URL for wiki ingestion by filing a GitHub issue on this repo.
---

# Wiki Request

File a GitHub issue requesting that a source be ingested into the wiki via `/wiki-ingest`.

## Process

### 1. Parse the argument

The argument can be:
- An arxiv URL (e.g. `https://arxiv.org/abs/2305.07428`)
- An arxiv ID (e.g. `2305.07428`)
- Any URL
- A paper title or free-form description

Normalize arxiv IDs to full URLs: `https://arxiv.org/abs/<id>`.

### 2. Resolve title and URL

Either a URL or a paper name/title may be provided — resolve whichever is missing:

- **URL provided, no title:** Fetch the URL and extract the title automatically.
  - For arxiv URLs/IDs: extract from the `<title>` tag (strip the "[] " prefix arxiv adds).
  - For other URLs: extract from `<title>` or `<h1>`.

- **Title/name provided, no URL:** Search the web for the paper or resource and find a canonical URL (prefer arxiv for papers). Present the found URL to the user and ask for confirmation before proceeding.

- **Both provided:** Use as-is.

Show the resolved (title, URL) pair to the user and ask for confirmation before filing the issue.

### 3. Determine tags

Tag the issue with **venue**, **year**, and **topic** labels.

- **Venue** (e.g., `icml`, `iclr`, `neurips`): infer from the paper's publication metadata. For arxiv papers, check the comments field (often says e.g. "Accepted at NeurIPS 2024"). If unknown, skip the venue label.
- **Year** (e.g., `2025`, `2026`): the year of publication (arxiv submission year is fine if no venue is known).
- **Topic** (e.g., `diffusion`, `tabular`, `survey`): pick the best-fitting topic(s) from the existing labels in the issue's target repo. Run `gh label list --repo <owner>/<repo> --limit 100` to see what exists. If no existing topic fits, propose a new one to the user and create it with `gh label create <name> --repo <owner>/<repo> --description "<desc>"` after confirmation.

Create any missing venue/year labels on the fly (e.g. `gh label create 2026 --repo <owner>/<repo> --description "2026"`).

Show the proposed tag set to the user along with the (title, URL) pair in step 2's confirmation.

### 4. File the GitHub issue

Use `gh issue create` on the current repo with:
- **Title:** `wiki-request: <human-readable title>`
- **Body:**

```
Please ingest the following source into the wiki using `/wiki-ingest`.

**Source:** <full URL or description>
**Title:** <human-readable title>

Run in this repo:
\`\`\`
/wiki-ingest <source>
\`\`\`
```

- **Labels:** the venue / year / topic labels from step 3. Pass multiple labels with repeated `--label` flags.

### 5. Report

Print the issue URL so the user can see it.
