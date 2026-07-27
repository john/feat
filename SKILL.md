---
name: feat
description: Create and execute versioned repository feature briefs. The create command scaffolds a numbered brief at docs/features/NNNN-name.md from a template. The run command executes one existing brief through planning, implementation, verification, commit, push, and pull request. Use only when the user explicitly invokes or requests the feat workflow; do not use for general feature discussion, planning-only work, code review, merge, release, or deployment. Run writes docs/plans/NNNN-name.md, establishes a practical pre-change verification baseline, traces every acceptance criterion to implementation and evidence, compares final results with the baseline, and stops after opening or updating the pull request. An optional assume mode resolves material ambiguity conservatively without pausing.
---

# Feat

Two commands:

- `create` scaffolds a numbered feature brief from a template.
- `run` executes one brief through: brief -> branch -> baseline -> plan -> implementation -> verification -> pull request

The pull request is the terminal action. Never merge, release, or deploy as part of this skill.

## Parse the request

Read arguments from the user request or an `ARGUMENTS:` block appended by the client. The first argument is the command:

| Command | Arguments |
|---|---|
| `create` | `NAME ["description"]` |
| `run` | `NAME [assume]` |
| `list` | none |

Match the command case-insensitively. When the first argument is missing or is not one of these commands, print the usage below along with the sorted stems from `docs/features/*.md`, and stop.

```text
/feat create NAME ["description"]   scaffold a new brief
/feat run NAME [assume]             execute an existing brief
/feat list                          list available briefs
```

Never treat a bare first argument as a brief name. Earlier versions accepted `/feat NAME`; that form is no longer supported.

Reject extra arguments, and any second argument to `run` other than `assume`, matched case-insensitively.

`list` prints the sorted stems from `docs/features/*.md` and stops. Say so plainly when there are none.

### Names and paths

Accept only a name matching `[A-Za-z0-9][A-Za-z0-9_-]*`. Reject path separators, `..`, absolute paths, and path overrides.

Briefs live at `docs/features/{NNNN}-{name}.md`. For `docs/features/0007-account-export.md`:

- Stem: `0007-account-export`
- Number: `0007`
- Name: `account-export`
- Plan: `docs/plans/0007-account-export.md`
- Branch: `feat/0007-account-export`

### Resolve a run argument

Match the argument against the stems of `docs/features/*.md`, case-insensitively, in this order:

1. Exact stem, such as `0007-account-export`.
2. An all-digit argument, compared numerically against each leading number, so `7`, `07`, and `0007` all resolve to `0007-account-export`.
3. The name alone: the stem with its `NNNN-` prefix removed, or a stem that never had one.

With no match, list every available brief and stop without creating files or branches. With more than one match, list only the matching stems and stop. Briefs written before numbering existed carry no prefix and resolve by rules 1 and 3.

## Preserve repository and user intent

Follow applicable `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, and scoped repository instructions. More specific instructions override broader ones.

Treat the brief as the source of truth. Do not implement adjacent improvements without explicit approval. Never discard, reset, clean, stash, overwrite, or commit unrelated user work. Never force-push.

Before branch changes, commits, pushes, or pull-request work, read [references/git-pr-workflow.md](references/git-pr-workflow.md).

## Create a brief

`/feat create NAME ["description"]`

Scaffold one brief and stop. Do not create branches, commits, or pull requests, and do not implement anything.

1. Create `docs/features/` if it does not exist.
2. Strip any leading `NNNN-` or `NNNN_` from the name; this command assigns the number.
3. If a brief with the same name already exists at any number, stop and report its path. Names stay unique so that `run` can resolve them.
4. Determine the next number: take the highest leading number across `docs/features/*.md` and add one, starting at `0001` when no numbered brief exists. Zero-pad to four digits, widening only when the number no longer fits. Never reuse a gap.
5. Read [references/brief-template.md](references/brief-template.md). Replace `{Title}` with the name, hyphens and underscores as spaces and the first letter capitalized.
6. Without a description, write the template unchanged; its italic placeholders are the author's prompts. With a description, replace the placeholders under Outcome, Scope, and Acceptance criteria with content drawn from it, and leave every section the description does not support as a placeholder. Never invent constraints, dependencies, or acceptance criteria the user did not state.
7. Write `docs/features/{NNNN}-{name}.md`, report the path and the `run` command that would execute it, and stop.

## Run a brief

`/feat run NAME [assume]`

Resolve the argument to a brief, then work through the run mode and the numbered steps below.

### Apply the run mode

**Default:** Ask one consolidated set of blocking questions before planning or implementation when ambiguity would materially change scope, observable behavior, acceptance criteria, data, security, privacy, migration, or compatibility. Do not ask about choices settled by the brief or clear repository conventions.

**Assume:** Do not pause for clarification. Choose the narrowest reasonable, reversible interpretation; record each material assumption in the plan; and defer adjacent work instead of expanding scope.

Assume mode does not bypass permissions, sandboxing, credentials, repository policy, branch protection, verification, safety constraints, or hard blockers. Stop when safe completion requires unavailable access, information, tooling, credentials, or repository state.

### 1. Read and frame the brief

Read `docs/features/{stem}.md`.

Extract the intended outcome, in-scope behavior, explicit exclusions, constraints, dependencies, rollout or migration requirements, risks, and acceptance criteria. Give each criterion a stable ID such as `AC-1`; preserve existing IDs and wording where practical.

Treat ambiguity as blocking when plausible interpretations would produce meaningfully different behavior, interfaces, data, security posture, migration requirements, or acceptance results. Resolve it according to the active run mode. A brief still carrying the italic placeholder text from `references/brief-template.md` is unfinished; treat each unfilled section as material ambiguity of exactly this kind.

### 2. Preflight and explore

Follow the preflight and unrelated-work rules in `references/git-pr-workflow.md`.

Read the smallest useful set of files, widening only when evidence requires it. Identify analogous implementations, architecture and coding conventions, affected interfaces or schemas, relevant tests, standard verification commands, documentation, accessibility, security, telemetry, migrations, and any existing plan, branch, or pull request for the brief.

### 3. Create or resume the feature branch

Follow repository branch conventions; otherwise use `feat/{stem}`. Resume clearly matching work instead of creating duplicates. Create or resume the branch before writing the plan or changing product code. Never work directly on the default branch.

### 4. Establish the verification baseline

Before product-code changes:

- Record the base commit.
- Run the narrowest relevant existing tests, build, typecheck, lint, or other practical checks.
- Prefer commands that can be rerun after implementation.
- Record every exact command, result, and identifiable pre-existing failure.
- Call a failure pre-existing only when the same command produced a materially equivalent failure before the feature changes.

Do not expand scope to fix unrelated baseline failures. If a useful baseline is unavailable, prohibitively expensive, flaky, or environmentally blocked, record why and do not later claim that no regressions were introduced.

### 5. Write the plan before product code

Create or update `docs/plans/{stem}.md`. Preserve useful history when resuming. Use repository conventions when available; otherwise use:

```markdown
# Plan: [feature title]

Source brief: docs/features/{stem}.md
Status: in progress
Base commit: [commit]

## Outcome

## Scope
### In scope
### Out of scope

## Assumptions and decisions

## Acceptance-criteria traceability

| ID | Acceptance criterion | Implementation | Verification | Status |
|---|---|---|---|---|
| AC-1 | [criterion] | [planned files or components] | [planned test or check] | planned |

## Verification baseline

| Command | Baseline result | Notes |
|---|---|---|
| `[exact command]` | [result] | [pre-existing failure or limitation] |

## Implementation steps
1. ...

## Files likely to change

## Risks and follow-ups
```

Add one traceability row per acceptance criterion before coding. Keep the plan concrete and sequenced. Update it whenever implementation or scope differs from the plan.

### 6. Implement the smallest coherent feature

Implement only what is needed to satisfy the brief and criteria. Reuse repository patterns, prefer incremental changes over broad refactors, preserve public behavior unless change is required, and address relevant errors, compatibility, migrations, accessibility, security, documentation, and telemetry.

Do not leave dead code, debug output, secrets, or unrelated formatting churn. In default mode, ask before scope expansion. In assume mode, choose the narrower implementation and record deferred work.

### 7. Verify, trace, and review

Add or update tests that directly cover the new behavior. Run, in increasing scope:

1. Checks for the changed behavior.
2. The baseline commands.
3. Affected package or subsystem checks.
4. Broader repository checks when practical.

Record every command and outcome. Compare baseline and final results explicitly. Distinguish verified pre-existing failures from new regressions; do not infer pre-existence without evidence.

Update every traceability row with actual implementation references, exact verification evidence, and status `pass`, `fail`, or `blocked`. Never mark `pass` without direct evidence. Capture screenshots or equivalent evidence for user-facing changes when supported; otherwise record that visual verification was unavailable.

Re-read the brief and plan, inspect the full branch and uncommitted diffs, confirm all criteria are covered, and check for omissions, scope creep, regressions, unsafe behavior, migration issues, missing tests, and documentation gaps. Set plan status to `implemented` only when all required criteria pass and relevant local verification is complete; otherwise set it to `blocked` and identify why.

### 8. Commit, push, and open or update the pull request

Follow `references/git-pr-workflow.md`. Include the brief and plan with the implementation. Open or update exactly one pull request for the branch.

Use a ready-for-review pull request only when all required criteria pass and no relevant new failure remains. Otherwise use a clearly blocked draft when repository policy permits preserving the work in a pull request.

After opening or updating the pull request, stop. Treat any requested merge, release, or deployment as separate follow-up work outside this skill.

### 9. Report the result

Report:

- Status, brief path, plan path, branch, and pull-request URL or blocker.
- Main files changed and material assumptions or deviations.
- One acceptance-criteria table with implementation, evidence, and `pass`/`fail`/`blocked` status.
- One verification comparison table with command, baseline, final result, and assessment.
- New regressions, pre-existing failures, unrun checks, visual-verification status, remaining risks, and follow-ups.
- That the workflow stopped at the pull request and did not merge, release, or deploy.

Do not claim completion, criterion coverage, test success, CI success, or absence of regressions without direct evidence.