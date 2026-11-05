# GitHub Issue Flow

A small, stack-agnostic Agent Skill for **issue-first AI coding**: parent GitHub Issues → real sub-issues → parallel worktrees → non-interactive cross-model reviews → CI-gated squash merges.

Compatible with **Cursor, Codex, and Claude Code** through the open [Agent Skills CLI](https://github.com/vercel-labs/skills).

## Install

Run this from any project after publishing the repository as `assb-lab/github-issue-flow`:

```bash
npx skills add assb-lab/github-issue-flow --skill github-issue-flow -a cursor -a codex -a claude-code -y
```

Add `-g` to install globally for all projects. Requires Node.js 22.20+ and Git. The installer places the skill where each agent expects it; no duplicate skill definitions are required.

## Usage

Tell your agent:

> Use github-issue-flow to implement this specification via parent Issues and sub-issues: ...

Or request a plan without implementation:

> Use github-issue-flow to create only the GitHub Issues, sub-issues and parallel execution plan: ...

Codex can explicitly invoke `$github-issue-flow`; Claude Code and Cursor can invoke `/github-issue-flow` where available.

## Requirements for the full workflow

- GitHub CLI (`gh`) authenticated with repository issue/PR permissions; v2.94+ recommended for native `--parent` and issue dependencies.
- Git and sufficient permission to create branches, PRs, and merge.
- Both Codex CLI (`codex`) and Claude Code CLI (`claude`) authenticated for mandatory cross-model reviews.
- Existing CI and branch protection rules are respected.

The skill itself is only instructions; it installs no extra runtime packages or privileged hooks. If a required review tool is unavailable, the agent must report the blocker instead of automatically merging.

## Layout

```text
skills/
  github-issue-flow/
    SKILL.md
```

## License

MIT
