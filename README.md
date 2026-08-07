# Claude Development Workflow

**A disciplined operating system for building software with AI agents —
contracts before code, evidence over claims, and enforcement that doesn't
depend on anyone remembering the rules.**

Copy this repo into a new project and you get a complete loop: every slice
of work starts as a written contract, gets tests derived from that contract
before any code exists, and passes two human gates before it merges. The
rules are wired into hooks and CI rather than living in a document, so they
hold at 3am and under deadline — which is exactly when they'd otherwise be
skipped.

Built for Claude Code, but the method is harness-agnostic; only the
`.claude/` directory is Claude-specific.

**What makes it different from a prompt library:**

- **Tests are frozen.** They're written from the brief before the
  implementation, and a hook stops the builder from editing them to make
  them pass — the single failure mode that makes agent-written code look
  finished when it isn't.
- **The agent can't grade its own homework.** It can't claim "done" while
  tests fail, *or* while they can't be run at all. It can't edit its own
  hooks, config, or CI workflow. CI re-checks everything on a clean machine.
- **It runs unattended.** Gates push to your phone, a queue feeds the next
  slice automatically, and a deadman switch pages you when the agent goes
  silent — the one failure nothing else can report.
- **It compounds.** Every correction made twice gets promoted up a ladder:
  chat → rule → hook → skill → subagent → eval. Each project ends with the
  kit stronger than it started.

Full method and reasoning: [RUNBOOK.md](RUNBOOK.md).
One-page cheat sheet: [WORKFLOW.md](WORKFLOW.md).

## Start a new project

1. Copy this whole folder and rename it to your project.
2. Fill in PLAN.md (goal + milestones) and ARCHITECT.md (irreversible
   decisions only).
3. Fill the "What this project is" and Commands sections of CLAUDE.md.
4. `git init` and commit the docs.
5. Open Claude Code and run `/brief` to write the first slice — or fill
   the brief inside HANDOFF.md by hand.
6. Follow the loop in WORKFLOW.md.

## What's wired up out of the box

| Piece | What it does |
|---|---|
| `/brief` | Interviews you into a slice contract; refuses vague done checks |
| `/gate` | Runs the reviewer subagent, shows diff + fresh proof, records your verdict |
| `/capture` | Writes the slice log; mechanics pre-filled, judgment asked |
| `/handoff` | Rewrites HANDOFF.md with verified state + the next brief |
| `/next` | Pops the next brief from `plans/backlog/` so a run continues unattended |
| test-writer agent | Writes tests from the brief only, before implementation |
| no-slop-reviewer agent | Read-only diff review against templates/no-slop.md |
| danger_block hook | Blocks destructive shell commands (deletes, force-push, history rewriting, pipe-to-shell, credential reads) |
| guard_writes hook | Freezes existing tests, and blocks the agent from editing its own hooks, config, CI workflow, or `.env` |
| stop_verify hook | Agent can't claim "done" while tests fail — or while they can't be run at all; 3 strikes → forced re-plan |
| capture_commit hook | Heartbeat for the watchdog + appends each commit's stat and diff size |
| notify_gate hook | Pushes gates to your phone so you can approve without sitting at the machine |
| session_start hook | Warns at session start if the machinery is broken or a breaker flag is set |
| `.claude/watchdog.py` | Deadman switch — pages you when the agent goes *silent*, the one failure nothing else can report |
| `.github/workflows/gate.yml` | Re-runs the gate on a clean machine, and regression-tests the hooks themselves |

## Adapting the kit

- **Different test runner?** Set `test_command` in `.claude/config.json`.
  That is the only place it lives — hooks and CI both read it. (Hooks are
  invoked with `python3`; never change this to `python`, which does not
  exist on macOS or most modern Linux and would make every hook silently
  no-op while still looking like it works.)
- **Want it to run unattended?** Export `NTFY_TOPIC` (install the ntfy app
  and subscribe to an unguessable topic) or `NOTIFY_WEBHOOK`, run
  `python3 .claude/watchdog.py` in a second terminal, and queue briefs in
  `plans/backlog/`.
- **Adding an eval suite?** Set `eval_command` in config and it becomes
  gating — `stop_verify` and CI both run it. Keep it OFF push-triggered CI:
  evals cost money and are non-deterministic, so they belong on manual
  dispatch or a nightly schedule. Unit tests must never make a live model
  call, which is why CI needs no API keys.
- **New slop pattern caught twice?** Add a line to `templates/no-slop.md` —
  the reviewer agent reads it as its rubric, so reviews improve instantly.
- **New standing rule?** One line in CLAUDE.md. Rule keeps getting violated?
  Promote it to a hook.
- Templates are the single source of truth: skills and agents reference
  them, never copy them. Edit the template, everything downstream follows.

## Layout

```
CLAUDE.md  PLAN.md  ARCHITECT.md  HANDOFF.md   the project's head
RUNBOOK.md  WORKFLOW.md                        the method
templates/                                     blank forms (source of truth)
plans/briefs/  plans/logs/                     contracts and evidence
plans/backlog/                                 queued briefs for /next
artifacts/reviews/  artifacts/design/          gate records, visual contracts
evals/  tests/                                 quality checks
.claude/skills|agents|hooks + settings.json    the machinery
.claude/config.json                            single source: test cmd, caps, paths
.claude/watchdog.py                            deadman switch (run separately)
.github/workflows/gate.yml                     CI that doesn't trust the agent
```