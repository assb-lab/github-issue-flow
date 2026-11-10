# GitHub Issue Flow

[日本語](README_ja.md)

A small, stack-agnostic Agent Skill for **issue-first AI coding**: Epics → **fine-grained, real GitHub sub-issues** → dependency/PR-stack plan → isolated parallel worktrees → **stacked PRs** → non-interactive cross-model reviews → CI-gated, bottom-up squash merges and restacking.

Compatible with **Cursor, Codex, and Claude Code** through the open [Agent Skills CLI](https://github.com/vercel-labs/skills).

## What makes this workflow different

**Issue granularity is not PR granularity.** The agent tracks each small, verifiable deliverable as a native GitHub sub-issue, but groups related sub-issues into cohesive PRs. Dependent PRs target their immediate predecessor's still-open branch, instead of waiting for every predecessor to merge. Each incremental diff is reviewed against its own PR base; squash-merged stacks are then carefully rebased and retargeted.

[sekigae-app](https://github.com/ks250206/sekigae-app) is the reference pattern: **13 Epics, 146 native sub-issues and 11 stacked implementation PRs**, starting with [PR #161](https://github.com/ks250206/sekigae-app/pull/161) → [PR #162](https://github.com/ks250206/sekigae-app/pull/162) → [PR #163](https://github.com/ks250206/sekigae-app/pull/163). Those counts are an example, **not fixed quotas** for smaller projects.

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
