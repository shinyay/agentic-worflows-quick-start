# GitHub Agentic Workflows — Step-by-Step Getting Started Tutorial

This tutorial walks you through setting up and running your first GitHub Agentic Workflow in this repository (`agentic-worflows-quick-start`).

---

## Prerequisites

Before you begin, make sure you have:

| Requirement | Check Command | Status |
|------------|---------------|--------|
| **GitHub CLI** v2.0.0+ | `gh --version` | ✅ |
| **GitHub repository** with write access | You're in one! | ✅ |
| **GitHub Actions** enabled on the repo | Settings → Actions | Check |
| **AI Provider Account** (one of the following): | | |
| — GitHub Copilot subscription | [github.com/features/copilot](https://github.com/features/copilot) | |
| — Anthropic Claude API key | [console.anthropic.com](https://console.anthropic.com/) | |
| — OpenAI API key | [platform.openai.com](https://platform.openai.com/api-keys) | |
| — Google Gemini API key | [aistudio.google.com](https://aistudio.google.com/api-keys) | |

---

## Step 1 — Install the `gh aw` CLI Extension

```bash
gh extension install github/gh-aw
```

Verify the installation:

```bash
gh aw version
```

> **Tip**: To pin a specific version for reproducibility:
> ```bash
> gh extension install github/gh-aw@v0.1.0
> ```

---

## Step 2 — Initialize Your Repository

Run `gh aw init` from the repository root. This sets up `.gitattributes`, creates the dispatcher agent file, and configures your engine.

```bash
cd /path/to/your/repo
gh aw init
```

The interactive wizard will:
1. Configure `.gitattributes` for lock file merging
2. Create `.github/agents/agentic-workflows.agent.md`
3. Ask you to select an AI engine (Copilot, Claude, Codex, or Gemini)
4. Prompt you to configure the required API secret

---

## Step 3 — Set Up Your AI Engine Secret

Depending on which engine you chose, set the required secret:

### Option A: GitHub Copilot (default, recommended)

1. Create a **fine-grained PAT** at [github.com/settings/personal-access-tokens/new](https://github.com/settings/personal-access-tokens/new)
   - Resource owner: **your user account** (not an organization)
   - Permissions → Account permissions → **Copilot Requests: Read**
2. Add it as a repository secret:

```bash
gh aw secrets set COPILOT_GITHUB_TOKEN --value "ghp_your_token_here"
```

### Option B: Claude by Anthropic

```bash
gh aw secrets set ANTHROPIC_API_KEY --value "sk-ant-your_key_here"
```

### Option C: OpenAI Codex

```bash
gh aw secrets set OPENAI_API_KEY --value "sk-your_key_here"
```

### Option D: Google Gemini

```bash
gh aw secrets set GEMINI_API_KEY --value "your_gemini_key_here"
```

> **Verify**: Run `gh aw secrets bootstrap` to check all required secrets are configured.

---

## Step 4 — Add Your First Workflow (Pre-Built)

The easiest way to start is by adding a pre-built workflow from the **Agentics Collection**:

```bash
gh aw add-wizard githubnext/agentics/daily-repo-status
```

This interactive wizard will:
1. Check prerequisites
2. Confirm engine selection
3. Add the workflow `.md` + compiled `.lock.yml` to `.github/workflows/`
4. Optionally trigger the first run

After completion, you'll see:
```
.github/
└── workflows/
    ├── daily-repo-status.md           # Your workflow (natural language)
    └── daily-repo-status.lock.yml     # Compiled GitHub Actions YAML
```

---

## Step 5 — Trigger Your First Run

```bash
gh aw run daily-repo-status
```

This dispatches the workflow immediately. Track it at the URL printed in the output, or:

```bash
gh aw status --ref main
```

Wait 2–3 minutes for completion. The workflow will create a **GitHub Issue** with a daily status report for your repository.

---

## Step 6 — Create Your Own Custom Workflow

Now let's write one from scratch.

### 6a. Create the workflow file

```bash
gh aw new issue-greeter
```

This creates `.github/workflows/issue-greeter.md`. Edit it:

```markdown
---
on:
  issues:
    types: [opened]
  reaction: "eyes"
permissions:
  contents: read
  issues: read
safe-outputs:
  add-comment:
---

# Issue Greeter

When a new issue is opened, welcome the author and help them.

## Instructions

1. Read the newly opened issue #${{ github.event.issue.number }}
2. Greet the issue author warmly
3. If the issue is unclear, politely ask clarifying questions
4. If the issue is clear, suggest relevant files or documentation they might find helpful
5. Keep the tone friendly and encouraging
```

### 6b. Compile the workflow

```bash
gh aw compile issue-greeter
```

This generates `.github/workflows/issue-greeter.lock.yml` with all security hardening applied.

### 6c. Commit and push

```bash
git add .github/
git commit -m "Add issue-greeter agentic workflow"
git push
```

### 6d. Test it

Open a new issue on your repository — the workflow will automatically run and post a friendly comment!

Or trigger manually:

```bash
gh aw run issue-greeter
```

---

## Step 7 — Explore More Workflow Examples

Here are more useful workflows to try:

### CI Failure Doctor

Automatically analyzes CI failures and suggests fixes:

```bash
gh aw add-wizard githubnext/agentics/ci-doctor
```

### PR Reviewer

Reviews pull requests and provides code feedback:

```markdown
---
on:
  pull_request:
    types: [opened, synchronize]
permissions:
  contents: read
  pull-requests: read
safe-outputs:
  add-comment:
  add-labels:
    allowed: [needs-review, approved, changes-requested]
---

# PR Reviewer

Review the pull request and provide constructive feedback.

## What to review
- Code quality and best practices
- Potential bugs or edge cases
- Documentation completeness
- Test coverage
```

### Weekly Summary Report

```markdown
---
on:
  schedule: weekly on monday around 9am
permissions:
  contents: read
  issues: read
  pull-requests: read
safe-outputs:
  create-issue:
    title-prefix: "[weekly] "
    labels: [report]
    close-older-issues: true
---

# Weekly Summary

Create a weekly summary of repository activity as a GitHub issue.

## Include
- Issues opened, closed, and in progress
- Pull requests merged
- Notable code changes
- Recommendations for the upcoming week
```

---

## Step 8 — Key CLI Commands Cheat Sheet

| Command | What It Does |
|---------|-------------|
| `gh aw init` | Set up repo for agentic workflows |
| `gh aw new <name>` | Create a new workflow from template |
| `gh aw add-wizard <source>` | Add pre-built workflow interactively |
| `gh aw compile` | Compile all `.md` → `.lock.yml` |
| `gh aw compile --watch` | Auto-recompile on changes |
| `gh aw validate` | Validate without generating output |
| `gh aw run <workflow>` | Trigger a workflow immediately |
| `gh aw run <workflow> --push` | Commit, push, then trigger |
| `gh aw status` | Check workflow states |
| `gh aw logs <workflow>` | View workflow execution logs |
| `gh aw audit <run-id>` | Deep-dive into a specific run |
| `gh aw health` | View success rates and metrics |
| `gh aw list` | List all workflows |
| `gh aw enable / disable` | Toggle workflows on/off |
| `gh aw secrets bootstrap` | Check and set up missing secrets |

---

## Step 9 — Understanding the Security Model

Every agentic workflow runs with these guardrails **by default**:

```
┌─────────────────────────────────────────────────────┐
│                  YOUR WORKFLOW                        │
│                                                      │
│  1. Agent runs READ-ONLY (no write access)          │
│  2. Network is FIREWALLED (domain allowlist only)   │
│  3. Writes go through SAFE OUTPUTS only             │
│  4. THREAT DETECTION scans output before writing    │
│  5. PROTECTED FILES block changes to package.json,  │
│     .github/workflows/, etc.                        │
│  6. All actions SHA-PINNED at compile time          │
└─────────────────────────────────────────────────────┘
```

### Key security principles

- **Read-only by default**: The AI agent cannot write to your repository directly
- **Safe outputs**: Write operations (create issue, PR, comment) are performed by separate jobs with scoped permissions
- **Sandboxed execution**: The agent runs inside a container with network egress controlled by the Agent Workflow Firewall (AWF)
- **Threat detection**: An AI-powered security scan runs between the agent and any write operations
- **Strict mode** (default): Enforces SHA-pinned actions, no wildcard domains, no direct write permissions

---

## Step 10 — Next Steps

1. **Read the full documentation**: [github.github.io/gh-aw](https://github.github.io/gh-aw/)
2. **Browse pre-built workflows**: [githubnext/agentics](https://github.com/githubnext/agentics)
3. **Explore patterns**:
   - [Orchestration](https://github.github.io/gh-aw/patterns/orchestration/) — Multi-workflow coordination
   - [MultiRepoOps](https://github.github.io/gh-aw/patterns/multi-repo-ops/) — Cross-repo automation
   - [ChatOps](https://github.github.io/gh-aw/patterns/chat-ops/) — Slash command driven workflows
4. **Add MCP tools**: Integrate Slack, Jira, or custom APIs via [MCP servers](https://github.github.io/gh-aw/guides/mcps/)
5. **Join the community**: [GitHub Next Discord](https://gh.io/next-discord) | [Discussions](https://github.com/orgs/community/discussions/186451)

---

## Quick Reference: Workflow Anatomy

```markdown
---
# FRONTMATTER (YAML) — Compiled into GitHub Actions
on:                    # When to run (triggers)
  issues:
    types: [opened]
permissions:           # GitHub token permissions (read-only default)
  contents: read
engine: copilot        # AI engine (copilot/claude/codex/gemini)
tools:                 # Available tools for the agent
  github:
    toolsets: [issues]
  bash: ["echo", "ls"]
network:               # Network access control
  allowed:
    - defaults
safe-outputs:          # Allowed write operations
  add-comment:
  create-issue:
    labels: [automated]
---

# MARKDOWN BODY — Natural language instructions (editable at runtime)
# Workflow Title

Describe what the AI agent should do in plain English.
The agent reads this at runtime and executes accordingly.
```

> **Remember**: You can edit the markdown body directly on GitHub.com — no recompilation needed! Only frontmatter changes require `gh aw compile`.
