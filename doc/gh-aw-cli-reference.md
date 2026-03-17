# `gh aw` CLI Extension — Complete Reference Guide

> **Version**: v0.61.0 | **Source**: [github/gh-aw](https://github.com/github/gh-aw) | **Docs**: [github.github.io/gh-aw](https://github.github.io/gh-aw/)

---

## Table of Contents

- [1. Introduction \& Overview](#1-introduction--overview)
- [2. Installation \& Setup](#2-installation--setup)
- [3. Setup Commands](#3-setup-commands)
- [4. Development Commands](#4-development-commands)
- [5. Execution Commands](#5-execution-commands)
- [6. Analysis Commands](#6-analysis-commands)
- [7. Utility Commands](#7-utility-commands)
- [8. Command Quick-Reference Table](#8-command-quick-reference-table)
- [9. Common Workflows \& Recipes](#9-common-workflows--recipes)
- [10. Environment Variables \& Configuration](#10-environment-variables--configuration)
- [11. Troubleshooting](#11-troubleshooting)

---

## 1. Introduction & Overview

### What Is `gh aw`?

`gh aw` is a **GitHub CLI extension** that brings GitHub Agentic Workflows to your terminal. It is the primary tool for creating, compiling, running, and monitoring AI-powered automation workflows that are written in natural language Markdown and executed by AI coding agents in GitHub Actions.

The extension is developed by **GitHub Next** and **Microsoft Research** as part of the broader [GitHub Agentic Workflows](https://github.github.io/gh-aw/) project. It is distributed as a Go binary installed via the `gh` CLI extension system.

### How It Fits in the Workflow Lifecycle

```
 ┌─────────┐     ┌──────────┐     ┌─────────┐     ┌──────────┐
 │  AUTHOR  │────▶│ COMPILE  │────▶│   RUN   │────▶│ MONITOR  │
 │          │     │          │     │         │     │          │
 │ gh aw    │     │ gh aw    │     │ gh aw   │     │ gh aw    │
 │ init     │     │ compile  │     │ run     │     │ logs     │
 │ new      │     │ validate │     │ trial   │     │ audit    │
 │ add      │     │ fix      │     │ enable  │     │ health   │
 │ add-wizard│    │          │     │ disable │     │ status   │
 │ secrets  │     │          │     │         │     │ checks   │
 └─────────┘     └──────────┘     └─────────┘     └──────────┘
```

| Phase | Purpose | Key Commands |
|-------|---------|-------------|
| **Author** | Create and configure workflows | `init`, `new`, `add`, `add-wizard`, `secrets`, `update`, `upgrade` |
| **Compile** | Transform Markdown → GitHub Actions YAML | `compile`, `validate`, `fix` |
| **Run** | Execute and test workflows | `run`, `trial`, `enable`, `disable` |
| **Monitor** | Observe, debug, and maintain | `logs`, `audit`, `health`, `status`, `checks` |

### Command Categories (29 Commands Total)

The CLI organizes its 29 commands into five categories:

| Category | Commands | Count |
|----------|----------|-------|
| **Setup** | `init`, `new`, `add`, `add-wizard`, `remove`, `secrets`, `update`, `upgrade` | 8 |
| **Development** | `compile`, `validate`, `fix`, `list`, `status`, `domains`, `mcp` | 7 |
| **Execution** | `run`, `trial`, `enable`, `disable` | 4 |
| **Analysis** | `audit`, `logs`, `health`, `checks` | 4 |
| **Utilities** | `completion`, `hash-frontmatter`, `mcp-server`, `pr`, `project`, `version` | 6 |

### Global Flags

These flags are available on **every** `gh aw` command:

| Flag | Description |
|------|-------------|
| `-h`, `--help` | Show help for the command |
| `-v`, `--verbose` | Enable verbose output with debugging details |
| `--banner` | Display ASCII logo banner with purple GitHub color theme |
| `--version` | Show version (top-level only) |

### Version Check

```bash
gh aw version
# Output: gh aw version v0.61.0
```

---

## 2. Installation & Setup

### Standard Installation (via GitHub CLI)

```bash
gh extension install github/gh-aw
```

**Prerequisites**:
- GitHub CLI (`gh`) v2.0.0+ — [Install here](https://cli.github.com)
- Authenticated session — `gh auth login`
- OS: Linux, macOS, or Windows with WSL

### Version Pinning

Pin to a specific version for production environments or team consistency:

```bash
# Pin to a release tag
gh extension install github/gh-aw@v0.1.0

# Pin to a commit SHA
gh extension install github/gh-aw@abc123def456

# Check current version
gh aw version

# Upgrade a pinned version
gh extension remove gh-aw
gh extension install github/gh-aw@v0.2.0
```

### Standalone Installer (curl)

Use when extension installation fails (common in Codespaces, restricted networks, or with auth issues):

```bash
# Install latest
curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash

# Install pinned version
curl -sL https://raw.githubusercontent.com/github/gh-aw/main/install-gh-aw.sh | bash -s v0.1.0
```

Installs to `~/.local/share/gh/extensions/gh-aw/gh-aw`. Supports Linux, macOS, FreeBSD, Windows, and Android (Termux). Works behind corporate firewalls.

### GitHub Actions Setup Action

Install the CLI in GitHub Actions workflows:

```yaml
- name: Install gh-aw CLI
  uses: github/gh-aw/actions/setup-cli@main
  with:
    version: v0.37.18
```

Includes automatic checksum verification and platform detection.

### GitHub Enterprise Server Support

```bash
# Set hostname
export GH_HOST="github.enterprise.com"

# Authenticate
gh auth login --hostname github.enterprise.com

# Use with commands
gh aw logs workflow --repo github.enterprise.com/owner/repo
```

Commands like `add`, `add-wizard`, `init`, `update`, and `upgrade` auto-detect the enterprise host from the git remote.

### Upgrading

```bash
# Standard upgrade
gh extension upgrade gh-aw

# Or reinstall at a specific version
gh extension remove gh-aw
gh extension install github/gh-aw@v0.62.0
```

### Shell Completions

```bash
# Auto-detect and install
gh aw completion install

# Or install manually
gh aw completion bash > ~/.bash_completion.d/gh-aw    # Bash
gh aw completion zsh > "${fpath[1]}/_gh-aw"            # Zsh
gh aw completion fish > ~/.config/fish/completions/gh-aw.fish  # Fish
```

Completions provide tab-completion for command names, workflow names, engine names (`--engine`), and directory paths (`--dir`).

---

## 3. Setup Commands

### 3.1 `gh aw init` — Initialize Repository

Configures a repository for agentic workflows by setting up required files and configurations.

**What it creates**:
- `.gitattributes` — marks `.lock.yml` files as generated
- `.github/agents/agentic-workflows.agent.md` — dispatcher agent file
- `.github/workflows/copilot-setup-steps.yml` — MCP installation steps (unless `--no-mcp`)
- `.vscode/mcp.json` — MCP server configuration (unless `--no-mcp`)
- `.vscode/settings.json` — VSCode settings

**Interactive Mode** (default, no flags):
- Prompts to select an AI engine (Copilot, Claude, Codex, Gemini)
- Configures engine-specific settings
- Detects and configures secrets from your environment
- Sets up repository Actions secrets automatically

```bash
# Interactive mode (recommended for first-time setup)
gh aw init

# Skip MCP configuration
gh aw init --no-mcp

# Configure GitHub Codespaces (creates devcontainer.json)
gh aw init --codespaces

# Codespaces with additional repos from the same org
gh aw init --codespaces repo1,repo2

# Install shell completions
gh aw init --completions

# Create a pull request with changes
gh aw init --create-pull-request

# Verbose output
gh aw init -v
```

**Flags**:

| Flag | Description |
|------|-------------|
| `--codespaces [repos]` | Create `devcontainer.json` for Codespaces. Comma-separated repos for multi-repo |
| `--completions` | Install shell completion (auto-detects bash/zsh/fish/PowerShell) |
| `--create-pull-request` | Create a PR with the initialization changes |
| `--no-mcp` | Skip MCP server integration for GitHub Copilot Agent |

**After running**, you can:
- Use GitHub Copilot Chat: type `/agent` and select `agentic-workflows`
- Add workflows: `gh aw add <workflow-name>`
- Create new workflows: `gh aw new <workflow-name>`

---

### 3.2 `gh aw new` — Create Workflow Template

Creates a new agentic workflow file with commented examples and explanations.

```bash
# Interactive wizard
gh aw new

# Create named template
gh aw new my-workflow

# With .md extension (stripped automatically)
gh aw new my-workflow.md

# Overwrite existing
gh aw new my-workflow --force

# Pre-select engine
gh aw new my-workflow --engine claude

# Interactive wizard mode explicitly
gh aw new --interactive
```

**Template contents**: The generated file includes commented examples of all trigger types, permissions, AI engine settings, tools configuration, and all frontmatter options.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--engine` | `-e` | Override AI engine (`copilot`, `claude`, `codex`, `custom`) |
| `--force` | `-f` | Overwrite existing files without confirmation |
| `--interactive` | `-i` | Launch interactive workflow creation wizard |

---

### 3.3 `gh aw add` — Add Workflows (Non-Interactive)

Adds one or more workflows from repositories directly without interactive prompts.

**Workflow Specification Formats**:

| Format | Example |
|--------|---------|
| Three parts (`owner/repo/name`) | `githubnext/agentics/daily-repo-status` |
| With version | `githubnext/agentics/ci-doctor@v1.0.0` |
| Full path with `.md` | `githubnext/agentics/workflows/ci-doctor.md@main` |
| GitHub URL | `https://github.com/githubnext/agentics/blob/main/workflows/ci-doctor.md` |
| Local file | `./my-workflow.md` |
| Local wildcard | `./*.md` or `./dir/*.md` |

```bash
# Add single workflow
gh aw add githubnext/agentics/daily-repo-status

# Add with version pin
gh aw add githubnext/agentics/ci-doctor@v1.0.0

# Add from URL
gh aw add https://github.com/githubnext/agentics/blob/main/workflows/ci-doctor.md

# Add local file
gh aw add ./my-workflow.md

# Add all local markdown files
gh aw add ./*.md

# Add to subdirectory
gh aw add githubnext/agentics/ci-doctor --dir shared

# Create PR with changes
gh aw add githubnext/agentics/ci-doctor --create-pull-request --force

# Override engine
gh aw add githubnext/agentics/ci-doctor --engine claude

# Custom name for the workflow file
gh aw add githubnext/agentics/ci-doctor --name my-ci-doctor

# Set a stop-after deadline
gh aw add githubnext/agentics/ci-doctor --stop-after "+48h"
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--append` | | Append extra content to the workflow on installation |
| `--create-pull-request` | | Create a PR with workflow changes |
| `--dir` | `-d` | Subdirectory under `.github/workflows/` |
| `--disable-security-scanner` | | Disable security scanning of workflow markdown |
| `--engine` | `-e` | Override AI engine |
| `--force` | `-f` | Overwrite existing files without confirmation |
| `--name` | `-n` | Custom name for the added workflow |
| `--no-gitattributes` | | Skip updating `.gitattributes` |
| `--no-stop-after` | | Remove any `stop-after` field from the workflow |
| `--repo` | `-r` | Source repository (`owner/repo` format) |
| `--stop-after` | | Override `stop-after` value (e.g., `+48h`, `2025-12-31`) |

---

### 3.4 `gh aw add-wizard` — Add Workflows (Interactive)

Interactively adds workflows with guided setup — the recommended way for first-time additions.

**The wizard walks through**:
1. Selecting an AI engine (Copilot, Claude, Codex)
2. Configuring API keys and secrets
3. Creating a pull request with the workflow
4. Optionally running the workflow immediately

```bash
# Guided setup
gh aw add-wizard githubnext/agentics/daily-repo-status

# With version pin
gh aw add-wizard githubnext/agentics/ci-doctor@v1.0.0

# Local workflow
gh aw add-wizard ./my-workflow.md

# Pre-select engine
gh aw add-wizard githubnext/agentics/ci-doctor --engine copilot

# Skip secret prompt (secret already at org/repo level)
gh aw add-wizard githubnext/agentics/ci-doctor --skip-secret
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--dir` | `-d` | Subdirectory under `.github/workflows/` |
| `--engine` | `-e` | Override AI engine |
| `--no-gitattributes` | | Skip updating `.gitattributes` |
| `--no-stop-after` | | Remove `stop-after` field |
| `--skip-secret` | | Skip API secret prompt |
| `--stop-after` | | Override `stop-after` value |

> **Note**: Requires an interactive terminal. Use `gh aw add` for CI/automation environments.

---

### 3.5 `gh aw remove` — Remove Workflows

Removes workflow files matching a pattern (both `.md` and `.lock.yml`).

```bash
# Remove specific workflow
gh aw remove my-workflow

# Remove all workflows starting with 'test-'
gh aw remove test-

# Remove but keep orphaned include files
gh aw remove old- --keep-orphans
```

**Flags**:

| Flag | Description |
|------|-------------|
| `--keep-orphans` | Skip removal of orphaned include files no longer referenced by any workflow |

By default, orphaned include files that are no longer referenced are also cleaned up.

---

### 3.6 `gh aw secrets` — Secret Management

Manage GitHub Actions secrets required by agentic workflows.

#### `gh aw secrets set` — Create/Update a Secret

```bash
# From stdin (interactive)
gh aw secrets set MY_SECRET

# From flag
gh aw secrets set MY_SECRET --value "secret123"

# From environment variable
export MY_TOKEN="secret123"
gh aw secrets set MY_SECRET --value-from-env MY_TOKEN

# Specify target repository
gh aw secrets set MY_SECRET --repo myorg/myrepo

# Custom API URL (for GHES)
gh aw secrets set MY_SECRET --value "xyz" --api-url https://github.enterprise.com/api/v3
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--value` | | Secret value (if empty, read from stdin) |
| `--value-from-env` | | Environment variable to read secret value from |
| `--repo` | `-r` | Target repository (`owner/repo`) |
| `--api-url` | | GitHub API base URL (default: `https://api.github.com`) |

#### `gh aw secrets bootstrap` — Analyze and Set Up Missing Secrets

Scans all workflows to determine required secrets, checks which exist, and prompts for missing ones.

```bash
# Check and set up all required secrets
gh aw secrets bootstrap

# Check for a specific engine
gh aw secrets bootstrap --engine copilot

# Display-only mode (no prompts)
gh aw secrets bootstrap --non-interactive

# Target different repository
gh aw secrets bootstrap --repo myorg/myrepo
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--engine` | `-e` | Check tokens for specific engine (`copilot`, `claude`, `codex`) |
| `--non-interactive` | | Check secrets without prompting (display-only) |
| `--repo` | `-r` | Target repository |

---

### 3.7 `gh aw update` — Update Workflows from Upstream

Fetches the latest version from source repositories and merges changes.

**Update behavior by ref type**:
- **Tag ref**: Updates to the latest release within the same major version
- **Branch ref**: Fetches the latest commit from that branch
- **Commit SHA ref**: Fetches the latest commit from the default branch

```bash
# Update all workflows with a 'source' field
gh aw update

# Update specific workflow
gh aw update repo-assist

# Override local changes with upstream (no merge)
gh aw update --no-merge

# Allow major version updates
gh aw update repo-assist --major

# Force update even if no changes detected
gh aw update --force

# Skip action version bumps
gh aw update --disable-release-bump

# Skip recompilation
gh aw update --no-compile

# Custom workflow directory
gh aw update --dir custom/workflows

# Create a pull request
gh aw update --create-pull-request

# Override engine
gh aw update --engine claude
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--create-pull-request` | | Create a PR with update changes |
| `--dir` | `-d` | Workflow directory (default: `.github/workflows`) |
| `--disable-release-bump` | | Skip auto major version bumps for all actions |
| `--engine` | `-e` | Override AI engine |
| `--force` | `-f` | Force update even if no changes detected |
| `--major` | | Allow major version updates for tagged releases |
| `--no-compile` | | Skip recompiling workflows |
| `--no-merge` | | Override local changes with upstream instead of merging |
| `--no-stop-after` | | Remove `stop-after` field |
| `--stop-after` | | Override `stop-after` value |

> **Note**: By default, `update` also force-updates all GitHub Actions referenced in workflows to their latest major version. Use `--disable-release-bump` to restrict this.

---

### 3.8 `gh aw upgrade` — Full Repository Upgrade

Performs a comprehensive upgrade of the repository's agentic workflows setup.

**What it does (in order)**:
1. Updates the dispatcher agent file to the latest template
2. Applies automatic codemods to fix deprecated fields (like `fix --write`)
3. Updates GitHub Actions versions in `.github/aw/actions-lock.json`
4. Compiles all workflows to generate lock files

```bash
# Upgrade everything
gh aw upgrade

# Skip codemods, actions, and compilation (agent files only)
gh aw upgrade --no-fix

# Skip updating GitHub Actions versions
gh aw upgrade --no-actions

# Skip recompilation
gh aw upgrade --no-compile

# Create a pull request
gh aw upgrade --create-pull-request

# Custom workflow directory
gh aw upgrade --dir custom/workflows

# Dependency health audit (without upgrading)
gh aw upgrade --audit

# Audit with JSON output
gh aw upgrade --audit --json
```

**Dependency Health Audit** (`--audit`):

Checks dependency health without performing upgrades:
- Outdated Go dependencies with available updates
- Security advisories from GitHub Security Advisory API
- Dependency maturity analysis (v0.x vs stable versions)
- Comprehensive dependency health report

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--audit` | | Check dependency health without performing upgrades |
| `--create-pull-request` | | Create a PR with upgrade changes |
| `--dir` | `-d` | Workflow directory (default: `.github/workflows`) |
| `--json` | `-j` | Output results in JSON format |
| `--no-actions` | | Skip updating GitHub Actions versions |
| `--no-compile` | | Skip recompiling workflows |
| `--no-fix` | | Skip codemods, action updates, and compilation (only update agent files) |

---

## 4. Development Commands

### 4.1 `gh aw compile` — Compile Workflows (Core Command)

The **most important command** — transforms Markdown workflow files (`.md`) into hardened GitHub Actions YAML (`.lock.yml`).

**Compilation pipeline**:
1. Schema validation of frontmatter
2. Import resolution (BFS traversal of local/remote imports)
3. Expression safety checking
4. Action SHA pinning
5. Security scanning (optional: actionlint, zizmor, poutine)
6. Tool configuration generation
7. Network hardening
8. Lock file generation

```bash
# Compile all workflows in .github/workflows/
gh aw compile

# Compile specific workflow
gh aw compile ci-doctor

# Compile multiple workflows
gh aw compile ci-doctor daily-plan

# Compile by file path
gh aw compile workflow.md

# Watch mode — auto-recompile on changes
gh aw compile --watch ci-doctor

# Validation mode — check without generating lock files
gh aw compile --validate --no-emit

# Strict mode — enforce security best practices
gh aw compile --strict

# Run security scanners
gh aw compile --zizmor                    # Zizmor scanner
gh aw compile --actionlint               # Actionlint
gh aw compile --poutine                  # Poutine scanner
gh aw compile --strict --zizmor          # Strict + security scan (fails on findings)

# Generate Dependabot manifests
gh aw compile --dependabot
gh aw compile --dependabot --force       # Overwrite existing dependabot.yml

# Apply codemods before compiling
gh aw compile --fix

# Remove orphaned lock files
gh aw compile --purge

# Custom directory
gh aw compile --dir custom/workflows

# Trial mode compilation
gh aw compile --trial --logical-repo owner/repo

# JSON output with statistics
gh aw compile --json --stats

# Stop at first error
gh aw compile --fail-fast

# Force refresh all action SHA pins
gh aw compile --force-refresh-action-pins

# Override action mode
gh aw compile --action-mode release
gh aw compile --action-tag v1.0.0
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--action-mode` | | Action script inlining mode (`inline`, `dev`, `release`) |
| `--action-tag` | | Override action SHA or tag for actions/setup |
| `--actionlint` | | Run actionlint linter on generated `.lock.yml` files |
| `--dependabot` | | Generate dependency manifests and Dependabot config |
| `--dir` | `-d` | Workflow directory (default: `.github/workflows`) |
| `--engine` | `-e` | Override AI engine |
| `--fail-fast` | | Stop at first validation error |
| `--fix` | | Apply codemod fixes before compiling |
| `--force` | | Force overwrite existing dependency files |
| `--force-refresh-action-pins` | | Force refresh all action SHAs from GitHub API |
| `--json` | `-j` | Output results in JSON format |
| `--logical-repo` | | Repository to simulate against (trial mode) |
| `--no-check-update` | | Skip checking for gh-aw updates |
| `--no-emit` | | Validate without generating lock files |
| `--poutine` | | Run poutine security scanner |
| `--purge` | | Delete orphaned `.lock.yml` files |
| `--refresh-stop-time` | | Force regeneration of `stop-after` times |
| `--stats` | | Display statistics table (jobs, steps, scripts, shells) |
| `--strict` | | Enforce strict mode (action pinning, network, no write perms) |
| `--trial` | | Enable trial mode compilation |
| `--validate` | | Enable schema, container, and action SHA validation |
| `--watch` | `-w` | Watch and auto-recompile on changes |
| `--zizmor` | | Run zizmor security scanner |

> **Tip**: Workflows default to strict mode unless frontmatter explicitly sets `strict: false`. The `--strict` CLI flag overrides all frontmatter settings.

---

### 4.2 `gh aw validate` — Validate Without Generating Output

Equivalent to running the compiler with all linters enabled but no output:

```
gh aw compile --validate --no-emit --zizmor --actionlint --poutine
```

```bash
# Validate all workflows
gh aw validate

# Validate specific workflow
gh aw validate ci-doctor

# Validate multiple
gh aw validate ci-doctor daily

# Strict mode
gh aw validate --strict

# JSON output
gh aw validate --json

# Stop at first error
gh aw validate --fail-fast

# Custom directory
gh aw validate --dir custom/workflows

# Override engine
gh aw validate --engine copilot

# Show compilation statistics
gh aw validate --stats
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--dir` | `-d` | Workflow directory |
| `--engine` | `-e` | Override AI engine |
| `--fail-fast` | | Stop at first error |
| `--json` | `-j` | JSON output |
| `--no-check-update` | | Skip checking for gh-aw updates |
| `--stats` | | Display statistics table |
| `--strict` | | Enforce strict mode |

> **Note**: All linters (`zizmor`, `actionlint`, `poutine`), `--validate`, and `--no-emit` are always-on defaults and cannot be disabled in validate.

---

### 4.3 `gh aw fix` — Auto-Fix Deprecated Fields

Applies automatic codemod-style fixes to workflow files to migrate deprecated fields.

**Default mode**: Dry-run (no files modified). Use `--write` to apply changes.

```bash
# Check all workflows (dry-run)
gh aw fix

# Apply fixes to all workflows
gh aw fix --write

# Check specific workflow
gh aw fix my-workflow

# Fix specific workflow
gh aw fix my-workflow --write

# Custom directory
gh aw fix --dir custom/workflows

# List all available codemods
gh aw fix --list-codemods
```

**What `--write` additionally does**:
- Writes updated files back to disk
- Deletes deprecated `.github/aw/schemas/agentic-workflow.json`
- Deletes old template files from `pkg/cli/templates/`
- Deletes old workflow-specific `.agent.md` files from `.github/agents/`

**Notable codemods**:
- `expires-integer-to-string` — Converts bare integer `expires: 7` to string `expires: 7d`

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--dir` | `-d` | Workflow directory |
| `--list-codemods` | | List all available codemods and exit |
| `--write` | | Write changes to files (default is dry-run) |

---

### 4.4 `gh aw list` — Quick Workflow Listing

Fast enumeration of workflows **without GitHub API queries**. Shows name, engine, and compilation status.

```bash
# List all workflows
gh aw list

# Filter by pattern (case-insensitive)
gh aw list ci-

# List from remote repository
gh aw list --repo github/gh-aw

# List from remote with pattern
gh aw list --repo github/gh-aw ci-

# List from custom path in remote repo
gh aw list --repo org/repo --path workflows

# Custom local directory
gh aw list --dir custom/workflows

# JSON output
gh aw list --json

# Filter by label
gh aw list --label automation
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--dir` | `-d` | Workflow directory |
| `--json` | `-j` | JSON output |
| `--label` | | Filter by label |
| `--path` | | Path in the repository (default: `.github/workflows`) |
| `--repo` | `-r` | Target repository (`[HOST/]owner/repo`) |

> **Difference from `status`**: `list` is fast (no API calls), while `status` queries GitHub for enabled/disabled state and run info.

---

### 4.5 `gh aw status` — Detailed Workflow Status

Shows workflow status including enabled/disabled state, schedules, labels, and optionally latest run information.

```bash
# Show all workflow status
gh aw status

# Filter by name pattern
gh aw status ci-

# Include latest run status for a branch
gh aw status --ref main

# Filter by label
gh aw status --label automation

# Check different repository
gh aw status --repo owner/other-repo

# JSON output
gh aw status --json
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--json` | `-j` | JSON output |
| `--label` | | Filter by label |
| `--ref` | | Filter runs by branch or tag (e.g., `main`, `v1.0.0`) |
| `--repo` | `-r` | Target repository |

---

### 4.6 `gh aw domains` — Network Domain Inspection

Lists network domains configured in workflows, including expanded ecosystem identifiers.

```bash
# Summary of all workflows with domain counts
gh aw domains

# Detailed domains for a specific workflow
gh aw domains weekly-research

# JSON output (summary)
gh aw domains --json

# JSON output (specific workflow)
gh aw domains weekly-research --json
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--json` | `-j` | JSON output |

When showing domains for a specific workflow, it expands ecosystem identifiers (e.g., `python` → list of PyPI/pip domains) and includes engine defaults.

---

### 4.7 `gh aw mcp` — MCP Server Management

Manage Model Context Protocol servers configured in workflows. Has four subcommands.

#### `gh aw mcp list` — List MCP Servers

```bash
# List all workflows with MCP servers
gh aw mcp list

# List MCP servers in a specific workflow
gh aw mcp list weekly-research

# Verbose (includes type and command/URL)
gh aw mcp list weekly-research -v
```

Displays: Server Name, Status (✓ Ready / ⚠ Incomplete), Tools Count, Network Access.

#### `gh aw mcp list-tools` — List Available Tools

```bash
# Find workflows with a specific MCP server
gh aw mcp list-tools github

# List tools for a server in a specific workflow
gh aw mcp list-tools github weekly-research

# Verbose with descriptions
gh aw mcp list-tools playwright test-workflow -v
```

#### `gh aw mcp inspect` — Deep Inspection

Starts each MCP server, queries capabilities, and displays results.

```bash
# List workflows with MCP servers (no specific workflow)
gh aw mcp inspect

# Inspect all MCP servers in a workflow
gh aw mcp inspect weekly-research

# Inspect only one server
gh aw mcp inspect daily-news --server tavily

# Show details for a specific tool
gh aw mcp inspect weekly-research --server github --tool create_issue

# Launch the official MCP Inspector tool
gh aw mcp inspect weekly-research --inspector

# Check GitHub Actions secrets
gh aw mcp inspect weekly-research --check-secrets

# Verbose output
gh aw mcp inspect weekly-research -v
```

**Flags**:

| Flag | Description |
|------|-------------|
| `--check-secrets` | Check repository secrets for missing secrets |
| `--inspector` | Launch `@modelcontextprotocol/inspector` tool |
| `--server` | Filter to inspect only the specified MCP server |
| `--tool` | Show details about a specific tool (requires `--server`) |

#### `gh aw mcp add` — Add MCP Tool from Registry

```bash
# List available MCP servers from registry
gh aw mcp add

# Add Notion MCP server to a workflow
gh aw mcp add weekly-research makenotion/notion-mcp-server

# Prefer stdio transport
gh aw mcp add weekly-research makenotion/notion-mcp-server --transport stdio

# Use custom registry
gh aw mcp add weekly-research server-name --registry https://custom.registry.com/v1

# Custom tool ID
gh aw mcp add weekly-research server-name --tool-id my-notion
```

**Flags**:

| Flag | Description |
|------|-------------|
| `--registry` | MCP registry URL (default: `https://api.mcp.github.com/v0.1`) |
| `--tool-id` | Custom tool ID in the workflow |
| `--transport` | Preferred transport (`stdio`, `http`, `docker`) |

---

## 5. Execution Commands

### 5.1 `gh aw run` — Trigger Workflows

Triggers workflows immediately via the `workflow_dispatch` mechanism.

**Interactive mode** (no arguments): Shows a list of eligible workflows, displays required/optional inputs, collects input with validation.

```bash
# Interactive mode
gh aw run

# Run specific workflow
gh aw run daily-perf-improver

# Run on specific branch
gh aw run daily-perf-improver --ref main

# Run multiple times (1 initial + 3 repeats = 4 total)
gh aw run daily-perf-improver --repeat 3

# Pass workflow inputs
gh aw run daily-perf-improver -F name=value -F env=prod

# Auto-commit, push, then dispatch
gh aw run daily-perf-improver --push

# Push to specific branch
gh aw run daily-perf-improver --push --ref main

# Enable disabled workflow, run, then restore state
gh aw run daily-perf-improver --enable-if-needed

# Auto-merge any PRs created
gh aw run daily-perf-improver --auto-merge-prs

# Validate without running
gh aw run daily-perf-improver --dry-run

# JSON output
gh aw run daily-perf-improver --json

# Override engine
gh aw run daily-perf-improver --engine claude

# Target different repository
gh aw run daily-perf-improver --repo owner/repo
```

**The `--push` flag**: Automatically recompiles outdated `.lock.yml` files, stages all transitive imports, commits, and pushes before dispatching. Without `--push`, warnings appear for missing/outdated lock files.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--auto-merge-prs` | | Auto-merge PRs created during execution |
| `--dry-run` | | Validate without actually running |
| `--enable-if-needed` | | Enable if disabled, run, then restore state |
| `--engine` | `-e` | Override AI engine |
| `--json` | `-j` | JSON output |
| `--push` | | Commit, push, then dispatch |
| `--raw-field` | `-F` | Add input parameter (`key=value`, repeatable) |
| `--ref` | | Branch or tag to run on (default: current branch) |
| `--repeat` | | Additional runs after initial (e.g., `--repeat 3` = 4 total) |
| `--repo` | `-r` | Target repository |

---

### 5.2 `gh aw trial` — Test in Temporary Repositories

Tests workflows by creating a temporary private repository and running them in "trial mode" with safe outputs captured but not applied to a real repository.

**Repository modes**:

| Mode | Flag | Behavior |
|------|------|----------|
| **Default** | (none) | Creates temp repo, simulates against current repo context |
| **Logical repo** | `--logical-repo REPO` | Simulates against specified repo (runs in temp repo) |
| **Host repo** | `--host-repo REPO` or `--repo REPO` | Runs directly in the specified repo |
| **Clone repo** | `--clone-repo REPO` | Clones repo contents into trial repo before execution |

```bash
# Test remote workflow (creates temp private repo)
gh aw trial githubnext/agentics/weekly-research

# Simulate running against a specific repo
gh aw trial githubnext/agentics/my-workflow --logical-repo myorg/myrepo

# Run directly in a specified repository
gh aw trial githubnext/agentics/my-workflow --repo myorg/myrepo

# Clone repo contents into trial repo
gh aw trial githubnext/agentics/my-workflow --clone-repo myorg/myrepo

# Multiple workflows (comparison)
gh aw trial githubnext/agentics/daily-plan githubnext/agentics/weekly-research

# Run 4 times total
gh aw trial githubnext/agentics/my-workflow --repeat 3

# Delete temp repo after completion
gh aw trial githubnext/agentics/my-workflow --delete-host-repo-after

# Custom host repo name
gh aw trial githubnext/agentics/my-workflow --host-repo my-trial

# Use current repo as host
gh aw trial githubnext/agentics/my-workflow --host-repo .

# Dry run (preview without changes)
gh aw trial githubnext/agentics/my-workflow --dry-run

# Auto-merge PRs
gh aw trial githubnext/agentics/my-workflow --auto-merge-prs

# Override engine and timeout
gh aw trial githubnext/agentics/my-workflow --engine claude --timeout 60

# Test local workflow with cloned repo
gh aw trial ./local-workflow.md --clone-repo upstream/repo --repeat 2

# Provide trigger context (e.g., for issue-triggered workflows)
gh aw trial githubnext/agentics/issue-triage --trigger-context https://github.com/org/repo/issues/42
```

Results are saved to `trials/` directory locally and in the host repository.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--append` | | Append extra content to workflow on installation |
| `--auto-merge-prs` | | Auto-merge PRs created during trial |
| `--clone-repo` | | Clone specified repo contents into host |
| `--delete-host-repo-after` | | Delete host repo after completion |
| `--disable-security-scanner` | | Disable security scanning |
| `--dry-run` | | Show what would happen without changes |
| `--engine` | `-e` | Override AI engine |
| `--force-delete-host-repo-before` | | Force delete host repo before creation |
| `--host-repo` | | Custom host repo slug (default: `<user>/gh-aw-trial`) |
| `--logical-repo` | `-s` | Repo to simulate execution against |
| `--repeat` | | Additional runs after initial |
| `--repo` | | Alias for `--host-repo` |
| `--timeout` | | Execution timeout in minutes (default: 30) |
| `--trigger-context` | | Trigger context URL (e.g., issue URL) |
| `--yes` | `-y` | Skip confirmation prompts |

---

### 5.3 `gh aw enable` — Enable Workflows

```bash
# Enable all workflows
gh aw enable

# Enable specific workflow
gh aw enable ci-doctor

# Enable multiple
gh aw enable ci-doctor daily

# Enable in specific repository
gh aw enable ci-doctor --repo owner/repo
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--repo` | `-r` | Target repository |

---

### 5.4 `gh aw disable` — Disable Workflows

Disables workflows and **cancels any in-progress runs** before disabling.

```bash
# Disable all workflows
gh aw disable

# Disable specific workflow
gh aw disable ci-doctor

# Disable multiple
gh aw disable ci-doctor daily

# Disable in specific repository
gh aw disable ci-doctor --repo owner/repo
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--repo` | `-r` | Target repository |

---

## 6. Analysis Commands

### 6.1 `gh aw audit` — Deep-Dive Run Analysis

Audits a single workflow run by downloading artifacts/logs, detecting errors, analyzing MCP tool usage, and generating a detailed Markdown report.

**Accepted input formats**:
- Numeric run ID: `1234567890`
- Workflow run URL: `https://github.com/owner/repo/actions/runs/123`
- Job URL: `https://github.com/owner/repo/actions/runs/123/job/456`
- Job URL with step: `https://github.com/owner/repo/actions/runs/123/job/456#step:7:1`
- Workflow run URL (short): `https://github.com/owner/repo/runs/123`
- GitHub Enterprise URLs: `https://github.example.com/owner/repo/actions/runs/123`

**What it analyzes**:
- Errors and warnings in logs
- MCP tool usage statistics
- Missing tool reports
- Firewall activity
- Noops and artifacts

**Job URL behavior**:
- With step number (`#step:7:1`): Extracts that specific step's output
- Without step number: Finds and extracts the first failing step's output

```bash
# Audit by run ID
gh aw audit 1234567890

# Audit from run URL
gh aw audit https://github.com/owner/repo/actions/runs/1234567890

# Audit job and extract first failing step
gh aw audit https://github.com/owner/repo/actions/runs/123/job/456

# Extract specific step output
gh aw audit https://github.com/owner/repo/actions/runs/123/job/456#step:7:1

# Parse logs into Markdown reports
gh aw audit 1234567890 --parse

# Custom output directory
gh aw audit 1234567890 -o ./audit-reports

# Specify repository (required for bare run ID)
gh aw audit 1234567890 --repo owner/repo

# JSON output
gh aw audit 1234567890 --json

# Verbose
gh aw audit 1234567890 -v
```

Output is saved to `logs/run-{id}/` by default.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--json` | `-j` | JSON output |
| `--output` | `-o` | Output directory (default: `.github/aw/logs`) |
| `--parse` | | Parse agent/firewall logs into Markdown (log.md, firewall.md) |
| `--repo` | `-r` | Target repository (required for bare run IDs) |

---

### 6.2 `gh aw logs` — Log Download and Analysis

Downloads workflow run logs and artifacts with aggregated metrics including duration, token usage, and cost.

**Downloaded artifacts**:

| File | Description |
|------|-------------|
| `aw_info.json` | Engine configuration and workflow metadata |
| `safe_output.jsonl` | Agent's final output content |
| `agent_output/` | Agent logs directory |
| `agent-stdio.log` | Agent standard output/error logs |
| `aw-{branch}.patch` | Git patch per branch (one file per PR/push) |
| `workflow-logs/` | GitHub Actions workflow run logs |
| `summary.json` | Complete metrics for all downloaded runs |

**Workflow name matching**: Accepts workflow IDs, display names, or case-insensitive variants.

```bash
# Download logs for all workflows
gh aw logs

# Download logs for specific workflow
gh aw logs weekly-research

# Limit to last 10 runs
gh aw logs -c 10

# Date filtering
gh aw logs --start-date 2024-01-01
gh aw logs --end-date 2024-01-31
gh aw logs --start-date -1w              # Last week
gh aw logs --start-date -1w -c 5         # Last week, max 5
gh aw logs --end-date -1d                # Until yesterday
gh aw logs --start-date -1mo             # Last month

# Content filtering
gh aw logs --engine claude               # By engine
gh aw logs --firewall                    # With firewall enabled
gh aw logs --no-firewall                 # Without firewall
gh aw logs --safe-output create-issue    # With specific safe output type
gh aw logs --safe-output missing-tool    # With missing tool reports
gh aw logs --ref main                    # By branch

# Run ID range filtering
gh aw logs --after-run-id 1000
gh aw logs --before-run-id 2000
gh aw logs --after-run-id 1000 --before-run-id 2000

# Output options
gh aw logs -o ./my-logs                  # Custom output directory
gh aw logs --tool-graph                  # Generate Mermaid tool sequence graph
gh aw logs --parse                       # Parse logs into Markdown reports
gh aw logs --json                        # JSON metrics output
gh aw logs --parse --json                # Both Markdown and JSON

# Exclude staged runs
gh aw logs --no-staged

# Cross-repository
gh aw logs weekly-research --repo owner/repo
```

> **Performance**: Results are cached for 10-100x speedup on subsequent runs.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--after-run-id` | | Filter runs after this ID (exclusive) |
| `--before-run-id` | | Filter runs before this ID (exclusive) |
| `--count` | `-c` | Max workflow runs to return (default: 10) |
| `--end-date` | | Runs before date (`YYYY-MM-DD` or `-1d`, `-1w`, `-1mo`) |
| `--engine` | `-e` | Filter by engine |
| `--firewall` | | Only runs with firewall enabled |
| `--json` | `-j` | JSON output |
| `--no-firewall` | | Only runs without firewall |
| `--no-staged` | | Exclude staged workflow runs |
| `--output` | `-o` | Output directory (default: `.github/aw/logs`) |
| `--parse` | | Parse logs into Markdown (log.md, firewall.md) |
| `--ref` | | Filter by branch or tag |
| `--repo` | `-r` | Target repository |
| `--safe-output` | | Filter by safe output type |
| `--start-date` | | Runs after date |
| `--summary-file` | | Summary JSON path (empty string to disable) |
| `--timeout` | | Download timeout in seconds (0 = no timeout) |
| `--tool-graph` | | Generate Mermaid tool sequence graph |

---

### 6.3 `gh aw health` — Health Metrics Dashboard

Displays workflow health metrics, success rates, and execution trends.

**Metrics shown**:
- Success/failure rates over time period
- Trend indicators: ↑ improving, → stable, ↓ degrading
- Average execution duration
- Token usage and costs
- Alerts when success rate drops below threshold

```bash
# Summary of all workflows (last 7 days)
gh aw health

# Detailed metrics for specific workflow
gh aw health issue-monster

# Last 30 days
gh aw health --days 30

# Last 90 days for specific workflow
gh aw health issue-monster --days 90

# Alert if below 90% success rate
gh aw health --threshold 90

# JSON output
gh aw health --json

# Different repository
gh aw health --repo owner/repo
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--days` | | Analysis period: 7, 30, or 90 (default: 7) |
| `--json` | `-j` | JSON output |
| `--repo` | `-r` | Target repository |
| `--threshold` | | Success rate warning threshold in % (default: 80) |

---

### 6.4 `gh aw checks` — PR CI Check Classification

Classifies CI check state for a pull request into normalized states.

**Normalized states**:

| State | Meaning |
|-------|---------|
| `success` | All checks passed |
| `failed` | One or more checks failed |
| `pending` | Checks still running or queued |
| `no_checks` | No checks configured or triggered |
| `policy_blocked` | Policy or account gates blocking the PR |

**JSON output includes two state fields**:
- `state` — Aggregate across all check runs and commit statuses
- `required_state` — Ignores optional third-party statuses (e.g., Vercel, Netlify); use this as the authoritative CI verdict

```bash
# Classify checks for PR #42
gh aw checks 42

# Specify repository
gh aw checks 42 --repo owner/repo

# JSON output
gh aw checks 42 --json
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--json` | `-j` | JSON output |
| `--repo` | `-r` | Target repository |

---

## 7. Utility Commands

### 7.1 `gh aw completion` — Shell Completions

Generate and manage shell completion scripts.

**What gets completed**: Command names, workflow names, engine names (`--engine`), directory paths (`--dir`), and helpful descriptions.

#### `gh aw completion install`

Auto-detects your shell and installs completions:

```bash
gh aw completion install
gh aw completion install --verbose  # Show detailed steps
```

**Supported shells**: Bash (`~/.bash_completion.d/`), Zsh (`~/.zsh/completions/`), Fish (`~/.config/fish/completions/`), PowerShell.

#### `gh aw completion uninstall`

```bash
gh aw completion uninstall
```

#### Manual Generation

```bash
# Bash
gh aw completion bash > ~/.bash_completion.d/gh-aw
source ~/.bash_completion.d/gh-aw

# Zsh
gh aw completion zsh > "${fpath[1]}/_gh-aw"
compinit

# Fish
gh aw completion fish > ~/.config/fish/completions/gh-aw.fish

# PowerShell
gh aw completion powershell | Out-String | Invoke-Expression
```

---

### 7.2 `gh aw hash-frontmatter` — Deterministic Frontmatter Hashing

Computes a SHA-256 hash of workflow frontmatter for change detection.

**Hash includes**:
- All frontmatter fields from the main workflow
- Frontmatter from all imported workflows (BFS traversal)
- Template expressions containing `env.` or `vars.` from the markdown body
- Version information (gh-aw, awf, agents)

```bash
gh aw hash-frontmatter my-workflow.md
gh aw hash-frontmatter .github/workflows/audit-workflows.md
```

**Use case**: Detect configuration changes between compilation and execution — if the hash changed, a recompile is needed.

---

### 7.3 `gh aw mcp-server` — Run gh-aw as MCP Server

Starts an MCP server that wraps `gh aw` CLI commands as MCP tools, enabling AI agents (like GitHub Copilot) to manage agentic workflows.

**Available tools**: `status`, `compile`, `logs`, `audit`, `mcp-inspect`, `add`, `update`, `fix`

**Transports**:
- **stdio** (default): For MCP clients
- **HTTP** (`--port`): SSE transport for web-based clients

**Access control**: The `GITHUB_ACTOR` environment variable determines which tools are available based on repository role. Tools like `logs` and `audit` require write+ access.

```bash
# Run with stdio transport (default)
gh aw mcp-server

# Run HTTP server on port 8080
gh aw mcp-server --port 8080

# With actor validation enforced
gh aw mcp-server --validate-actor

# Set actor for access control
GITHUB_ACTOR=octocat gh aw mcp-server

# Use custom gh-aw binary
gh aw mcp-server --cmd ./gh-aw

# Debug mode
DEBUG=mcp:* GITHUB_ACTOR=octocat gh aw mcp-server
```

> **Security**: Spawns subprocess calls for each tool invocation, ensuring GitHub tokens and secrets are not shared with the MCP server process.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--cmd` | | Path to `gh aw` command (default: `gh aw`) |
| `--port` | `-p` | Port for HTTP server (uses stdio if not set) |
| `--validate-actor` | | Enforce actor validation for elevated commands |

---

### 7.4 `gh aw pr` — Pull Request Utilities

#### `gh aw pr transfer` — Transfer PR Between Repositories

Transfers a pull request from one repository to another (e.g., from a trial repo to production).

**What it does**:
1. Fetches PR details (title, body, changes)
2. Applies changes as a single squashed commit
3. Creates a new PR in the target repository
4. Copies the original title and description

```bash
# Transfer to current repository
gh aw pr transfer https://github.com/trial/repo/pull/234

# Transfer to specific target repository
gh aw pr transfer https://github.com/source/repo/pull/123 --repo owner/target

# Transfer from trial repo to production
gh aw pr transfer https://github.com/gh-aw-trial/repo/pull/5 --repo owner/prod-repo
```

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--repo` | `-r` | Target repository (default: current) |

---

### 7.5 `gh aw project` — GitHub Projects V2

#### `gh aw project new` — Create Project Board

Creates a new GitHub Projects V2 board.

```bash
# Create user project
gh aw project new "My Project" --owner @me

# Create org project
gh aw project new "Team Board" --owner myorg

# Create and link to repository
gh aw project new "Bugs" --owner myorg --link myorg/myrepo

# With standard project setup (views, fields)
gh aw project new "Project Q1" --owner myorg --with-project-setup
```

**`--with-project-setup`** creates:
- Standard views: Progress Board, Task Tracker, Roadmap
- Custom fields: Tracker Id, Worker Workflow, Target Repo, Priority, Size, dates
- Enhanced Status field with "Review Required" option

> **Note**: The default `GITHUB_TOKEN` cannot create projects — additional authentication is required.

**Flags**:

| Flag | Short | Description |
|------|-------|-------------|
| `--link` | `-l` | Repository to link project to (`owner/repo`) |
| `--owner` | `-o` | Project owner: `@me` for current user or org name (required) |
| `--with-project-setup` | | Create standard project views and fields |

---

### 7.6 `gh aw version` — Version Information

```bash
gh aw version
# Output: gh aw version v0.61.0
```

---

## 8. Command Quick-Reference Table

### By Lifecycle Phase

#### Setup & Author

| Command | Description | Key Flags |
|---------|-------------|-----------|
| `gh aw init` | Initialize repository | `--no-mcp`, `--codespaces`, `--completions`, `--create-pull-request` |
| `gh aw new [name]` | Create workflow template | `-e/--engine`, `-f/--force`, `-i/--interactive` |
| `gh aw add <spec>` | Add workflow (non-interactive) | `-d/--dir`, `-e`, `-f`, `-n/--name`, `--create-pull-request` |
| `gh aw add-wizard <spec>` | Add workflow (interactive) | `-e`, `--skip-secret`, `-d/--dir` |
| `gh aw remove [pattern]` | Remove workflows | `--keep-orphans` |
| `gh aw secrets set <name>` | Set a secret | `--value`, `--value-from-env`, `-r/--repo` |
| `gh aw secrets bootstrap` | Check/set missing secrets | `-e/--engine`, `--non-interactive` |
| `gh aw update [wf]` | Update from upstream | `--no-merge`, `--major`, `-f`, `--create-pull-request` |
| `gh aw upgrade` | Full repository upgrade | `--audit`, `--no-fix`, `--no-actions`, `--create-pull-request` |

#### Compile & Validate

| Command | Description | Key Flags |
|---------|-------------|-----------|
| `gh aw compile [wf]` | Compile MD → YAML | `--strict`, `--watch`, `--validate`, `--zizmor`, `--dependabot`, `--purge` |
| `gh aw validate [wf]` | Validate without output | `--strict`, `--fail-fast`, `--stats` |
| `gh aw fix [wf]` | Auto-fix deprecated fields | `--write`, `--list-codemods` |

#### Execute & Test

| Command | Description | Key Flags |
|---------|-------------|-----------|
| `gh aw run [wf]` | Trigger workflow | `--push`, `--repeat`, `-F key=value`, `--ref`, `--dry-run` |
| `gh aw trial <spec>` | Test in temp repo | `--logical-repo`, `--clone-repo`, `--repeat`, `--delete-host-repo-after` |
| `gh aw enable [wf]` | Enable workflows | `-r/--repo` |
| `gh aw disable [wf]` | Disable + cancel runs | `-r/--repo` |

#### Monitor & Debug

| Command | Description | Key Flags |
|---------|-------------|-----------|
| `gh aw list [pattern]` | Quick listing (no API) | `--label`, `-r/--repo`, `-j/--json` |
| `gh aw status [pattern]` | Detailed status | `--ref`, `--label`, `-r/--repo` |
| `gh aw logs [wf]` | Download logs + metrics | `-c`, `--start-date`, `--engine`, `--parse`, `--tool-graph` |
| `gh aw audit <id>` | Deep-dive run analysis | `--parse`, `-r/--repo`, `-o/--output` |
| `gh aw health [wf]` | Health metrics | `--days`, `--threshold` |
| `gh aw checks <pr>` | PR CI classification | `-j/--json`, `-r/--repo` |
| `gh aw domains [wf]` | Network domain listing | `-j/--json` |

#### Utilities

| Command | Description | Key Flags |
|---------|-------------|-----------|
| `gh aw mcp list` | List MCP servers | |
| `gh aw mcp inspect [wf]` | Inspect MCP servers | `--server`, `--tool`, `--inspector`, `--check-secrets` |
| `gh aw mcp add [wf] [srv]` | Add MCP from registry | `--transport`, `--registry` |
| `gh aw mcp list-tools <srv>` | List MCP tools | |
| `gh aw completion install` | Install shell completions | |
| `gh aw hash-frontmatter <wf>` | Compute frontmatter hash | |
| `gh aw mcp-server` | Run as MCP server | `--port`, `--validate-actor`, `--cmd` |
| `gh aw pr transfer <url>` | Transfer PR to repo | `-r/--repo` |
| `gh aw project new <title>` | Create Projects V2 board | `-o/--owner`, `-l/--link`, `--with-project-setup` |
| `gh aw version` | Show version | |

---

## 9. Common Workflows & Recipes

### Recipe 1: First-Time Repository Setup

```bash
# 1. Install the extension
gh extension install github/gh-aw

# 2. Initialize your repository
cd your-repo
gh aw init

# 3. Add a pre-built workflow with guided setup
gh aw add-wizard githubnext/agentics/daily-repo-status

# 4. Workflow is automatically compiled and optionally triggered
# Check the Actions tab for your first run!
```

### Recipe 2: Create and Deploy a Custom Workflow

```bash
# 1. Create a workflow template
gh aw new pr-reviewer

# 2. Edit .github/workflows/pr-reviewer.md with your instructions

# 3. Compile the workflow
gh aw compile pr-reviewer

# 4. Commit and push
git add .github/
git commit -m "Add PR reviewer agentic workflow"
git push

# 5. Trigger manually to test
gh aw run pr-reviewer
```

### Recipe 3: Update All Workflows from Upstream

```bash
# 1. Update all workflows with 'source' field
gh aw update

# 2. Review changes
git diff

# 3. Or create a PR directly
gh aw update --create-pull-request
```

### Recipe 4: Debug a Failed Workflow Run

```bash
# 1. Check workflow status
gh aw status --ref main

# 2. Download and analyze logs
gh aw logs my-workflow -c 5

# 3. Deep-dive into a specific run
gh aw audit https://github.com/owner/repo/actions/runs/12345 --parse

# 4. Review the generated Markdown report
cat .github/aw/logs/run-12345/log.md
```

### Recipe 5: Test Before Deploying to Production

```bash
# 1. Trial the workflow in a temporary repo
gh aw trial githubnext/agentics/ci-doctor

# 2. Review results in trials/ directory
ls trials/

# 3. If satisfied, add to your repo
gh aw add githubnext/agentics/ci-doctor --create-pull-request

# 4. Or transfer the trial PR to production
gh aw pr transfer https://github.com/user/gh-aw-trial/pull/1 --repo myorg/myrepo
```

### Recipe 6: Monitor Workflow Health Over Time

```bash
# Quick status check
gh aw status --ref main

# 7-day health summary
gh aw health

# 30-day detailed metrics for specific workflow
gh aw health issue-triage --days 30

# Alert if any workflow drops below 90% success
gh aw health --threshold 90

# JSON export for dashboards
gh aw health --json > health-report.json
```

### Recipe 7: Full Repository Upgrade

```bash
# 1. Check dependency health first
gh aw upgrade --audit

# 2. Perform full upgrade
gh aw upgrade --create-pull-request

# 3. Review the generated PR for all changes
```

### Recipe 8: Multi-Repository Workflow Management

```bash
# List workflows in another repo
gh aw list --repo org/other-repo

# Check status
gh aw status --repo org/other-repo --ref main

# Download logs
gh aw logs weekly-research --repo org/other-repo

# Enable/disable
gh aw enable ci-doctor --repo org/other-repo
gh aw disable old-workflow --repo org/other-repo

# Run workflow in another repo
gh aw run daily-plan --repo org/other-repo
```

### Recipe 9: Compile with Maximum Security Validation

```bash
# Strict mode + all security scanners + validation
gh aw compile --strict --zizmor --actionlint --poutine --validate

# Or use validate (which has all linters always-on)
gh aw validate --strict

# Generate Dependabot integration
gh aw compile --dependabot
```

### Recipe 10: Set Up MCP Integration

```bash
# List available MCP servers from registry
gh aw mcp add

# Add a server to your workflow
gh aw mcp add my-workflow makenotion/notion-mcp-server

# Inspect what tools are available
gh aw mcp inspect my-workflow --server notion

# Launch the MCP Inspector for debugging
gh aw mcp inspect my-workflow --inspector

# Check if required secrets are configured
gh aw mcp inspect my-workflow --check-secrets
```

---

## 10. Environment Variables & Configuration

### Environment Variables

| Variable | Description | Used By |
|----------|-------------|---------|
| `GH_HOST` | GitHub Enterprise Server hostname | All commands (GHES support) |
| `GH_AW_ACTION_MODE` | Override action mode (`dev`, `release`, `action`, `script`) | `compile` |
| `GITHUB_ACTOR` | GitHub username for role-based access control | `mcp-server` |
| `GITHUB_API_URL` | Custom API base URL | `secrets set` |
| `GITHUB_SERVER_URL` | Auto-set by GitHub Actions on GHES | Agent job (auto-detected) |

### Configuration Files Created by `gh aw init`

| File | Purpose |
|------|---------|
| `.gitattributes` | Marks `.lock.yml` as generated for merge handling |
| `.github/agents/agentic-workflows.agent.md` | Dispatcher agent for GitHub Copilot Chat |
| `.github/workflows/copilot-setup-steps.yml` | MCP installation steps |
| `.vscode/mcp.json` | MCP server configuration for VSCode |
| `.vscode/settings.json` | VSCode settings |

### Files Created by Compilation

| File | Purpose |
|------|---------|
| `.github/workflows/*.lock.yml` | Compiled GitHub Actions workflow |
| `.github/aw/imports/` | Cached remote imports (by commit SHA) |
| `.github/aw/actions-lock.json` | Pinned GitHub Actions versions |
| `.github/workflows/agentics-maintenance.yml` | Auto-generated if `expires` is used |

### Workflow ID Convention

The **workflow-id** is the basename of the Markdown file without the `.md` extension:
- File: `.github/workflows/ci-doctor.md` → ID: `ci-doctor`
- You can use either format: `gh aw run ci-doctor` or `gh aw run ci-doctor.md`

---

## 11. Troubleshooting

### Lock File Out of Sync

**Symptom**: Workflow behavior doesn't match your markdown changes.

```bash
# Recompile all workflows
gh aw compile

# Or recompile specific workflow
gh aw compile my-workflow

# Purge orphaned lock files
gh aw compile --purge
```

> **Remember**: Markdown body changes take effect immediately (loaded at runtime). Only **frontmatter** changes require recompilation.

### Missing Secrets

**Symptom**: Workflow fails at engine authentication step.

```bash
# Check which secrets are missing
gh aw secrets bootstrap --non-interactive

# Set missing secret
gh aw secrets set COPILOT_GITHUB_TOKEN --value "ghp_..."

# Or interactively fix all missing secrets
gh aw secrets bootstrap
```

### Compilation Errors

**Symptom**: `gh aw compile` fails with validation errors.

```bash
# See detailed error messages with file paths and line numbers
gh aw compile -v

# Run validation with all linters
gh aw validate --strict

# Check and fix deprecated fields first
gh aw fix --write
gh aw compile
```

### Network/Firewall Blocking

**Symptom**: Agent cannot reach required APIs at runtime.

```bash
# Check which domains are configured
gh aw domains my-workflow

# Review firewall logs from a run
gh aw audit <run-id> --parse
# Check .github/aw/logs/run-<id>/firewall.md
```

**Fix**: Add missing domains to `network.allowed` in frontmatter:

```yaml
network:
  allowed:
    - defaults
    - python                # Add ecosystem
    - "api.example.com"    # Add custom domain
```

### Workflow Not Triggering

**Symptom**: Workflow doesn't run on expected events.

```bash
# Check if workflow is enabled
gh aw status

# Enable if disabled
gh aw enable my-workflow

# Check if lock file is committed and up to date
gh aw list

# Force a manual run to test
gh aw run my-workflow
```

### MCP Server Connection Issues

```bash
# Inspect MCP server configuration
gh aw mcp inspect my-workflow

# Check if required secrets exist
gh aw mcp inspect my-workflow --check-secrets

# Use the MCP Inspector for interactive debugging
gh aw mcp inspect my-workflow --inspector

# Check available tools
gh aw mcp list-tools github my-workflow
```

### Upgrading and Migration

```bash
# Check what codemods are available
gh aw fix --list-codemods

# Dry-run to see what would change
gh aw fix

# Apply all fixes
gh aw fix --write

# Full upgrade (agent files + codemods + actions + compile)
gh aw upgrade

# Check dependency health
gh aw upgrade --audit
```

---

> **Document version**: Based on `gh aw v0.61.0` — March 2026  
> **Source**: [github/gh-aw](https://github.com/github/gh-aw) | **Docs**: [github.github.io/gh-aw](https://github.github.io/gh-aw/)
