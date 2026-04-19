# Frontmatter リファレンス

> _gh-aw v0.61.0 を基準 · 最終レビュー 2026-04_

このドキュメントは、エージェントワークフローの YAML frontmatter に書ける
すべてのフィールドの包括的リファレンスです。検索用途で参照してください。
_なぜ_ そうなっているかは [Architecture](./01-architecture-and-security.ja.md)
と [Engines](./02-engines.ja.md) の詳解を参照してください。

## TL;DR

- frontmatter は `.github/workflows/*.md` ファイルの先頭にある 2 本の `---` の
  間に置かれます。それより下はエージェントのプロンプトです。
- トップレベルのフィールドは **トリガー**、**識別情報/メタデータ**、
  **エンジン/ランタイム**、**ツール/MCP**、**出力**、**脅威検知**、
  **ネットワーク**、**インポート**、**権限**、**機能フラグ**、
  **チェックアウト**、**セキュリティ (`strict`)** をカバーします。
- `strict: true` は v0.61.0 では **デフォルト** で、7 つの具体的なルールを
  強制します（[Strict モード](#セキュリティ-strict)参照）。
  個別のエコシステムメンバードメインには **警告** を出しますが、カスタム
  ドメインは **警告なしで許可** します。
- 再利用可能なライブラリは `imports:`（APM パッケージも同様）で取り込みます。
  レガシーの `plugins:` フィールドは廃止されました。
- `runtimes:` は **ランタイム名をキーとするオブジェクト** であり、リスト
  ではありません。

## 主要な概念

| 用語 | 定義 |
|------|------|
| **Frontmatter** | エージェントワークフロー Markdown ファイルの先頭にある YAML ブロック。`---` で区切られます。 |
| **Strict モード** | リスクのある設定を拒否するコンパイラモード（v0.61.0 デフォルト `true`）。 |
| **エコシステム識別子** | `python`、`node`、`github` のような短い名前で、キュレーション済みのネットワーク allowlist に展開される。strict モードで個別メンバードメインより推奨。 |
| **Safe output** | エージェントが出力する構造化アーティファクト。別のゲート付きジョブが適用する。 |
| **脅威検知** | エージェントジョブとライタージョブの間で実行される AI スキャン（`safe-outputs:` と並列のトップレベルフィールド）。 |
| **Imports** | `imports:` で取り込む再利用可能な frontmatter / プロンプト断片（および APM パッケージ）。 |
| **APM** | Agentic Package Manager — `imports: [{uses: shared/apm.md, with: {packages: [...]}}]` を介して、ピン留め済み・再現可能・SHA ロックされたコンポーネントを共有する仕組み。 |

## 詳解

### トップレベルの Frontmatter フィールド

| グループ | フィールド | 型 | デフォルト | 注意 |
|----------|------------|----|------------|------|
| Trigger | `on` | object | _必須_ | 標準 Actions の `on:` に gh-aw 拡張を追加（[Triggers 詳解](./04-triggers-and-scheduling.ja.md)参照） |
| Identity | `description` | string | _なし_ | 人間向け説明。生成された `.lock.yml` 先頭にコメントとして描画される |
| Identity | `source` | string | _なし_ | `owner/repo/path@ref` — `gh aw add` が自動設定 |
| Identity | `redirect` | string | _なし_ | `owner/repo/path@ref` — 移動・改名されたワークフローへのポインタ |
| Identity | `private` | bool | `false` | このワークフローを `gh aw add` で他リポジトリへインストールさせない |
| Identity | `resources` | list | _なし_ | `gh aw add` が一緒に取得する **コンパニオンファイル**（CPU/メモリヒントではない） |
| Identity | `labels` | list | _なし_ | **ワークフローの分類**（`gh aw status --label` が使用、issue ラベルではない — それは `safe-outputs.create-issue.labels` ） |
| Identity | `metadata` | object | _なし_ | 自由形式のキー/値文字列、Copilot custom-agent-spec 互換 |
| Engine | `engine` | string/object | `copilot` | [Engines](./02-engines.ja.md) 参照 |
| Engine | `runtimes` | object | _なし_ | **ランタイム名をキーとするオブジェクト** — [Runtimes](#runtimes) 参照 |
| Tools | `tools` | object | _キュレート済みデフォルト_ | 組み込みツールトグル + allowlist |
| Tools | `mcp-servers` | object | _なし_ | 外部 MCP サーバ（Docker は `container:` を使用、`image:` ではない） |
| Tools | `mcp-scripts` | object | _なし_ | **インラインのカスタム MCP ツール**（JS/シェル）— [MCP Scripts](https://github.github.io/gh-aw/reference/mcp-scripts/) 参照。`safe-outputs.scripts:`（safe-output 化）とは別物 |
| Outputs | `safe-outputs` | object | _なし_ | 宣言されたライター出力（[Safe-Outputs カタログ](./06-safe-outputs-catalog.ja.md)参照） |
| Outputs | `threat-detection` | bool/object | `safe-outputs:` がある場合は有効 | トップレベルフィールドで `safe-outputs:` と **並列** |
| Network | `network` | string/object | `defaults` | `defaults`、`{}`（ネットワーク無効）、または `{allowed, blocked, firewall}` |
| Imports | `imports` | list | _なし_ | 再利用可能コンポーネントと APM パッケージ |
| Permissions | `permissions` | object | `read` | 読み取り専用スコープ。書き込みは `safe-outputs:` 経由 |
| Security | `strict` | bool | **`true`** | マスター安全トグル |
| Features | `features` | object | _なし_ | 実験的フラグ |
| Checkout | `checkout` | list/false | リポジトリのデフォルト | マルチリポジトリ チェックアウト設定 |
| Run config | `run-name` | string | ワークフロー名 | 生成される GH Actions ワークフローの `run-name:` を上書き |
| Run config | `runs-on` | string | `ubuntu-latest` | v0.61.0 では生成される **すべての** ジョブで共通のランナーラベル。`ubuntu-24.04-arm` を指定すると ARM64 で実行（[ランナー選択](#runner-selection)参照） |
| Run config | `runs-on-slim` | string | _`runs-on` を継承_ | **公式ドキュメントには記載されているが v0.61.0 のコンパイラには拒否される**。フレームワーク/safe-output ジョブのみのランナー上書きを意図した項目。v0.61.0 では設定しないこと |
| Run config | `timeout-minutes` | int | エンジン依存 | エージェントジョブの最大実時間 |
| Concurrency | `concurrency` | object | _なし_ | 標準 Actions の `concurrency:` を生成ワークフローへ反映 |

> [!IMPORTANT]
> レガシーの `plugins:` フィールドは v0.61.0 で **廃止** されました。代わりに
> Agentic Package Manager を `imports: [{uses: shared/apm.md, with: {packages: [...]}}]`
> として使ってください。

### 識別情報とメタデータ

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

| フィールド | 用途 |
|------------|------|
| `description` | 生成されたロックファイルの先頭コメントやダッシュボードに表示される。 |
| `source` | `owner/repo/path@ref` — `gh aw add` でインストールされた際のワークフローの出所。 |
| `redirect` | `owner/repo/path@ref` — ワークフローを soft-deprecate。`gh aw` がポインタに従う。 |
| `private` | `true` のとき、`gh aw add` は他リポジトリへのインストールを拒否する。 |
| `resources` | `gh aw add` が対象リポジトリへ一緒に取得すべきコンパニオンファイル（ランナー CPU/メモリヒントではない）。 |
| `labels` | ワークフローレベルの分類 — `gh aw status --label <name>` でフィルタ。 |
| `metadata` | 自由形式のキー/値文字列。Copilot custom-agent-spec と互換。 |

> [!WARNING]
> ここでの `labels:` は **ワークフローの分類** であって、作成された issue に
> 付与されるラベルではありません。safe outputs が作成する issue にラベルを
> 付けるには `safe-outputs.create-issue.labels:` を使ってください。

### エンジンとランタイム

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

`runtimes:` は **ランタイム名をキーとするオブジェクト** です（リストでは
ありません）。各エントリは `version:` を指定でき、`action-repo:` /
`action-version:` でセットアップアクションを上書きできます。

サポートされているランタイムキー（11 個）:

`node`、`python`、`go`、`uv`、`bun`、`deno`、`ruby`、`java`、`dotnet`、`elixir`、`haskell`

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

`engine:` の中身については [Engines](./02-engines.ja.md) を参照してください。

### ツールと MCP

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
> Docker ベースの MCP サーバは `container:` フィールドを使います。
> **`image:` は使わないでください** — v0.61.0 では有効な MCP フィールドでは
> ありません。

> [!TIP]
> `mcp-servers.<name>.allowed:` でエージェントを MCP サーバの特定サブセットに
> ロックダウンしてください。最も安価で効果的なハードニングです。

### 出力 (`safe-outputs:` と `threat-detection:`)

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
> `threat-detection:` は **`safe-outputs:` と同じインデントのトップレベル
> frontmatter フィールド** です — safe-outputs の中にネストされた
> ものではありません。

各 safe-output タイプには独自の上限と検証があります。
[Safe Outputs Catalog](./06-safe-outputs-catalog.ja.md) を参照してください。

### ネットワーク

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

| フィールド | 挙動 |
|------------|------|
| `network: defaults` | 未指定時のデフォルト — キュレート済みのベースライン allowlist。 |
| `network: {}` | **ネットワークアクセスなし**（完全にブロック）。 |
| `network.allowed` | ドメインまたはエコシステム allowlist。strict モードは裸の `*` を拒否。 |
| `network.blocked` | 明示的な deny リスト。`allowed` の後に評価。 |
| `network.firewall.ssl-bump` | `true` なら AWF が TLS を終端し URL を検査。 |
| `network.firewall.allow-urls` | パス対応 allowlist (`ssl-bump: true` 時のみ意味あり)。 |
| `network.firewall.log-level` | `debug` / `info` / `warn` / `error`。 |

> [!WARNING]
> `ssl-bump` を有効にすると AWF プロキシがエージェントの通信に対する TLS
> man-in-the-middle になります。本当に URL レベルのフィルタリングが必要な
> ときだけ有効にしてください。一部の上流 API は SSL bump 配下のピン留め
> 証明書クライアントを拒否します。

### Imports と APM

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

- `imports:` は frontmatter をマージしプロンプト内容を先頭に追加します
  （BFS による解決、循環検出あり）。
- 共有本文の中で `with:` パラメータを参照するには
  `${{ github.aw.import-inputs.X }}` を使います。
- Agentic Package Manager (APM) は `imports:` を再利用して `apm.lock` から
  SHA ピン留めパッケージを取り込みます。レガシーの `plugins:` フィールドは
  もうありません。

### 権限

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

トップレベルの `permissions:` は **activation + agent** の上限になります。
**strict モード** ではここでの書き込み権限は拒否されます。代わりに
`safe-outputs:` を使ってください。コンパイラが必要最小限のスコープでライター
ジョブごとに権限を設定します。

### Features

```yaml
features:
  action-mode: dev          # dev | release | action | script
  byok-copilot: true        # bundle BYOK behaviors
  awf-diagnostic-logs: true # AWF Docker diagnostics on failure
  integrity-reactions: true # reaction-based trust signals (v0.68.2+)
```

`features:` は実験的またはオプトインのコンパイラ動作を切り替える
キー/値マップです。

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
# 完全に無効化:
# checkout: false
```

### ランナー選択 {#runner-selection}

```yaml
runs-on: ubuntu-24.04-arm   # ARM64 ホストランナー
# runs-on-slim: …           # v0.61.0 では未対応 — コンパイラに拒否される
timeout-minutes: 20
```

**v0.61.0 で実測確認した挙動。** トップレベルの `runs-on:` ひとつで、生成
される **4 つすべて** のジョブ（`activation`、`agent`、threat-detection
スロット、`safe_outputs`）に伝播します。公式ドキュメントは `runs-on:` を
エージェントジョブのみ、`runs-on-slim:` をフレームワークジョブ向けと説明
していますが、現行 v0.61.0 では `runs-on-slim:` は
`Unknown property: runs-on-slim` で拒否されます。このバージョンでは
`runs-on:` のみを設定してください。

**ARM64 (`ubuntu-24.04-arm`)。** 本リポジトリで End-to-End で確認済み —
ランナーイメージ `Ubuntu 24.04 by Arm Limited`、カーネル `aarch64`、
ARM Neoverse-N2 / 4 コア、リージョン eastus2。AWF（`v0.24.2`）と
Copilot CLI はいずれも ARM64 ビルド（`copilot-linux-arm64.tar.gz`）を提供
しており、MCP ゲートウェイ用コンテナ（`awf-squid`、`awf-api-proxy`、
`awf-agent`）もマルチアーキです。ARM64 はパブリックリポジトリでは無料、
プライベートリポジトリでは x64 比 **約 37% 安い** 単価で、エージェント
実行 1 回あたり 4 ジョブ発火するためコスト差が積み上がります。

**safe-output 単位での上書き。** `safe-outputs.<name>.runs-on:` で個別
の safe-output ジョブのランナーを上書きできます。ワークフロー全体は
ARM64 にしつつ、特定のカスタムジョブだけ x64 専用バイナリのために
`ubuntu-latest` に逃がす、といった構成に有効です。

### セキュリティ: `strict:`

```yaml
strict: true   # default in v0.61.0
```

strict モードは以下の **7 つの強制エリア** を適用します。

1. トップレベル `permissions:` での **書き込み権限を拒否** — safe outputs の
   利用を強制。
2. **明示的なネットワーク設定を要求** — 暗黙の「全部許可」はなし。
3. `network.allowed` 内の **ワイルドカード `*` を拒否**。
4. **エコシステム識別子を推奨** — 個別のエコシステムメンバードメイン
   （例: `pypi.org`）に対して警告を出し、対応する識別子（例: `python`）の
   利用を提案します。`api.example.com` のような **カスタムドメイン** は
   **警告なしで許可** されます。
5. **コンテナを伴うカスタム MCP サーバにはネットワーク設定を要求**。
6. 注入されたステップで **GitHub Actions の SHA ピン留めを強制**。
7. **deprecated な frontmatter フィールドを拒否**（例: `plugins:`）。

> [!IMPORTANT]
> strict モードは `network.allowed` 内の **カスタムドメインを拒否しません**。
> 個別エコシステムメンバードメインに対してのみ警告を出し、エコシステム
> 識別子への切り替えを提案します。strict モードを無効化することはサポート
> されていますが、コードレビューに記録される意図的な決断であるべきです。

## 例

### 最小の有効ワークフロー

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

### strict、スケジュール、imports・APM・runtimes 付き

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

### カスタム GitHub App による承認ゲート付き PR ワークフロー

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

## 落とし穴と FAQ

> [!WARNING]
> **`permissions: contents: write` は strict モードでコンパイル失敗します。**
> 代わりに `safe-outputs:` を使い、コンパイラにスコープ付き権限を持つライター
> ジョブを生成させてください。

> [!WARNING]
> **`network.allowed` にすべての PyPI ミラーを並べると strict モードで警告が
> 出ます。** 代わりに `python` エコシステム識別子を使ってください
> （エコシステム外のカスタムドメインは警告なしで許可されます）。

> [!WARNING]
> **`plugins:` は廃止されました。** 残っている `plugins:` エントリは
> ルール 7（deprecated frontmatter）で strict モードに失敗します。
> `imports:` + APM に移行してください。

**Q: strict モードを無効化するには?**
A: トップレベルで `strict: false`。プロトタイプ用途に限定してください。本番
ワークフローでは有効のままにします。

**Q: `on:` のイベントが gh-aw でサポートされていない場合は?**
A: Actions がサポートするものは何でも動きます。gh-aw 拡張（`reaction:`、
`stop-after:`、`manual-approval:`、`slash_command:`、`label_command:` など）
は加算的で、標準フィールドと共存します。

**Q: `imports:` の競合解決は?**
A: BFS による解決と循環検出 — スカラーは last write wins、リストは連結、
オブジェクトはマージされます。`with:` でパラメータを渡し、共有本文では
`${{ github.aw.import-inputs.X }}` で読み出してください。

**Q: `safe-outputs:` を `threat-detection:` なしで使えますか?**
A: 技術的には可能（`threat-detection: false`）ですが、
[Architecture](./01-architecture-and-security.ja.md) のガードレール #5 を
無効化することになります。やめてください。

**Q: `tools:` と `mcp-servers:` の違いは?**
A: `tools:` は gh-aw に組み込まれた能力（`edit`、`bash`、`web-fetch`、
`web-search`、`github`、`playwright` など）をトグルします。`mcp-servers:`
は MCP ゲートウェイ経由でエージェントが対話する外部 Model Context Protocol
サーバを登録します。両方ともトップレベルフィールドです。

**Q: このページにないフィールドはありますか?**
A: このリファレンスは v0.61.0 の表面をカバーします。エンジン固有のノブは
`engine:` の内側にあります — [Engines](./02-engines.ja.md) を参照してください。
トリガー固有の拡張は `on:` の中にあります — [Triggers](./04-triggers-and-scheduling.ja.md)
を参照してください。

## 関連ドキュメント

- [Architecture & Security](./01-architecture-and-security.ja.md)
- [Engines](./02-engines.ja.md)
- [Triggers & Scheduling](./04-triggers-and-scheduling.ja.md)
- [Safe-Outputs カタログ](./06-safe-outputs-catalog.ja.md)
- 公式: [Frontmatter reference](https://github.github.io/gh-aw/reference/frontmatter/)
- 公式: [Strict mode](https://github.github.io/gh-aw/reference/strict-mode/)
- 公式: [Network permissions](https://github.github.io/gh-aw/reference/network/)
- 公式: [Safe outputs](https://github.github.io/gh-aw/reference/safe-outputs/)
