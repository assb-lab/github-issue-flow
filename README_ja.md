# GitHub Issue Flow

[English](README.md)

**Issueを起点とするAIコーディング**のための、技術スタックに依存しないAgent Skillです。Epic → **細粒度のGitHubネイティブSub-issue** → 独立worktreeでの並列実装 → **stacked PR** → AIレビュー → **テーマ別 `stack/*` で人間が確認** → **`dev` に統合** → **`main` にリリース**、という流れです。GitHub Actionsは利用許可を事前に確認します。

オープンな[Agent Skills CLI](https://github.com/vercel-labs/skills)を通じて、**Cursor、Codex、Claude Code**で利用できます。

## ワークフローの特徴

**Sub-issueの粒度とPRの粒度は別です。** 各Sub-issueは独立して完了判定できる小さな実装成果に分けつつ、関連する複数のSub-issueを1本の機能別PRにまとめます。依存するPRは、前のPRのマージを待たずにその機能ブランチをbaseとして積みます。各PRの差分はそのPRの直接のbaseと比較してレビューし、squash merge後は後続PRを安全にrebase・retargetします。最初のfeature PRのbaseは`main`ではなく、そのテーマ専用の`stack/<topic>`です。

参考事例は[sekigae-app](https://github.com/ks250206/sekigae-app)です。**13 Epic・146個の実Sub-issue・11本のstacked PR** に分け、[PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163) のように積んでいます。これは**目安となる構造であり、小さな案件でも同じ数を強制するものではありません**。

## Epic Issueは詳細な設計書として作成

親Epicは単なるTODOの一覧ではなく、**詳細な仕様・設計Issue**にします。背景と課題、目的、スコープ（含む／含まない）、機能要件・利用シナリオ・例外ケース、アーキテクチャとデータ／API等の契約、依存関係、テスト方針、検証可能な完了条件、紐付くネイティブSub-issueを記載します。

**すべてのEpic IssueにMermaidの`flowchart`を最低1つ必須**とし、実際のユーザー操作、処理分岐、データの流れ、システム構成・処理ライフサイクルなどを図示します。単なる「計画→実装→テスト」図だけでは不可。依存関係が複雑なEpicでは、実際のSub-issue番号を使った依存DAGも追加します。GitHub上での表示とリンクの整合性を確認します。

具体的な記述例とMermaid図を含む[親Epic Issueテンプレート](skills/github-issue-flow/references/epic-issue-template.md)を追加しました。Epic作成後にSub-issueを発行し、親Issueへリンク・依存関係を追記する運用です。
## テーマ別レビュースタックと承認フロー

単一の`stack`ブランチを使う代わりに、**Epicまたは関連する開発テーマごとに`stack/<topic>`を作成**します。テーマごとに独立したPRとプレビューを用意できるので、複数の開発を並行してレビューできます。

```text
main（安定版・リリース）
└── dev（承認済み開発内容の統合先）
    ├── stack/room-editor    → dev（人間のレビューPR A）
    │   └── feat/room-foundation → stack/room-editor
    │       └── feat/seat-layout → feat/room-foundation
    └── stack/solver         → dev（人間のレビューPR B）
        └── feat/rule-evaluator  → stack/solver
            └── feat/optimization → feat/rule-evaluator
```

1. `dev`がなければ安定版ブランチから作ります。**新しい**`stack/<topic>`は原則として、その時点の最新の`dev`から分岐させます。細かなネイティブSub-issueを個別worktreeで実装し、関連するものを`feat/*`のPRにまとめます。
2. 各feature PRを別モデルがレビューし、問題がなければ**所属する`stack/<topic>`へSquash merge**します。依存する後続PRは必要に応じてrestackします。
3. `stack/<topic> → dev`は**テーマ専用の人間向けレビューPR**です。対応するfeature PR、Sub-issue、画面、検証結果、懸念点を一覧化します。**あなたが各スタックを明示的に承認してから、通常のMerge commitで`dev`へ統合**します。自動マージはしません。
4. 承認・統合済みの`feat/*`と`stack/<topic>`は、後続PRの依存や未統合の作業が残っていない場合に限って削除します。
5. 別途`dev → main`のPRを作り、**再度人間が承認してからSquash merge**します。`main`の新しいコミットは、リリース後に`dev`へ通常マージして履歴を整合させます。レビュー途中の`stack/*`を強制的に更新・破棄しません。

**既存の`stack`からの移行：** Gitでは`stack`と`stack/xxx`を同時に作れません。差分・既存PR・ブランチ保護を確認し、[GitHubのBranches画面からリネーム](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-branches-in-your-repository/renaming-a-branch)して、例えば`archive/legacy-stack-20261111`に退避してから`stack/*`を作成します。安全性が確認でき、許可がある場合は旧ブランチを削除する方法もあります。**Skillを書き換えただけでは、既存ブランチの名前は変更されません。**

## GitHub Actionsの利用確認

GitHub Actionsを設定する前、またはPrivateリポジトリの既存ワークフローを発火させる前に、次のいずれかを**必ず確認**します。

- **使わない**：ローカルでlint・型検査・テスト・ビルドを実施
- **無料枠の範囲で使う**：利用枠と設定を確認し、コスト抑制策を入れる
- **有料利用も許可する**：事前に上限予算を指定してもらう

Publicリポジトリの標準GitHub-hosted runnerは基本無料です。Privateリポジトリもプランに応じた無料利用枠がありますが、超過すると課金され得ます（[GitHub公式の料金説明](https://docs.github.com/en/actions/concepts/billing-and-usage)）。既存のworkflow・必須チェックを調査し、許可がない状態で新たなActionsワークフローを追加したり、PrivateリポジトリのCIを意図的に起動したりしません。許可を得た場合は、発火条件・並列実行のキャンセル・タイムアウト・予算アラート等で消費を抑えます。

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
