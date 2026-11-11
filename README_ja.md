# GitHub Issue Flow

[English](README.md)

**Issueを起点とするAIコーディング**のための、技術スタックに依存しないAgent Skillです。Epic → **細粒度のGitHubネイティブSub-issue** → 独立worktreeでの並列実装 → **stacked PR** → AIレビュー → **`stack` で人間が確認** → **`dev` に統合** → **`main` にリリース**、という流れです。GitHub Actionsは利用許可を事前に確認します。

オープンな[Agent Skills CLI](https://github.com/vercel-labs/skills)を通じて、**Cursor、Codex、Claude Code**で利用できます。

## ワークフローの特徴

**Sub-issueの粒度とPRの粒度は別です。** 各Sub-issueは独立して完了判定できる小さな実装成果に分けつつ、関連する複数のSub-issueを1本の機能別PRにまとめます。依存するPRは、前のPRのマージを待たずにその機能ブランチをbaseとして積みます。各PRの差分はそのPRの直接のbaseと比較してレビューし、squash merge後は後続PRを安全にrebase・retargetします。最初のfeature PRのbaseは`main`ではなく`stack`です。

参考事例は[sekigae-app](https://github.com/ks250206/sekigae-app)です。**13 Epic・146個の実Sub-issue・11本のstacked PR** に分け、[PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163) のように積んでいます。これは**目安となる構造であり、小さな案件でも同じ数を強制するものではありません**。

## ブランチ構成とレビューフロー

```text
main              安定版・リリース用（人間の承認が必要）
└── dev           人間が承認済みの開発成果の統合先
    └── stack     レビュー用ステージング（レビュー窓口PR: stack → dev）
        └── feat/foundation  PR A → stack
            └── feat/domain PR B → feat/foundation
```

1. 対象リポジトリで`dev`と`stack`がなければ、安定ブランチから安全に作成します。すでにあるブランチの履歴は消しません。
2. 小さなSub-issueを別々のworktreeで実装し、関連するものを`feat/*`のPRにまとめます。前提となるfeature PRが未マージでも、そのブランチをbaseにして次のPRを積めます。異なるAIによるレビューとチェックの後、`feat/* → stack`にsquash mergeします。
3. `stack → dev`のPRが**人間のレビュー窓口**です。各feature PRへのリンク、対象Sub-issue、スクリーンショット、検証結果、懸念点を一覧化します。**あなたの明示的な承認があるまで`dev`にマージしません**。自動マージもしません。なおGitHubではPR作成者が自分のPRを「Approve」できないため、AIがあなたのアカウントでPRを作った場合は、あなた自身のPRコメントや明示的な指示を承認として扱います（AIが承認を代行することはありません）。
4. リリース時は別の`dev → main` PRを作り、**再度、人間の承認を得てSquash merge**します。これにより**mainには1リリースにつき1コミット**だけを追加できます。Squash merge後は新しい`main`を`dev`へ通常のmerge commitで取り込み、レビュー待ちの変更がない場合に限って`stack`も`dev`へfast-forward同期します。長期運用ブランチをreset・force-pushしてはいけません。

`stack → dev`が承認・統合された後、対象の`feat/*`ブランチは、**依存する後続stacked PRがすべてrestackまたはmerge済みで、他のopen PRのbaseにも使われていないこと**を確認してから削除します。`stack`・`dev`・`main`は削除しません。

**ブランチ名の注意：** `stack`というブランチが存在すると`stack/xxx`はGit上で作れません。機能ブランチには`feat/xxx`、個別作業には`work/issue-123`を使います.

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
