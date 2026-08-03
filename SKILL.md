---
name: feat
description: Create, plan, and execute versioned repository feature briefs. The create command scaffolds a numbered brief, plan turns one brief into a reviewable implementation plan, and run implements an approved plan and stops at the pull request. Use only when the user explicitly invokes feat; not for general feature discussion, code review, merge, release, or deployment.
---

# Feat

Three commands, in order:

- `create` scaffolds a numbered feature brief from a template.
- `plan` turns one brief into a reviewable plan and stops.
- `run` implements an approved plan through: branch -> baseline -> implementation -> verification -> pull request

The plan is the review point. `create` and `plan` each write one file and stop; `run` performs every branch, commit, push, and pull-request action. The pull request is the terminal action. Never merge, release, or deploy as part of this skill.

## Parse the request

Read arguments from the user request or an `ARGUMENTS:` block appended by the client. The first argument is the command:

| Command | Arguments |
|---|---|
| `create` | `NAME ["description"]` |
| `plan` | `NAME [assume]` |
| `run` | `NAME` |
| `list` | none |

Match the command case-insensitively. When the first argument is missing or is not one of these commands, print the usage below along with the sorted stems from `docs/features/*.md`, and stop.

```text
/feat create NAME ["description"]   scaffold a new brief
/feat plan NAME [assume]            turn a brief into a reviewable plan
/feat run NAME                      implement an approved plan
/feat list                          list available briefs
```

Never treat a bare first argument as a brief name. Earlier versions accepted `/feat NAME`; that form is no longer supported.

Reject extra arguments, any second argument to `plan` other than `assume`, matched case-insensitively, and any second argument to `run` at all. Earlier versions accepted `run NAME assume`; `assume` now belongs to `plan`, which is where clarifying questions are asked.

`list` prints the sorted stems from `docs/features/*.md` and stops. Say so plainly when there are none.

### Names and paths

Accept only a name matching `[A-Za-z0-9][A-Za-z0-9_-]*`. Reject path separators, `..`, absolute paths, and path overrides.

Briefs live at `docs/features/{NNNN}-{name}.md`. For `docs/features/0007-account-export.md`:

- Stem: `0007-account-export`
- Number: `0007`
- Name: `account-export`
- Plan: `docs/plans/0007-account-export.md`
- Branch: `feat/0007-account-export`

### Resolve a brief argument

`plan` and `run` resolve their argument identically. Match it against the stems of `docs/features/*.md`, case-insensitively, in this order:

1. Exact stem, such as `0007-account-export`.
2. An all-digit argument, compared numerically against each leading number, so `7`, `07`, and `0007` all resolve to `0007-account-export`.
3. The name alone: the stem with its `NNNN-` prefix removed, or a stem that never had one.

With no match, list every available brief and stop without creating files or branches. When `docs/features/` is missing or contains no briefs, say that plainly and name the `create` command that would scaffold one. With more than one match, list only the matching stems and stop. Briefs written before numbering existed carry no prefix and resolve by rules 1 and 3.

## Preserve repository and user intent

These rules apply to `plan` and `run`. `create` writes one templated file into `docs/features/` and needs none of them; it must not read repository instructions, product code, or documentation.

Follow applicable `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `README.md`, and scoped repository instructions. More specific instructions override broader ones.

Treat the brief as the source of truth for intent, and the approved plan as the source of truth for what `run` builds. Do not implement adjacent improvements without explicit approval. Never discard, reset, clean, stash, overwrite, or commit unrelated user work. Never force-push.

Before branch changes, commits, pushes, or pull-request work, read [references/git-pr-workflow.md](references/git-pr-workflow.md).

## Create a brief

`/feat create NAME ["description"]`

Scaffold one brief and stop. Do not create branches, commits, or pull requests, and do not implement anything.

1. In a single tool block, run `mkdir -p docs/features && ls docs/features` and read [references/brief-template.md](references/brief-template.md). These are independent; issue them together rather than in sequence. Read nothing else.
2. Strip any leading `NNNN-` or `NNNN_` from the name; this command assigns the number. From the listing: if a brief with the same name already exists at any number, stop and report its path, since names stay unique so that `plan` and `run` can resolve them. Otherwise take the highest leading number and add one, starting at `0001` when no numbered brief exists. Zero-pad to four digits, widening only when the number no longer fits. Never reuse a gap.
3. Replace `{Title}` with the name, hyphens and underscores as spaces and the first letter capitalized. Without a description, write the template unchanged; its italic placeholders are the author's prompts. With a description, replace the placeholders under Outcome, Scope, and Acceptance criteria with content drawn from it, and leave every section the description does not support as a placeholder. Never invent constraints, dependencies, or acceptance criteria the user did not state.
4. Write `docs/features/{NNNN}-{name}.md`, report the path and the `plan` command that would turn it into a plan, and stop.

## Plan a brief

`/feat plan NAME [assume]`

Resolve the argument to a brief, then follow [references/plan.md](references/plan.md).

## Run a plan

`/feat run NAME`

Resolve the argument to a brief, then follow [references/run.md](references/run.md).
