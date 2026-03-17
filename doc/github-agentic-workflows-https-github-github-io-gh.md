# GitHub Agentic Workflows — Ultra-Deep Research Report

## Executive Summary

GitHub Agentic Workflows (gh-aw) is a new automation paradigm developed by **GitHub Next** and **Microsoft Research** that enables repository automation through natural language markdown files executed by AI coding agents (GitHub Copilot CLI, Claude by Anthropic, OpenAI Codex, or Google Gemini) within GitHub Actions[^1][^2]. Unlike traditional GitHub Actions workflows written in complex YAML, agentic workflows let you describe automation intent in plain English markdown, and a compiler (`gh aw compile`) translates them into hardened GitHub Actions YAML (`.lock.yml` files) with built-in security guardrails including sandboxed execution, network isolation, read-only default permissions, output sanitization, and threat detection[^3][^4]. The system is currently in **technical preview** as of February 2026[^5], and the core implementation is open source at [github/gh-aw](https://github.com/github/gh-aw).

---

## Table of Contents

1. [Core Concepts and Philosophy](#1-core-concepts-and-philosophy)
2. [Architecture Overview](#2-architecture-overview)
3. [Workflow Structure](#3-workflow-structure)
4. [The Compilation Pipeline](#4-the-compilation-pipeline)
5. [AI Engines (Coding Agents)](#5-ai-engines-coding-agents)
6. [Security Architecture (Defense-in-Depth)](#6-security-architecture-defense-in-depth)
7. [Safe Outputs System](#7-safe-outputs-system)
8. [Tools and MCP Integration](#8-tools-and-mcp-integration)
9. [Network Controls and Sandbox](#9-network-controls-and-sandbox)
10. [Threat Detection Pipeline](#10-threat-detection-pipeline)
11. [Trigger System](#11-trigger-system)
12. [Imports and Modularity](#12-imports-and-modularity)
13. [Operational Patterns](#13-operational-patterns)
14. [CLI Reference (`gh aw`)](#14-cli-reference-gh-aw)
15. [Authentication](#15-authentication)
16. [Key Repositories Summary](#16-key-repositories-summary)
17. [Confidence Assessment](#17-confidence-assessment)
18. [Footnotes](#18-footnotes)

---

## 1. Core Concepts and Philosophy

### What Are Agentic Workflows?

Agentic workflows are **AI-powered automation** that can understand context, make decisions, and take meaningful actions — all from natural language instructions written in markdown[^6]. The term "agentic" comes from "agent" + "-ic" (having the characteristics of), meaning **having agency** — the ability to act independently, make context-aware decisions, and adapt behavior based on circumstances[^7].

### How They Differ from Traditional CI/CD

| Aspect | Traditional GitHub Actions | Agentic Workflows |
|--------|---------------------------|-------------------|
| **Language** | Complex YAML | Natural language Markdown |
| **Logic** | Fixed if-then rules | AI-driven contextual reasoning |
| **Adaptation** | Requires explicit programming for each scenario | Flexibly adapts to different scenarios |
| **Permissions** | Developer-granted | Read-only by default + safe outputs |
| **Decision Making** | Deterministic | Context-aware, AI-interpreted |

Agentic workflows are **100% additive** to existing CI/CD — they don't replace deterministic build, test, or release pipelines. They are described as **"Continuous AI"** alongside Continuous Integration and Continuous Deployment: a new automation layer for tasks where exact reproducibility doesn't matter — triaging issues, drafting documentation, researching dependencies, or proposing code improvements for human review[^8].

### Design Principles

1. **Security-first**: Read-only defaults, sandboxed execution, output sanitization
2. **Natural language over YAML**: Describe intent, not implementation
3. **Engine-agnostic**: Support for multiple AI providers (Copilot, Claude, Codex, Gemini)
4. **Composable**: Import and reuse workflow components
5. **Observable**: Full audit trail through compiled lock files and workflow logs

---

## 2. Architecture Overview

### High-Level System Flow

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Workflow (.md)   │────▶│   Compilation    │────▶│  .lock.yml       │
│  Natural Language │     │   (gh aw compile)│     │  GitHub Actions  │
│  + YAML Frontmatter│    │  Schema Validate │     │  YAML            │
└──────────────────┘     │  SHA Pin Actions │     └────────┬─────────┘
                          │  Security Harden │              │
                          └──────────────────┘              │ GitHub Event
                                                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       GitHub Actions Runtime                         │
│                                                                      │
│  ┌──────────────┐   ┌─────────────────┐   ┌───────────────────────┐ │
│  │ Pre-Activation│──▶│  Agent Job       │──▶│  Threat Detection    │ │
│  │ Role Checks  │   │  (Read-Only)    │   │  Job                 │ │
│  │ Permission   │   │  AI Engine      │   └──────────┬────────────┘ │
│  │ Validation   │   │  + MCP Tools    │              │ Approved     │
│  └──────────────┘   │  + AWF Firewall │              ▼              │
│                      └─────────────────┘   ┌───────────────────────┐ │
│                                             │  Safe Output Jobs    │ │
│                                             │  (Scoped Write       │ │
│                                             │   Permissions)       │ │
│                                             │  Create Issue/PR/    │ │
│                                             │  Comment/Label       │ │
│                                             └───────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### Three-Layer Security Model

The architecture implements defense-in-depth across three trust layers[^9]:

**Layer 1 — Substrate-Level Trust**: Hardware, kernel, container runtime, AWF firewall, API proxy, and MCP Gateway provide memory isolation, CPU isolation, mediation of privileged operations, and kernel-enforced communication boundaries.

**Layer 2 — Configuration-Level Trust**: Declarative configuration artifacts (Action steps, network-firewall policies, MCP server configurations) constrain which components are loaded, how they communicate, which channels are permitted, and what privileges are assigned.

**Layer 3 — Plan-Level Trust**: The trusted compiler decomposes a workflow into stages. For each stage, the plan specifies active components, their permissions, data produced, and how it may be consumed by subsequent stages. The **SafeOutputs** subsystem is the primary instantiation of this layer.

---

## 3. Workflow Structure

### Anatomy of an Agentic Workflow File

Each workflow is a **Markdown file** (`.md`) stored in `.github/workflows/` with two parts[^10]:

```markdown
---
# 1. YAML Frontmatter (Configuration)
on:
  issues:
    types: [opened]
permissions:
  contents: read
  issues: read
safe-outputs:
  add-comment:
tools:
  github:
    toolsets: [issues]
---

# 2. Markdown Body (Natural Language Instructions)
# Issue Clarifier

Analyze the current issue and ask for additional details
if the issue is unclear.
```

### File Organization

```
.github/
└── workflows/
    ├── ci-doctor.md           # Agentic Workflow (source)
    ├── ci-doctor.lock.yml     # Compiled GitHub Actions Workflow
    ├── issue-triage.md        # Another workflow
    ├── issue-triage.lock.yml
    └── shared/                # Shared components (no `on:` field)
        ├── common-tools.md
        └── mcp/
            └── tavily.md
```

**Key property**: The **markdown body** is loaded at runtime and can be edited directly on GitHub.com without recompilation. Only **frontmatter changes** require recompilation via `gh aw compile`[^10].

### Frontmatter Elements

The YAML frontmatter supports an extensive set of configuration fields[^11]:

| Field | Purpose |
|-------|---------|
| `on:` | Trigger events (issues, PRs, schedule, dispatch, etc.) |
| `permissions:` | GitHub Actions permissions (read-only by default) |
| `safe-outputs:` | Allowed write operations (issues, PRs, comments, labels) |
| `tools:` | Available tools (GitHub API, bash, web-fetch, edit, etc.) |
| `engine:` | AI engine selection (copilot, claude, codex, gemini) |
| `network:` | Network access controls (domain allowlists) |
| `imports:` | Shared component imports |
| `mcp-servers:` | Custom MCP server configurations |
| `mcp-scripts:` | Inline custom MCP tools (JS, shell, Python, Go) |
| `strict:` | Enhanced security validation (default: true) |
| `plugins:` | Engine plugins to install |
| `dependencies:` | APM (Agent Package Manager) packages |
| `runtimes:` | Runtime version overrides (Node, Python, Go, etc.) |
| `sandbox:` | Sandbox configuration (AWF, MCP Gateway) |
| `threat-detection:` | Threat detection configuration |
| `labels:` | Workflow categorization labels |
| `metadata:` | Custom key-value metadata |
| `private:` | Prevent external installation |
| `source:` | Track workflow origin |
| `resources:` | Companion files to fetch |

---

## 4. The Compilation Pipeline

The `gh aw compile` command transforms markdown workflows into hardened GitHub Actions YAML[^12]:

### Compilation Steps

1. **Schema Validation**: Validates frontmatter against the workflow schema
2. **Import Resolution**: Resolves and merges local/remote imports via BFS traversal
3. **Expression Safety Check**: Validates GitHub Actions expressions
4. **Action SHA Pinning**: Pins all referenced actions to specific commit SHAs
5. **Security Scanning**: Runs actionlint, zizmor, and poutine scanners
6. **Tool Configuration**: Generates MCP server configs and tool allowlists
7. **Network Hardening**: Configures AWF firewall domain allowlists
8. **Lock File Generation**: Produces the `.lock.yml` GitHub Actions workflow

### The Lock File

The `.lock.yml` is the compiled output that GitHub Actions actually executes[^13]. It contains:
- Complete GitHub Actions YAML with all jobs and steps
- SHA-pinned action references
- Resolved imports and merged configurations
- Embedded frontmatter configuration
- Security hardening (permissions, network, tool restrictions)
- Runtime reference to the markdown body (loaded at execution time)

Both `.md` and `.lock.yml` files should be committed to version control.

### Strict Mode

Strict mode (enabled by default) enforces security best practices[^14]:

1. Refuses write permissions — forces use of safe outputs instead
2. Requires explicit network configuration
3. Refuses wildcard `*` in `network.allowed` domains
4. Requires ecosystem identifiers instead of individual ecosystem domains
5. Requires network config for custom MCP servers with containers
6. Enforces GitHub Actions pinned to commit SHAs
7. Refuses deprecated frontmatter fields

---

## 5. AI Engines (Coding Agents)

GitHub Agentic Workflows supports four AI engines[^15]:

| Engine | `engine:` Value | Required Secret | Default |
|--------|----------------|-----------------|---------|
| GitHub Copilot CLI | `copilot` | `COPILOT_GITHUB_TOKEN` | ✓ (default) |
| Claude by Anthropic | `claude` | `ANTHROPIC_API_KEY` | |
| OpenAI Codex | `codex` | `OPENAI_API_KEY` | |
| Google Gemini CLI | `gemini` | `GEMINI_API_KEY` | |

### Extended Configuration

```yaml
engine:
  id: copilot
  version: "0.0.422"         # Pin specific version
  model: gpt-5               # Model selection
  command: /usr/local/bin/copilot  # Custom executable
  args: ["--add-dir", "/workspace"]  # CLI arguments
  agent: technical-doc-writer  # Custom agent file
  api-target: api.acme.ghe.com  # Enterprise API endpoint
  env:
    DEBUG_MODE: "true"
```

### Custom Copilot Agents

For the Copilot engine, you can reference custom agent files located in `.github/agents/`[^15]:

```yaml
engine:
  id: copilot
  agent: technical-doc-writer  # References .github/agents/technical-doc-writer.agent.md
```

### Enterprise Support

The `api-target` field supports GitHub Enterprise Cloud (GHEC) and GitHub Enterprise Server (GHES) deployments[^15]:

```yaml
engine:
  id: copilot
  api-target: api.acme.ghe.com
```

Custom API endpoints are also supported via `OPENAI_BASE_URL` (for Codex) and `ANTHROPIC_BASE_URL` (for Claude), enabling internal LLM routers and Azure OpenAI deployments.

---

## 6. Security Architecture (Defense-in-Depth)

### Adversary Model

The security architecture considers an adversary that may compromise untrusted user-level components (containers) and cause them to behave arbitrarily within granted privileges. The adversary may attempt to[^9]:

- Access or corrupt memory/state of other components
- Communicate over unintended channels
- Abuse legitimate channels for unintended actions
- Confuse higher-level control logic

### Compilation-Time Security

- **Schema Validation**: Comprehensive frontmatter validation
- **Expression Safety**: Allowlisted expressions only
- **Action SHA Pinning**: All GitHub Actions pinned to exact commit SHAs
- **Security Scanners**: actionlint, zizmor, poutine run during compilation
- **Permission Validation**: Strict mode enforces least-privilege

### Runtime Security

- **Pre-Activation Checks**: Role and permission validation before agent execution
- **Content Sanitization**: Input sanitization during activation
- **Read-Only Agent**: Agent job runs with minimal read-only permissions
- **Secret Redaction**: Credentials protected from agent access

### Isolation Mechanisms

- **Agent Workflow Firewall (AWF)**: Network egress control via domain allowlist
- **API Proxy**: Agent auth-token isolation (tokens kept out of agent container)
- **MCP Server Sandboxing**: Each MCP server runs in isolated Docker containers
- **Tool Allowlisting**: Only explicitly permitted tools are available

### Output Security

- **Threat Detection**: AI-powered analysis of agent output
- **Safe Outputs**: Permission separation between read (agent) and write (safe output jobs)
- **Output Sanitization**: Content validation, secret redaction, URL filtering
- **Protected Files**: Supply chain protection against modification of sensitive files

### Cross-Prompt Injection Attack (XPIA) Mitigation

XPIA attacks occur when malicious instructions are embedded in external data (issue bodies, PR descriptions, file contents) to hijack an AI agent's behavior. Mitigations include[^7]:

- System prompt hardening
- Threat detection scanning
- Lockdown mode (filtering content in public repos to trusted users)
- `min-integrity` guard policies filtering by author trust level

---

## 7. Safe Outputs System

The SafeOutputs subsystem is the core mechanism enabling agentic workflows to perform write operations while maintaining security[^16]. The AI agent generates structured output describing desired actions, which are processed by **separate permission-controlled jobs** that execute only after the agent completes.

### Architecture

```
┌────────────────────┐
│  Agent Job          │  (Read-Only Permissions)
│  AI generates       │
│  structured output  │
│  → agent_output.json│
└────────┬───────────┘
         │ artifacts
         ▼
┌────────────────────┐
│  Threat Detection   │  (Analyzes for security issues)
│  Job                │
└────────┬───────────┘
         │ approved
         ▼
┌────────────────────┐
│  Safe Output Jobs   │  (Scoped Write Permissions)
│  • create-issue     │  (issues: write)
│  • add-comment      │  (issues: write)
│  • create-pr        │  (contents: write, pull-requests: write)
│  • add-labels       │  (issues: write)
│  • dispatch-workflow│  (actions: write)
└────────────────────┘
```

### Available Safe Output Types

#### Issues & Discussions
| Type | Description | Default Max |
|------|-------------|-------------|
| `create-issue` | Create GitHub issues | 1 |
| `update-issue` | Update issue status, title, body | 1 |
| `close-issue` | Close issues with comment | 1 |
| `link-sub-issue` | Link issues as sub-issues | 1 |
| `create-discussion` | Create GitHub discussions | 1 |
| `update-discussion` | Update discussion | 1 |
| `close-discussion` | Close discussions | 1 |

#### Pull Requests
| Type | Description | Default Max |
|------|-------------|-------------|
| `create-pull-request` | Create PRs with code changes | 1 |
| `update-pull-request` | Update PR title or body | 1 |
| `close-pull-request` | Close PRs without merging | 10 |
| `create-pull-request-review-comment` | Create code review comments | 10 |
| `push-to-pull-request-branch` | Push changes to PR branch | 1 |

#### Labels, Assignments & Reviews
| Type | Description | Default Max |
|------|-------------|-------------|
| `add-comment` | Post comments on issues/PRs/discussions | 1 |
| `hide-comment` | Hide/minimize comments | 5 |
| `add-labels` | Add labels | 3 |
| `remove-labels` | Remove labels | 3 |
| `add-reviewer` | Add PR reviewers | 3 |
| `assign-milestone` | Assign issues to milestones | 1 |
| `assign-to-agent` | Assign Copilot coding agent | 1 |
| `assign-to-user` | Assign users to issues | 1 |

#### Projects, Releases & Security
| Type | Description | Default Max |
|------|-------------|-------------|
| `create-project` | Create GitHub Projects boards | 1 |
| `update-project` | Manage Projects boards | 10 |
| `update-release` | Update release descriptions | 1 |
| `upload-asset` | Upload files to orphaned git branch | 10 |
| `dispatch-workflow` | Trigger other workflows | 3 |
| `call-workflow` | Call reusable workflows (compile-time fan-out) | 1 |
| `create-code-scanning-alert` | Generate SARIF security advisories | unlimited |
| `create-agent-session` | Create Copilot coding agent sessions | 1 |

### Key Features

**Cross-Repository Operations**: Most safe outputs support `target-repo` for creating issues, PRs, and comments in external repositories[^17].

**Protected Files**: Supply chain protection prevents AI from modifying sensitive files (dependency manifests, CI/CD config, agent instruction files) by default. Policies: `blocked` (default), `allowed`, or `fallback-to-issue`[^18].

**Text Sanitization**: All outputs undergo secret redaction, URL domain filtering, XML escaping, size limits, control character stripping, GitHub reference escaping, and HTTPS enforcement[^8].

**Workflow-ID Markers**: All created items include hidden `<!-- gh-aw-workflow-id: WORKFLOW_NAME -->` markers for tracking[^16].

**Custom Safe Output Jobs**: Create custom post-processing jobs registered as MCP tools, supporting standard GitHub Actions properties with access to agent output via `$GH_AW_AGENT_OUTPUT`[^16].

---

## 8. Tools and MCP Integration

### Built-in Tools

Tools are configured in frontmatter to specify available capabilities[^19]:

| Tool | Purpose | Configuration |
|------|---------|--------------|
| `edit:` | File editing in workspace | `tools: edit:` |
| `github:` | GitHub API operations (MCP) | `tools: github: toolsets: [repos, issues]` |
| `bash:` | Shell command execution | `tools: bash: ["echo", "ls", "git status"]` |
| `web-fetch:` | Fetch web content | `tools: web-fetch:` |
| `web-search:` | Search the web | `tools: web-search:` |
| `playwright:` | Browser automation | `tools: playwright:` |
| `cache-memory:` | Persistent memory across runs | `tools: cache-memory:` |
| `repo-memory:` | Repository-specific memory | `tools: repo-memory:` |
| `agentic-workflows:` | Workflow introspection/debugging | `tools: agentic-workflows:` |

### GitHub Tools (MCP)

The GitHub MCP server provides fine-grained API access[^20]:

**Available Toolsets**: `context`, `repos`, `issues`, `pull_requests`, `users`, `actions`, `code_security`, `discussions`, `labels`, `notifications`, `orgs`, `projects`, `gists`, `search`, `dependabot`, `experiments`, `secret_protection`, `security_advisories`, `stargazers`

**Modes**:
- **Local** (default): Docker container for isolation
- **Remote**: Hosted MCP server for faster startup (requires additional auth)

**Guard Policies**: Fine-grained access control restricting which repositories and content integrity levels the agent can read[^20]:

```yaml
tools:
  github:
    repos: ["myorg/*", "partner/shared-repo"]
    min-integrity: approved  # Only OWNER, MEMBER, COLLABORATOR content
```

### Custom MCP Servers

Integrate third-party MCP servers via Docker, command, HTTP, or registry[^19]:

```yaml
mcp-servers:
  slack:
    command: "npx"
    args: ["-y", "@slack/mcp-server"]
    env:
      SLACK_BOT_TOKEN: "${{ secrets.SLACK_BOT_TOKEN }}"
    allowed: ["send_message", "get_channel_history"]
```

### MCP Scripts (Inline Tools)

Define custom MCP tools inline in frontmatter using JavaScript, shell, Python, or Go[^21]. These run **outside the agent container** on the GitHub Actions runner:

```yaml
mcp-scripts:
  fetch-data:
    description: "Fetch data from API"
    inputs:
      endpoint:
        type: string
        required: true
    script: |
      const apiKey = process.env.API_KEY;
      const response = await fetch(`https://api.example.com/${endpoint}`, {
        headers: { Authorization: `Bearer ${apiKey}` }
      });
      return await response.json();
    env:
      API_KEY: "${{ secrets.API_KEY }}"
```

### MCP Gateway

The MCP Gateway routes all MCP server calls through a unified HTTP gateway for centralized management, logging, and authentication[^22]:

```
┌─────────────────────────────────────┐
│  Agent Container (AWF Network)       │
│  Copilot CLI + MCP Client           │
│  172.30.0.20                        │
│           │                          │
│           │ CONNECT host.docker.internal:80
│           ▼                          │
│  ┌────────────────┐                 │
│  │ Squid Proxy    │                 │
│  │ 172.30.0.10    │                 │
│  └────────┬───────┘                 │
└───────────┼─────────────────────────┘
            │ allowed domain
            ▼
┌───────────────────────┐
│ gh-aw-mcpg            │ (Host machine)
│ Docker container      │
│ Port 80 → 8000        │
│       │               │
│       │ spawns        │
│       ▼               │
│ ┌─────────────────┐   │
│ │ GitHub MCP Server│   │
│ │ (Docker socket)  │   │
│ └─────────────────┘   │
└───────────────────────┘
```

---

## 9. Network Controls and Sandbox

### Agent Workflow Firewall (AWF)

AWF is the default sandbox that containerizes the agent, binds it to a Docker network, and uses **iptables to redirect HTTP/HTTPS traffic through a Squid proxy container**. The Squid proxy controls the agent's egress traffic via a configurable domain allowlist[^23].

AWF separates two concerns:
- **Filesystem**: Controlled access to host binaries and runtimes via chroot mode
- **Network**: All traffic routed through proxy enforcing the domain allowlist

### Network Configuration

```yaml
network:
  allowed:
    - defaults              # Basic infrastructure
    - python               # Python/PyPI ecosystem
    - node                 # npm ecosystem
    - "api.example.com"    # Custom domain
  blocked:
    - "cdn.untrusted.com"  # Blocked domains take precedence
```

### Ecosystem Identifiers

Named shorthand references to predefined domain sets[^24]:

| Identifier | Includes |
|-----------|----------|
| `defaults` | Certificates, JSON schema, Ubuntu, package mirrors |
| `github` | GitHub domains (`github.com`, `*.githubusercontent.com`, etc.) |
| `python` | PyPI, pip, Conda |
| `node` | npm, yarn, pnpm |
| `go` | proxy.golang.org |
| `containers` | Docker Hub, GHCR, Quay |
| `dev-tools` | Codecov, Shields.io, Snyk, Renovate, CircleCI |
| `default-safe-outputs` | Compound: defaults + dev-tools + github + local |
| `local` | Loopback addresses |

Also supported: `dotnet`, `dart`, `haskell`, `java`, `julia`, `perl`, `php`, `ruby`, `rust`, `swift`, `terraform`, `playwright`, `linux-distros`

### SSL Bump for HTTPS Inspection

AWF supports SSL bump for deep packet inspection of HTTPS traffic, allowing URL-path-level filtering[^24]:

```yaml
network:
  firewall:
    ssl-bump: true
    allow-urls:
      - "https://github.com/githubnext/*"
      - "https://api.github.com/repos/*/issues"
```

### Supported Runtimes

The sandbox provides access to 12 language runtimes[^11]:

| Runtime | Default Version | Setup Action |
|---------|----------------|-------------|
| Node.js | 24 | `actions/setup-node@v6` |
| Python | 3.12 | `actions/setup-python@v5` |
| Go | 1.25 | `actions/setup-go@v5` |
| Ruby | 3.3 | `ruby/setup-ruby@v1` |
| Java | 21 | `actions/setup-java@v4` |
| .NET | 8.0 | `actions/setup-dotnet@v4` |
| Rust | via `uv` | `astral-sh/setup-uv@v5` |
| Bun | 1.1 | `oven-sh/setup-bun@v2` |
| Deno | 2.x | `denoland/setup-deno@v2` |
| Elixir | 1.17 | `erlef/setup-beam@v1` |
| Haskell | 9.10 | `haskell-actions/setup@v2` |

---

## 10. Threat Detection Pipeline

Threat detection is **automatically enabled** when safe outputs are configured. It runs as a separate job between the agent job and safe output jobs[^18].

### Detection Architecture

```
┌────────────────────┐
│ Agent Job Artifacts │
│ • agent_output.json│  (Buffered actions)
│ • aw.patch         │  (Git diff from agent)
│ • prompt.txt       │  (Original workflow context)
└────────┬───────────┘
         │ Download
         ▼
┌────────────────────┐
│ Threat Detection   │
│ ├─ AI Analysis     │  (Security-focused prompt)
│ │  • Prompt injection detection
│ │  • Secret leak detection
│ │  • Malicious patch detection
│ ├─ Custom Steps    │  (Semgrep, TruffleHog, etc.)
│ └─ Verdict         │
└────────┬───────────┘
         │
    ┌────┴────┐
    │Threats? │
    └────┬────┘
    No   │   Yes
    ▼         ▼
┌────────┐ ┌────────┐
│Safe Out│ │Workflow│
│Jobs    │ │Fails   │
│Proceed │ │Blocked │
└────────┘ └────────┘
```

### Configuration

```yaml
safe-outputs:
  create-pull-request:
  threat-detection:
    enabled: true
    prompt: |
      Focus on SQL injection vulnerabilities
      and authentication bypass attempts
    engine: copilot          # Override engine for detection
    steps:
      - name: Run TruffleHog
        run: trufflehog filesystem /tmp/gh-aw --only-verified
      - name: Run Semgrep
        run: semgrep scan /tmp/gh-aw/aw.patch --config=auto
```

### Supply Chain Protection (Protected Files)

Default-blocked file categories[^18]:

1. **Runtime dependency manifests**: `package.json`, `go.mod`, `requirements.txt`, `Gemfile`, `pom.xml`, etc.
2. **Engine instruction files**: `AGENTS.md` (Copilot), `CLAUDE.md`/`.claude/` (Claude), `.codex/` (Codex)
3. **Repository security config**: `.github/` prefix, `.agents/` prefix

---

## 11. Trigger System

### Standard GitHub Actions Triggers (Enhanced)

All standard GitHub Actions triggers are supported plus additional enhancements[^25]:

| Trigger | Description | Key Enhancements |
|---------|-------------|-----------------|
| `workflow_dispatch:` | Manual trigger | Custom input parameters |
| `schedule:` | Cron/fuzzy schedule | Fuzzy scheduling with time scattering |
| `issues:` | Issue events | `lock-for-agent`, `names:` label filtering |
| `pull_request:` | PR events | `forks:` filtering, code availability |
| `issue_comment:` | Comment events | Lock-for-agent support |
| `workflow_run:` | Post-workflow trigger | Auto security protections |
| `push:` | Push events | Standard |

### Novel Trigger Types

**Slash Commands** (`slash_command:`): Respond to `/command-name` mentions in issues, PRs, and comments[^25].

**Label Commands** (`label_command:`): Activate when a label is applied, auto-removes the label for re-triggering[^25]:

```yaml
on:
  label_command: deploy     # Treats label as one-shot command
```

### Fuzzy Scheduling

Human-friendly schedule syntax with automatic time scattering to avoid load spikes[^25]:

```yaml
on:
  schedule: daily                           # Scattered time
  schedule: daily around 14:00              # ±1 hour window
  schedule: daily between 9:00 and 17:00   # Business hours
  schedule: weekly on friday around 5pm     # Weekly
  schedule: every 2h                        # Interval
```

The compiler assigns each workflow a unique, deterministic execution time based on the file path.

### Advanced Trigger Features

- **`reaction:`** — Emoji reactions on triggering items for visual status feedback
- **`stop-after:`** — Auto-disable triggers after a deadline (cost control)
- **`manual-approval:`** — Require human approval via environment protection rules
- **`skip-if-match:`** — Skip when a GitHub search query has matches
- **`skip-if-no-match:`** — Skip when a search query has no matches
- **`on.roles:`** — Restrict triggers by repository permission level (default: admin, maintainer, write)
- **`on.bots:`** — Allow specific bot accounts to trigger workflows
- **`on.skip-roles:`** — Exempt users with specific roles
- **`on.skip-bots:`** — Skip for specific bot actors
- **`on.steps:`** — Inject custom deterministic pre-activation steps

---

## 12. Imports and Modularity

### Import System

Workflows can import shared configurations and components both from local files and remote repositories[^26]:

```yaml
# In frontmatter
imports:
  - shared/common-tools.md                           # Local
  - acme-org/shared-workflows/mcp/tavily.md@v1.0.0  # Remote

# In markdown body
{{#import shared/common-tools.md}}
{{#import? optional/file.md}}  # Optional (won't fail if missing)
```

### Shared Workflows

Workflows **without an `on:` field** are shared workflow components — validated but not compiled, designed to be imported by other workflows[^26].

### Import Merging Rules

| Field | Merge Behavior |
|-------|---------------|
| `tools:` | Deep merge with array concatenation; `allowed` arrays deduplicate |
| `mcp-servers:` | Imported servers override by name; first-wins ordering |
| `network:` | Union of `allowed` domains, deduplicated and sorted |
| `permissions:` | Validation only — must be explicitly declared in main workflow |
| `safe-outputs:` | One definition per type; main overrides imports |
| `runtimes:` | Main overrides imported versions |
| `services:` | Must be unique; duplicates fail compilation |
| `steps:` | Imported steps prepended to main workflow steps |

### Remote Import Caching

Remote imports are cached in `.github/aw/imports/` by commit SHA for offline compilation. The cache is git-tracked with `.gitattributes` configured for conflict-free merges[^26].

---

## 13. Operational Patterns

### Pattern Overview

GitHub Agentic Workflows supports several operational patterns[^27][^28]:

| Pattern | Description |
|---------|-------------|
| **IssueOps** | Single-repo issue automation |
| **ChatOps** | Command-driven workflows via slash commands |
| **LabelOps** | Label-based workflow triggering |
| **MultiRepoOps** | Cross-repository coordination |
| **SideRepoOps** | Running workflows from a separate sidecar repository |
| **ProjectOps** | GitHub Projects board automation |
| **Orchestration** | Multi-workflow coordination (orchestrator/worker pattern) |

### Orchestration Pattern

The orchestrator/worker pattern enables complex multi-workflow coordination[^27]:

```
┌─────────────────────┐
│  Orchestrator        │  Decides what to do,
│  Workflow            │  dispatches workers
└──────┬──────────────┘
       │ dispatch-workflow / call-workflow
       ├────────────────────────┐
       ▼                        ▼
┌──────────────┐    ┌──────────────┐
│  Worker A     │    │  Worker B     │
│  (triage)     │    │  (analysis)   │
└──────────────┘    └──────────────┘
```

**`dispatch-workflow`**: Workers run asynchronously as independent workflow runs
**`call-workflow`**: Workers run as part of the same workflow run (compile-time fan-out), preserving `github.actor` and billing attribution

### MultiRepoOps

Cross-repository operations using `target-repo` parameter and GitHub App/PAT authentication[^28]:

```yaml
safe-outputs:
  github-token: ${{ secrets.GH_AW_CROSS_REPO_PAT }}
  create-issue:
    target-repo: "org/tracking-repo"
    title-prefix: "[component-a] "
    labels: [tracking, multi-repo]
```

### Example Use Cases

- **AI-driven issue triage and labeling**
- **Automated release note generation**
- **CI failure root cause analysis** (CI Doctor)
- **Daily/weekly team status reports**
- **Documentation maintenance**
- **Test coverage improvement**
- **Dependency research and updates**
- **Code review automation**
- **Security audit workflows**
- **Onboarding and compliance automation**

---

## 14. CLI Reference (`gh aw`)

The `gh aw` CLI extension is the primary interface for managing agentic workflows[^29].

### Installation

```bash
gh extension install github/gh-aw           # Latest
gh extension install github/gh-aw@v0.1.0   # Pinned version
```

### Core Commands

| Command | Description |
|---------|-------------|
| `gh aw init` | Set up repository for agentic workflows |
| `gh aw new` | Create a workflow template |
| `gh aw add` | Add workflows from external repositories |
| `gh aw add-wizard` | Interactive guided workflow setup |
| `gh aw compile` | Convert markdown to GitHub Actions YAML |
| `gh aw validate` | Validate workflows (compile with all linters, no output) |
| `gh aw run` | Execute workflows immediately in GitHub Actions |
| `gh aw trial` | Test workflows in temporary repositories |

### Management Commands

| Command | Description |
|---------|-------------|
| `gh aw list` | Quick listing of all workflows |
| `gh aw status` | Check state of all workflows with run info |
| `gh aw enable` | Enable workflows |
| `gh aw disable` | Disable workflows (cancels in-progress runs) |
| `gh aw remove` | Remove workflows (both `.md` and `.lock.yml`) |
| `gh aw update` | Update workflows from upstream sources |
| `gh aw upgrade` | Upgrade CLI and recompile workflows |

### Observability Commands

| Command | Description |
|---------|-------------|
| `gh aw logs` | Download and analyze workflow logs |
| `gh aw audit` | Analyze specific runs with detailed metrics |
| `gh aw health` | Display workflow health metrics and success rates |

### Utility Commands

| Command | Description |
|---------|-------------|
| `gh aw fix` | Auto-fix deprecated workflow fields |
| `gh aw secrets set` | Create/update repository secrets |
| `gh aw secrets bootstrap` | Analyze workflows and prompt for missing secrets |
| `gh aw version` | Show current version |

### Key Flags

- `--push`: Auto-commit, push, and dispatch workflow
- `--create-pull-request`: Create a PR with changes
- `--strict`: Enforce strict security validation
- `--watch`: Auto-recompile on file changes
- `--json`: JSON output format
- `--label`: Filter workflows by label

---

## 15. Authentication

### Engine Authentication

Each AI engine requires a specific GitHub Actions secret[^30]:

| Engine | Secret | Source |
|--------|--------|--------|
| Copilot | `COPILOT_GITHUB_TOKEN` | Fine-grained PAT with "Copilot Requests: Read" permission |
| Claude | `ANTHROPIC_API_KEY` | Anthropic API key |
| Codex | `OPENAI_API_KEY` | OpenAI API key |
| Gemini | `GEMINI_API_KEY` | Google AI Studio API key |

### Additional Authentication Scenarios

- **Cross-repo reading**: PAT or GitHub App with access to target repos
- **Cross-repo writing**: PAT or GitHub App configured in `safe-outputs.github-token`
- **Remote GitHub MCP mode**: PAT or GitHub App
- **Lockdown mode**: Automatically enabled for public repos with custom tokens
- **GitHub Projects**: Requires additional token with Projects permission

### GitHub App Support

For enhanced security with short-lived, auto-revocable tokens[^30]:

```yaml
tools:
  github:
    github-app:
      app-id: ${{ vars.APP_ID }}
      private-key: ${{ secrets.APP_PRIVATE_KEY }}
      owner: "my-org"
      repositories: ["repo1", "repo2"]  # Optional scoping
```

### Magic Secrets

Several "magic" secret names are automatically detected without explicit frontmatter references[^30]:

- `GH_AW_GITHUB_TOKEN` — General-purpose fallback token
- `GH_AW_AGENT_TOKEN` — Fallback for assign-to-agent operations
- `GH_AW_GITHUB_MCP_SERVER_TOKEN` — GitHub MCP server authentication

---

## 16. Key Repositories Summary

| Repository | Purpose | Key Technologies |
|-----------|---------|-----------------|
| [github/gh-aw](https://github.com/github/gh-aw) | Core CLI extension and compiler | Go, GitHub Actions, MCP |
| [github/gh-aw-firewall](https://github.com/github/gh-aw-firewall) | Agent Workflow Firewall (AWF) — network egress control | Docker, iptables, Squid proxy |
| [github/gh-aw-mcpg](https://github.com/github/gh-aw-mcpg) | MCP Gateway — unified HTTP gateway for MCP servers | Docker, HTTP, MCP |
| [github/gh-aw-actions](https://github.com/github/gh-aw-actions) | Shared custom GitHub Actions for compiled workflows | GitHub Actions |
| [githubnext/agentics](https://github.com/githubnext/agentics) | Sample workflow collection (The Agentics Collection) | Markdown workflows |

### Documentation & Community

- **Official Documentation**: [github.github.io/gh-aw](https://github.github.io/gh-aw/)
- **GitHub Changelog Announcement**: [Technical Preview (Feb 2026)](https://github.blog/changelog/2026-02-13-github-agentic-workflows-are-now-in-technical-preview/)
- **GitHub Next**: [Agentic Workflows Project](https://githubnext.com/projects/agentic-workflows/)
- **Microsoft Research**: [Agentic Workflows Project](https://www.microsoft.com/en-us/research/project/agentic-workflows/)
- **Community Feedback**: [GitHub Discussions](https://github.com/orgs/community/discussions/186451)
- **Discord**: [GitHub Next Discord](https://gh.io/next-discord)

---

## 17. Confidence Assessment

### High Confidence (Directly verified from official documentation)
- Overall architecture and security model
- Frontmatter configuration fields and syntax
- Safe outputs system and all available types
- CLI commands and their flags
- Network configuration and ecosystem identifiers
- Trigger system including novel trigger types
- Import system and merging rules
- Threat detection pipeline
- Authentication requirements and setup
- Supported AI engines and their configuration

### Medium Confidence (Cross-referenced from multiple sources)
- The technical preview status and timeline
- Specific default values (verified from docs but could change)
- Runtime versions and setup actions (documented but may be updated)
- Enterprise deployment patterns (documented but not all verified in practice)

### Low Confidence / Assumptions
- Internal implementation details of the Go compiler (source code not fully explored due to repository size)
- Exact production-readiness timeline beyond "technical preview" status
- The `githubnext/agentics` sample collection could not be accessed (SAML-protected); descriptions are based on documentation references

---

## 18. Footnotes

[^1]: [GitHub Agentic Workflows Homepage](https://github.github.io/gh-aw/) — "Developed by GitHub Next and Microsoft Research, workflows run with added guardrails"
[^2]: [github/gh-aw README.md](https://github.com/github/gh-aw) — "Write agentic workflows in natural language markdown, and run them in GitHub Actions"
[^3]: [Overview Documentation](https://github.github.io/gh-aw/introduction/overview/) — "The `gh aw compile` command this markdown file into a hardened GitHub Actions Workflow .lock.yml file"
[^4]: [FAQ](https://github.github.io/gh-aw/reference/faq/) — "guardrails are foundational to the design"
[^5]: [GitHub Blog Changelog](https://github.blog/changelog/2026-02-13-github-agentic-workflows-are-now-in-technical-preview/) — Technical Preview announcement
[^6]: [Overview Documentation](https://github.github.io/gh-aw/introduction/overview/) — "Agentic workflows are AI-powered automation that can understand context, make decisions, and take meaningful actions"
[^7]: [Glossary](https://github.github.io/gh-aw/reference/glossary/) — Definitions of "agentic", XPIA, lockdown mode, etc.
[^8]: [FAQ — Determinism](https://github.github.io/gh-aw/reference/faq/#i-like-deterministic-cicd-isnt-this-non-deterministic) — "100% additive" and sanitization details
[^9]: [Security Architecture](https://github.github.io/gh-aw/introduction/architecture/) — Three-layer security model, adversary model, component overview
[^10]: [Workflow Structure](https://github.github.io/gh-aw/reference/workflow-structure/) — File organization and editing behavior
[^11]: [Frontmatter Reference](https://github.github.io/gh-aw/reference/frontmatter/) — Complete frontmatter fields including runtimes table
[^12]: [CLI Commands — Compile](https://github.github.io/gh-aw/setup/cli/#compile) — Compilation options, `--validate`, `--strict`, `--zizmor`
[^13]: [Glossary — Workflow Lock File](https://github.github.io/gh-aw/reference/glossary/#workflow-lock-file-lockyml) — Lock file definition
[^14]: [Frontmatter — Strict Mode](https://github.github.io/gh-aw/reference/frontmatter/#strict-mode-strict) — Strict mode enforcement areas
[^15]: [AI Engines Reference](https://github.github.io/gh-aw/reference/engines/) — Available engines, extended configuration, enterprise support
[^16]: [Safe Outputs Reference](https://github.github.io/gh-aw/reference/safe-outputs/) — All safe output types, workflow-id markers, custom safe output jobs
[^17]: [MultiRepoOps Pattern](https://github.github.io/gh-aw/patterns/multi-repo-ops/) — Cross-repository operations
[^18]: [Threat Detection Reference](https://github.github.io/gh-aw/reference/threat-detection/) — Detection pipeline, protected files, supply chain protection
[^19]: [Tools Reference](https://github.github.io/gh-aw/reference/tools/) — Built-in tools and custom MCP servers
[^20]: [GitHub Tools Reference](https://github.github.io/gh-aw/reference/github-tools/) — Toolsets, modes, guard policies, integrity levels
[^21]: [MCP Scripts Reference](https://github.github.io/gh-aw/reference/mcp-scripts/) — Inline tool definition in JS, shell, Python, Go
[^22]: [Sandbox Reference](https://github.github.io/gh-aw/reference/sandbox/) — MCP Gateway configuration
[^23]: [Security Architecture — AWF](https://github.github.io/gh-aw/introduction/architecture/#agent-workflow-firewall-awf) — AWF architecture and chroot mode
[^24]: [Network Permissions Reference](https://github.github.io/gh-aw/reference/network/) — Ecosystem identifiers, SSL bump, domain filtering
[^25]: [Trigger Events Reference](https://github.github.io/gh-aw/reference/triggers/) — All trigger types including slash_command, label_command, fuzzy scheduling
[^26]: [Imports Reference](https://github.github.io/gh-aw/reference/imports/) — Import system, merging rules, caching
[^27]: [Orchestration Pattern](https://github.github.io/gh-aw/patterns/orchestration/) — Orchestrator/worker pattern, dispatch vs call
[^28]: [MultiRepoOps Pattern](https://github.github.io/gh-aw/patterns/multi-repo-ops/) — Cross-repo patterns and authentication
[^29]: [CLI Reference](https://github.github.io/gh-aw/setup/cli/) — All CLI commands and flags
[^30]: [Authentication Reference](https://github.github.io/gh-aw/reference/auth/) — Engine secrets, GitHub App setup, magic secrets
