# Plan: Break out plan and implementation

Source brief: docs/features/0001-break-out-plan-and-implementation.md
Status: implemented
Planned against commit: 7e23a8a
Base commit: 7e23a8a

## Outcome

`feat` becomes a three-stage workflow with a human review point between planning and
implementation:

- `create` writes a brief and stops.
- `plan` reads a brief, writes `docs/plans/{stem}.md`, and stops. No branches, no commits.
- `run` reads an approved plan and implements it, ending at a pull request.

Today `run` does planning and implementation in one uninterruptible pass, so the plan is
only ever seen after the code already exists.

## Scope

### In scope

- Add a `plan` command that resolves a brief by number, name, or stem, writes the plan file, and stops.
- Move brief framing, ambiguity resolution, repository exploration, and the run modes from `run` to `plan`.
- Re-base `run` on the plan file: it requires `docs/plans/{stem}.md` and refuses to proceed without it.
- Move branch creation and the verification baseline entirely into `run`.
- Update the plan template for a two-command lifecycle (status values, planned-against vs base commit, pending verification rows).
- Update `SKILL.md` frontmatter, `README.md`, and `agents/openai.yaml` to describe three commands.

### Out of scope

- Changing `create` behavior or `references/brief-template.md`.
- Changing `references/git-pr-workflow.md`; branch, commit, push, and PR handling are unchanged, only relocated under `run`.
- Teaching `list` to show which briefs have plans (recorded as a follow-up).
- Plan staleness detection beyond existence, such as comparing the plan's recorded commit against HEAD.
- Committing the working-tree edit to `references/brief-template.md`.

## Assumptions and decisions

Confirmed with the user before planning:

1. **`run` owns all git work.** `plan` is side-effect-free apart from writing its own file, mirroring `create`. The verification baseline stays in `run` so it is measured immediately before implementation and cannot go stale between the two commands.
2. **`run` resolves against briefs, then requires the plan.** Both `plan` and `run` share one resolution rule over `docs/features/*.md`. A missing `docs/plans/{stem}.md` stops `run` with an instruction to run `plan` first.
3. **`plan` asks; `run` executes.** `assume` moves to `plan` and is removed from `run`. `run` keeps only the repository-safety prompt from `git-pr-workflow.md` about isolating unrelated uncommitted changes, which is a git-state question rather than a scope question.
4. **`.claude/skills/feat/` is synced but not committed.** It is the untracked local copy the `/feat` command loads. `.gitignore` does not cover `.claude/skills/`, so staging must name paths explicitly.
5. The brief labels two criteria `AC-1`; renumbered to AC-1, AC-2, AC-3 preserving the original wording.
6. The brief's closing line asks to merge. The skill's terminal action is the pull request, so this stops there.

## Acceptance-criteria traceability

| ID | Acceptance criterion | Implementation | Verification | Status |
|---|---|---|---|---|
| AC-1 | Adds a `plan` command that takes a brief name or number, same as `run` currently does, uses it to create a plan file, and then stops | `SKILL.md` command table row `plan / NAME [assume]`, usage block, and `## Plan a brief` with steps 1–4; `### Resolve a brief argument` states `plan` and `run` resolve identically | Inspected: `## Plan a brief` states it "writes exactly one file, `docs/plans/{stem}.md`, and stops" and "creates no branches, commits, or pull requests"; step 4 "Report and stop" repeats the prohibition. Vocabulary check shows `plan` at 42 references, up from 0 | pass |
| AC-2 | `plan` fails with a useful message if there is no correctly-named brief for it to work with | `SKILL.md` `### Resolve a brief argument`, shared by `plan` and `run` | Inspected: no match lists every available brief and stops "without creating files or branches"; a missing or empty `docs/features/` is called out plainly and answered with the `create` command that would scaffold one | pass |
| AC-3 | `run` only works if `plan` has already been run against the brief, otherwise it tells you to run `plan` first | `SKILL.md` `## Run a plan` step 1, "Load the brief and the plan" | Inspected: a missing `docs/plans/{stem}.md` stops with a literal message naming `/feat plan {name}`, followed by "Stop before preflight. Do not create a branch, change files, or open a pull request." The stop is step 1, ahead of preflight (2) and branch creation (3) | pass |

## Verification

No test, build, lint, or typecheck tooling exists in this repository; it ships Markdown
instructions and one YAML adapter. The checks below are ad-hoc structural commands run
from the repository root, not committed tooling.

| Command | Purpose | Baseline result | Final result |
|---|---|---|---|
| Relative-link resolution over `SKILL.md`, `README.md`, `references/*.md` | No documentation link breaks | 0 broken | 0 broken, unchanged |
| `head -1 SKILL.md` plus `grep -cE '^(name\|description):' SKILL.md` | Frontmatter intact | opens `---`, 2 fields | opens `---`, 2 fields, unchanged |
| Command-vocabulary counts across `SKILL.md`, `README.md`, `agents/openai.yaml` | `plan` is documented everywhere `run` is | create 15, plan 0, run 25, list 4 | create 18, plan 42, run 46, list 4, expected increase |
| `ruby -ryaml -e "YAML.safe_load(...)"` on `agents/openai.yaml` | Adapter still parses | parses ok | parses ok, unchanged |
| `grep -niE 'run mode'` across all instruction files | No mode name orphaned by the split | not run at baseline | 0 hits after rename |

`python3 -c "import yaml"` failed with `ModuleNotFoundError`; the Ruby equivalent was
used instead. No behavioral execution harness exists for skill instructions, so the
acceptance criteria are verified by direct inspection of the instruction text rather
than by running the workflow.

## Implementation steps

1. Rewrite the `SKILL.md` header and command table for `create` / `plan` / `run` / `list`.
2. Update argument rejection: `plan` accepts an optional `assume`; `run` accepts no second argument.
3. Generalize "Resolve a run argument" into a shared brief-resolution section used by `plan` and `run`.
4. Add `## Plan a brief` with the run modes and steps: read and frame, explore, write the plan, report and stop.
5. Rewrite `## Run a plan` with steps: load brief and plan (stop if absent), preflight, branch, baseline, implement, verify, PR, report.
6. Update the plan template: `Status: planned`, `Planned against commit`, `Base commit` filled by `run`, verification table with pending result columns.
7. Update the `SKILL.md` frontmatter description.
8. Update `README.md` throughout: intro, layout prose, per-command sections, arguments, run modes, planning, verification, invocation examples.
9. Update `agents/openai.yaml` `short_description` and `default_prompt`.
10. Re-run the four baseline checks and record final results.
11. Sync `.claude/skills/feat/` from the repository root without staging it.

## Files likely to change

- `SKILL.md` — the substantive change.
- `README.md` — user-facing documentation of the three commands.
- `agents/openai.yaml` — OpenAI adapter prompt text.
- `docs/features/0001-break-out-plan-and-implementation.md` — the brief, committed with the change.
- `docs/plans/0001-break-out-plan-and-implementation.md` — this plan.
- `.claude/skills/feat/**` — local install copy, synced but deliberately not staged.

## Deviations from this plan

Two edits went slightly beyond the scope written above, both forced by the split rather than chosen:

1. **`references/git-pr-workflow.md` was edited**, though listed out of scope. Its unrelated-changes rule branched on "Default mode" versus "Assume mode", and `run` no longer has a mode, so the rule was unreachable as written. Collapsed to a single instruction that `run` always asks, matching decision 3. Its related-work note now mentions the plan file `plan` leaves behind as well as the brief.
2. **"Run mode" was renamed to "planning mode"** in `SKILL.md`, `README.md`, and `git-pr-workflow.md`. Leaving the old name would have attached the word "run" to the one mode `run` does not have.

## Risks and follow-ups

- **Existing briefs have no plan.** Anyone with a half-finished `run` habit now needs an extra command. Mitigated by the explicit "run `plan` first" message.
- **Plan staleness is unchecked.** A plan written against an old commit still runs. Deferred by decision 4; a freshness check is a candidate follow-up.
- **No executable verification.** The acceptance criteria describe agent behavior, and this repository has no harness that executes skill instructions, so evidence is textual inspection only.
- **Follow-up:** teach `list` to show whether each brief has a plan, now that the two stages are separate.
- **Follow-up:** the uncommitted `references/brief-template.md` guidance line overlaps this change, since asking questions is now `plan`'s job. Left untouched at the user's direction.
