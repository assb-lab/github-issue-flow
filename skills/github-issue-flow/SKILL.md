---
name: github-issue-flow
description: Plan and execute issue-driven GitHub development. Break specifications into parent issues and real GitHub sub-issues, schedule parallel subagents in isolated git worktrees, require non-interactive cross-model review, and squash-merge validated PRs. Use when asked to develop a feature or repository with GitHub Issues, sub-issues, or an AI-agent GitHub workflow.
---

# GitHub Issue Flow

Apply this workflow to the **requested scope**, not unrelated repository backlog. Follow existing repository instructions and preserve its tech stack. Use `gh` (v2.94+ for issue hierarchy) or equivalent GitHub tools. Verify repository access, discover the default branch, and inspect existing issues/PRs before creating anything; reuse rather than duplicate work.

## 1. Decompose before coding

1. Convert the specification into a few parent GitHub Issues, each with a goal and acceptance criteria.
2. Split each parent into independently testable **real sub-issues**. Record scope, acceptance criteria, dependencies, tests, and likely file ownership. Avoid overly small or overlapping tasks.
3. Use `gh issue create --title "..." --body "..."` for parents and `gh issue create --parent <PARENT> --title "..." --body "..."` for children. Record dependencies with `--blocked-by` or `gh issue edit --add-blocked-by` when appropriate.
4. Build a dependency graph and publish an execution plan in an issue comment. Group sub-issues into parallel waves; establish shared interfaces before dependent work. If the user requests *planning only*, stop here.

## 2. Implement isolated waves

1. Assign each independent sub-issue to an available subagent, each using its **own Git worktree and feature branch** (`feat/issue-<N>`); never share a writable worktree. If parallel agents are unavailable, process the same plan sequentially.
2. Implement only the assigned issue. Run applicable tests/checks, commit, push, and open a PR linked with `Closes #<N>`.
3. Integrate dependent work only after its prerequisites land. Keep worktree boundaries and shared-file ownership explicit.

## 3. Cross-model review is mandatory

For every PR, use a **different AI model from the implementer**, run **non-interactively**, and report findings with severity, file/line, and evidence. Review correctness, security, edge cases, tests, and acceptance criteria—not style alone. In the examples below, replace `main` with the actual default branch.

- **Codex implementation → Claude review:** from the PR worktree run `claude -p --permission-mode plan "Review the changes against origin/main. Do not edit files. Report actionable findings by severity with file:line evidence."`
- **Claude implementation → Codex review:** from the PR worktree run `codex exec review --base origin/main`.
- **Other/unknown implementer:** choose an independently different model using one of these commands.

Keep review read-only using available sandbox/permissions; verify the review did not modify the worktree. Post the results to the PR. Have the implementer fix actionable findings, rerun checks, then repeat the independent review until blockers are resolved. **Never mark an unavailable/failed review as passed.**

## 4. Validate, squash, repeat

1. Require the repository's CI, type checks, lint, tests, and build as applicable; do not invent commands. Verify PR base and review the final diff after rebases or material changes.
2. After successful checks and cross-model review, squash-merge (`gh pr merge <PR> --squash --delete-branch`). Respect branch protections and required human approvals.
3. Confirm the linked sub-issue is closed. Close a parent only when all its children and acceptance criteria are complete.
4. Refresh the issue graph and repeat parallel waves until **all issues created/selected for this request** are complete. Do not close unrelated issues.
5. Summarize issue/PR links, review results, CI status, blockers, and remaining work.

## Guardrails

- Do not force-push protected branches, bypass permissions, expose credentials, or silently skip verification.
- If GitHub access, cross-model CLI, permissions, or CI is unavailable, continue independent safe work and report the specific blocker; do not falsely claim completion or merge unreviewed work.
- Prefer concise GitHub Issues and PR descriptions over long planning documents.
