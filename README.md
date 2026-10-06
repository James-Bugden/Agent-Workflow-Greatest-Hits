# Coding Workflow — Greatest Hits

A reusable `AGENTS.md` workflow template for running coding agents
(any tool, any model) against a real codebase in a disciplined,
ticket-driven way. Generalized from a working project's day-to-day workflow.

It covers the parts that actually matter for agent-driven development:

- **Change tiering** — not every change deserves full ceremony; classify
  before you start, and escalate the moment scope grows.
- **A phased workflow** — brainstorm → plan → (design, if user-visible) →
  TDD → review → finish, with clear per-tier skip rules.
- **Auto-merge rules** — agents merge their own PRs on green CI by default,
  with a short, explicit list of carve-outs (access control, payments) that
  always require a human.
- **Testing** — a strict red → green → refactor loop, which tests belong at
  which layer, access-control test requirements, rules against skipping or
  mocking your way to green, and how local runs map onto the CI backstop.
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

1. Copy `AGENTS.md` into the root of your project's repo.
2. Fill in every `{{DOUBLE_BRACE}}` placeholder — project description, tech
   stack, file-path conventions, issue-tracker ticket prefix, default
   branch name, test/lint/typecheck commands, and your own auto-merge
   carve-outs.
3. Delete any section marked **(optional)** that doesn't apply to your
   setup (e.g. local-only orchestration tooling, a design-system section,
   worktree isolation if you never run concurrent agents against one
   checkout).
4. Add your own team-specific rules under `Hard Rules` and
   `Data-Layer Rules` — the ones here are a reasonable default set, not a
   complete list.
5. Commit it. Tools that read `AGENTS.md` (Codex, Cursor, Copilot, Gemini
   CLI, Aider, and others) pick it up automatically. For a tool that expects
   a different filename, point it at this file rather than forking it, e.g.
   `ln -s AGENTS.md CLAUDE.md` (Claude Code) or `ln -s AGENTS.md GEMINI.md`.

## What this is *not*

This is a workflow document, not a framework or a package — there's nothing
to `npm install`. It's meant to be copied, edited, and owned by your repo,
the same way you'd own a `CONTRIBUTING.md`.

## How this template is tested

There is no application code here, so "tests" means validating the template
itself. `.github/workflows/validate.yml` runs on every push and PR to
`main` and checks that:

1. **Markdown lints clean** (`markdownlint-cli2`, config in
   `.markdownlint-cli2.jsonc`).
2. **Required sections are present** in `AGENTS.md` — including `## Testing`
   — so a future edit can't silently drop part of the contract.
3. **Placeholders are well-formed** — every `{{TOKEN}}` closes on its line
   and matches `{{UPPER_SNAKE_CASE}}`.
4. **No placeholders leak into filled-in copies** — anything under
   `examples/` must have zero `{{...}}` tokens left.

Run the same checks locally:

```bash
npx --yes markdownlint-cli2 "**/*.md" "#node_modules"
```

The workflow can't judge whether your filled-in `AGENTS.md` makes sense for
your project; that remains a human call. For *your* project's testing
rules, see the `## Testing` section of `AGENTS.md`.

## License

MIT — see `LICENSE`. Copy, adapt, and redistribute freely.
