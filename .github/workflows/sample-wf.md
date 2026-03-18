---
on:
  workflow_dispatch:

permissions:
  contents: read
  issues: read

network: defaults

safe-outputs:
  create-issue:
    title-prefix: "[hello] "
    labels: [sample]

engine: copilot
---

# Hello World

Create a GitHub issue that says "Hello World! 👋"

## Instructions

1. Create a single GitHub issue
2. The issue body should contain a friendly "Hello World" greeting
3. Include today's date and a fun emoji
4. Keep it short and simple — this is just a test!
