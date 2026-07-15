# Feat

`feat` is a portable Agent Skill that executes a repository feature from a stored brief through planning, implementation, verification, and a GitHub pull request. After it creates a plan based on the brief, it adds it to the codebase too. As a result when a PR is cut, it contains both the prompt and the plan behind it.

## Package layout

```text
feat/
├── SKILL.md
├── README.md
└── agents/
    └── openai.yaml
```

`SKILL.md` is the single source of truth and follows the open Agent Skills format. `agents/openai.yaml` is optional OpenAI metadata; it improves the Codex and ChatGPT desktop experience and disables implicit invocation there. Claude ignores that adapter and uses the same `SKILL.md` directly.

No duplicate `CLAUDE.md`, prompt file, or `.claude/commands` wrapper is needed. Current Claude Code versions load Agent Skills natively and expose a skill directory named `feat` as `/feat`.

## Requirements in the target repository

- Git repository with a GitHub remote.
- Authenticated GitHub access through `gh` or another available GitHub integration.
- Feature briefs at `docs/features/SLUG.md`.
- Permission to create `docs/plans/SLUG.md`, branches, commits, pushes, and pull requests.

The skill follows repository-local instructions such as `AGENTS.md`, `CLAUDE.md`, and `CONTRIBUTING.md`.

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
/feat account-export yolo
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

In Codex CLI or the IDE extension, mention the skill with:

```text
$feat account-export
$feat account-export yolo
```

In the ChatGPT desktop app, select Feat from the Skills interface and provide the slug.

Copying the directory instead of symlinking also works. Symlinking keeps one checkout as the source of truth.

## Run modes

Default mode pauses once for material scope or behavior questions before coding. Yolo mode makes narrow, reversible assumptions and records them in the plan instead of pausing.

Yolo mode does not bypass client permissions, sandboxing, credentials, branch protection, CI, merge authorization, or deployment authorization.

## Distribution

Keep the source files at the root of a repository named `feat`; do not nest them inside another `feat` directory in that repository.

A packaged archive should contain one top-level `feat/` directory. The archive filename is not part of the open Agent Skills standard. This repository can be distributed through GitHub without committing a zip. Generate a zip only for an uploader or release artifact; the accompanying download uses `skill.zip` for ChatGPT packaging compatibility.

## Portability choice

Claude-specific frontmatter such as `disable-model-invocation`, `argument-hint`, and named `arguments` is intentionally omitted from the canonical `SKILL.md`. That keeps the file valid across skills-compatible clients. The description explicitly limits invocation, and OpenAI's adapter enforces `allow_implicit_invocation: false` for Codex and ChatGPT desktop.
