# Run a plan

`/feat run NAME`

Work through the numbered steps below.

`run` implements an already-approved plan. It does not reframe the brief and does not ask scope questions; that happened during `plan`, and the plan file is the agreement. `run` still asks the repository-safety question in `references/git-pr-workflow.md` about isolating unrelated uncommitted changes, which concerns repository state rather than feature scope.

When implementation reveals that the plan is wrong or unworkable, do not silently redesign. Implement what the plan states where it holds, stop at the point it breaks down, record the conflict in the plan, and report it so the user can revise the plan and run again.

## 1. Load the brief and the plan

Read `docs/features/{stem}.md` and `docs/plans/{stem}.md`.

When `docs/plans/{stem}.md` does not exist, stop immediately and say so, naming the exact command that would create it:

```text
No plan found at docs/plans/{stem}.md.
Run /feat plan {name} first, review the plan, then run /feat run {name}.
```

Stop before preflight. Do not create a branch, change files, or open a pull request when the plan is missing.

Treat the plan as the specification for what to build and the brief as the source of truth for why and for the acceptance criteria. When the plan omits a criterion the brief states, add a traceability row for it, record the addition as a deviation, and report it. When the plan is materially unfinished, stop and ask for it to be revised through `plan` rather than filling the gaps here.

## 2. Preflight

Follow the preflight and unrelated-work rules in `references/git-pr-workflow.md`.

Confirm the repository, remote, and pull-request mechanism are usable before changing anything. Read enough of the code the plan names to implement it correctly; the plan's exploration notes are a starting point, not a substitute for reading the files being changed.

## 3. Create or resume the feature branch

Follow repository branch conventions; otherwise use `feat/{stem}`. Resume clearly matching work instead of creating duplicates. Create or resume the branch before changing product code. Never work directly on the default branch.

Record the branch's starting commit as the base commit in the plan.

## 4. Establish the verification baseline

Before product-code changes:

- Run the baseline commands the plan records, adjusting when the repository has changed since planning.
- Prefer commands that can be rerun after implementation.
- Record every exact command, result, and identifiable pre-existing failure in the plan's verification table.
- Call a failure pre-existing only when the same command produced a materially equivalent failure before the feature changes.

Do not expand scope to fix unrelated baseline failures. If a useful baseline is unavailable, prohibitively expensive, flaky, or environmentally blocked, record why and do not later claim that no regressions were introduced.

Set the plan status to `in progress` once the baseline is recorded.

## 5. Implement the smallest coherent feature

Follow the plan's implementation steps. Implement only what is needed to satisfy the plan and the acceptance criteria. Reuse repository patterns, prefer incremental changes over broad refactors, preserve public behavior unless change is required, and address relevant errors, compatibility, migrations, accessibility, security, documentation, and telemetry.

Do not leave dead code, debug output, secrets, or unrelated formatting churn. Choose the narrower implementation when the plan permits either, and record deferred work rather than expanding scope. Update the plan whenever implementation differs from what it describes.

## 6. Verify, trace, and review

Add or update tests that directly cover the new behavior. Run, in increasing scope:

1. Checks for the changed behavior.
2. The baseline commands.
3. Affected package or subsystem checks.
4. Broader repository checks when practical.

Record every command and outcome. Compare baseline and final results explicitly. Distinguish verified pre-existing failures from new regressions; do not infer pre-existence without evidence.

Update every traceability row with actual implementation references, exact verification evidence, and status `pass`, `fail`, or `blocked`. Never mark `pass` without direct evidence. Capture screenshots or equivalent evidence for user-facing changes when supported; otherwise record that visual verification was unavailable.

Re-read the brief and plan, inspect the full branch and uncommitted diffs, confirm all criteria are covered, and check for omissions, scope creep, regressions, unsafe behavior, migration issues, missing tests, and documentation gaps. Set plan status to `implemented` only when all required criteria pass and relevant local verification is complete; otherwise set it to `blocked` and identify why.

## 7. Commit, push, and open or update the pull request

Follow `references/git-pr-workflow.md`. Include the brief and plan with the implementation. Open or update exactly one pull request for the branch.

Use a ready-for-review pull request only when all required criteria pass and no relevant new failure remains. Otherwise use a clearly blocked draft when repository policy permits preserving the work in a pull request.

After opening or updating the pull request, stop. Treat any requested merge, release, or deployment as separate follow-up work outside this skill.

## 8. Report the result

Report:

- Status, brief path, plan path, branch, and pull-request URL or blocker.
- Main files changed and material assumptions or deviations, including anything that diverged from the plan.
- One acceptance-criteria table with implementation, evidence, and `pass`/`fail`/`blocked` status.
- One verification comparison table with command, baseline, final result, and assessment.
- New regressions, pre-existing failures, unrun checks, visual-verification status, remaining risks, and follow-ups.
- That the workflow stopped at the pull request and did not merge, release, or deploy.

Do not claim completion, criterion coverage, test success, CI success, or absence of regressions without direct evidence.
