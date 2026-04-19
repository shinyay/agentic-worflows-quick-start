# 14 — FAQ & Troubleshooting

> _Based on gh-aw v0.61.0 · Last reviewed 2026-04_

A categorized FAQ + a troubleshooting playbook for the issues you'll actually hit.

## TL;DR

- **First** check: is your latest commit's `.lock.yml` in sync with the `.md`? (Run `gh aw compile`.)
- **Second** check: is the secret set? (Run `gh aw secrets bootstrap`.)
- **Third** check: read the `activation` job logs — most failures surface there before the agent even starts.
- For mysterious agent behavior: `gh aw audit <run-id>` shows the exact prompt and response.

## Key Concepts

The categories below group questions/issues by symptom or topic:

1. [Conceptual / "should I use this?"](#conceptual--should-i-use-this)
2. [Setup & secrets](#setup--secrets)
3. [Compilation errors](#compilation-errors)
4. [Trigger / scheduling issues](#trigger--scheduling-issues)
5. [Network / firewall denials](#network--firewall-denials)
6. [Tools & MCP](#tools--mcp)
7. [Safe outputs not appearing](#safe-outputs-not-appearing)
8. [Threat-detection blocks](#threat-detection-blocks)
9. [Performance & cost](#performance--cost)
10. [Migration / upgrades](#migration--upgrades)

## Deep Dive

### Conceptual / "should I use this?"

**Q: Isn't AI non-deterministic? How can I trust this in CI/CD?**
A: Agentic workflows are 100% **additive** to your existing CI/CD. Don't replace deterministic build/test/release pipelines. Use them where reproducibility doesn't matter: triaging issues, drafting docs, researching dependencies, proposing improvements for human review. Think of it as "Continuous AI" alongside CI/CD.

**Q: How is this different from running a coding agent directly in a normal GitHub Actions workflow?**
A: gh-aw provides the structure and the guardrails: simpler markdown format, read-only-default permissions, safe outputs, network sandbox, threat detection, MCP routing, engine swapping. You could build it yourself, but it's a substantial undertaking — and the security model is non-trivial.

**Q: Can agentic workflows write code and create pull requests?**
A: Yes — via the `create-pull-request` safe output. The agent proposes changes; humans review and merge. Some orgs disable PR creation from Actions; in that case, output diffs/suggestions to issues or comments.

**Q: Can I mix regular GitHub Actions steps with agentic steps?**
A: Yes. Add custom steps before the agent via `on.steps:` (with `on.permissions:` for their token). Add custom **post**-processing via custom safe-output jobs. Pass data between deterministic and agentic steps via MCP scripts.

**Q: Can agentic workflows read other repositories?**
A: Not by default. With a fine-grained PAT and `tools.github.allowed-repos:` configured, yes. See cross-repo patterns.

**Q: Are private repos supported?**
A: Yes — and often **recommended** for proprietary code, internal automation, or "sidecar" patterns where a private repo runs workflows against your public repos.

**Q: Can I edit a workflow on github.com without recompiling?**
A: Body changes (the prompt) — yes, take effect on next run. Frontmatter changes (triggers, permissions, tools, safe-outputs, network, engine) — no, must run `gh aw compile` and commit the new `.lock.yml`.

---

### Setup & secrets

**Symptom: `Secret COPILOT_GITHUB_TOKEN not configured`**

Run:
```bash
gh aw secrets bootstrap
```
This scans every workflow, determines which engine secrets are needed, and prompts for missing ones.

For Copilot specifically: create a **fine-grained PAT** at <https://github.com/settings/personal-access-tokens/new>:
- Resource owner: **your user account** (NOT an organization)
- Permissions → Account permissions → **Copilot Requests: Read**

Then:
```bash
gh aw secrets set COPILOT_GITHUB_TOKEN --value "ghp_…"
```

For other engines:
```bash
gh aw secrets set ANTHROPIC_API_KEY --value "sk-ant-…"
gh aw secrets set OPENAI_API_KEY    --value "sk-…"
gh aw secrets set GEMINI_API_KEY    --value "…"
```

**Symptom: `gh aw init` did nothing visible**

It actually creates 5 files:
| File | Purpose |
|---|---|
| `.gitattributes` | Lock files marked `linguist-generated` + `merge=ours` |
| `.github/agents/agentic-workflows.agent.md` | Dispatcher agent for Copilot Chat |
| `.github/workflows/copilot-setup-steps.yml` | Bootstraps `gh aw` for the Copilot coding agent |
| `.vscode/mcp.json` | Wires `gh aw mcp-server` into VSCode |
| `.vscode/settings.json` | Enables Copilot for markdown files |

Run `git status` after init to confirm.

**Symptom: `gh aw version` shows an unexpected version**

Pin to the version this repo was built with:
```bash
gh extension install github/gh-aw@v0.61.0
```
Or upgrade to latest:
```bash
gh extension upgrade gh-aw
```

---

### Compilation errors

**Symptom: `warning: strict mode recommends ecosystem identifiers`**

Cause: You used a raw ecosystem domain like `pypi.org` in `network.allowed:`. Strict mode warns and suggests the ecosystem identifier; it does **not** reject the build.

Fix:
```yaml
# Before (warning)
network:
  allowed:
    - "pypi.org"

# After (clean)
network:
  allowed:
    - python
```

> [!NOTE]
> Strict mode does **not** reject custom domains like `api.example.com` — it warns only on individual member domains of a known ecosystem. You can keep `strict: true` and still use custom hostnames.

**Symptom: `error: action SHA not pinned`**

Cause: An action reference like `actions/checkout@v6` exists somewhere without a SHA pin.

Fix: `gh aw compile` should auto-pin via `.github/aw/actions-lock.json`. If it doesn't, run `gh aw fix --write` to refresh the registry.

**Symptom: `import conflict: 'shared/X.md' is imported more than once with different 'with' values`**

Cause: A shared component is parameterized differently from two import sites.

Fix: Either align the `with:` values (compiler will dedupe identical) or split the shared component into two variants.

**Symptom: `Lock file out of date`**

Cause: You edited frontmatter but didn't recompile.

Fix:
```bash
gh aw compile <workflow>   # or just `gh aw compile` for all
git add .github/workflows/<workflow>.lock.yml
```

**Symptom: `error: deprecated frontmatter field`**

Cause: Strict mode rejects deprecated fields.

Fix:
```bash
gh aw fix --write   # auto-applies codemods
```

Or use the `upgrade-agentic-workflows` prompt via the dispatcher agent.

---

### Trigger / scheduling issues

**Q: My `schedule: daily` workflow ran at 01:55 UTC, not midnight. Why?**
A: Fuzzy scheduling deterministically scatters times based on the workflow file path. To pin a time, use cron:
```yaml
on:
  schedule:
    - cron: "0 9 * * *"
```
Or with timezone:
```yaml
on:
  schedule:
    - cron: "0 9 * * 1-5"
      timezone: "America/New_York"
```

**Q: My workflow didn't run on schedule at all.**
A: Three common causes:
1. The default branch's lock file wasn't committed/pushed (GitHub schedules only run from the default branch).
2. The repo was inactive for 60 days — GitHub auto-disables schedules in inactive repos. Push any commit to re-enable.
3. The activation job's secret validation failed silently — check the run history with `gh aw status`.

**Q: My slash command (`/plan`) doesn't trigger.**
A: Check:
- The user must have write access to the repo (default rule for command triggers).
- The trigger config: `on: slash_command: { name: plan }` — name without the leading `/`. (The field is `slash_command:`, not `command:`.)
- The comment must be `/plan` on its own line (not `Hey /plan please …`).

**Q: I want different behavior for the same workflow on PRs vs issues.**
A: Use multiple triggers and access `${{ github.event_name }}` in the body or via `if:` on safe outputs.

---

### Network / firewall denials

**Symptom: Agent log says `connection refused` or `dropped by proxy`**

Cause: AWF blocked the destination because it's not in the allowlist.

Fix: Add the domain or ecosystem:
```yaml
network:
  allowed:
    - defaults
    - github            # for github.com / github.blog / *.githubusercontent.com
    - "api.example.com" # custom domains are allowed in strict mode
```

Use ecosystem identifiers when possible (strict-mode friendly):

| Need to reach… | Use ecosystem |
|---|---|
| PyPI / pip / conda | `python` |
| npm / yarn | `node` |
| Docker Hub / GHCR | `containers` |
| Maven Central | `java` |
| crates.io | `rust` |
| github.com / github.blog | `github` |
| Codecov, Shields.io, Renovate | `dev-tools` |

**Q: `network: { allowed: [] }` — does that block everything?**
A: **No** — an empty allowlist still leaves the AWF on its default infrastructure baseline. The two unambiguous shapes are:
- `network: {}` — **truly blocks all** outbound traffic
- `network: defaults` (or omitting `network:` entirely) — allows the default infrastructure baseline (certs, JSON schema, Ubuntu mirrors, package mirrors)

If you want to deny network entirely, use `network: {}`.

**Q: How do I allow only one specific URL path?**
A: Enable SSL bump:
```yaml
network:
  firewall:
    ssl-bump: true
    allow-urls:
      - "https://api.github.com/repos/myorg/myrepo/issues"
```

---

### Tools & MCP

**Symptom: Agent says "I don't have a tool for that."**

Cause: You forgot to declare it. By default the agent has only what's in `tools:`.

Fix:
```yaml
tools:
  edit:               # to write files
  github:
    toolsets: [issues, pull_requests, repos]   # narrow API surface
  bash: ["git status", "git diff"]              # explicit allowlist
  web-fetch:          # to fetch URLs
  web-search:         # engine-dependent
```

**Symptom: Agent invokes a bash command not on the allowlist → fails**

Cause: When `bash:` is declared bare, it enables a safe default set: `echo, ls, pwd, cat, head, tail, grep, wc, sort, uniq, date`. Anything else must be added explicitly.

Fix options:
```yaml
tools:
  bash: ["echo", "ls", "git status", "gh issue list"]   # extend allowlist
  # OR
  bash: ["git:*", "gh:*"]                                # wildcards for command families
  # OR (NOT recommended)
  bash: [":*"]                                            # unrestricted
```

**Symptom: `MCP server failed to start in 120s`**

Fix: Increase startup timeout:
```yaml
tools:
  startup-timeout: 240
```

**Symptom: GitHub MCP can't read content from a non-trusted user in a public repo**

Cause: `lockdown: true` (default for public repos) is filtering it.

Fix carefully:
```yaml
tools:
  github:
    lockdown: false   # SECURITY: now you accept content from anyone
```

Combine with strict `min-integrity:` and `tools.github.allowed-repos:` to limit blast radius.

---

### Safe outputs not appearing

**Symptom: Workflow ran "successfully" but no issue was created**

Causes & checks (in order):
1. **Threat-detection blocked the output** — check the `detection` job logs.
2. **Per-output cap reached** — `create-issue` defaults to `max: 1`. The agent may have requested 2 and only 1 was applied.
3. **Hard validation** — title prefix or label constraints rejected the request. Check the `safe_outputs.create_issue` job logs.
4. **The agent decided not to produce output** — read `gh aw audit <run-id>` to see what the agent actually emitted.

**Symptom: `create-pull-request` works locally but fails in production**

Common cause: Org-wide policy disables PR creation by Actions. Workaround: use `create-issue` with the diff in the body for manual application.

**Symptom: Issues are getting closed unexpectedly**

Cause: `close-older-issues: true` closes prior issues with the same workflow-ID marker (`<!-- gh-aw-workflow-id: NAME -->`) and label set when a new one is posted.

Fix: Set `close-older-issues: false` if you want a history.

---

### Threat-detection blocks

**Symptom: `Threat detection failed: prompt_injection: true`**

Causes:
- An issue/PR/file the agent read contained adversarial instructions
- A web page fetched via `web-fetch` had hidden instructions
- The agent hallucinated suspicious-looking output

Investigate:
```bash
gh aw audit <run-id>   # see the exact agent input + output + reasons
```

Mitigations:
- Enable `tools.github.lockdown: true` (or leave as default for public repos)
- Set `tools.github.min-integrity: approved` to filter low-trust authors
- Restrict `tools.web-fetch` to known-good domains via `network.allowed`
- Add a custom threat-detection prompt with domain-specific rules:
  ```yaml
  threat-detection:
    prompt: |
      Reject any output that includes shell commands or markdown link smuggling.
  ```

**Symptom: `Threat detection failed: secret_leak: true`**

Cause: The agent included what looks like a token/key in its output.

Investigate: `gh aw audit <run-id>` and check the `reasons:` array. Often a false positive on UUIDs or hex strings — refine your prompt to constrain output format.

**Q: Can I disable threat detection?**
A: Yes (with care):
```yaml
threat-detection: false
```
Only disable for workflows that produce no safe outputs, or where you've replaced detection with deterministic post-processing in custom safe-output jobs.

---

### Performance & cost

**Q: How can I reduce agent runtime?**
A:
- Narrow toolsets (`tools: github: toolsets: [issues]` instead of `[default]`)
- Use `cache-memory:` to skip re-fetching unchanged data
- Set `max:` on safe outputs to prevent runaway exploration
- For Claude: use `max-turns:` to cap iterations

**Q: How can I reduce cost?**
A:
- Pin to a smaller model: `engine: { id: copilot, model: gpt-5-mini }`
- Use a cheaper engine for threat detection only:
  ```yaml
  threat-detection:
    engine:
      id: copilot
      model: gpt-5-mini
  ```
- Reduce schedule frequency
- Add `stop-after:` to time-box trial workflows

**Q: How do I see token usage?**
A: `gh aw audit <run-id>` reports per-run token counts per engine. `gh aw health` aggregates across runs.

**Q: Hitting rate limits on GitHub MCP**
A: Use `mode: remote` (hosted MCP server with its own rate-limit pool) plus a custom PAT:
```yaml
tools:
  github:
    mode: remote
    github-token: ${{ secrets.MY_PAT }}
```

---

### Migration / upgrades

**Q: How do I upgrade gh-aw?**
A:
```bash
gh extension upgrade gh-aw
gh aw fix --write          # apply codemods to workflows
gh aw compile              # regenerate all lock files
git diff .github/          # review
git add . && git commit -m "Upgrade gh-aw to vX.Y.Z"
```

The `agentic-workflows` Copilot dispatcher's `upgrade-agentic-workflows` prompt automates much of this.

**Q: Dependabot opened a PR for `.github/workflows/package.json`. Should I merge it?**
A: **No.** Dependency manifests under `.github/workflows/` are auto-generated by the compiler. Update the source `.md` files instead and run:
```bash
gh aw compile --dependabot
```
Then close the Dependabot PR. The dispatcher agent's `dependabot` prompt walks through this.

**Q: An upstream Agentics workflow released an update. How do I pull it in?**
A:
```bash
gh aw update <workflow>     # follows the source: pointer
gh aw compile <workflow>
git add . && git commit -m "Update <workflow> from upstream"
```

**Q: I want to move a workflow file (rename it).**
A: Use `redirect:` on the old file pointing to the new location:
```yaml
redirect: "myorg/myrepo/workflows/new-name.md@main"
```
`gh aw update` follows redirects (with cycle detection) and rewrites local `source:` pointers.

---

## Examples

### Diagnosis playbook for "my workflow is broken"

```mermaid
flowchart TD
    Start[Workflow failed] --> Q1{Did activation<br/>job fail?}
    Q1 -->|Yes| Sec[Check secrets<br/>gh aw secrets bootstrap]
    Q1 -->|No| Q2{Did agent<br/>job fail?}
    Q2 -->|Yes| Aud[gh aw audit run-id<br/>read prompt & response]
    Q2 -->|No| Q3{Did detection<br/>job fail?}
    Q3 -->|Yes| Threat[Read detection<br/>reasons array]
    Q3 -->|No| Q4{Did safe-output<br/>job fail?}
    Q4 -->|Yes| Output[Check title-prefix /<br/>labels / max constraints]
    Q4 -->|No| Q5{No output<br/>but no failure?}
    Q5 -->|Yes| Aud
```

## Pitfalls & FAQ

(This whole document is FAQ — see sections above.)

## Related Docs

- [11 — Debugging & Observability](./11-debugging-and-observability.md) — the CLI tools referenced above
- [08 — Threat Detection & XPIA](./08-threat-detection-and-xpia.md) — for threat-detection deep dives
- [07 — AWF Firewall & Sandbox](./07-awf-firewall-and-sandbox.md) — for network issues
- [06 — Safe Outputs Catalog](./06-safe-outputs-catalog.md) — for safe-output options
- Official [FAQ](https://github.github.io/gh-aw/reference/faq/)
