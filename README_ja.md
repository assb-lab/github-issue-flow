# GitHub Issue Flow

[English](README.md)

**Issueを起点とするAIコーディング**のための、技術スタックに依存しない小さなAgent Skillです。親GitHub Issue → GitHubのサブIssue → worktreeでの並行作業 → 異なるAIモデルによる非対話型レビュー → CI通過を条件としたスカッシュマージ、という流れで開発を進めます。

オープンな[Agent Skills CLI](https://github.com/vercel-labs/skills)を通じて、**Cursor、Codex、Claude Code**で利用できます。

## インストール

リポジトリを `assb-lab/github-issue-flow` として公開した後、任意のプロジェクトで次のコマンドを実行します。

```bash
npx skills add assb-lab/github-issue-flow --skill github-issue-flow -a cursor -a codex -a claude-code -y
```

すべてのプロジェクトで使えるようにグローバルインストールする場合は、`-g` を追加します。Node.js 22.20以上とGitが必要です。インストーラーが各エージェントの所定の場所にスキルを配置するため、スキル定義を重複して用意する必要はありません。

## 使い方

エージェントに次のように依頼します。

> github-issue-flowを使って、次の仕様を親IssueとサブIssueに分けて実装してください: ...

実装せずに計画だけを作る場合は、次のように依頼します。

> github-issue-flowを使って、GitHub Issue、サブIssue、並行実行計画の作成だけを行ってください: ...

Codexでは `$github-issue-flow` で明示的に呼び出せます。Claude CodeとCursorでは、対応している環境で `/github-issue-flow` を使えます。

## ワークフロー全体を実行するための要件

- GitHub CLI（`gh`）が認証済みで、リポジトリのIssueとPRを操作する権限があること。ネイティブの `--parent` とIssueの依存関係を利用するには、v2.94以上を推奨します。
- Gitが利用でき、ブランチとPRの作成およびマージに必要な権限があること。
- 必須の異なるAIモデルによるレビューを実行するため、Codex CLI（`codex`）とClaude Code CLI（`claude`）の両方が認証済みであること。
- 既存のCIとブランチ保護ルールに従うこと。

スキル自体は手順書のみで、追加のランタイムパッケージや特権を必要とするフックはインストールしません。必要なレビューツールを利用できない場合、エージェントは自動マージせず、作業を妨げている要因を報告する必要があります。

## ディレクトリ構成

```text
skills/
  github-issue-flow/
    SKILL.md
```

## ライセンス

MIT
