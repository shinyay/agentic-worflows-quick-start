---
description: |
  Checks the GitHub Changelog (https://github.blog/changelog/) for recent
  updates, summarizes them by category, and creates a GitHub issue with
  the highlights. Useful for staying on top of new GitHub features,
  API changes, and deprecations.

on:
  schedule: weekly on monday around 9am
  workflow_dispatch:

permissions:
  contents: read
  issues: read

tools:
  web-fetch:
  github:
    toolsets: [repos, issues]

network:
  allowed:
    - defaults
    - github

safe-outputs:
  create-issue:
    title-prefix: "[changelog] "
    labels: [github-changelog, weekly-summary]
    close-older-issues: true

engine: copilot
---

# GitHub Changelog Weekly Summary

Fetch the GitHub Changelog and create a concise weekly summary as a GitHub issue.

## Goal

Help the team stay informed about new GitHub features, changes, and deprecations
without having to read every changelog entry individually.

## Instructions

1. Fetch the GitHub Changelog page at https://github.blog/changelog/
2. Identify entries from the **last 7 days**
3. For each entry, capture:
   - The title and date
   - A 1-2 sentence summary of what changed
   - The link to the full changelog entry
4. Group entries into these categories:
   - 🚀 **New Features** — new capabilities and products
   - 🔄 **Changes & Improvements** — updates to existing features
   - ⚠️ **Deprecations & Removals** — things being removed or deprecated
   - 🔒 **Security** — security-related updates
   - 📦 **API & Integrations** — API changes, webhook updates, integrations
5. Create a GitHub issue with the summary

## Output Format

The issue should follow this structure:

```
## GitHub Changelog Summary — Week of [DATE]

### 🚀 New Features
- **[Title]** (Date) — Brief summary. [Read more](link)

### 🔄 Changes & Improvements
- **[Title]** (Date) — Brief summary. [Read more](link)

### ⚠️ Deprecations & Removals
- **[Title]** (Date) — Brief summary. [Read more](link)

### 🔒 Security
- **[Title]** (Date) — Brief summary. [Read more](link)

### 📦 API & Integrations
- **[Title]** (Date) — Brief summary. [Read more](link)

---
> 💡 **Highlights**: Call out 1-3 most impactful changes at the top.
> 📋 Source: https://github.blog/changelog/
```

## Rules

- Only include entries from the last 7 days
- Skip categories that have no entries (don't show empty sections)
- Keep summaries concise — 1-2 sentences per entry
- If no entries found for the week, create an issue noting "No changelog updates this week"
- Use emojis for visual scanning but keep it professional
