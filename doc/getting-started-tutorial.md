# GitHub Agentic Workflows — Step-by-Step Getting Started Tutorial

> _Based on gh-aw v0.61.0 · Last reviewed 2026-04_

This tutorial walks you through setting up and running your first GitHub Agentic Workflow in this repository (`agentic-worflows-quick-start`).

> 📚 **See also — Deep Dive:** For conceptual background and reference material that complements this tutorial, see the [Deep Dive documentation set](./deep-dive/README.md). Most relevant to this tutorial:
> - [10 — Writing Workflows Cookbook](./deep-dive/10-writing-workflows-cookbook.md) — patterns and recipes
> - [03 — Frontmatter Reference](./deep-dive/03-frontmatter-reference.md) — every field explained
> - [11 — Debugging & Observability](./deep-dive/11-debugging-and-observability.md) — when your first run misbehaves

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

### 🔍 What Just Happened?

`gh aw` is a **GitHub CLI extension** — a Go binary from the [github/gh-aw](https://github.com/github/gh-aw) repository that adds the `gh aw` command family to your terminal.

It provides **three core capabilities** that map to the workflow lifecycle:

```
 ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
 │  AUTHOR   │────▶│ COMPILE  │────▶│   RUN    │────▶│ MONITOR  │
 │           │     │          │     │          │     │          │
 │ init, new │     │ compile  │     │ run      │     │ logs     │
 │ add, add- │     │ validate │     │ trial    │     │ audit    │
 │ wizard    │     │ fix      │     │ enable   │     │ health   │
 │ secrets   │     │          │     │ disable  │     │ status   │
 └──────────┘     └──────────┘     └──────────┘     └──────────┘
```

**Key insight**: The `gh aw` extension runs **locally on your machine**. It creates and compiles workflow files. The actual AI agents (Copilot, Claude, Codex, Gemini) run **remotely in GitHub Actions** — your local CLI never executes AI inference itself.

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

### 🔍 What Just Happened?

`gh aw init` created **5 files** that prepare your repository for agentic workflows:

| File | Purpose |
|------|---------|
| **`.gitattributes`** | Marks `.lock.yml` files as `linguist-generated` (excluded from GitHub language stats and diffs) and sets `merge=ours` strategy to prevent merge conflicts when multiple branches compile the same workflow |
| **`.github/agents/agentic-workflows.agent.md`** | **Dispatcher agent** for GitHub Copilot Chat. When you type `/agent` in Copilot Chat and select `agentic-workflows`, this file routes your request to specialized prompts (create, debug, upgrade, report, etc.) |
| **`.github/workflows/copilot-setup-steps.yml`** | A GitHub Actions workflow that installs the `gh aw` CLI (SHA-pinned to the exact version) so the **Copilot coding agent** can use it as an MCP server during automated runs |
| **`.vscode/mcp.json`** | Configures `gh aw mcp-server` as a **Model Context Protocol server** for VSCode. This lets Copilot Chat in VSCode invoke `gh aw` commands (status, compile, logs, audit, etc.) directly |
| **`.vscode/settings.json`** | Enables GitHub Copilot for markdown files, so you get AI assistance when editing `.md` workflow files |

**What this means for your repository**:
- ✅ Copilot Chat can now interact with `gh aw` commands via MCP
- ✅ Lock files (`.lock.yml`) won't cause merge conflicts
- ✅ You're ready to **create or add agentic workflows**

> **Note**: No agentic workflows exist yet — the next steps add actual workflow files.

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

### 🔍 What Just Happened?

**Why do you need a secret?** Agentic workflows run AI coding agents (Copilot, Claude, Codex, Gemini) inside GitHub Actions. These are **external AI services** that require authentication — the secret tells the service "this user is authorized to make AI requests."

**What each secret authenticates**:

| Secret | What It Does |
|--------|-------------|
| `COPILOT_GITHUB_TOKEN` | A fine-grained **GitHub PAT** with "Copilot Requests: Read" permission. Associates workflow runs with your Copilot subscription. Must be owned by your **user account** (not an org) |
| `ANTHROPIC_API_KEY` | An **Anthropic API key** that authenticates against Anthropic's Claude API. Billed to your Anthropic account |
| `OPENAI_API_KEY` | An **OpenAI API key** for Codex access. Billed to your OpenAI account |
| `GEMINI_API_KEY` | A **Google AI Studio API key** for Gemini access. Billed to your Google account |

**Where secrets are stored**: Secrets are encrypted and stored in **GitHub Actions secret storage** — they're never exposed in logs, workflow files, or to the AI agent itself. The agent runs in a read-only sandbox and authenticates through a separate, isolated process. (See [Deep Dive 07 — AWF Firewall & Sandbox](./deep-dive/07-awf-firewall-and-sandbox.md) for how the sandbox isolates the agent.)

**`gh aw secrets bootstrap`** is a verification tool that:
1. Scans all your workflow files to determine which engine secrets are needed
2. Checks which secrets already exist in the repository
3. Interactively prompts you to set any missing ones

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

### 🔍 What Just Happened?

**`add-wizard` vs `add`**: The `add-wizard` command is the **interactive, guided** version — it walks you through engine selection, secret setup, and optionally triggers the first run. The `add` command does the same thing **non-interactively** (better for CI/automation).

**Two files were created** — this is the fundamental pattern of agentic workflows:

| File | Role | Editable? |
|------|------|-----------|
| **`daily-repo-status.md`** | **Source file** — your natural language workflow with YAML frontmatter (config) + Markdown body (AI instructions) | ✅ Yes — this is what you edit |
| **`daily-repo-status.lock.yml`** | **Compiled output** — hardened GitHub Actions YAML that GitHub Actions actually executes | ⚠️ Auto-generated — don't edit directly |

**What "compilation" means**: The `gh aw compile` command (run automatically by the wizard) transforms your `.md` file into a production-ready GitHub Actions workflow with:
- SHA-pinned action references (no supply chain attacks)
- Security hardening (read-only permissions, network controls)
- Resolved imports and merged configurations
- Tool configurations and MCP server setups

**The Agentics Collection** ([githubnext/agentics](https://github.com/githubnext/agentics)) is GitHub's official repository of pre-built agentic workflows. It includes workflows for daily reports, CI diagnostics, issue triage, PR reviews, and more — ready to add with a single command.

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

### 🔍 What Just Happened?

**Under the hood**, `gh aw run` calls GitHub's `workflow_dispatch` API — the same mechanism as clicking "Run workflow" in the GitHub Actions UI. This only works with workflows that have a `workflow_dispatch` trigger (which compiled agentic workflows include by default).

**The execution pipeline** inside GitHub Actions follows this security-layered flow:

```
┌──────────────────┐
│ 1. PRE-ACTIVATION │  Checks user roles and permissions.
│    Job            │  Verifies the triggering user is authorized.
└────────┬─────────┘  Posts a 👀 reaction on the triggering item.
         ▼
┌──────────────────┐
│ 2. AGENT JOB      │  The AI coding agent runs HERE.
│    (Read-Only)    │  It reads your repo, interprets the markdown
│                   │  instructions, and generates structured output.
│                   │  🔒 No write permissions — cannot modify the repo.
└────────┬─────────┘
         ▼
┌──────────────────┐
│ 3. THREAT         │  AI-powered security scan of agent output.
│    DETECTION      │  Checks for: prompt injection, secret leaks,
│                   │  malicious code patches.
└────────┬─────────┘  ❌ Blocks if threats detected.
         ▼
┌──────────────────┐
│ 4. SAFE OUTPUT    │  Separate jobs with SCOPED write permissions.
│    JOBS           │  Creates issues, comments, PRs, labels —
│                   │  only the operations declared in safe-outputs.
└──────────────────┘
```

**Monitoring your run**:
- `gh aw status --ref main` — Check if the workflow is running/completed
- `gh aw logs daily-repo-status` — Download and analyze execution logs
- `gh aw audit <run-id>` — Deep-dive into a specific run with error analysis

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

### 🔍 What Just Happened?

**You just authored your first custom agentic workflow.** Let's break down the anatomy:

```
┌─────────────────────────────────────────────────────────┐
│  .github/workflows/issue-greeter.md                      │
│                                                          │
│  ┌─── FRONTMATTER (YAML between --- markers) ─────────┐ │
│  │ on:              ← WHEN to run (trigger)            │ │
│  │ permissions:     ← WHAT the agent can read          │ │
│  │ safe-outputs:    ← WHAT the workflow can write      │ │
│  │ tools:           ← WHAT tools the agent can use     │ │
│  │ engine:          ← WHICH AI to use                  │ │
│  │ network:         ← WHICH domains are accessible     │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  ┌─── MARKDOWN BODY ──────────────────────────────────┐ │
│  │ # Issue Greeter                                     │ │
│  │                                                     │ │
│  │ Natural language instructions that the AI agent     │ │
│  │ reads and follows at runtime.                       │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

**What `gh aw compile` produced** (the `.lock.yml` file):
- Full GitHub Actions YAML with multiple jobs (pre-activation, agent, [threat detection](./deep-dive/08-threat-detection-and-xpia.md), [safe outputs](./deep-dive/06-safe-outputs-catalog.md))
- All referenced GitHub Actions **SHA-pinned** to exact commit hashes
- Network firewall configuration with domain allowlists (see [Deep Dive 07 — AWF Firewall & Sandbox](./deep-dive/07-awf-firewall-and-sandbox.md))
- MCP server configurations for GitHub API tools (see [Deep Dive 05 — Tools & MCP](./deep-dive/05-tools-and-mcp.md))
- Permission separation: agent job is read-only, safe output jobs get scoped write permissions

**💡 Key insight — the "edit without recompile" property**:
- The **markdown body** (your AI instructions) is loaded **at runtime** from the `.md` file
- You can edit it directly on GitHub.com → changes take effect on the **next workflow run**
- **No recompilation needed** for instruction changes!

**⚠️ When you MUST recompile** (`gh aw compile`):
- Any change to the **frontmatter** (triggers, permissions, tools, safe-outputs, network, engine) — see [Deep Dive 03 — Frontmatter Reference](./deep-dive/03-frontmatter-reference.md)
- Adding or removing **imports** — see [Deep Dive 09 — Imports & Shared Components](./deep-dive/09-imports-and-shared-components.md)
- Changing **MCP server configurations**

> **Rule of thumb**: If you changed anything between the `---` markers, run `gh aw compile`.

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
