---
name: github-issue-flow
description: Plan and execute issue-driven development using granular native GitHub sub-issues, parallel isolated worktrees, dependent feature PR stacks grouped into topic-specific stack/* review branches, human-approved stack/*-to-dev merges, squash dev-to-main releases, optional GitHub Actions, and cross-model reviews. Use for GitHub Issue and AI-agent workflows.
---

# GitHub Issue Flow

Apply this workflow only to the **requested scope**, respecting repository instructions and its tech stack. Inspect the repository, default branch, existing Issues, and PRs before creating work; reuse existing work instead of duplicating it. Use `gh` (v2.94+ for native Issue hierarchy) or equivalent GitHub tooling.

**Core rule: finely decomposed sub-issues and PR boundaries are different.** Create small, verifiable **real GitHub sub-issues** first; then group several related children into a cohesive PR and **stack dependent PRs on unmerged predecessor branches**. Do not default to one giant sub-issue per feature, one PR per sub-issue, or waiting for every prerequisite PR to merge before beginning dependent work.

**Promotion rule: `feat/* → stack/<topic> → dev → main`.** `stack/<topic>` is a **short-lived, topic- or review-batch-specific** human-review lane. Each stack has its own feature PR chain and its own cumulative `stack/<topic> → dev` review PR. Different review stacks can coexist and be approved independently. `dev` is the accepted integration branch and `main` is stable/released. **Merge strategy:** squash feature PRs into their own `stack/<topic>`; after explicit human approval, merge `stack/<topic> → dev` with a regular merge commit; after separate explicit approval, squash `dev → main` into one release commit. Do not bypass either human gate or automatically merge unapproved code.

## 0. Bootstrap named review stacks and confirm GitHub Actions policy

1. Detect repository/default branch, visibility, permissions, existing workflows, required checks, `dev`, open PRs, and any existing `stack/*` review branches. Do not silently change branch protection or existing workflows. If adopting this pattern for a new repository, create `dev` from the stable default branch when missing. The default branch is `main` in examples; adapt to the actual default branch.
2. **Use named `stack/<topic>` branches, not a permanent branch named `stack`.** Choose a short kebab-case review unit such as `stack/room-editor`, `stack/solver`, or `stack/release-1`. The unit can be one Epic or a deliberately grouped batch of closely related Epics; it need not be exactly one Epic. Create each new review stack from the **current reviewed `dev` head**, unless there is a concrete reason for another reviewed ancestor, which must be documented. Never combine unrelated batches solely for convenience.
3. **Legacy namespace migration:** Git cannot have both `refs/heads/stack` and any `refs/heads/stack/<topic>`. If a repository already has `stack`, first inspect its differences from `main`/`dev`, open PRs, and branch protections. **Preserve the current tip and rename** the legacy branch to a nonconflicting archive name (for example `archive/legacy-stack-20261111`) via GitHub's Rename branch UI or an authorized branch-rename tool; only delete it if genuinely redundant, permission is available, and the owner approves. Existing open PRs may be affected; never force-update or silently drop changes. Only after the conflicting `stack` ref is gone can `stack/<topic>` be created. Do not claim the migration finished if renaming is unavailable.
4. Before configuring Actions or knowingly triggering an **existing private-repository workflow** by push/PR, **ask the user explicitly**: "Use GitHub Actions? (A) No—local verification only; (B) Yes—within included usage; (C) Yes—with an explicitly approved paid budget. Repository: [public/private]; existing workflows: [summary]." Reuse prior project policy if already answered. Explain that standard hosted runners for public repositories are generally free; private repositories receive plan-dependent included usage and may incur costs above it ([GitHub billing](https://docs.github.com/en/actions/concepts/billing-and-usage)).
5. Until authorized, do not introduce workflows, opt into cloud CI, or knowingly trigger private-repo Actions. If a push/PR would trigger an existing workflow, stop at the boundary and explain; do not disable existing workflows unilaterally. With authorization, use scoped triggers, `concurrency` with cancellation, finite `timeout-minutes`, small matrices, and budget alerts/limits. Without Actions, run local lint, type checks, unit/E2E tests and build as appropriate and explicitly label them "local only"; never report absent CI as passing.

## 1. Author detailed Epic specifications, then fine-grained native sub-issues

**Every parent Epic is a detailed design Issue, not a two-line ticket or only a list of children.** Before creating the parent, read and follow [the Epic template](references/epic-issue-template.md). Use the user's working language and adapt scope/technical depth to the actual feature; omit unrelated boilerplate, not meaningful detail.

1. Convert the requested specification into coherent Epic/parent Issues by feature domain (foundation, data model, UI, service/algorithm, integration, quality). Write each parent as a self-contained specification that includes **background/problem, target users and outcomes, in-scope/out-of-scope behavior, numbered functional requirements and scenarios, architecture/module boundaries, data/interfaces/contracts, relevant non-functional constraints, tradeoffs/open decisions, dependencies, test plan, and objective Epic acceptance criteria**. Link existing designs without replacing the Issue with a mere link.
2. **Every Epic MUST include at least one real Mermaid `flowchart` in its GitHub Issue body.** Diagram the Epic's actual **feature/user workflow, architecture/data flow, lifecycle, or pipeline** (choose what genuinely clarifies the Epic), with named transitions, success/decision branches and relevant error/exception behavior. **A generic Issue-planning chart (`spec → code → test`) is not a substitute.** For nontrivial child sequencing, add a **second Mermaid dependency DAG** using verified GitHub child Issue numbers. Use native GitHub Markdown ` ```mermaid ` code fences, concise node IDs and readable labels. Check diagram syntax and preview rendering when possible; never report unverified rendering as successful.
3. Split **each Epic into concrete implementation deliverables**, not broad workstreams. From the Epic's requirements, separate independently demonstrable types/contracts, stores, algorithms, components/interactions, validation/failure handling, integrations, and E2E/performance checks as appropriate. Include tests with their implementation; cross-feature E2E tests can be dedicated children.
4. **Run a second granularity pass** before creating child Issues. Split any composite child spanning several independently testable behaviors; reject umbrella child titles such as "implement the editor", "build the backend", or "add all screens". Avoid separate Issues for trivial lines/renames or steps without independently reviewable results.
5. Each child must specify **one bounded outcome**, checkboxes, objective acceptance criteria, tests/verification, actual blockers, and likely code/interface ownership where known. A reviewer should be able to mark the child complete without evaluating the entire Epic.
6. For substantial multi-feature work, **roughly 8–20 meaningful children per large Epic when justified** is a useful calibration, **never a quota**. Small Epics legitimately need fewer; complex Epics may need more. Reference [sekigae-app](https://github.com/ks250206/sekigae-app): 13 parent Epics and 146 native sub-issues, e.g. [foundation #14](https://github.com/ks250206/sekigae-app/issues/14), [schema #24](https://github.com/ks250206/sekigae-app/issues/24), [heatmap #63](https://github.com/ks250206/sekigae-app/issues/63).
7. Create parent Issues **with the detailed text and Mermaid feature/system flow already present** (`gh issue create`). Then create **real native sub-issues** (`gh issue create --parent <PARENT>`) and true blocker links (`--blocked-by` or `gh issue edit --add-blocked-by` when supported). **Update the Epic body** with the verified child Issue links, mapping to requirements/acceptance, and a Mermaid dependency graph when sequencing warrants it. Markdown checklists alone do not replace actual parent-child GitHub relationships.
8. **Epic quality gate before coding:** review every parent for completeness, measurable acceptance, a feature-specific Mermaid flowchart (not a generic software-process flow), valid links, real child references and correct dependency arrows. Validate that all functional requirements are covered by children and that no unresolved design choices have been silently invented. Follow the template's final checklist; fix issues before publishing the implementation plan.
9. Publish the full decomposition, dependencies and review-stack plan. If the user requested *planning only*, stop here; otherwise begin implementation only after the Epic quality gate passes.

**Parent Epic template:** [`references/epic-issue-template.md`](references/epic-issue-template.md) (includes illustrative Mermaid feature flow, optional child dependency diagram, required headings, and publication checks).

**Child sub-issue body template**

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
## 2. Plan PR groups AND named review stacks

1. Map every real sub-issue to a coherent implementation PR group or mark it deferred. One PR can deliver **several small sub-issues** (e.g. 5–20 when reviewable); retain the native GitHub issue hierarchy. If a group becomes unwieldy, split the PR without collapsing its child Issues.
2. Assign each PR group to **exactly one named review stack**, e.g. `stack/room-editor` or `stack/solver`. Keep independent topics in separate review stacks when they can be approved individually. Show a table: `Epic | sub-issues | review stack | feature PR group | PR position | base/head | dependencies | validation | owner`. Document cross-stack dependencies; prefer publishing stable interfaces or waiting for needed integration over silently chaining unrelated stacks.
3. For each named review stack, the **bottom feature PR targets `stack/<topic>`**. Subsequent feature PRs target the immediate unmerged predecessor feature branch (e.g., `feat/domain → feat/foundation`) and are opened *before* that predecessor merges. Independent PR groups can each target their corresponding `stack/<topic>` branch. Stack only genuine code dependencies.
4. Make a **separate cumulative human review PR per stack**: `stack/room-editor → dev`, `stack/solver → dev`. It lists the constituent feature PRs and sub-issues, evidence, screenshots, known gaps, and approval status. No single global `stack` branch or global review PR.
5. Reference [sekigae-app](https://github.com/ks250206/sekigae-app): 13 Epic Issues, 146 native sub-issues, 11 stacked implementation PRs. Reproduce its issue/PR granularity, not its PR root branch or exact counts.

```text
main  (stable)
└── dev  (human-accepted integration)
    ├── stack/room-editor       review PR: stack/room-editor → dev
    │   └── feat/room-foundation     PR A → stack/room-editor
    │       └── feat/seat-layout    PR B → feat/room-foundation
    └── stack/solver            review PR: stack/solver → dev
        └── feat/rule-evaluator     PR C → stack/solver
            └── feat/optimization  PR D → feat/rule-evaluator
```

## 3. Implement in isolated worktrees and stage per-topic feature PRs

1. Create the designated `stack/<topic>` branch from the current `dev` tip **after verifying no legacy `stack` ref blocks the namespace**. For each PR group, create one integration branch/worktree owned by a single integrator. The first PR-group branch starts from `origin/stack/<topic>`; dependent PR groups start from the published head of their immediate preceding `feat/*` PR. Unrelated groups can each start from the same reviewed stack head.
2. Parallel sub-issues use `work/issue-<N>` branches with **separate writable worktrees**. Establish shared contracts and file ownership first. Each agent commits its focused changes and runs relevant local checks. If parallel agents are unavailable, implement sequentially.
3. The integrator cherry-picks each completed child issue's commits (or explicitly integrates its branch) into the PR-group `feat/*` branch in dependency order, resolves conflicts and runs combined verification. No agent directly edits another agent's writable worktree or pushes issue branches to `dev` or `main`.
4. Open the feature PR (draft until ready) with `gh pr create --draft --base <stack/topic-or-immediate-feat-parent> --head feat/<group>`. Recheck that the PR diff relative to its **actual direct base** contains only its group's work. Record stack ID/position, parent/child PR links, native sub-issues, validation, screenshots where relevant, and independent review status.
5. Maintain a review index in the cumulative `stack/<topic> → dev` PR and update it as feature PRs land. Do not count staging merges into `stack/<topic>` as approval to promote to `dev` or close release-scoped Issues before their work reaches `main`.

## 4. Mandatory non-interactive independent review of EACH incremental PR

Use an AI model **different from the implementer**, non-interactively and read-only. Obtain the real PR base (`gh pr view <PR> --json baseRefName --jq .baseRefName`) and fetch it. Always review against `origin/<ACTUAL_PR_BASE>`, **not automatically `origin/main`** (bottom feature PRs target their own `stack/<topic>`):

- **Codex implemented → Claude reviews:** in that PR's worktree, `claude -p --permission-mode plan "Review only the changes in origin/<ACTUAL_PR_BASE>...HEAD. Do not edit files. Report actionable correctness, security, test, edge-case and acceptance findings by severity and file:line."`
- **Claude implemented → Codex reviews:** `codex exec review --base origin/<ACTUAL_PR_BASE>`.
- **Other implementer:** choose an independently different model.

Inspect `git diff origin/<ACTUAL_PR_BASE>...HEAD` to ensure the review scope excludes preceding PRs. Verify no files were edited by the reviewer, publish severity/evidence on the PR, fix actionable findings, run checks, and repeat until blockers are resolved. **Unavailable/failed review is never a passing review.**

## 5. Review `stack/*`, promote to `dev`, and squash-release to `main`

**Feature staging (`feat/* → stack/<topic>`)**

1. Verify local checks, approved CI when enabled, acceptance criteria, and mandatory independent non-interactive cross-model review. Respect requested human reviews. Unavailable checks/reviews are not passing checks/reviews.
2. **Squash-merge feature PRs bottom-up into their assigned `stack/<topic>` branch**, not into `dev` or `main`. When a parent feature PR squashes, save its prior head SHA and restack dependent children so each incremental PR still contains **only its own commits**. For example, with a linear child chain: `git rebase --onto origin/stack/<topic> <OLD_PARENT_TIP> feat/child`; push only nonprotected `feat/*` with `--force-with-lease`, retarget the PR, verify the diff, and repeat. Never delete a parent `feat/*` branch until all descendants have been restacked or merged.
3. Keep feature work for separate review topics in separate `stack/*` branches. Build/preview each topic independently so the human can review one topic without gating all other topics.

**Human review (`stack/<topic> → dev`)**

4. As soon as a review stack contains staged changes not yet in `dev`, create/update its own **review hub PR**, with `base=dev` and `head=stack/<topic>`. List all feature PRs, covered sub-issues, expected behaviors, cumulative diff, checks, screenshots, known issues, and reviewer decisions. Draft until ready.
5. **Explicit user approval is mandatory** for **each** review hub PR. Approval can be a real GitHub review or the user's explicit PR comment/direct instruction naming the PR when self-approval is unavailable. The agent must not impersonate the owner's approval, use auto-merge, or interpret silence/AI review as permission.
6. After approval and checks, **merge `stack/<topic> → dev` using a normal merge commit** to preserve the review stack's history. If another review stack was merged into `dev` meanwhile, fetch and review the updated cumulative diff, resolve genuine conflicts and rerun affected checks before merging. **Do not squash the review-hub PR.**
7. Clean up `feat/*` branches belonging to an accepted review stack only after all their descendants are successfully restacked/merged, no open PR uses those branches as a base, and no unmerged work remains. The approved `stack/<topic>` branch is also **disposable after its PR has merged and no dependents remain**; archive/reuse only if the project has an explicit reason, otherwise safely delete it. Never delete `dev` or `main`, and do not delete a still-open review stack.

**Release (`dev → main`, one squash commit per release)**

8. Continue integrating **approved** review stacks into `dev`. When ready, open a separate release PR `dev → main` with the release summary, Issue links and validation. Require **another explicit human approval**; no automatic release.
9. After approval and required checks, **squash-merge `dev → main`**, e.g. `gh pr merge <RELEASE_PR> --squash`. Do **not** delete `dev`. Write a clear release title/version. Do not force-push long-lived branches.
10. **Mandatory post-release reconciliation:** because the `main` release squash creates a new commit not already in `dev`, fetch `origin/main` and integrate it **back into `dev` with a normal merge commit** (for example from a clean `dev` worktree, `git merge --no-ff origin/main`), preserving detailed development history. Inspect unexpected diffs or conflicts and use a reviewed sync PR if branch rules require it; **never reset `dev` to `main`**.
11. For any **still-open** `stack/*` branches, **preserve unreviewed changes**. If a review stack has no pending differences and is an ancestor of the synchronized `dev`, it can be safely fast-forwarded to `dev` or deleted. If it contains unreviewed changes, re-evaluate its PR diff against the updated `dev`, resolve/rebase/reconcile with human visibility, and rerun affected checks before promotion. Never blindly reset/force-push a review stack or pull unreviewed features into a release.
12. Only after the release reaches the default branch, verify Issue closure and Epic acceptance criteria. Report the review stack and feature PR links, human approvals, CI/local verification, release squash SHA, synchronization status, and outstanding work.

## Guardrails

- Never rewrite `main`, `dev`, or protected review branches; never bypass branch protection or human approval, expose credentials, or falsely report CI/review/merge success.
- Never configure or knowingly trigger private-repository GitHub Actions without the user's explicit policy; do not disable pre-existing workflows without authorization.
- If access, native sub-issues, cross-model review CLI, required checks, or merge permissions are unavailable, continue independent safe work and report the precise blocker; no unreviewed or unauthorized promotion.
- Keep Issues focused and PR descriptions scannable, but **do not reduce the number of meaningful children merely to shorten planning**.
