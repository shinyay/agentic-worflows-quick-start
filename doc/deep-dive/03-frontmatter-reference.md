# Frontmatter Reference

> _Based on gh-aw v0.61.0 · Last reviewed 2026-04_

This document is the comprehensive reference for every field you can put in the
YAML frontmatter of an agentic workflow. Use it as a lookup; the
[Architecture](./01-architecture-and-security.md) and
[Engines](./02-engines.md) deep-dives explain the _why_.

## TL;DR

- Frontmatter sits between two `---` lines at the top of a `.github/workflows/*.md`
  file. Everything below is the agent prompt.
- Top-level fields cover **triggers**, **identity/metadata**,
  **engine/runtimes**, **tools/MCP**, **outputs**, **threat detection**,
  **network**, **imports**, **permissions**, **features**, **checkout**, and
  **security (`strict`)**.
- `strict: true` is the **default** in v0.61.0. It enforces 7 specific rules
  (see [Strict mode](#security-strict)). It **warns** on individual ecosystem
  member domains but **allows** custom domains without warning.
- Reusable libraries are pulled in through `imports:` (with APM packages too) —
  the legacy `plugins:` field has been removed.
- `runtimes:` is an **object keyed by runtime name** — not a list.

## Key Concepts

| Term | Definition |
|------|------------|
| **Frontmatter** | YAML block at the top of an agentic workflow Markdown file, delimited by `---`. |
| **Strict mode** | Compiler mode (default `true` in v0.61.0) that refuses risky configurations. |
| **Ecosystem identifier** | A short name like `python`, `node`, `github` that expands to a curated network allowlist. Recommended over individual member domains in strict mode. |
| **Safe output** | A structured artifact the agent emits; applied by a separate gated job. |
| **Threat detection** | The AI scan run between agent and writer jobs (top-level field, parallel to `safe-outputs:`). |
| **Imports** | Reusable frontmatter / prompt fragments (and APM packages) brought in via `imports:`. |
| **APM** | Agentic Package Manager — pinned, reproducible, SHA-locked sharing of components via `imports: [{uses: shared/apm.md, with: {packages: [...]}}]`. |

## Deep Dive

### Top-Level Frontmatter Fields

| Group | Field | Type | Default | Notes |
|-------|-------|------|---------|-------|
| Trigger | `on` | object | _required_ | Standard Actions `on:` plus gh-aw extensions (see [Triggers deep-dive](./04-triggers-and-scheduling.md)) |
| Identity | `description` | string | _none_ | Human description; rendered as a comment in the generated `.lock.yml` |
| Identity | `source` | string | _none_ | `owner/repo/path@ref` — auto-populated by `gh aw add` |
| Identity | `redirect` | string | _none_ | `owner/repo/path@ref` — pointer for moved/renamed workflows |
| Identity | `private` | bool | `false` | Prevents `gh aw add` from installing this workflow into other repos |
| Identity | `resources` | list | _none_ | **Companion files** fetched alongside this workflow by `gh aw add` (NOT CPU/memory hints) |
| Identity | `labels` | list | _none_ | **Workflow categorization** used by `gh aw status --label` (NOT issue labels — those go in `safe-outputs.create-issue.labels`) |
| Identity | `metadata` | object | _none_ | Free-form key/value strings, Copilot custom-agent-spec compatible |
| Engine | `engine` | string/object | `copilot` | See [Engines](./02-engines.md) |
| Engine | `runtimes` | object | _none_ | **Object keyed by runtime name** — see [Runtimes](#runtimes) |
| Tools | `tools` | object | _curated defaults_ | Built-in tool toggles + allowlists |
| Tools | `mcp-servers` | object | _none_ | External MCP servers (Docker uses `container:`, NOT `image:`) |
| Tools | `mcp-scripts` | object | _none_ | **Inline custom MCP tools** in JS/shell — see [MCP Scripts](https://github.github.io/gh-aw/reference/mcp-scripts/). Distinct from `safe-outputs.scripts:` (which exposes scripts as safe-output writers) |
| Outputs | `safe-outputs` | object | _none_ | Declared writer outputs (see [Safe-Outputs Catalog](./06-safe-outputs-catalog.md)) |
| Outputs | `threat-detection` | bool/object | enabled when `safe-outputs:` present | Top-level field, **parallel to** `safe-outputs:` |
| Network | `network` | string/object | `defaults` | `defaults`, `{}` (no network), or `{allowed, blocked, firewall}` |
| Imports | `imports` | list | _none_ | Reusable components and APM packages |
| Permissions | `permissions` | object | `read` | Read-only scopes; write goes via `safe-outputs:` |
| Security | `strict` | bool | **`true`** | Master safety toggle |
| Features | `features` | object | _none_ | Experimental flags |
| Checkout | `checkout` | list/false | repo default | Multi-repo checkout configuration |
| Run config | `run-name` | string | workflow name | Override the `run-name:` of the generated GH Actions workflow |
| Run config | `runs-on` | string | `ubuntu-latest` | Runner label for **all** generated jobs in v0.61.0. Set to `ubuntu-24.04-arm` to run on ARM64 (see [Runner Selection](#runner-selection)) |
| Run config | `runs-on-slim` | string | _inherits `runs-on`_ | **Documented but rejected by the v0.61.0 compiler.** Intended to override the runner for framework/safe-output jobs only. Do not set on v0.61.0 |
| Run config | `timeout-minutes` | int | engine default | Maximum wall-clock for the agent job |
| Concurrency | `concurrency` | object | _none_ | Standard Actions `concurrency:` block applied to the generated workflow |

> [!IMPORTANT]
> The legacy `plugins:` field has been **removed** in v0.61.0. Use the
> Agentic Package Manager instead via `imports: [{uses: shared/apm.md, with: {packages: [...]}}]`.

### Identity & Metadata

```yaml
description: Triage incoming issues
source: githubnext/agentics/workflows/triage.md@1199e4a230756fb94a382496a73e689091aa4b6b
redirect: githubnext/agentics/workflows/triage-v2.md@main
private: true
resources:
  - triage-issue.md
  - shared/helper-action.yml
labels: [automation, triage, diagnostics]
metadata:
  owner: team-platform
  on-call: "@platform-oncall"
```

| Field | Use case |
|-------|----------|
| `description` | Surfaced in the generated lock file as a leading comment, and in dashboards. |
| `source` | `owner/repo/path@ref` — origin of the workflow when installed via `gh aw add`. |
| `redirect` | `owner/repo/path@ref` — soft-deprecate a workflow; `gh aw` follows the pointer. |
| `private` | When `true`, `gh aw add` refuses to install this workflow into other repos. |
| `resources` | Companion files `gh aw add` should also fetch into the target repo (NOT runner CPU/memory hints). |
| `labels` | Workflow-level categorization — filter with `gh aw status --label <name>`. |
| `metadata` | Free-form key/value strings. Compatible with the Copilot custom-agent-spec. |

> [!WARNING]
> `labels:` here is **workflow categorization**, not labels applied to created
> issues. To label issues created by safe outputs, use
> `safe-outputs.create-issue.labels:` instead.

### Engine & Runtimes

```yaml
engine:
  id: copilot
  version: "0.0.422"
  model: gpt-5
  max-turns: 10
  env:
    MY_VAR: ${{ vars.MY_VAR }}

runtimes:
  node:
    version: "22"
  python:
    version: "3.12"
    action-repo: "actions/setup-python"
    action-version: "v5"
```

#### `runtimes`

`runtimes:` is an **object keyed by runtime name** (NOT a list). Each entry can
specify a `version:` and optionally override the setup action with
`action-repo:` / `action-version:`.

Supported runtime keys (11 total):

`node`, `python`, `go`, `uv`, `bun`, `deno`, `ruby`, `java`, `dotnet`, `elixir`, `haskell`

```yaml
runtimes:
  node:
    version: "22"
  python:
    version: "3.12"
  go:
    version: "1.23"
  uv: {}
  bun: {}
  deno: {}
  ruby:
    version: "3.3"
  java:
    version: "21"
  dotnet:
    version: "8.0"
  elixir:
    version: "1.17"
  haskell: {}
```

See [Engines](./02-engines.md) for everything inside `engine:`.

### Tools & MCP

```yaml
tools:
  edit:
  bash: ["echo", "git status", "gh:*"]
  web-fetch:
  web-search:                       # Codex requires explicit declaration
  github:
    toolsets: [repos, issues, pull_requests]
    mode: remote
    allowed-repos: ["myorg/*"]

mcp-servers:
  slack:
    command: "npx"
    args: ["-y", "@slack/mcp-server"]
    env:
      SLACK_BOT_TOKEN: "${{ secrets.SLACK_BOT_TOKEN }}"
    allowed: ["send_message", "get_channel_history"]
  notion:
    container: "mcp/notion"          # Docker — field is `container:`, NOT `image:`
    env:
      NOTION_TOKEN: "${{ secrets.NOTION_TOKEN }}"
    allowed: ["search_pages"]
  remote-server:
    url: "https://example.com/mcp"
    headers:
      Authorization: "Bearer ${{ secrets.TOKEN }}"
```

> [!WARNING]
> Docker-backed MCP servers use the `container:` field. **Do not use `image:`** —
> it is not a valid MCP field in v0.61.0.

> [!TIP]
> Lock the agent to a specific subset of an MCP server's capabilities with
> `mcp-servers.<name>.allowed:`. This is the cheapest, most effective hardening
> you can apply.

### Outputs (`safe-outputs:` and `threat-detection:`)

```yaml
safe-outputs:
  create-issue:
    title-prefix: "[ai] "
    labels: [ai-triaged]
    max: 2
  add-comment:
    max: 5
  create-pull-request:
    max: 1
    draft: true
    protected-files: blocked

threat-detection: true               # explicit enable (default when safe-outputs exist)
# or, advanced:
threat-detection:
  enabled: true
  prompt: "Focus on SQL injection"
  engine: copilot
  steps:
    - name: Setup gateway
      run: echo "preflight"
  post-steps:
    - name: Custom scan
      run: echo "post"
```

> [!IMPORTANT]
> `threat-detection:` is a **top-level frontmatter field at the same indentation
> as `safe-outputs:`** — it is NOT nested under safe-outputs.

Each safe-output type has its own caps and validation. The
[Safe Outputs Catalog](./06-safe-outputs-catalog.md) enumerates them.

### Network

```yaml
network:
  allowed:
    - defaults
    - github                         # ecosystem identifier (recommended in strict mode)
    - python
    - "api.example.com"              # custom domain — ALLOWED in strict mode without warning
    - "https://secure.api.example.com"
    - "*.cdn.example.com"
  blocked:
    - "*.suspicious.example"
  firewall:
    ssl-bump: true                   # HTTPS deep inspection (AWF 0.9.0+)
    allow-urls:
      - "https://api.github.com/repos/*/issues"
    log-level: info
```

| Field | Behaviour |
|-------|-----------|
| `network: defaults` | Default if not specified — curated baseline allowlist. |
| `network: {}` | **No network access** (truly blocks). |
| `network.allowed` | Domain or ecosystem allowlist. Strict mode rejects bare `*`. |
| `network.blocked` | Explicit deny list, evaluated after `allowed`. |
| `network.firewall.ssl-bump` | If `true`, AWF terminates TLS to inspect URLs. |
| `network.firewall.allow-urls` | Path-aware allowlist (only meaningful with `ssl-bump: true`). |
| `network.firewall.log-level` | `debug` / `info` / `warn` / `error`. |

> [!WARNING]
> Enabling `ssl-bump` makes the AWF proxy a TLS man-in-the-middle for the
> agent's traffic. Only enable when you genuinely need URL-level filtering;
> some upstream APIs reject pinned-cert clients under SSL bump.

### Imports & APM

```yaml
imports:
  - uses: shared/component.md
    with:
      param1: value1
  - .github/workflows/shared/safety-base.md   # bare path also valid
  - uses: shared/apm.md
    with:
      packages:
        - microsoft/apm-sample-package
        - github/awesome-copilot/skills/review-and-refactor
        - microsoft/apm-sample-package#v2.0     # version-pinned (tag/branch/SHA)
```

- `imports:` merges frontmatter and prepends prompt content (BFS resolution,
  cycle detection).
- Use `${{ github.aw.import-inputs.X }}` inside a shared body to consume
  `with:` parameters.
- The Agentic Package Manager (APM) reuses `imports:` to pull SHA-pinned
  packages from `apm.lock`. The legacy `plugins:` field is gone.

### Permissions

```yaml
permissions:
  contents: read
  issues: read
  pull-requests: read
  discussions: read
  actions: read
  id-token: write          # only valid as 'write' or 'none' (OIDC)
# Shortcuts:
# permissions: read-all
# permissions: {}
```

Top-level `permissions:` becomes the **activation + agent** ceiling. In
**strict mode**, write permissions here are refused — you must use
`safe-outputs:` instead, which generates per-writer jobs with the minimum
required scope.

### Features

```yaml
features:
  action-mode: dev          # dev | release | action | script
  byok-copilot: true        # bundle BYOK behaviors
  awf-diagnostic-logs: true # AWF Docker diagnostics on failure
  integrity-reactions: true # reaction-based trust signals (v0.68.2+)
```

`features:` is a key/value map of experimental or opt-in compiler behaviours.

### Checkout

```yaml
checkout:
  - fetch-depth: 0
    fetch: ["refs/pulls/open/*"]
  - repository: org/other-repo
    path: ./libs/other
    github-token: ${{ secrets.CROSS_REPO_PAT }}
    sparse-checkout: |
      defaults/
    current: true
# Disable entirely:
# checkout: false
```

### Runner Selection

```yaml
runs-on: ubuntu-24.04-arm   # ARM64 hosted runner
# runs-on-slim: …           # NOT supported in v0.61.0 — rejected by compiler
timeout-minutes: 20
```

**v0.61.0 behavior (verified empirically).** A single top-level `runs-on:`
propagates to **all four** generated jobs (`activation`, `agent`,
threat-detection slot, `safe_outputs`), even though the upstream docs
describe `runs-on:` as agent-only and `runs-on-slim:` as covering the
framework jobs. The `runs-on-slim:` field exists in the official
frontmatter reference but is rejected by the v0.61.0 compiler with
`Unknown property: runs-on-slim`. Set only `runs-on:` on this version.

**ARM64 (`ubuntu-24.04-arm`).** Confirmed end-to-end on this repo —
runner image `Ubuntu 24.04 by Arm Limited`, kernel `aarch64`, ARM
Neoverse-N2, 4 cores, eastus2. AWF (`v0.24.2`) and the Copilot CLI both
ship ARM64 builds (`copilot-linux-arm64.tar.gz`); MCP gateway containers
(`awf-squid`, `awf-api-proxy`, `awf-agent`) are multi-arch. ARM64 is
free on public repos and **~37 % cheaper per minute** than x64 on
private repos — and that discount stacks across the four jobs every
agentic run produces.

**Per-safe-output runner.** `safe-outputs.<name>.runs-on:` overrides the
runner for an individual safe-output job. Useful when one custom-job
output needs an x64-only binary even though the rest of the workflow is
on ARM64.

### Security: `strict:`

```yaml
strict: true   # default in v0.61.0
```

Strict mode applies the following **7 enforcement areas**:

1. **Refuses write permissions** in top-level `permissions:` — forces use of
   safe outputs.
2. **Requires explicit network configuration** — no implicit "allow everything".
3. **Refuses wildcard `*`** in `network.allowed`.
4. **Recommends ecosystem identifiers** — warns on individual ecosystem member
   domains (e.g. `pypi.org`) and suggests the matching identifier (e.g.
   `python`). **Custom domains** like `api.example.com` are **allowed without
   warning**.
5. **Requires network config for custom MCP servers with containers.**
6. **Enforces SHA-pinned actions** in any injected steps.
7. **Refuses deprecated frontmatter fields** (e.g. `plugins:`).

> [!IMPORTANT]
> Strict mode does **not reject custom domains** in `network.allowed`. It only
> warns about individual ecosystem member domains, suggesting you switch to
> the ecosystem identifier. Disabling strict mode is supported but should be a
> deliberate decision logged in code review.

## Examples

### Minimal valid workflow

```yaml
---
on:
  workflow_dispatch:
engine:
  id: copilot
  version: "0.0.422"
network:
  allowed: [defaults, github]
safe-outputs:
  add-comment:
    max: 1
---

# Hello

Post a friendly comment summarising the latest commit.
```

### Strict, scheduled, with imports, APM, and runtimes

```yaml
---
on:
  schedule: daily around 5:00
  stop-after: +30m
description: Daily dependency report
strict: true
labels: [dependencies, daily]
imports:
  - uses: shared/safety-base.md
  - uses: shared/apm.md
    with:
      packages:
        - microsoft/apm-sample-package
engine:
  id: claude
  version: "2.1.70"
  model: claude-opus-4.7
  max-turns: 6
runtimes:
  python:
    version: "3.12"
network:
  allowed: [defaults, github, python, "api.anthropic.com"]
safe-outputs:
  create-issue:
    max: 1
    labels: [dependencies, ai-generated]
threat-detection: true
---
```

### Approval-gated PR workflow with custom GitHub App

```yaml
---
on:
  pull_request:
    types: [opened]
  manual-approval: production-agent
  forks: ["myorg/*"]
  github-app:
    app-id: ${{ vars.AGENT_APP_ID }}
    private-key: ${{ secrets.AGENT_APP_PRIVATE_KEY }}
strict: true
engine:
  id: copilot
  version: "0.0.422"
network:
  allowed: [defaults, github]
safe-outputs:
  add-comment:
    max: 3
  create-pull-request:
    max: 1
    draft: true
    protected-files: blocked
---
```

## Pitfalls & FAQ

> [!WARNING]
> **`permissions: contents: write` will fail to compile in strict mode.** Use
> `safe-outputs:` and let the compiler generate the writer job with scoped
> permissions.

> [!WARNING]
> **Listing every PyPI mirror in `network.allowed` will trigger a strict-mode
> warning.** Use the `python` ecosystem identifier instead. (Custom non-ecosystem
> domains are allowed without warning.)

> [!WARNING]
> **`plugins:` is gone.** Any leftover `plugins:` entries fail strict mode under
> rule 7 (deprecated frontmatter). Migrate to `imports:` + APM.

**Q: How do I disable strict mode?**
A: `strict: false` at the top level. Only do this for prototypes. Production
workflows should keep it on.

**Q: What if an `on:` event isn't supported by gh-aw?**
A: Anything Actions supports works. The gh-aw extensions (`reaction:`,
`stop-after:`, `manual-approval:`, `slash_command:`, `label_command:`, etc.)
are additive — they live alongside the standard fields.

**Q: Where do `imports:` go when conflicts happen?**
A: BFS resolution with cycle detection — last write wins for scalars; lists
are concatenated; objects are merged. Pass parameters through `with:` and
read them in shared bodies via `${{ github.aw.import-inputs.X }}`.

**Q: Can I have `safe-outputs:` without `threat-detection:`?**
A: Technically yes (`threat-detection: false`) — but you are disabling
Guardrail #5 from the [Architecture](./01-architecture-and-security.md) doc.
Don't.

**Q: What is the difference between `tools:` and `mcp-servers:`?**
A: `tools:` toggles built-in capabilities baked into gh-aw (`edit`, `bash`,
`web-fetch`, `web-search`, `github`, `playwright`, etc.). `mcp-servers:`
registers external Model Context Protocol servers that the agent talks to via
the MCP gateway. Both are top-level fields.

**Q: Are there fields not on this page?**
A: This reference covers the v0.61.0 surface. Engine-specific knobs live
inside `engine:` — see [Engines](./02-engines.md). Trigger-specific
extensions live inside `on:` — see [Triggers](./04-triggers-and-scheduling.md).

## Related Docs

- [Architecture & Security](./01-architecture-and-security.md)
- [Engines](./02-engines.md)
- [Triggers & Scheduling](./04-triggers-and-scheduling.md)
- [Safe-Outputs Catalog](./06-safe-outputs-catalog.md)
- Official: [Frontmatter reference](https://github.github.io/gh-aw/reference/frontmatter/)
- Official: [Strict mode](https://github.github.io/gh-aw/reference/strict-mode/)
- Official: [Network permissions](https://github.github.io/gh-aw/reference/network/)
- Official: [Safe outputs](https://github.github.io/gh-aw/reference/safe-outputs/)
