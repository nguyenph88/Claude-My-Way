# Subagent model baseline

Rules for which model a Claude Code subagent runs on, so that delegated work
stops inheriting the most expensive model in the session.

This is a baseline. Adjust the table when something doesn't work.

## Why

A subagent with no model set runs on the main session's model. With the
session on a top-tier model, every search, recon run and copy-the-template
build runs on it too. In the usage this baseline was measured against, the
`general-purpose` agent was about half of all spend, 96% of everything ran on
one top-tier model, and 70% of agent spend was builds that followed an
existing template.

## How to apply (instructions for Claude)

When the user asks you to apply this file on a machine:

1. Read `~/.claude/CLAUDE.md` and `~/.claude/settings.json` first. Create
   either one if it is missing.
2. Copy everything between the `BEGIN` and `END` markers below into
   `~/.claude/CLAUDE.md`, markers included. If a block with these markers is
   already there, replace it; never add a second copy.
3. In `~/.claude/settings.json`, merge the `env` key from "Safety net" below
   into the existing `env` object. Leave every other setting alone.
4. Show the user both diffs before writing, then write them.
5. Tell the user the setting takes effect in new sessions.

Do not apply it to a project's own `CLAUDE.md`; the rules are meant to be
global.

## The rules

<!-- BEGIN subagent-model-baseline -->
## Subagent model rules

Every Agent tool call names a `model`. A subagent never inherits the session
model by omission.

| Task | Model | Effort |
|---|---|---|
| Find a file or symbol; summarize docs, logs or files | haiku | low |
| Web or API recon, probing a site, competitor research | sonnet | medium |
| Build that follows an existing template or sibling | sonnet | medium |
| Run a script, crawl or test suite and report back | sonnet | low |
| Browser form-filling and web setup | sonnet | low |
| Routine code review, implementation plan | sonnet | medium |
| First-of-a-kind build, hard debugging, architecture | main session; opus only if it must be delegated | default |
| Top-tier model above opus | only when the user asks in that message | n/a |

1. **Small lookups stay inline.** About three tool calls or fewer: do it
   yourself, no agent.
2. **Bulk reading goes to a cheap agent.** Reading many files or pages inline
   on an expensive session model costs more than delegating it.
3. **Hard thinking stays in the main session.** It already has the context
   and the strongest model. Delegate first-of-a-kind work only when it has to
   run in the background or in parallel, then use opus and say why in one
   line before spawning.
4. **Start cheap, escalate once.** If a result comes back thin or wrong,
   rerun it one tier up and tell the user.
5. **No forks for reading, recon or form-filling.** A fork always runs on the
   session model and ignores a model override.
6. **Every brief caps the report.** Ask for the conclusion plus `file:line`
   references, with a word limit.
7. **At most three agents in parallel** without asking the user first.
8. **Skills and plugins that spawn agents follow the same table.**
<!-- END subagent-model-baseline -->

## Safety net

Rules are followed only as well as the model follows them. This setting
catches the case where a call names no model: the `general-purpose` agent
then runs on Sonnet instead of the session model.

```json
{
  "env": {
    "CLAUDE_CODE_SUBAGENT_MODEL": "sonnet"
  }
}
```

It does not change the built-in Explore and Plan agents, and a model named in
the call still wins. Adding `"CLAUDE_CODE_SUBAGENT_MODEL_FORCE": "1"` forces
Sonnet onto every subagent, which also blocks the opus row; add it only if
the rules keep leaking.

## Checking that it works

Subagent transcripts are in
`~/.claude/projects/<project>/<session>/subagents/agent-*.jsonl`, each with a
`.meta.json` holding the agent type and description. Each assistant message
records its `model` and `usage`. Summing those by agent type and model shows
whether agents still run on the session model.

## Reference

- Subagent docs, including the model resolution order:
  https://code.claude.com/docs/en/sub-agents
