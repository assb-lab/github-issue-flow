# GitHub Issue Flow

[English](README.md)

**Issueを起点とするAIコーディング**のための、技術スタックに依存しないAgent Skillです。Epic → **細粒度のGitHubネイティブSub-issue** → 依存関係・PRスタック計画 → 独立worktreeでの並列実装 → **stacked PR** → 異なるAIモデルによる非対話型レビュー → CI通過後に下から順にsquash merge・restack、という流れで開発を進めます。

オープンな[Agent Skills CLI](https://github.com/vercel-labs/skills)を通じて、**Cursor、Codex、Claude Code**で利用できます。

## ワークフローの特徴

**Sub-issueの粒度とPRの粒度は別です。** 各Sub-issueは独立して完了判定できる小さな実装成果に分けつつ、関連する複数のSub-issueを1本の機能別PRにまとめます。依存するPRは、前のPRのマージを待たずにその機能ブランチをbaseとして積みます。各PRの差分はそのPRの直接のbaseと比較してレビューし、squash merge後は後続PRを安全にrebase・retargetします。

参考事例は[sekigae-app](https://github.com/ks250206/sekigae-app)です。**13 Epic・146個の実Sub-issue・11本のstacked PR** に分け、[PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163) のように積んでいます。これは**目安となる構造であり、小さな案件でも同じ数を強制するものではありません**。

## インストール

リポジトリを `assb-lab/github-issue-flow` として公開した後、任意のプロジェクトで次のコマンドを実行します。

```bash
npx skills add assb-lab/github-issue-flow --skill github-issue-flow -a cursor -a codex -a claude-code -y
```

すべてのプロジェクトで使えるようにグローバルインストールする場合は、`-g` を追加します。Node.js 22.20以上とGitが必要です。インストーラーが各エージェントの所定の場所にスキルを配置するため、スキル定義を重複して用意する必要はありません。

## 使い方

エージェントに次のように依頼します。

> github-issue-flowを使って、次の仕様をEpicごとに細かいネイティブSub-issueへ分割し、関連する子Issueをまとめたstacked PRで実装してください: ...

実装せずに計画だけを作る場合は、次のように依頼します。

> github-issue-flowを使って、Epic・細粒度Sub-issue・依存関係DAG・PRスタック計画だけを作成してください: ...

Codexでは `$github-issue-flow` で明示的に呼び出せます。Claude CodeとCursorでは、対応している環境で `/github-issue-flow` を使えます。

## ワークフロー全体を実行するための要件

- GitHub CLI（`gh`）が認証済みで、リポジトリのIssueとPRを操作する権限があること。ネイティブの `--parent` とIssueの依存関係を利用するには、v2.94以上を推奨します。
- Gitが利用でき、ブランチとPRの作成およびマージに必要な権限があること。restackでは**保護されていない機能ブランチに限り** `--force-with-lease` が必要になる場合があります。
- 必須の異なるAIモデルによるレビューを実行するため、Codex CLI（`codex`）とClaude Code CLI（`claude`）の両方が認証済みであること。
- 既存のCIとブランチ保護・人間の承認ルールに従うこと。

スキル自体は手順書のみで、追加のランタイムパッケージや特権を必要とするフックはインストールしません。必要なレビューツールを利用できない場合、エージェントは自動マージせず、作業を妨げている要因を報告する必要があります。

## ディレクトリ構成

```text
skills/
  github-issue-flow/
    SKILL.md
```

## ライセンス

MIT
