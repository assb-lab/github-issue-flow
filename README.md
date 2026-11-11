# GitHub Issue Flow

[日本語](README_ja.md)

A stack-agnostic Agent Skill for **issue-first AI coding**: Epics → **fine-grained, real GitHub sub-issues** → parallel worktrees → **stacked feature PRs** → AI review and **topic-specific `stack/*` review lanes** → approved **`dev` integration** → reviewed **`main` release**. GitHub Actions are opt-in.

Compatible with **Cursor, Codex, and Claude Code** through the open [Agent Skills CLI](https://github.com/vercel-labs/skills).

## What makes this workflow different

**Issue granularity is not PR granularity.** The agent tracks each small, verifiable deliverable as a native GitHub sub-issue, but groups related sub-issues into cohesive PRs. Dependent PRs target their immediate predecessor's still-open branch, instead of waiting for every predecessor to merge. Each incremental diff is reviewed against its own PR base; squash-merged feature stacks are carefully restacked against their assigned `stack/<topic>` branch, not `main`.

[sekigae-app](https://github.com/ks250206/sekigae-app) is the reference pattern: **13 Epics, 146 native sub-issues and 11 stacked implementation PRs**, starting with [PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163). Those counts are an example, **not fixed quotas** for smaller projects.

## Epic Issues as detailed specifications

Every Epic/parent Issue must be a **substantial design specification**, not just a title and a task checklist. It contains the problem and goals, in/out scope, concrete behavior and edge cases, architecture/interface contracts, integration/dependency plan, test strategy, measurable acceptance criteria, and linked native sub-issues.

**At least one feature-specific Mermaid `flowchart` is required in every Epic Issue.** It should show the real user journey, data/control flow or system lifecycle, including meaningful decision and failure paths. Complex Epics should also have a second Mermaid dependency diagram with actual child Issue numbers. Agents must validate the diagrams and update the parent Issue after creating its real sub-issues.

See the [copyable Epic Issue template](skills/github-issue-flow/references/epic-issue-template.md), including a worked Mermaid example and an authoring checklist.
## Topic-specific review stacks and human promotion

Instead of one global `stack` branch, use **one short-lived review stack per independent Epic or related work batch**. This supports multiple concurrent human reviews without a shared review queue.

```text
main (released)
└── dev (human-approved integration)
    ├── stack/room-editor    → dev (review hub PR A)
    │   └── feat/room-foundation  → stack/room-editor
    │       └── feat/seat-layout  → feat/room-foundation
    └── stack/solver         → dev (review hub PR B)
        └── feat/rule-evaluator   → stack/solver
            └── feat/optimization → feat/rule-evaluator
```

1. The agent creates `dev` from the stable release branch when missing. Each **new** `stack/<topic>` branch starts from the current `dev` tip. Related native sub-issues are implemented in independent worktrees and grouped into feature PRs.
2. Each feature PR is independently cross-model reviewed, then **squash-merged into its own review stack**. Dependent feature PRs are restacked after an upstream feature PR squashes.
3. Each `stack/<topic> → dev` **review-hub PR** includes cumulative changes, links to the feature PRs/sub-issues, screenshots/previews, tests and known gaps. **The human reviews and explicitly approves each stack**; the agent then **uses a regular merge commit**, not a squash, to promote that topic to `dev`.
4. After promotion, delete completed `feat/*` branches and the associated `stack/<topic>` branch only if no open PR or dependent work still needs them. No auto-merge or artificial approvals.
5. The separate `dev → main` release PR requires **another explicit human approval** and uses **Squash merge**: one commit per release on `main`. Then merge the new `main` commit back into `dev` to reconcile ancestry without losing detailed development history; do not reset or overwrite ongoing review stacks.

**Migrating an existing `stack` branch:** Git cannot store `stack` and `stack/something` simultaneously. Inspect the legacy branch first, then [rename it in GitHub's Branches UI](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-branches-in-your-repository/renaming-a-branch) to a nonconflicting archive name such as `archive/legacy-stack-20261111` (or safely delete it after verifying it has no unique/unmerged work and getting permission). Only then create named review stacks. A rename is a **separate repository operation**; merely updating this Skill does not rename the old branch.

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
