# Git and pull-request workflow

Use this reference for `feat` preflight, branch isolation, commit, push, and pull-request handling. The workflow always stops at the pull request.

## Preflight and unrelated work

Confirm the current directory is a Git repository with a GitHub remote and an authenticated pull-request mechanism such as `gh` or an available GitHub integration.

Inspect the repository root, default and current branches, worktrees, remotes, authentication, complete working-tree status, applicable instructions, and existing plan, branch, remote branch, or pull request for the slug.

Never discard, reset, clean, stash, overwrite, or commit unrelated changes.

When unrelated uncommitted changes exist:

- Default mode: ask how the user wants the feature isolated before editing.
- Assume mode: use a separate worktree or other safe isolation without touching those changes.
- Stop if safe isolation is unavailable.

## Branch handling

Use the repository's branch convention; otherwise use `feat/{slug}`. Resume a clearly matching local branch, remote branch, worktree, or pull request instead of creating duplicates.

Create or resume the feature branch before writing the plan or changing product code. Never implement or commit on the default branch. Record the branch's starting commit as the baseline commit. When resuming and the original starting state cannot be reconstructed safely, record that limitation rather than rewriting history.

Do not amend unrelated commits, rewrite shared history, or force-push.

## Stage, commit, and push

Stage only intended paths; avoid broad staging when unrelated changes exist.

Before committing, inspect `git status`, the staged diff, the full branch diff against the default branch, and recent commit messages. Follow repository commit conventions, keep commits focused, and do not bypass hooks or conceal failures.

Push the named feature branch without force. On authentication, network, or permission failure, preserve local work and report the exact blocker.

## Pull-request handling

Look for the active feature template, including:

- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE/*.md`
- `docs/PULL_REQUEST_TEMPLATE.md`
- `PULL_REQUEST_TEMPLATE.md`

Choose the feature template when several exist and preserve required sections. Update an existing pull request for the branch rather than opening another.

If no template exists, use:

```markdown
## Summary

## Source brief
`docs/features/{slug}.md`

## Implementation plan
`docs/plans/{slug}.md`

## Acceptance criteria

| ID | Acceptance criterion | Implementation | Verification | Status |
|---|---|---|---|---|
| AC-1 | ... | ... | ... | pass/fail/blocked |

## Verification comparison

| Command | Baseline | Final | Assessment |
|---|---|---|---|
| `[exact command]` | ... | ... | unchanged/improved/regression/unavailable |

## Screenshots

## Risks and follow-ups
```

The pull request must describe implemented rather than aspirational scope; include the brief and plan; map every criterion to evidence; report exact baseline and final results; distinguish pre-existing failures from regressions; include screenshots or explain their absence; and identify assumptions, deviations, risks, follow-ups, and unrun checks.

Open ready-for-review only when all required criteria pass and no known failure is attributable to the change. Use a draft when blocked, materially unverified, or failing because of the change, when repository policy permits it.

After creating or updating the pull request, return its URL or number and stop. Do not merge, enable auto-merge, approve, release, deploy, or invoke a release workflow.