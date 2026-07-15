# Feat

`feat` is a portable Agent Skill that executes one repository feature from a versioned brief through branch isolation, planning, implementation, verification, commit, push, and a GitHub pull request.

The brief, implementation plan, code, and verification evidence are versioned together, creating an auditable chain from the original request to the resulting pull request. The workflow stops at the pull request. It never merges, releases, or deploys.

## Package layout

```text
feat/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    └── git-pr-workflow.md
```

`SKILL.md` is the cross-platform control plane and the single source of truth for the feature workflow. `references/git-pr-workflow.md` contains detailed branch, commit, push, and pull-request handling so the core instructions remain focused. `agents/openai.yaml` is optional OpenAI metadata that improves presentation and disables implicit invocation in supported OpenAI clients.

No duplicate prompt, `CLAUDE.md`, or `.claude/commands` wrapper is required. Install the same `feat` directory in each client's skills location.

## What the skill does

For a brief at `docs/features/SLUG.md`, the skill:

1. Reads the brief and assigns stable IDs such as `AC-1` to its acceptance criteria.
2. Inspects repository instructions, existing work, and relevant implementation patterns.
3. Creates or resumes an isolated feature branch.
4. Runs practical pre-change checks and records a verification baseline.
5. Writes or updates `docs/plans/SLUG.md` before changing product code.
6. Maps every acceptance criterion to planned implementation and verification evidence.
7. Implements the smallest coherent change that satisfies the brief.
8. Reruns relevant checks, compares final results with the baseline, and records any regressions or limitations.
9. Commits and pushes only the intended work, then opens or updates exactly one pull request.
10. Stops and reports the result without merging, releasing, or deploying.

## Requirements in the target repository

- A Git repository with a GitHub remote.
- Authenticated GitHub access through `gh` or another available GitHub integration.
- Feature briefs at `docs/features/SLUG.md`.
- Permission to create `docs/plans/SLUG.md`, branches, commits, pushes, and pull requests.
- Relevant build, test, lint, typecheck, or other verification tooling when the repository provides it.

The skill follows repository-local instructions such as `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, and scoped instruction files. More specific repository instructions take precedence over broader ones.

## Install for Claude Code

Personal installation:

```sh
mkdir -p ~/.claude/skills
ln -s /absolute/path/to/feat ~/.claude/skills/feat
```

Project-scoped installation:

```sh
mkdir -p /path/to/project/.claude/skills
ln -s /absolute/path/to/feat /path/to/project/.claude/skills/feat
```

Invoke it with:

```text
/feat account-export
/feat account-export assume
```

## Install for Codex or ChatGPT desktop

Personal installation:

```sh
mkdir -p ~/.agents/skills
ln -s /absolute/path/to/feat ~/.agents/skills/feat
```

Project-scoped installation:

```sh
mkdir -p /path/to/project/.agents/skills
ln -s /absolute/path/to/feat /path/to/project/.agents/skills/feat
```

In Codex CLI or an IDE integration, invoke it with:

```text
$feat account-export
$feat account-export assume
```

In ChatGPT desktop, select **Feat** from the Skills interface and provide the brief slug, optionally followed by `assume`.

Copying the directory instead of symlinking also works. Symlinking keeps one checkout as the source of truth across clients.

## Arguments

The required first argument is the brief slug: the filename in `docs/features/` without `.md`.

The slug may contain letters, numbers, underscores, and hyphens. Paths, path separators, `..`, absolute paths, extra arguments, and any second argument other than `assume` are rejected.

```text
/feat SLUG [assume]
$feat SLUG [assume]
```

When no slug is supplied, the skill lists the available briefs and stops.

## Run modes

### Default

Default mode asks one consolidated set of blocking questions before planning or implementation when an ambiguity would materially change scope, observable behavior, acceptance criteria, data handling, security, privacy, migration, or compatibility.

It does not ask about matters already settled by the brief or clear repository conventions.

### Assume

`assume` mode does not pause for clarification. It chooses the narrowest reasonable and reversible interpretation, records every material assumption in the plan, and defers adjacent work rather than expanding scope.

`assume` does not bypass client permissions, sandboxing, credentials, repository policy, branch protection, verification, safety constraints, or hard blockers. The skill stops when safe completion requires unavailable information, access, tooling, credentials, or repository state.

## Planning and traceability

Before changing product code, the skill writes or updates `docs/plans/SLUG.md`. The plan records:

- The source brief and base commit.
- Intended outcome and scope boundaries.
- Assumptions and decisions.
- One traceability row for every acceptance criterion.
- Exact baseline commands and their results.
- Sequenced implementation steps, expected files, risks, and follow-ups.

Each acceptance criterion begins as `planned` and ends as `pass`, `fail`, or `blocked`, with actual implementation references and direct verification evidence. The skill never marks a criterion `pass` without evidence.

## Verification baseline

Before implementation, the skill runs the narrowest practical existing checks that are relevant to the feature and can be rerun afterward. It records the base commit, exact commands, results, and identifiable pre-existing failures.

After implementation, it reruns those commands along with checks for the new behavior and compares the results. A failure is called pre-existing only when a materially equivalent failure was observed before the change.

The skill does not expand scope to fix unrelated baseline failures. When a meaningful baseline is unavailable, prohibitively expensive, flaky, or environmentally blocked, it records that limitation and does not claim that no regressions were introduced.

## Branch and pull-request behavior

The skill never implements or commits directly on the repository's default branch. It follows the repository's branch convention or uses `feat/SLUG`, safely resumes matching work, and avoids duplicate branches or pull requests.

It preserves unrelated user work, never force-pushes, does not bypass hooks, and stages only intended paths. A pull request is ready for review only when all required criteria pass and no relevant new failure remains. Otherwise, the skill may preserve the work in a clearly blocked draft pull request when repository policy permits it.

After opening or updating the pull request, the workflow stops. Merge, release, deployment, approval, and auto-merge are separate actions outside this skill.

## Distribution

Keep the source files at the root of a repository named `feat`; do not nest them inside another source-level `feat` directory.

A packaged archive should contain one top-level `feat/` directory. The archive filename is not part of the open Agent Skills format. The source repository does not need to commit a zip. Generate one only for an uploader or release artifact; this downloadable package uses `skill.zip` for ChatGPT packaging compatibility.

## Portability

The canonical `SKILL.md` uses only portable frontmatter and shared instructions. Client-specific invocation metadata belongs in adapters such as `agents/openai.yaml`, not in duplicated workflow prompts. This keeps the behavior aligned across Claude, Codex, and other clients that support the Agent Skills format.
