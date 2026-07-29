# Break out plan and implementation

## Outcome

Right now there's a 'create' step to create a brief, and then a 'run' step, which both creates the plan and then imlements it.

Break out 'plan' from implementation so that after the plan is created, it stops, and the user then has to call 'run' seperately to do the implementation. So the steps become:

- `create` creates the brief file. Accepts an optional descriptions. The user then fills in the brief.
- `plan` takes the brief created by the user, and uses it to make an implementable plan to fulfill it. The plan is written to /docs/plans/*, where the user can review or revise it.
- `run` takes the plan and implements it--unlike now, where `run` starts from the brief file.

## Scope

### In scope

- Add a `plan` command
- Edit `run` to base its work on the plan file, rather than starting from the brief.

### Out of scope

*Adjacent work this feature explicitly does not do.*

## Acceptance criteria

*Independently checkable, one stable ID each. These become the traceability rows in the plan and the pull request.*

- **AC-1** — *adds a `plan` command that takes a brief name or number, same as `run` currently does*, and uses it to create a plan file--and then stops.
- **AC-1** —*`plan` fails with a useful message if there is no correctly-named brief for it to work with
- **AC-2** — *`run` also should only work if `plan` has already been run against the brief, otherwise it should tell you to run `plan` first*

## Constraints and dependencies

None known

Ask any questions needed, then implement, but a PR, and merge