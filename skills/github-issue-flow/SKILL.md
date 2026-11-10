---
name: github-issue-flow
description: Plan and execute issue-driven GitHub development with fine-grained real GitHub sub-issues, dependency-aware parallel worktrees, coherent stacked pull requests, mandatory non-interactive cross-model reviews, and CI-gated squash merges. Use when asked to implement a specification with GitHub Issues, sub-issues, or an AI-agent GitHub workflow.
---

# GitHub Issue Flow

Apply this workflow only to the **requested scope**, respecting repository instructions and its tech stack. Inspect the repository, default branch, existing Issues, and PRs before creating work; reuse existing work instead of duplicating it. Use `gh` (v2.94+ for native Issue hierarchy) or equivalent GitHub tooling.

**Core rule: finely decomposed sub-issues and PR boundaries are different.** Create small, verifiable **real GitHub sub-issues** first; then group several related children into a cohesive PR and **stack dependent PRs on unmerged predecessor branches**. Do not default to one giant sub-issue per feature, one PR per sub-issue, or waiting for every prerequisite PR to merge before beginning dependent work.

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

1. Map **every** child to a PR group or explicitly mark it deferred. A PR group is a cohesive integration slice, often one Epic or one meaningful subset of it, and may close **several** children (e.g. 5–20 closely related sub-issues when reviewable). Do not collapse those children into a single Issue. Split unwieldy PR groups even if they belong to the same Epic.
2. Publish a table: `Epic | child Issue IDs | PR group | stack order | base/head branches | prerequisites | execution wave / owner`. Set a base branch for **each** PR, not just `main`. Keep the Issue dependency DAG separate from the PR stack graph.
3. **Stack only where code actually depends on an unmerged PR**. The first PR targets the default branch; the next depends on its feature branch, and so on. Start independent stacks from the default branch instead of serializing unrelated work.
4. Reference stack from sekigae-app: [PR #161](https://github.com/ks250206/sekigae-app/pull/161) (`feat/e0-foundation → main`) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) (`feat/e1-domain → feat/e0-foundation`) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163) (`feat/e2-persistence → feat/e1-domain`). Its implementation had 11 stacked PRs for 146 children; reproduce the **relationship**, not necessarily the counts or one very long chain.

```text
main
└── stack/foundation          PR A, base: main; closes #14, #15, ...
    └── stack/domain          PR B, base: stack/foundation; closes #24, #25, ...
        └── stack/persistence PR C, base: stack/domain; closes #33, #34, ...
```

## 3. Implement in isolated worktrees, integrate child work, and open stacked PRs

1. Create **one PR-group integration branch and worktree** per planned PR, owned by a single integrator. Branch from the **published tip of the immediate predecessor PR** (or from the default branch for the bottom PR), even when that predecessor is still open. Example, after fetching the published parent: `git worktree add -b stack/domain ../wt-domain origin/stack/foundation`. Do not quietly use `main` as the base for every PR.
2. For parallel sub-issues **within that PR group**, assign each available subagent a separate `work/issue-<N>` branch and writable worktree, based on the agreed PR-group starting commit. Establish shared contracts and file ownership first; avoid overlapping edits. Each agent commits its own child outcome and runs relevant focused checks. If agents are unavailable, implement sequentially in the group worktree.
3. The integrator **cherry-picks completed child commits (or explicitly integrates their branches)** into the group's integration branch in dependency order, resolves conflicts there, and runs combined checks. Do not let parallel agents write to the same integration worktree, silently overwrite each other's changes, or auto-merge child branches directly to `main`. This is how **many small sub-issues become one coherent PR** while retaining individual Issue tracking.
4. Push the integration branch and create a draft PR with `gh pr create --draft --base <immediate-parent-branch-or-default> --head <integration-branch>`. Open stacked PRs **before** upstream PRs merge; waiting for upstream merge defeats stacking. Keep the incremental PR diff limited to the group relative to its actual base.
5. In every PR body provide: stack position (`2/4`), **direct base branch** and predecessor PR link, child IDs completed (`Closes #N` for fully delivered children; `Refs #N` for partial work), scope, checks, and review status. Do not close an Issue for an unimplemented acceptance criterion. One small child may warrant its own PR, but this is an exception, not the default.
6. Update the published Issue → PR group → stack mapping as work lands. Keep child Issues open until requirements actually land on the default branch; a stacked PR targeting a feature branch does not necessarily auto-close them.

## 4. Mandatory non-interactive independent review of EACH incremental PR

Use an AI model **different from the implementer**, non-interactively and read-only. Obtain the real PR base (`gh pr view <PR> --json baseRefName --jq .baseRefName`) and fetch it. Always review against `origin/<ACTUAL_PR_BASE>`, **not `origin/main` unless this PR directly targets main**:

- **Codex implemented → Claude reviews:** in that PR's worktree, `claude -p --permission-mode plan "Review only the changes in origin/<ACTUAL_PR_BASE>...HEAD. Do not edit files. Report actionable correctness, security, test, edge-case and acceptance findings by severity and file:line."`
- **Claude implemented → Codex reviews:** `codex exec review --base origin/<ACTUAL_PR_BASE>`.
- **Other implementer:** choose an independently different model.

Inspect `git diff origin/<ACTUAL_PR_BASE>...HEAD` to ensure the review scope excludes preceding PRs. Verify no files were edited by the reviewer, publish severity/evidence on the PR, fix actionable findings, run checks, and repeat until blockers are resolved. **Unavailable/failed review is never a passing review.**

## 5. Verify, squash-merge in stack order, and restack

1. Run actual repo checks (CI, typecheck, lint, tests, build as applicable) and confirm the **incremental diff and base** before marking a PR ready. Respect required human approvals and branch protections.
2. Merge **bottom-up**: the PR directly targeting the default branch first. After it passes checks and independent review, squash-merge it (`gh pr merge <PR> --squash`); do not prematurely merge a child PR into an intermediate feature branch and call that final integration.
3. **Squash changes commit SHAs**. Before merging, save the old parent and child branch tip SHAs. After the parent lands on the default branch, rebase its immediate child to the updated default branch, replaying **only the child's own commits** (for a linear stack, `git rebase --onto origin/main <OLD_PARENT_TIP> <CHILD_BRANCH>` in the child's worktree), push the rewritten **feature branch only** with `--force-with-lease`, and retarget that child's PR to `main`. Never force-push `main` or protected branches.
4. Restack further descendants onto the rewritten immediate parent using the corresponding **old parent tip** as the rebase cut point; update their PR base refs. Re-verify all incremental diffs, conflicts, CI and cross-model review when diffs materially change. Never delete a base branch while descendants still require its old tip for restacking.
5. Repeat checks/review/merge/restack to the top of the stack. Verify linked children actually closed; close parent Epics only when **all** children and Epic-level criteria are met. Resolve partial/deferred issues explicitly; never mark unimplemented work done.
6. Summarize links to parents, children and stacked PRs, exact stack bases, CI, independent review results, merged status, and outstanding work.

## Guardrails

- Never force-push protected branches, override branch protection, expose credentials, or falsely report CI/review/merge success.
- If access, sub-issue creation, cross-model CLI, CI, or permissions are unavailable, perform independent safe work and report the precise blocker; do not silently replace native sub-issues with checklist text or merge unreviewed PRs.
- Keep Issues focused and PR descriptions scannable, but **do not reduce the number of meaningful children just to make planning shorter**.
