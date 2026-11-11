# GitHub Issue Flow

[日本語](README_ja.md)

A stack-agnostic Agent Skill for **issue-first AI coding**: Epics → **fine-grained, real GitHub sub-issues** → parallel worktrees → **stacked feature PRs** → AI review and a human-facing **`stack` review lane** → approved **`dev` integration** → reviewed **`main` release**. GitHub Actions are opt-in.

Compatible with **Cursor, Codex, and Claude Code** through the open [Agent Skills CLI](https://github.com/vercel-labs/skills).

## What makes this workflow different

**Issue granularity is not PR granularity.** The agent tracks each small, verifiable deliverable as a native GitHub sub-issue, but groups related sub-issues into cohesive PRs. Dependent PRs target their immediate predecessor's still-open branch, instead of waiting for every predecessor to merge. Each incremental diff is reviewed against its own PR base; squash-merged feature stacks are carefully restacked against `stack`, not `main`.

[sekigae-app](https://github.com/ks250206/sekigae-app) is the reference pattern: **13 Epics, 146 native sub-issues and 11 stacked implementation PRs**, starting with [PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163). Those counts are an example, **not fixed quotas** for smaller projects.

## Branch lanes and human review

```text
main              released/stable (release approval required)
└── dev           integrated and human-approved
    └── stack     review staging branch; human review hub = PR stack → dev
        └── feat/foundation  PR A, base: stack
            └── feat/domain PR B, base: feat/foundation
```

1. The agent creates `dev` and `stack` from the existing stable branch when missing, without resetting branches that already exist.
2. Related sub-issues are implemented in independent worktrees and grouped into feature PRs. Dependent PRs stack on each other's feature branches. After independent AI review and checks, feature PRs are **squash-merged into `stack`**, restacking any descendants.
3. A `stack → dev` PR acts as the **review hub**: it lists all feature PRs, related Issues, screenshots/previews, checks and outstanding concerns. **Human approval is mandatory** for promotion to `dev`; no auto-merge. If the agent opens PRs under your own GitHub account, GitHub does not allow you to approve your own PR, so an explicit human comment or direct instruction is accepted instead. The agent must not manufacture approval.
4. Release is a separate **human-approved `dev → main` Squash merge**, creating **one `main` commit per release**. Because a squash commit changes ancestry, merge the new `main` tip **back into `dev` with a normal merge commit** after release; when no unreviewed staging changes remain, fast-forward `stack` to the synchronized `dev`. Never reset or force-push these long-lived branches.

Once a `stack → dev` batch is human-approved and integrated, delete the batch's merged `feat/*` branches **only after all dependent stacked PRs have been restacked/merged and no open PR still uses those branches as its base**. Do not delete the permanent `stack`, `dev`, or `main` branches.

**Branch name warning:** the persistent branch is named `stack`, so feature branches **cannot** be named `stack/anything` (Git ref namespace collision). Use `feat/*` and `work/issue-*` instead.

## GitHub Actions cost decision

Before creating workflows or causing private-repository workflows to run, the skill asks:

> Use GitHub Actions? **No (local checks only)** / **Yes, within included usage** / **Yes, with an approved paid budget**.

Public repositories' standard GitHub-hosted runners are generally free. Private repositories get a plan-dependent included allowance and can incur costs after that allowance is used ([GitHub billing docs](https://docs.github.com/en/actions/concepts/billing-and-usage)). The agent checks existing workflows and respects required checks; without explicit consent, it will not add Actions workflows or knowingly trigger private-repo CI. Local verification is reported distinctly from GitHub CI. Enable scoped triggers, cancellation, timeouts, and budget alerts when CI is approved.

## Install

Run this from any project after publishing the repository as `assb-lab/github-issue-flow`:

```bash
npx skills add assb-lab/github-issue-flow --skill github-issue-flow -a cursor -a codex -a claude-code -y
```

Add `-g` to install globally for all projects. Requires Node.js 22.20+ and Git. The installer places the skill where each agent expects it; no duplicate skill definitions are required.

## Usage

Tell your agent:

> Use github-issue-flow to implement this specification. Create fine-grained native sub-issues for every Epic and group related children into dependency-aware stacked PRs: ...

Or request a plan without implementation:

> Use github-issue-flow to create only the Epics, fine-grained sub-issues, dependency DAG and PR stack plan: ...

Codex can explicitly invoke `$github-issue-flow`; Claude Code and Cursor can invoke `/github-issue-flow` where available.

## Requirements for the full workflow

- GitHub CLI (`gh`) authenticated with repository issue/PR permissions; v2.94+ recommended for native `--parent` and issue dependencies.
- Git and permission to create branches, PRs, and merge. Stacked PR restacking may require `--force-with-lease` **on non-protected feature branches only**.
- Both Codex CLI (`codex`) and Claude Code CLI (`claude`) authenticated for mandatory cross-model reviews.
- Existing CI, branch protection, and human approval rules are respected.

The skill itself is only instructions; it installs no extra runtime packages or privileged hooks. If a required review tool is unavailable, the agent must report the blocker instead of automatically merging.

## Layout

```text
skills/
  github-issue-flow/
    SKILL.md
```

## License

MIT
