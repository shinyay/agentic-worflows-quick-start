---
description: |
  GitHub Changelog (https://github.blog/changelog/) の最新情報を取得し、
  カテゴリ別に要約して GitHub Issue として作成します。
  新機能、API変更、廃止予定などの最新情報を日本語でチームに共有するのに便利です。

on:
  schedule: weekly on monday around 9am
  workflow_dispatch:

permissions:
  contents: read
  issues: read

tools:
  web-fetch:
  github:
    toolsets: [repos, issues]

network:
  allowed:
    - defaults
    - github

safe-outputs:
  create-issue:
    title-prefix: "[changelog-jp] "
    labels: [github-changelog, weekly-summary, japanese]
    close-older-issues: true

engine: copilot
---

# GitHub Changelog 週次サマリー（日本語版）

GitHub Changelog を取得し、日本語で簡潔な週次サマリーを GitHub Issue として作成してください。

## 目的

チームが GitHub の新機能、変更点、廃止予定などの最新情報を、個々の Changelog エントリを読まずに把握できるようにします。
**すべての出力は日本語で記述してください。**

## 手順

1. GitHub Changelog ページ https://github.blog/changelog/ を取得する
2. **過去7日間**のエントリを特定する
3. 各エントリについて以下を記録する：
   - タイトルと日付
   - 変更内容の1〜2文の要約（日本語）
   - 元の Changelog エントリへのリンク
4. エントリを以下のカテゴリに分類する：
   - 🚀 **新機能** — 新しい機能やプロダクト
   - 🔄 **変更・改善** — 既存機能のアップデート
   - ⚠️ **廃止・削除** — 廃止予定または削除された機能
   - 🔒 **セキュリティ** — セキュリティ関連のアップデート
   - 📦 **API・インテグレーション** — API変更、Webhook更新、連携機能
5. サマリーを GitHub Issue として作成する

## 出力フォーマット

Issue は以下の構造に従ってください：

```
## GitHub Changelog サマリー — [日付] の週

### 🚀 新機能
- **[タイトル]**（日付）— 要約。[詳細はこちら](リンク)

### 🔄 変更・改善
- **[タイトル]**（日付）— 要約。[詳細はこちら](リンク)

### ⚠️ 廃止・削除
- **[タイトル]**（日付）— 要約。[詳細はこちら](リンク)

### 🔒 セキュリティ
- **[タイトル]**（日付）— 要約。[詳細はこちら](リンク)

### 📦 API・インテグレーション
- **[タイトル]**（日付）— 要約。[詳細はこちら](リンク)

---
> 💡 **今週の注目ポイント**: 最もインパクトのある変更を1〜3件、冒頭にピックアップしてください。
> 📋 ソース: https://github.blog/changelog/
```

## ルール

- 過去7日間のエントリのみ含めること
- エントリがないカテゴリは省略すること（空のセクションを表示しない）
- 要約は簡潔に — 各エントリ1〜2文
- 該当週にエントリがない場合は「今週の Changelog 更新はありませんでした」と記載した Issue を作成する
- 絵文字は視認性のために使用するが、プロフェッショナルなトーンを保つこと
- **すべてのテキストは日本語で記述すること**（タイトル、要約、ハイライトを含む）
