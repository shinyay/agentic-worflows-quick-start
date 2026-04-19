# 13 — Glossary & Concepts

> _Based on gh-aw v0.61.0 · Last reviewed 2026-04_

A reference dictionary for every term you'll encounter in GitHub Agentic Workflows. Skim once; come back when you hit an unfamiliar word.

## TL;DR

- Terms grouped by theme: **core concepts**, **architecture**, **security**, **tools/MCP**, **outputs**, **CLI/lifecycle**
- Each entry: one-line definition + ≥1 sentence of context + cross-link to the deep-dive doc that covers it in full
- If you only learn 5 terms: **agentic workflow**, **lock file**, **safe outputs**, **AWF**, **MCP**

## Key Concepts (The Five You Must Know)

| Term | One-liner |
|---|---|
| **Agentic workflow** | A markdown file with YAML frontmatter that an AI agent executes inside GitHub Actions, with security guardrails. |
| **Lock file** (`.lock.yml`) | The hardened, compiled GitHub Actions YAML that GitHub actually runs. Generated from the `.md` source by `gh aw compile`. |
| **Safe outputs** | Structured write requests an agent emits as an artifact; a separate, scoped-permission job applies them. The agent itself is read-only. |
| **AWF** (Agent Workflow Firewall) | Squid+iptables Docker sandbox that drops all egress traffic not on the configured domain allowlist. |
| **MCP** (Model Context Protocol) | The standard the agent uses to talk to its tools (GitHub API, web fetch, custom integrations, etc.). |

## Deep Dive — A to Z Glossary

### A

**Agency** — Having the ability to act independently and adapt behavior to context. The "agentic" in "agentic workflow" comes from this.

**Agent (Coding Agent / AI Engine)** — The AI system that executes natural-language instructions in a workflow. One of: GitHub Copilot CLI (default), Claude Code, OpenAI Codex, Google Gemini CLI, Crush (experimental). See [02 — Engines](./02-engines.md).

**Agent file** — A markdown file in `.github/agents/` referenced by `engine.agent:` (Copilot only). Defines a custom agent persona with description, instructions, and `disable-model-invocation:` flag. The dispatcher pattern (this repo's `agentic-workflows.agent.md`) uses these.

**Agentic workflow** — An AI-powered automation defined in markdown that reasons, decides, and acts using natural-language instructions. Contrast with deterministic GitHub Actions YAML.

**APM** (Agent Package Manager) — Mechanism for declaring agent dependencies via `dependencies:` in frontmatter.

**Artifact** — In gh-aw context, the structured `agent_output.json` the agent produces, which threat detection scans and safe-output jobs consume.

**Author** — The first phase of the workflow lifecycle: write `.md` source. Then **Compile**, **Run**, **Monitor**.

**AWF** (Agent Workflow Firewall) — The default sandbox. Three Docker containers: Squid proxy, agent, optional API proxy sidecar. Source: [github/gh-aw-firewall](https://github.com/github/gh-aw-firewall). See [07 — AWF Firewall & Sandbox](./07-awf-firewall-and-sandbox.md).

### C

**Cache memory** (`tools.cache-memory`) — Persistent key-value storage across workflow runs. Useful for trend tracking ("what was the issue count last week?"). See [05 — Tools & MCP](./05-tools-and-mcp.md).

**Chroot mode** — AWF feature allowing workflows to use host binaries with network isolation. Filesystem and network controls are separately configurable.

**`call-workflow`** — Safe-output type that triggers a reusable workflow at compile time (compile-time fan-out). Different from `dispatch-workflow` (runtime fan-out).

**Claude (Claude Code)** — Anthropic's coding agent. `engine: claude`, requires `ANTHROPIC_API_KEY`. Supports `max-turns` to limit chat iterations.

**Codex (OpenAI Codex)** — OpenAI's coding agent. `engine: codex`, requires `OPENAI_API_KEY`. `web-search` is disabled by default; explicitly enable via `tools: web-search:`.

**Compilation** — Translating `.md` workflows into `.lock.yml` GitHub Actions YAML via `gh aw compile`. Includes schema validation, import resolution, action SHA pinning, security scans (actionlint, zizmor, poutine), tool-config generation, network hardening.

**Compile-time vs runtime** — Compile-time = `gh aw compile` on your machine or in CI. Runtime = GitHub Actions executing the lock file. Most security guarantees apply at both layers.

**Continuous AI** — Marketing term for the new automation layer agentic workflows add alongside CI (Continuous Integration) and CD (Continuous Deployment). Coined by GitHub Next.

**Copilot CLI** — The default engine. `engine: copilot`, requires `COPILOT_GITHUB_TOKEN` (a fine-grained PAT with "Copilot Requests: Read" on your user account, not an org). Supports `engine.agent:` (custom agent) and `max-continuations:` (autopilot mode).

**Crush** — Experimental engine (`engine: crush`). Reuses `COPILOT_GITHUB_TOKEN`.

**Cross-Prompt Injection Attack (XPIA)** — Adversarial prompts hidden in repository data (issue bodies, PR descriptions, file contents, fetched web content) that try to hijack the agent. Mitigated by lockdown mode, min-integrity, threat detection, AWF, and read-only-default. See [08 — Threat Detection & XPIA](./08-threat-detection-and-xpia.md).

**Cross-repo operation** — Most safe outputs accept a `target-repo:` field, allowing the workflow to operate on a different repo (requires appropriate auth).

### D

**Defense-in-depth** — The architectural philosophy: stack multiple independent security controls so no single failure compromises the system. The five guardrails (read-only token, zero secrets, AWF, safe outputs, threat detection) are layered defense-in-depth.

**Dependabot bundler** — A specific workflow pattern (and Agentics workflow) that consolidates Dependabot PRs. Notable because Dependabot may modify generated manifest files (`.github/workflows/package.json`, etc.) — never merge those PRs directly; instead update the source `.md` files and run `gh aw compile --dependabot`.

**`description:`** — Frontmatter field for human-readable description; rendered as a comment in the lock file.

**Dispatcher agent** — A custom Copilot agent file with `disable-model-invocation: true` that routes user requests to one of several specialist prompts. This repo's `.github/agents/agentic-workflows.agent.md` is an example.

**`dispatch-workflow`** — Safe-output type that fires a `workflow_dispatch` for another workflow. Used for orchestrator/worker patterns. Default max: 3.

### E

**Ecosystem identifier** — A named shorthand for a predefined set of network domains. E.g., `python` = PyPI + conda + pythonhosted.org. Strict mode requires you use these instead of raw domains. See [07 — AWF Firewall & Sandbox](./07-awf-firewall-and-sandbox.md).

**Edit tool** (`tools.edit`) — Allows file editing in the GitHub Actions workspace. Required for any workflow that pushes commits or pull requests.

**Engine** (`engine:`) — The AI coding agent that runs the workflow. See **Agent**.

**Enterprise endpoint** — Custom API host for GHEC/GHES via `engine.api-target:`. Plus `OPENAI_BASE_URL` / `ANTHROPIC_BASE_URL` env vars route engines through internal LLM gateways.

### F

**Fine-grained PAT** — GitHub's modern personal access token. Required for `COPILOT_GITHUB_TOKEN` (with `Copilot Requests: Read`) and for cross-repo GitHub MCP access.

**Frontmatter** — The YAML block between `---` markers at the top of a workflow file. Contains all configuration. See [03 — Frontmatter Reference](./03-frontmatter-reference.md).

**Fuzzy schedule** — Human-friendly schedule expression (`schedule: daily`, `weekly on monday around 5pm`) where the compiler deterministically scatters the exact run time per workflow file path. Avoids load spikes when many workflows say "daily". See [04 — Triggers & Scheduling](./04-triggers-and-scheduling.md).

### G

**Gemini (Google Gemini CLI)** — Google's coding agent. `engine: gemini`, requires `GEMINI_API_KEY`.

**`gh aw`** — The GitHub CLI extension. Installed with `gh extension install github/gh-aw`. Provides ~29 commands across the author/compile/run/monitor lifecycle.

**`gh aw audit <run-id>`** — Reconstructs the full prompt, response, tool calls, safe-output artifact, and threat-detection result for a run. The single most useful debug command. See [11 — Debugging & Observability](./11-debugging-and-observability.md).

**`gh aw mcp-server`** — Subcommand that exposes `gh aw` capabilities as an MCP server. Used by VSCode (`.vscode/mcp.json`) and the Copilot coding agent (`copilot-setup-steps.yml`).

**`gh aw trial <workflow-spec>`** — Run a workflow in a sandbox-like test mode without affecting production state. Good for prompt iteration.

**GHEC / GHES** — GitHub Enterprise Cloud / Server. Both supported via `engine.api-target:`.

**GitHub MCP server** — The MCP server that exposes the GitHub API as tools. Available toolsets: `context, repos, issues, pull_requests, users, actions, code_security, discussions, labels, notifications, orgs, projects, gists, search, dependabot, experiments, secret_protection, security_advisories, stargazers`. Default: `context, repos, issues, pull_requests, users`. Shorthand `default` and `all`.

**Guardrail** — Built-in security control. The five named guardrails: read-only token, zero secrets in agent, containerized + firewalled, safe outputs, threat detection.

### I

**Imports** (`imports:`) — Frontmatter field for including shared workflow components. Paths can be relative, absolute, or cross-repo (`owner/repo/path@ref`). See [09 — Imports & Shared Components](./09-imports-and-shared-components.md).

**Import schema** (`import-schema:`) — Typed parameter contract a shared component declares. Callers pass values via `with:` (alias `inputs:`), accessed in the component via `${{ github.aw.import-inputs.<key> }}`.

**Integrity filtering** (`tools.github.min-integrity`) — Restricts which user content (issues/PRs/comments) the agent can read, by author trust level. Hierarchy (highest → lowest): `merged > approved > unapproved > none > blocked`. Auto-set to `approved` for public repos. See [17 — Min-Integrity & Content Trust](./17-min-integrity-and-content-trust.md).

### L

**Layer 1 / 2 / 3 trust** — The three trust layers in the architecture:
- **Layer 1 (Substrate)** — hardware, kernel, container runtime, AWF, API proxy, MCP Gateway
- **Layer 2 (Configuration)** — declarative configs and toolchains that interpret them; auth tokens treated as imported capabilities
- **Layer 3 (Plan)** — trusted compiler decomposes workflow into stages; SafeOutputs is the primary instantiation

See [01 — Architecture & Security](./01-architecture-and-security.md).

**Lock file** (`.lock.yml`) — Compiled output of `gh aw compile`. Hardened GitHub Actions YAML that GitHub Actions executes. Both `.md` source and `.lock.yml` should be committed. Don't edit directly.

**Lockdown mode** (`tools.github.lockdown: true`) — In public repos, filters issues/PRs/comments from untrusted authors before the agent sees them. XPIA mitigation. Default: on for public repos.

### M

**`max:`** — Per-safe-output cap. E.g., `create-issue: { max: 5 }`. Prevents runaway agents.

**`max-continuations`** — Copilot-only feature: enables autopilot mode with multiple consecutive runs per dispatch.

**`max-turns`** — Claude-only feature: limits the number of AI chat iterations per run.

**MCP** (Model Context Protocol) — Standard protocol for AI agents to securely connect to external tools. The agent uses MCP for everything (GitHub API, web fetch, file edit, Slack, etc.).

**MCP Gateway** (gh-aw-mcpg) — Transparent proxy routing all MCP server calls through a unified HTTP gateway for centralized management, logging, auth, and isolation. Spawns isolated MCP server containers (e.g., GitHub MCP server with Docker socket).

**MCP scripts** (`mcp-scripts:`) — Inline custom MCP tools defined in JS, shell, Python, or Go. Run **outside** the agent container on the GitHub Actions runner. Useful for secret-bearing operations.

**MCP server** — A program implementing MCP that provides tools to the agent. Examples: GitHub MCP, Playwright MCP, custom Slack MCP.

**`mentions:`** (`safe-outputs.mentions: false`) — Strips `@user` mentions from agent output. Prevents notification spam.

**Min-integrity** — See **Integrity filtering**.

**Monitor** — The fourth phase of the workflow lifecycle: `gh aw logs`, `gh aw audit`, `gh aw health`, `gh aw status`.

### O

**Orchestrator workflow** — A workflow that fans out to other workflows (workers) using `safe-outputs.dispatch-workflow:`, aggregates results, and posts summaries.

**`owner/repo/path@ref`** — Cross-repo reference format used for `imports:`, `source:`, `redirect:`, and the `gh aw add` command.

### P

**PAT** — Personal Access Token. Modern fine-grained PATs are preferred. Required for cross-repo GitHub tools and for `COPILOT_GITHUB_TOKEN`.

**Plan-Level Trust** — Layer 3 of the trust model. The compiler decomposes a workflow into stages; for each stage, the plan specifies active components, permissions, data produced, and how subsequent stages may consume it. SafeOutputs is the primary instantiation.

**Playwright** (`tools.playwright`) — Built-in tool for browser automation/testing. Optional `version:` field.

**Plugins** (`plugins:`) — Engine-specific plugins to install.

**Private workflow** (`private: true`) — Marks a workflow as not installable into other repos via `gh aw add`. Use for internal tooling.

**Protected files** — Supply-chain-protection mechanism preventing the agent from modifying sensitive files (dependency manifests, CI/CD config, agent instruction files). Policies: `blocked` (default), `allowed`, `fallback-to-issue`.

**Pre-activation** — The first job in a compiled workflow. Validates secrets, checks roles, runs any `on.steps:` custom steps with `on.permissions:` token, sets up the agent.

### Q

**QMD** (`tools.qmd`) — Experimental built-in tool that builds a local vector search index over docs and exposes it as an MCP search tool. Built in a dedicated indexing job (no `contents: read` needed in the agent job). Source: tobi/qmd.

### R

**Read-only token** — The agent's GitHub token has only the read permissions you declare in `permissions:`. Any write must go through a safe output.

**Reaction** (`on.reaction:`) — Trigger enhancement that adds an emoji reaction to the triggering item (issue, PR, comment). Useful UX feedback.

**`redirect:`** — Frontmatter field marking a workflow as moved/renamed. `gh aw update` follows the redirect chain (with cycle detection).

**Repo memory** (`tools.repo-memory`) — Repository-specific persistent storage for context across workflow executions. Backed by an orphan git branch.

**Resources** (`resources:`) — Companion files the workflow needs at runtime; fetched into the workspace.

**Run** — Third phase of the lifecycle: `gh aw run`, `gh aw enable`, `gh aw disable`, `gh aw trial`.

**Runtimes** (`runtimes:`) — Override default runtime versions (Node, Python, Go, etc.).

### S

**SafeOutputs subsystem** — The Layer 3 component that buffers agent write requests as an artifact, processes them through filters/sanitization, and applies them via scoped-permission jobs.

**Sanitization** — The pipeline applied to all safe-output text: secret redaction, URL domain filtering (against allowed-domains), XML escaping, size limits, control-char stripping, GitHub reference escaping, HTTPS enforcement.

**Sandbox** (`sandbox:`) — Frontmatter field for AWF/MCP-Gateway configuration overrides.

**Schedule** — See **Fuzzy schedule** and standard cron syntax with `timezone:`.

**Shared component** — A workflow file with no `on:` field. Validated but not compiled to GitHub Actions; only imported by other workflows. Conventional location: `.github/workflows/shared/`.

**Slash command** (`on.command: { name: ... }`) — Trigger fires when someone comments `/<name>` on an issue or PR. `status-comment:` is auto-enabled.

**`source:`** — Frontmatter field tracking workflow origin in `owner/repo/path@ref` format. Auto-populated by `gh aw add`.

**SSL bump** — AWF feature for HTTPS deep-packet inspection, enabling URL-path-level filtering. Configured via `network.firewall.ssl-bump: true` and `allow-urls:`.

**Status comment** (`on.status-comment:`) — Auto-posted "started/completed" comment on the triggering item with a workflow run link. Auto-on for slash/label commands.

**`stop-after:`** (in `on:`) — Disables triggers after a deadline. Useful for time-boxed trial workflows.

**Strict mode** (`strict: true`, default) — Enforces:
1. No write permissions in `permissions:`
2. Explicit network configuration
3. No `*` wildcards in `network.allowed`
4. Ecosystem identifiers, not raw domains
5. Network config required for custom MCP servers with containers
6. SHA-pinned GitHub Actions
7. No deprecated frontmatter fields

**Substrate-Level Trust** — Layer 1: hardware, kernel, container runtime, plus AWF, API proxy, and MCP Gateway privileged containers.

### T

**Threat detection** — Auto-enabled job between agent and safe-output jobs when safe-outputs are configured. AI-powered scan for prompt injection, secret leaks, malicious patches. Output: `{ "prompt_injection": bool, "secret_leak": bool, "malicious_patch": bool, "reasons": [] }`. Any `true` → workflow fails. See [08 — Threat Detection & XPIA](./08-threat-detection-and-xpia.md).

**Toolset** — Named bundle of GitHub MCP API tools. Examples: `repos`, `issues`, `pull_requests`, `actions`, `discussions`. Configured via `tools.github.toolsets:`.

**Trusted bots** (`sandbox.mcp.trusted-bots:`) — Additive list of GitHub bot identities the MCP Gateway recognizes as trusted (e.g. `github-actions[bot]`, `copilot-swe-agent[bot]`). Cannot remove built-in entries.

### W

**Web fetch** (`tools.web-fetch`) — Built-in tool for HTTP(S) GETs. Available on all engines.

**Web search** (`tools.web-search`) — Engine-dependent. Codex requires explicit declaration (`tools: web-search:`) — otherwise disabled. Other engines need a third-party MCP server.

**Worker workflow** — A workflow dispatched by an orchestrator that performs a focused unit of work.

**Workflow dispatch** (`on.workflow_dispatch:`) — Manual trigger from GitHub UI, API, or `gh aw run`/`gh aw trial`. Supports input types: `string`, `boolean`, `choice`, `environment`.

**Workflow ID marker** — Hidden HTML comment `<!-- gh-aw-workflow-id: WORKFLOW_NAME -->` in every safe-output-created item. Enables forensic tracing back to the source workflow.

### X

**XPIA** — See **Cross-Prompt Injection Attack**.

### Z

**Zero secrets in agent** — Security guardrail #2: write tokens, API keys, and other sensitive credentials never enter the agent process. They live only in isolated safe-output jobs that run after the agent finishes.

## Examples

### Decoding a frontmatter snippet

```yaml
on:
  schedule: daily around 9am utc-5
permissions:
  contents: read
tools:
  github:
    toolsets: [issues, pull_requests]
    min-integrity: approved
network:
  allowed: [defaults, github]
safe-outputs:
  create-issue:
    title-prefix: "[daily] "
    close-older-issues: true
threat-detection: true
strict: true
```

Translated to plain English:
> "Run daily at scattered times around 14:00 UTC (=09:00 EST). Read repo contents only. Use the GitHub MCP for issues + PR APIs, but only show me content from approved authors. Network is locked to basic infrastructure + github.com. After the agent runs, create one issue with `[daily]` prefix, close yesterday's. AI threat-detection runs before the issue is posted. Strict mode is on (no surprises)."

## Pitfalls & FAQ

**Q: I keep mixing up `dispatch-workflow` and `call-workflow`.**
A: `dispatch-workflow` = runtime fan-out (the agent decides at runtime to fire N other workflows). `call-workflow` = compile-time fan-out (the compiler emits a `workflow_call` that runs as part of this workflow's job graph).

**Q: Is "engine" the same as "model"?**
A: No. Engine = the CLI/agent (Copilot CLI, Claude Code, etc.). Model = the LLM behind it (gpt-5, claude-sonnet, etc.) — set via `engine.model:`.

**Q: What's the difference between `tools` and `mcp-servers`?**
A: `tools:` are first-class built-ins gh-aw knows about (edit, github, bash, web-fetch, etc.). `mcp-servers:` is the escape hatch for any other MCP server (Slack, Notion, custom integrations).

**Q: I see `${{ github.aw.import-inputs.X }}`. Is that a GitHub Actions expression?**
A: It's a gh-aw extension to GitHub Actions expression syntax. Resolved at compile time when an imported component is parameterized. Not available in a top-level workflow.

**Q: Why "fuzzy" scheduling? My boss wants exact times.**
A: Fuzziness is a feature, not a bug — it scatters load across the GitHub Actions runner pool. If you need exact, use cron syntax: `schedule: - cron: "0 9 * * *"`.

## Related Docs

- [01 — Architecture & Security](./01-architecture-and-security.md)
- [03 — Frontmatter Reference](./03-frontmatter-reference.md)
- [14 — FAQ & Troubleshooting](./14-faq-and-troubleshooting.md)
- Official [Glossary](https://github.github.io/gh-aw/reference/glossary/)
