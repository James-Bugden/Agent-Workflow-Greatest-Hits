# Bugden's Coding Agent Workflow

A reusable `CLAUDE.md` workflow template for running coding agents
(Claude Code, or similar) against a real codebase in a disciplined,
ticket-driven way. Generalized from a working project's day-to-day workflow.

It covers the parts that actually matter for agent-driven development:

- **Change tiering** — not every change deserves full ceremony; classify
  before you start, and escalate the moment scope grows.
- **A phased workflow** — brainstorm → plan → (design, if user-visible) →
  TDD → review → finish, with clear per-tier skip rules.
- **Auto-merge rules** — agents merge their own PRs on green CI by default,
  with a short, explicit list of carve-outs (access control, payments) that
  always require a human.
- **Context management** — automatic compaction and handoff-doc triggers so
  a session interrupted mid-task (rate limit, timeout, crash) is always
  recoverable from a clean commit.
- **Environment-awareness** — the same file works whether the agent is
  running on a local machine with custom tooling, or in a fresh, isolated
  cloud container. It tells the agent how to detect which one it's in and
  what changes as a result.
- **Ambiguity handling** — agents keep moving on ambiguous requirements by
  default, documenting assumptions instead of stopping, reserving pauses for
  genuine, hard-to-reverse blockers.

## How to use this

1. Copy `CLAUDE.md` into the root of your project's repo.
2. Fill in every `{{DOUBLE_BRACE}}` placeholder — project description, tech
   stack, file-path conventions, issue-tracker ticket prefix, default
   branch name, and your own auto-merge carve-outs.
3. Delete any section marked **(optional)** that doesn't apply to your
   setup (e.g. local-only orchestration tooling, a design-system section,
   worktree isolation if you never run concurrent agents against one
   checkout).
4. Add your own team-specific rules under `Hard Rules` and
   `Data-Layer Rules` — the ones here are a reasonable default set, not a
   complete list.
5. Commit it. Any Claude Code (or compatible) session opened against your
   repo will pick it up automatically.

## What this is *not*

This is a workflow document, not a framework or a package — there's nothing
to `npm install`. It's meant to be copied, edited, and owned by your repo,
the same way you'd own a `CONTRIBUTING.md`.

## CI

`.github/workflows/validate.yml` runs a basic sanity check on this
template's own `CLAUDE.md` — markdown lint plus a check that no
placeholder tokens were accidentally left in a *derived, filled-in* copy
you might add under `examples/`. It does not (and can't) validate that a
downstream project's filled-in `CLAUDE.md` makes sense; that's a human
judgment call for each adopting repo.

## License

MIT — see `LICENSE`. Copy, adapt, and redistribute freely.
