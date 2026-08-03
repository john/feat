# Plan a brief

`/feat plan NAME [assume]`

Work through the planning mode and the numbered steps below.

`plan` writes exactly one file, `docs/plans/{stem}.md`, and stops. It creates no branches, commits, or pull requests, runs no verification baseline, and changes no product code. Everything it learns that `run` will need belongs in the plan file, because `run` starts from the plan rather than from this conversation.

## Apply the planning mode

**Default:** Ask one consolidated set of blocking questions before writing the plan when ambiguity would materially change scope, observable behavior, acceptance criteria, data, security, privacy, migration, or compatibility. Do not ask about choices settled by the brief or clear repository conventions.

**Assume:** Do not pause for clarification. Choose the narrowest reasonable, reversible interpretation; record each material assumption in the plan; and defer adjacent work instead of expanding scope.

Assume mode does not bypass permissions, sandboxing, credentials, repository policy, safety constraints, or hard blockers. Stop when safe planning requires unavailable access, information, or tooling.

## 1. Read and frame the brief

Read `docs/features/{stem}.md`.

Extract the intended outcome, in-scope behavior, explicit exclusions, constraints, dependencies, rollout or migration requirements, risks, and acceptance criteria. Give each criterion a stable ID such as `AC-1`; preserve existing IDs and wording where practical. When the brief repeats or skips an ID, renumber into a consistent sequence, keep the original wording, and record the renumbering in the plan.

Treat ambiguity as blocking when plausible interpretations would produce meaningfully different behavior, interfaces, data, security posture, migration requirements, or acceptance results. Resolve it according to the active planning mode. A brief still carrying the italic placeholder text from `references/brief-template.md` is unfinished; treat each unfilled section as material ambiguity of exactly this kind.

## 2. Explore the repository

Read the smallest useful set of files, widening only when evidence requires it. Identify analogous implementations, architecture and coding conventions, affected interfaces or schemas, relevant tests, standard verification commands, documentation, accessibility, security, telemetry, migrations, and any existing plan, branch, or pull request for the brief.

Inspect repository state read-only. Note the current commit, the default branch, and any uncommitted work that `run` will have to isolate, but do not create or switch branches and do not modify the working tree.

Choose the commands `run` should use as its verification baseline: the narrowest existing checks relevant to the feature that can be rerun after implementation. Record them in the plan. When the repository provides no such tooling, say so in the plan rather than inventing a harness.

## 3. Write the plan

Create `docs/plans/` if it does not exist, then create or update `docs/plans/{stem}.md`. When a plan already exists, revise it in place and preserve useful history rather than discarding prior decisions. Use repository conventions when available; otherwise use:

```markdown
# Plan: [feature title]

Source brief: docs/features/{stem}.md
Status: planned
Planned against commit: [commit at planning time]
Base commit: [recorded by run]

## Outcome

## Scope
### In scope
### Out of scope

## Assumptions and decisions

## Acceptance-criteria traceability

| ID | Acceptance criterion | Implementation | Verification | Status |
|---|---|---|---|---|
| AC-1 | [criterion] | [planned files or components] | [planned test or check] | planned |

## Verification

| Command | Purpose | Baseline result | Final result |
|---|---|---|---|
| `[exact command]` | [what it protects] | pending | pending |

## Implementation steps
1. ...

## Files likely to change

## Risks and follow-ups
```

Add one traceability row per acceptance criterion, every row starting at status `planned`. Record every material assumption and every answer received while framing the brief, because the reader of the plan may not have seen this conversation. Keep the plan concrete and sequenced: someone else should be able to execute it.

## 4. Report and stop

Report the plan path, the acceptance criteria it covers, the assumptions or answers it records, and the `/feat run NAME` command that would implement it. Invite review or revision of the plan file.

Stop there. Do not create a branch, run a baseline, change product code, commit, or open a pull request.
