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

### 3. File the GitHub issue

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

- **Label:** `wiki` (create it if it doesn't exist — use `gh label create wiki --color 0075ca --description "Wiki ingestion requests"` first if needed)

### 4. Report

Print the issue URL so the user can see it.
