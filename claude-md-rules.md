# CLAUDE.md rules

How to write a `CLAUDE.md`, and how to clean up one that has grown.

## Why

`CLAUDE.md` is loaded into every session and every subagent, and re-read on
every turn. A 600-line file is paid for hundreds of times a day whether or
not the task needs it. Long files are also followed less reliably than short
ones.

## The test for every line

Keep a line in `CLAUDE.md` only if all three are true:

1. Claude would get something wrong without it.
2. It applies to most tasks in this repo, not one area or one workflow.
3. Claude can't learn it quickly from the code, `package.json`, the README
   or git history.

Anything that fails the test moves or goes.

## Where things go instead

| Content | Home | When it loads |
|---|---|---|
| Rules for most tasks | `CLAUDE.md` | Every session and agent |
| Rules for one part of the code | `.claude/rules/<topic>.md` with `paths:` frontmatter | When Claude touches matching files |
| Rules for one directory | `CLAUDE.md` inside that directory | When Claude reads files there |
| Step-by-step workflows (release, deploy, onboarding) | A skill, or `docs/runbooks/<name>.md` | When invoked or read |
| Reference material (tables, per-item notes, API details) | `docs/<topic>.md` | When Claude reads it |
| Why a decision was made, history, incident write-ups | `docs/decisions.md` or a changelog | When Claude reads it |
| Personal or machine-specific notes | `CLAUDE.local.md` (gitignored) | Every session, yours only |
| Things that must never happen | A hook or permission rule | Enforced, not remembered |

`@path` imports do not save anything: an imported file loads with the
`CLAUDE.md` that imports it. Use a plain-text pointer instead.

## Writing rules

- **Budget:** under 200 lines for a project file, under 100 for the global
  one.
- **One line per rule.** Say what to do, with the reason in a clause if it
  isn't obvious. Paragraphs of rationale go to `docs/`.
- **Pointers name a trigger.** Write "Before releasing an actor, read
  `docs/runbooks/release.md`", not "see docs". Claude follows a pointer only
  when it knows when the pointer applies.
- **Be specific.** "Run `npm test -- --run` before committing", not "test
  your changes".
- **No duplicates.** A rule lives in one file. Don't repeat the global file
  in a project file.
- **No stale content.** Remove dated notes, finished migrations and
  one-off fixes. Git history keeps them.
- **Emphasis is rare.** If everything is IMPORTANT, nothing is.

## Pipelines (create something new, release, deploy)

A pipeline is a fixed sequence of steps for a recurring job. The steps do not
belong in `CLAUDE.md`; the trigger and the hard gates do.

- **In `CLAUDE.md`:** one line per pipeline that names the trigger and the
  file, such as "Adding a new app? Follow `docs/runbooks/new-app.md` from
  step 1; don't work from memory or copy the last one."
- **Also in `CLAUDE.md`:** any gate that must hold even if the runbook isn't
  open, such as "Never deploy with failing tests." One line each.
- **In the runbook:** numbered steps, the exact command for each, how to
  check the step worked, and what to do when it fails. End with a "done
  means" checklist.
- **Runbook or skill?** Use a runbook in `docs/runbooks/` by default. Make it
  a skill when you want to start it by name (`/release`) or it needs scripts
  bundled with it.
- **Repeated mechanical steps become a script.** If every run creates the
  same files or runs the same commands, the runbook calls a script instead
  of describing the steps.
- **Gates that must never be skipped become hooks or CI checks.** A written
  rule is a reminder, not a guarantee.
- **Keep it current.** When a run shows a step is wrong or missing, fix the
  runbook in the same change.

## Cleanup procedure (instructions for Claude)

When the user asks you to clean up a `CLAUDE.md` with this file:

1. Read the whole `CLAUDE.md` and list its sections with line counts.
2. Sort every section, and every rule inside mixed sections, into one of:
   keep, move (name the destination from the table above), merge with a
   duplicate, or delete. Apply the three-part test.
3. Show the user that plan as a table, with the before and after line count,
   and wait for approval. Do not delete or move anything before that.
4. Move content verbatim into its new file first. Then replace it in
   `CLAUDE.md` with a one-line pointer that names its trigger.
5. Compress what stays to one line per rule.
6. Check nothing was lost: every removed rule is either in a destination
   file or on the approved delete list. `grep` the repo for references to
   moved headings and fix them.
7. Commit the moves and the trimmed file together.

## Template

```markdown
# <Project name>

<One or two sentences: what this is and who uses it.>

## Commands
- Install: `<command>`
- Dev: `<command>`
- Test: `<command>` (single test: `<command>`)
- Lint and typecheck: `<command>`

## Layout
- `<dir>/`: <what lives here>
- `<dir>/`: <what lives here>

## Rules
- <Rule that applies to most tasks.> <Reason, if not obvious.>
- <Rule.>

## Decided, don't reopen
- <Decision in one line.> Why: `docs/decisions.md#<anchor>`

## Never
- <Action that must not happen.>

## Read before you start
- Before <task>, read `docs/<file>.md`.
- Before <task>, read `docs/runbooks/<file>.md`.
- Working in `<dir>/`? Rules are in `.claude/rules/<topic>.md`.
```

A path-scoped rule file looks like this:

```markdown
---
paths:
  - "src/adapters/**"
---
- <Rule that only matters when editing these files.>
```

## Reference

- https://code.claude.com/docs/en/memory
