---
name: github-issue-flow
description: Plan and execute issue-driven development with granular native GitHub sub-issues, dependent stacked feature PRs, isolated worktrees, human-reviewed stack-to-dev merges, squash dev-to-main releases, safe post-release synchronization, opt-in GitHub Actions, and cross-model reviews. Use for GitHub Issue and AI-agent development workflows.
---

# GitHub Issue Flow

Apply this workflow only to the **requested scope**, respecting repository instructions and its tech stack. Inspect the repository, default branch, existing Issues, and PRs before creating work; reuse existing work instead of duplicating it. Use `gh` (v2.94+ for native Issue hierarchy) or equivalent GitHub tooling.

**Core rule: finely decomposed sub-issues and PR boundaries are different.** Create small, verifiable **real GitHub sub-issues** first; then group several related children into a cohesive PR and **stack dependent PRs on unmerged predecessor branches**. Do not default to one giant sub-issue per feature, one PR per sub-issue, or waiting for every prerequisite PR to merge before beginning dependent work.

**Promotion rule: `feat/* → stack → dev → main`.** `stack` is the shared, human-visible review staging branch; `dev` holds reviewed, accepted development; `main` is released/stable. **Merge strategy:** squash feature PRs into `stack`; after human approval merge `stack → dev` with a normal merge commit; after separate human approval **squash** `dev → main` into one release commit. Human approval is required at both promotion gates. Following a release squash, merge the new `main` commit back into `dev` and safely synchronize `stack` when it has no pending unreviewed changes. No automatic direct `feat → dev` or `feat → main` merge. GitHub has no native "stack channel": implement it with the `stack` branch plus a linked `stack → dev` review PR. Keep `stack`, `dev`, and `main` as **protected-from-rewrite, long-lived branches** (request appropriate branch rules where permissions allow).

## 0. Bootstrap branch lanes and confirm GitHub Actions policy

1. Detect the actual default branch, visibility (public/private), permissions, existing `.github/workflows/**`, required checks, and any existing `stack`/`dev` branches. Do not silently change existing branch protection, default branch, workflows, or commits. For repositories adopting this convention, **create `dev` from the stable default branch, then `stack` from `dev`** if missing; otherwise inspect divergence and preserve existing history. If the default branch is not `main`, use its actual name as the release branch and explain the mapping.
2. Before configuring Actions, or pushing/opening a PR that would trigger **existing Actions on a private repo**, **ask the user explicitly**: "Use GitHub Actions for this repository? (A) No—local verification only; (B) Yes—stay within the included allowance; (C) Yes—with an explicitly approved paid budget. The repository is [public/private] and existing workflows are [summary]." If the answer already exists in a project policy or this conversation, reuse it; do not ask again without a relevant change.
3. Explain accurately: standard hosted runners for public repositories are generally free; private repositories have plan-dependent **included usage** but can incur costs above it. Check the up-to-date [GitHub Actions billing guidance](https://docs.github.com/en/actions/concepts/billing-and-usage); do not assert that all private workflows cost money, or promise a quota without checking it. A self-hosted runner has operational costs/risks even when GitHub does not bill Actions minutes.
4. **Until explicitly authorized**, do not introduce Actions workflows, opt into cloud CI, or knowingly trigger an existing private-repo workflow by push/PR. Continue non-triggering local checks and preparatory work. If an existing workflow or required check makes a push/PR unavoidable, stop at that boundary and report it; **do not disable existing CI without consent**. Record the decision and any budget/runner/trigger limits in the parent Issue plan so future agents respect it.
5. If enabled, minimize usage: scope triggers to relevant branches/paths, use `concurrency` with `cancel-in-progress`, reasonable `timeout-minutes`, small matrices and artifact retention, and request budget alerts/limits for private repos. If disabled, run local type/lint/test/build checks and report "CI not run (user chose local-only)" instead of treating absent CI as a pass.

## 1. Design the Issue hierarchy before coding

1. Translate the specification into Epic/parent Issues by coherent feature area (foundation, domain, persistence, individual UI features, integration, quality). Each parent has a goal, scope, dependencies, and Epic-level acceptance criteria.
2. Decompose **each Epic into concrete implementation deliverables**, not merely a list of broad workstreams. Split separate contracts/types, state/store, algorithms, import/export, UI components and interactions, validation/error handling, integration, and E2E/performance checks where independently demonstrable. Unit tests normally belong with the implementation; separate cross-feature E2E tests can be their own child.
3. **Run a second granularity pass before creating Issues.** Split any candidate that covers multiple independently testable behaviors or cannot be reviewed/verified as one focused change. Reject umbrella child titles such as "implement the editor", "build the backend", or "add all screens". Avoid the opposite extreme: don't create separate Issues for trivial lines, renames, or implementation steps with no independent acceptance outcome.
4. Every child must specify **one bounded outcome**, concrete checkboxes, measurable acceptance criteria, required tests or verification, dependencies (actual blockers only), and likely files/interface ownership when known. A reviewer should be able to say whether that child is done without evaluating the entire Epic.
5. For a substantial multi-feature application, expect **roughly 8–20 meaningful children per large Epic when justified**, rather than only 2–3 vague children. This is a **granularity calibration, never a quota**: small Epics legitimately need fewer, complex Epics may need more. Reference example: [sekigae-app](https://github.com/ks250206/sekigae-app) used 13 Epic Issues and 146 native sub-issues; see [foundation child #14](https://github.com/ks250206/sekigae-app/issues/14), [schema child #24](https://github.com/ks250206/sekigae-app/issues/24), and [heatmap child #63](https://github.com/ks250206/sekigae-app/issues/63).
6. Create the actual hierarchy (`gh issue create --title "..." --body "..."` for parents; `gh issue create --parent <PARENT> --title "..." --body "..."` for children). Record real dependencies with `--blocked-by` / `gh issue edit --add-blocked-by` where supported. Verify each child is attached to the correct parent; plain Markdown checklists are not a substitute.
7. Publish a **complete decomposition + dependency graph** before implementation. Review every Epic for unsplit composite children and uncovered requirements. If asked for planning only, stop after publishing this plan.

**Sub-issue body template**

```md
## Goal
One verifiable outcome.

## Scope
- [ ] Concrete behavior or implementation slice
- [ ] Error/edge case and integration contract
- [ ] Appropriate unit/component test

## Acceptance criteria
Observable result or testable invariant.

## Depends on
#123 (only if a true blocker; otherwise "None")

## Ownership / validation
Likely files/interfaces; relevant test or manual check.
```

## 2. Plan PR groups AND their stack (separately from sub-issues)

1. Map **every** child to a PR group or explicitly mark it deferred. A PR group is a cohesive integration slice, often one Epic or meaningful subset of it, and may cover **several** children (e.g. 5–20 closely related sub-issues when reviewable). Never collapse those children into one Issue. Split unwieldy PR groups even within the same Epic.
2. Publish a table: `Epic | child Issue IDs | PR group | stack position | base/head branches | prerequisites | owner / wave | verification`. Keep the Issue dependency DAG separate from the PR stack graph, and show the human review checkpoints.
3. The **bottom feature PR always targets `stack`**, not `main` or `dev`. Dependent feature PRs target their immediate unmerged predecessor feature branch; unrelated PRs can independently target `stack`. Stack only genuine code dependencies. A published `feat` head can be the base for the next feature before it is merged.
4. Keep the **reserved branch name `stack` distinct from all feature branch names**: Git does not allow both `refs/heads/stack` and `refs/heads/stack/foo`. Use `feat/<group>` for PR-group integration and `work/issue-<N>` for parallel children; **never create `stack/*` branches**.
5. Historical reference: [sekigae-app](https://github.com/ks250206/sekigae-app) used [PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163), with 11 stacked PRs for 146 children. **Reproduce the fine-grained Issue / grouped PR relationship, but replace its `main` root with `stack` in this workflow.**

```text
main (stable; human-approved release)
└── dev (accepted integration; stack → dev approval required)
    └── stack (staged review queue; feat → stack after AI checks)
        └── feat/foundation       PR A, base: stack
            └── feat/domain       PR B, base: feat/foundation
                └── feat/storage  PR C, base: feat/domain

Review hub PR: stack → dev (human approval)
Release PR:    dev   → main (human approval)
```

## 3. Implement in isolated worktrees, integrate child work, and open stacked PRs

1. Create **one PR-group integration branch/worktree** per planned feature PR, owned by one integrator. The first group branches from the published `origin/stack` tip; dependent groups branch from their immediate predecessor's **published feature head**, even when that PR remains open. For example, after fetching: `git worktree add -b feat/domain ../wt-domain origin/feat/foundation`. Unrelated groups branch from `origin/stack`.
2. For parallel sub-issues within a PR group, give each agent a separate `work/issue-<N>` branch and writable worktree, based on the group's agreed starting commit. Establish shared contracts/file ownership; each agent commits its own bounded outcome and runs focused local checks. If agents are unavailable, implement sequentially.
3. The integrator cherry-picks completed child commits (or explicitly integrates child branches) into the group's feature branch in dependency order, resolves conflicts there, and runs combined checks. Agents never share writable worktrees or push child branches straight to `stack`, `dev`, or `main`.
4. Push the feature branch (subject to the **GitHub Actions permission gate** in section 0) and create a draft PR using `gh pr create --draft --base <actual-predecessor-feat-or-stack> --head feat/<group>`. **Create dependent PRs before predecessors merge.** Verify that each incremental PR diff contains only its own group's changes against the actual PR base.
5. Include in each PR: position (`2/4`), direct base and predecessor PR, links to all related children (`Refs #N`; only use auto-closing keywords when their effect is appropriate), scope, screenshots/preview if UI, test results, review status and the next/previous PR links. An agent-created **stack index** in the Epic plan or `stack → dev` PR lists every open group in order so the user can navigate and comment on the entire stack.
6. Keep Issue/PR mapping updated. **Do not treat merges into `stack` or `dev` as release completion.** Verify children truly landed in `main` before closing release-scoped issues; close parent Epics only when all their children and acceptance criteria are fulfilled.

## 4. Mandatory non-interactive independent review of EACH incremental PR

Use an AI model **different from the implementer**, non-interactively and read-only. Obtain the real PR base (`gh pr view <PR> --json baseRefName --jq .baseRefName`) and fetch it. Always review against `origin/<ACTUAL_PR_BASE>`, **not automatically `origin/main`** (the bottom feature PR targets `stack`):

- **Codex implemented → Claude reviews:** in that PR's worktree, `claude -p --permission-mode plan "Review only the changes in origin/<ACTUAL_PR_BASE>...HEAD. Do not edit files. Report actionable correctness, security, test, edge-case and acceptance findings by severity and file:line."`
- **Claude implemented → Codex reviews:** `codex exec review --base origin/<ACTUAL_PR_BASE>`.
- **Other implementer:** choose an independently different model.

Inspect `git diff origin/<ACTUAL_PR_BASE>...HEAD` to ensure the review scope excludes preceding PRs. Verify no files were edited by the reviewer, publish severity/evidence on the PR, fix actionable findings, run checks, and repeat until blockers are resolved. **Unavailable/failed review is never a passing review.**

## 5. Stage in `stack`, obtain human approval, and promote to `dev` / `main`

**Feature staging (`feat/* → stack`)**

1. Verify local checks, any **opted-in** Actions checks, the incremental PR diff, acceptance criteria and mandatory independent cross-model review. An unavailable review or failing required check remains a blocker; do not invent green CI for local-only runs. Any additional human reviews of feature PRs must be respected.
2. Stage **bottom-up**. After the bottom PR passes required checks and reviews, **squash-merge its feature group into `stack`**. This is an intermediate staging merge, **not** permission to promote to `dev`. Keep `stack` usable for preview and manual review.
3. Because a squash changes the parent commit SHA, save the old parent tip, then restack the immediate child: `git rebase --onto origin/stack <OLD_PARENT_TIP> feat/child`, push **only the feature branch** with `--force-with-lease`, and retarget the child PR base to `stack`. Restack descendants against their rewritten immediate predecessors as required, always replaying only child commits. Confirm GitHub's PR incremental diff before review/merge. Do not delete parent branches before descendants have restacked.
4. Continue until the intended review batch is staged on `stack`. Multiple independent groups may be staged in any tested dependency-safe order. Keep work for unfinished future batches out of `stack`. **Do not delete a `feat/*` head while any open stacked PR still uses it as its base**; wait until its descendants have been successfully restacked.

**Human review hub (`stack → dev`)**

5. As soon as `stack` contains staging changes not in `dev`, create/update a persistent **review hub PR** with `base=dev`, `head=stack`, normally as a **draft until ready**. List the feature PR stack, linked sub-issues, cumulative changes (`git diff origin/dev...origin/stack`), validation status, preview/screenshots, known limitations, and specific items requiring a human decision. This PR is the "stack channel"; individual feature PRs remain available for per-slice review.
6. **Explicit human approval is mandatory** before `stack → dev`. The user reviews the hub PR, requests changes if needed (fix via a new `feat` PR or committed corrections staged into `stack`), and approves when satisfied. Approval can be a genuine GitHub approval or an explicit human PR comment/direct instruction naming the PR; **PR authors cannot GitHub-Approve their own PR**, so do not require an impossible self-approval when the agent uses the user's account. Never let an agent impersonate the user's approval, enable auto-merge, or interpret silence, CI, or AI review as permission. After approval and required checks, merge with **a regular merge commit** (or another ancestry-preserving merge, not squash) to keep the long-lived branch history reconcilable.
7. Fetch and update `stack` to `dev` with a **fast-forward** after promotion, verifying there is no unreviewed work in `stack`. Do not force-push or hard-reset long-lived branches. If fast-forward is impossible (unexpected `dev` changes, protection rules, merge mode constraints), stop and reconcile safely before starting the next batch. **Clean up feature branches belonging to this approved batch** only after their contents are staged in `stack`, the batch has been accepted into `dev`, all dependent PRs have been restacked or merged, and no open PR uses those feature branches as a base. Verify there are no unmerged changes, then delete the now-unused remote `feat/*` branches and their disposable worktrees; never delete unrelated work or long-lived branch refs.

**Release (`dev → main`: one squash commit per release)**

8. Continue integrating accepted batches into `dev`. Open a separate `dev → main` release PR including the change summary since the previous release, linked Issues, local/opted-in CI checks, and release notes; request **another explicit human approval** (GitHub review where possible or explicit human instruction/PR comment when self-approval is unavailable). Do not release automatically after the `stack` review.
9. After that approval and required checks, **squash-merge `dev → main`**, e.g. `gh pr merge <RELEASE_PR> --squash` (do **not** pass `--delete-branch`; `dev` is permanent). Put a concise, descriptive release title/version in the squash commit. Do **not** merge this PR with a merge commit, do not force-push, and do not rewrite the long-lived branches.
10. **Mandatory post-release reconciliation.** Squash merging creates a new commit on `main` that is **not an ancestor of the current `dev`**. After the release lands, fetch the new `origin/main` and integrate it **back into `dev` with a normal ancestry-preserving merge commit**, without discarding the detailed development history (for example, in a clean `dev` worktree: `git fetch origin && git merge --no-ff origin/main`, then push if permitted). This should normally be content-neutral if the release contains exactly `dev`. If there are unexpected diffs, conflicts, concurrent pushes or branch protection requiring PRs, do not overwrite or bypass policy: inspect and open a normal synchronization PR to `dev` when necessary. **Never reset `dev` to `main`.**
11. After `dev` incorporates the new `main` release commit, safely **fast-forward `stack` to `dev`** *only if* there is no pending unreviewed work on `stack` and `origin/stack` is an ancestor of `origin/dev`. Verify the ancestry first (for example, `git merge-base --is-ancestor origin/stack origin/dev`). Otherwise, preserve outstanding staging changes and use a reviewed reconciliation/rebase plan; never force-update or reset `stack`. Recheck/restack any open feature PRs whose base changed and respect protected-branch rules.
12. The `dev → main` squash is **one commit for that release**, rather than one commit per feature on `main`. Verify the branch diff for the next release is limited to new work, check linked Issue closure and Epic criteria after the release reaches the default branch, then report feature PR links, human approvals, release squash commit, synchronization status, and any outstanding work. Never claim reconciliation completed if it remains blocked.

## Guardrails

- Never rewrite `main`, `dev`, or `stack`; never bypass branch protection or human approval, expose credentials, or falsely report CI/review/merge success.
- Never configure or knowingly trigger private-repository GitHub Actions without the user's explicit policy; do not disable pre-existing workflows without authorization.
- If access, native sub-issues, cross-model review CLI, required checks, or merge permissions are unavailable, continue independent safe work and report the precise blocker; no unreviewed or unauthorized promotion.
- Keep Issues focused and PR descriptions scannable, but **do not reduce the number of meaningful children merely to shorten planning**.
