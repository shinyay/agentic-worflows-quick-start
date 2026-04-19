# Agentic Workflows Quick Start

> **Get AI-powered repository automation running in minutes — write workflows in Markdown, not YAML.**

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/shinyay/agentic-worflows-quick-start)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/f3ac65a05ed8c8ea70b653875ccac0c6dbc10ba1/LICENSE)

A hands-on quick start repository for **[GitHub Agentic Workflows](https://github.github.io/gh-aw/)** — the new automation paradigm where you describe what you want in plain English Markdown, and AI coding agents execute it in GitHub Actions with built-in security guardrails. Includes 2 working workflows, a step-by-step tutorial, and comprehensive CLI reference documentation.

> [!NOTE]
> **GitHub Agentic Workflows is in technical preview** (as of February 2026). Using agentic workflows requires careful attention to security considerations and human supervision. See the [Security Architecture](https://github.github.io/gh-aw/introduction/architecture/) for details.

---

## 🚀 Quick Start

**Prerequisites:**

- [x] **GitHub CLI** v2.0.0+ — `gh --version`
- [x] **GitHub repository** with Actions enabled
- [x] **AI Provider Account** — [GitHub Copilot](https://github.com/features/copilot) (recommended), [Anthropic Claude](https://console.anthropic.com/), [OpenAI](https://platform.openai.com/api-keys), or [Google Gemini](https://aistudio.google.com/api-keys)

### 1. Install the `gh aw` CLI extension

```bash
gh extension install github/gh-aw
gh aw version  # Verify: v0.61.0
```

### 2. Initialize your repository

```bash
gh aw init
```

### 3. Set up your AI engine secret and add a workflow

```bash
# Set your Copilot token (or ANTHROPIC_API_KEY / OPENAI_API_KEY / GEMINI_API_KEY)
gh aw secrets set COPILOT_GITHUB_TOKEN --value "your_token_here"

# Add a pre-built workflow with guided setup
gh aw add-wizard githubnext/agentics/daily-repo-status

# Or trigger manually
gh aw run daily-repo-status
```

If you see a new **GitHub Issue** with a daily status report — your setup works! 🎉

> [!TIP]
> See [`doc/getting-started-tutorial.md`](doc/getting-started-tutorial.md) for the full 10-step walkthrough with detailed "🔍 What Just Happened?" explanations after each step.

---

## 💡 Overview

### What are GitHub Agentic Workflows?

**Agentic workflows** are AI-powered automation that you write in **natural language Markdown** instead of complex YAML. An AI coding agent (Copilot, Claude, Codex, or Gemini) reads your instructions and executes them in GitHub Actions with built-in security guardrails.

### How it works

```
 ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
 │   AUTHOR      │────▶│   COMPILE    │────▶│     RUN      │────▶│   MONITOR    │
 │               │     │              │     │              │     │              │
 │ Write .md     │     │ gh aw compile│     │ gh aw run    │     │ gh aw logs   │
 │ (Markdown +   │     │              │     │              │     │ gh aw audit  │
 │  frontmatter) │     │ Produces     │     │ AI agent     │     │ gh aw health │
 │               │     │ .lock.yml    │     │ runs in      │     │ gh aw status │
 │               │     │ (hardened    │     │ GitHub       │     │              │
 │               │     │  Actions     │     │ Actions      │     │              │
 │               │     │  YAML)       │     │              │     │              │
 └──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

**Key security properties:**
- 🔒 Agent runs **read-only** by default — no direct write access to your repository
- 🛡️ Write operations go through **safe outputs** — separate permission-controlled jobs
- 🔥 **Agent Workflow Firewall (AWF)** — network egress controlled by domain allowlist
- 🔍 **Threat detection** — AI-powered security scan before any writes are applied
- 📌 All GitHub Actions **SHA-pinned** at compile time — no supply chain attacks

---

## ✨ What's in This Repository

### Agentic Workflows

| Workflow | Trigger | What It Does |
|----------|---------|-------------|
| **[daily-repo-status](/.github/workflows/daily-repo-status.md)** | Daily schedule + manual | Analyzes repo activity and creates a status report issue with insights, highlights, and recommendations |
| **[github-changelog-summary](/.github/workflows/github-changelog-summary.md)** | Weekly on Monday + manual | Fetches [GitHub Changelog](https://github.blog/changelog/), categorizes entries (Features, Changes, Deprecations, Security, API), and creates a summary issue |

### Documentation

| Document | Description |
|----------|-------------|
| **[Getting Started Tutorial](doc/getting-started-tutorial.md)** | 10-step walkthrough with "🔍 What Just Happened?" explanations for each step |
| **[gh aw CLI Reference](doc/gh-aw-cli-reference.md)** | Complete reference for all 29 CLI commands with flags, examples, and recipes |
| **[GitHub Agentic Workflows Research](doc/github-agentic-workflows-https-github-github-io-gh.md)** | Deep research report covering architecture, security model, safe outputs, tools, triggers, and more |
| **[📚 Deep Dive Documentation](doc/deep-dive/README.md)** ([🇯🇵](doc/deep-dive/README.ja.md)) | 14 in-depth, bilingual (EN+JP) topic guides covering architecture, engines, frontmatter, triggers, MCP, safe outputs, AWF, threat detection, imports, cookbook, debugging, glossary, FAQ, and a walkthrough of this repo |

### 📚 Deep Dive Documentation (NEW)

A comprehensive, layered, bilingual (EN + 日本語) documentation set in [`doc/deep-dive/`](doc/deep-dive/). 14 topics, each with TL;DR → Key Concepts → Deep Dive → Examples → Pitfalls/FAQ, and Mermaid diagrams.

**Learning paths:**

- 🟢 **Beginner**: [12 Walkthrough](doc/deep-dive/12-this-repos-workflows-walkthrough.md) → [13 Glossary](doc/deep-dive/13-glossary-and-concepts.md) → [04 Triggers](doc/deep-dive/04-triggers-and-scheduling.md) → [06 Safe Outputs](doc/deep-dive/06-safe-outputs-catalog.md) → [14 FAQ](doc/deep-dive/14-faq-and-troubleshooting.md)
- 🔵 **Developer**: [01 Architecture](doc/deep-dive/01-architecture-and-security.md) → [03 Frontmatter](doc/deep-dive/03-frontmatter-reference.md) → [05 Tools/MCP](doc/deep-dive/05-tools-and-mcp.md) → [10 Cookbook](doc/deep-dive/10-writing-workflows-cookbook.md) → [09 Imports](doc/deep-dive/09-imports-and-shared-components.md) → [02 Engines](doc/deep-dive/02-engines.md) → [11 Debugging](doc/deep-dive/11-debugging-and-observability.md)
- 🔴 **Security/Platform**: [01 Architecture](doc/deep-dive/01-architecture-and-security.md) → [07 AWF](doc/deep-dive/07-awf-firewall-and-sandbox.md) → [08 Threat Detection](doc/deep-dive/08-threat-detection-and-xpia.md) → [06 Safe Outputs](doc/deep-dive/06-safe-outputs-catalog.md) → [02 Engines](doc/deep-dive/02-engines.md)

→ Start with the **[Deep Dive index](doc/deep-dive/README.md)** ([日本語版](doc/deep-dive/README.ja.md))

---

## 🏗️ Project Structure

```
agentic-worflows-quick-start/
├── .github/
│   ├── agents/
│   │   └── agentic-workflows.agent.md    # Copilot Chat dispatcher agent
│   ├── aw/
│   │   └── actions-lock.json             # Pinned GitHub Actions versions
│   └── workflows/
│       ├── copilot-setup-steps.yml       # MCP server setup for Copilot Agent
│       ├── daily-repo-status.md          # Workflow source (natural language)
│       ├── daily-repo-status.lock.yml    # Compiled GitHub Actions YAML
│       ├── github-changelog-summary.md   # Workflow source (natural language)
│       └── github-changelog-summary.lock.yml  # Compiled GitHub Actions YAML
├── .vscode/
│   ├── mcp.json                          # MCP server config for VSCode
│   └── settings.json                     # Copilot markdown support
├── doc/
│   ├── getting-started-tutorial.md       # Step-by-step tutorial
│   ├── gh-aw-cli-reference.md            # Complete CLI reference
│   └── github-agentic-workflows-*.md     # Deep research report
├── .gitattributes                        # Lock file merge strategy
└── README.md
```

---

## 📖 Usage

### Quick Reference: "I want to…"

| I want to… | Do this |
|------------|---------|
| Set up the repo for agentic workflows | `gh aw init` |
| Add a pre-built workflow | `gh aw add-wizard githubnext/agentics/<name>` |
| Create my own workflow | `gh aw new my-workflow` → edit → `gh aw compile` |
| Compile all workflows | `gh aw compile` |
| Run a workflow manually | `gh aw run <workflow>` |
| Check workflow status | `gh aw status --ref main` |
| View execution logs | `gh aw logs <workflow>` |
| Debug a failed run | `gh aw audit <run-id>` |
| Check health metrics | `gh aw health` |
| Set up missing secrets | `gh aw secrets bootstrap` |
| Test without affecting production | `gh aw trial <workflow-spec>` |
| List all workflows | `gh aw list` |

### Creating Your Own Workflow

```bash
# 1. Create a template
gh aw new my-custom-workflow

# 2. Edit .github/workflows/my-custom-workflow.md
#    - Configure frontmatter (triggers, permissions, tools, safe-outputs)
#    - Write your AI instructions in the markdown body

# 3. Compile
gh aw compile my-custom-workflow

# 4. Commit, push, and run
git add .github/
git commit -m "Add my-custom-workflow"
git push
gh aw run my-custom-workflow
```

> [!TIP]
> The **markdown body** (AI instructions) can be edited directly on GitHub.com without recompiling. Only **frontmatter changes** (triggers, permissions, tools) require `gh aw compile`.

---

## 📚 References

| Resource | Link |
|----------|------|
| GitHub Agentic Workflows Docs | [github.github.io/gh-aw](https://github.github.io/gh-aw/) |
| GitHub Next — Agentic Workflows | [githubnext.com/projects/agentic-workflows](https://githubnext.com/projects/agentic-workflows/) |
| gh-aw Source Repository | [github/gh-aw](https://github.com/github/gh-aw) |
| Agentics Collection (Pre-built Workflows) | [githubnext/agentics](https://github.com/githubnext/agentics) |
| Agent Workflow Firewall | [github/gh-aw-firewall](https://github.com/github/gh-aw-firewall) |
| MCP Gateway | [github/gh-aw-mcpg](https://github.com/github/gh-aw-mcpg) |
| Security Architecture | [Architecture Docs](https://github.github.io/gh-aw/introduction/architecture/) |
| Community Feedback | [GitHub Discussions](https://github.com/orgs/community/discussions/186451) |

---

## 🤝 Contributing

Found a bug? Have an idea? [Open an issue](https://github.com/shinyay/agentic-worflows-quick-start/issues/new) — contributions and suggestions are welcome.

---

## ⭐ Support

If this project helps you, please consider:
- ⭐ Starring this repository
- 🐛 [Reporting issues](https://github.com/shinyay/agentic-worflows-quick-start/issues/new)
- 📢 Sharing with others

---

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/f3ac65a05ed8c8ea70b653875ccac0c6dbc10ba1/LICENSE)

## Author

- github: <https://github.com/shinyay>
- bluesky: <https://bsky.app/profile/yanashin.bsky.social>
- twitter: <https://twitter.com/yanashin18618>
- mastodon: <https://mastodon.social/@yanashin>
- linkedin: <https://www.linkedin.com/in/yanashin/>
