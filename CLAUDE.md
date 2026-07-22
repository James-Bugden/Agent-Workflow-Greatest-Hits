# {{PROJECT_NAME}} — Agent Instructions

> **Template notice:** This file is a generalized coding-agent workflow,
> extracted from a working project (originally "HireSign"). Every
> `{{DOUBLE_BRACE}}` token is a placeholder — fill them in for your project
> before using this file as your repo's `CLAUDE.md`. Sections marked
> **(optional)** describe machinery that was specific to the original
> project's local setup; keep, adapt, or delete them.

## Project Overview
{{ONE_PARAGRAPH_PROJECT_DESCRIPTION}}

## Tech Stack
- Frontend: {{FRONTEND_STACK}}
- Backend: {{BACKEND_STACK}}
- Auth: {{AUTH_PROVIDER}}
- Deployment: {{DEPLOY_TARGET}} — state clearly whether a merge to your
  default branch goes live automatically or requires a manual promote step.
  Getting this wrong is the single most common cause of accidental
  production incidents in agent-driven workflows.
- Database migrations: state clearly whether your deploy pipeline runs
  migrations automatically, or whether they must be applied as a separate,
  explicit step (and by what tool/command).
- Issue tracker: {{ISSUE_TRACKER}} (ticket prefix: `{{TICKET_PREFIX}}`)
- Version control: {{VCS_HOST}}
- Package manager: {{PACKAGE_MANAGER}}

## Key Conventions
- Components / modules: {{COMPONENTS_PATH}}
- Hooks / shared logic: {{HOOKS_PATH}}
- Pages / routes: {{PAGES_PATH}}
- Utilities: {{UTILS_PATH}}
- Types: {{TYPES_PATH}}
- Styling approach: {{STYLING_CONVENTION}}
- Schema changes: {{SCHEMA_CHANGE_CONVENTION}} (e.g. "migration files in
  `db/migrations/` only, never via the provider's dashboard")
- {{ANY_OTHER_HARD_CONVENTION}} (e.g. "every new table must have RLS
  policies")

## Brand / Design Tokens (optional — delete if not applicable)
If your project has a design system, name the single source of truth here
(e.g. `src/index.css :root`, a Figma library, a tokens package) and require
agents to read it before producing any UI mockup or styled component,
rather than approximating from memory or from this file. Treat a live
reference implementation (a shipped page/component that best expresses the
current brand) as authoritative over prose descriptions when the two
disagree — and note that the prose is what gets corrected, not the shipped
code.

---

## Execution Environments — Where This Workflow Runs

Coding agents run in fundamentally different places: your local machine (with
its own tools, worktrees, and shell binaries) versus a managed cloud
container (a fresh, isolated clone with no shared state). **Detect which one
you're in before applying any path-, tool-, or worktree-specific rule.**
Following local-only rules in a cloud session — or vice versa — is the most
common way a workflow like this "doesn't work properly" in one environment.

### How to detect where you are
- **Local machine** if the working directory matches your known local repo
  path and any custom local tooling (see *Parallel-Agent Safety* and
  *Parallel Heavy Work*, both optional, below) resolves on `PATH`.
- **Managed cloud** (a hosted coding-agent session — web app, mobile app, or
  a CI/Action runner) if the repo was cloned fresh into an isolated
  container: an unfamiliar path, outbound network via a proxy, and none of
  your local-only binaries present.

### What applies where

| Concern | Local machine | Managed cloud |
|---|---|---|
| **Working tree isolation** | Use a dedicated worktree per session if multiple agents/bots share one checkout (see *Parallel-Agent Safety*) | Skip it — the container is already an isolated clone |
| **Pushing** | Through whatever local push gate you've set up (optional) | `git push -u origin <branch>` — let CI be the guard |
| **Quality gate** | Local gate (optional) **+** CI on the PR | CI on the PR (universal backstop) |
| **Parallel heavy work** | Local orchestration tooling (optional) | In-session subagents instead |
| **Design review** | A local browser-based review tool (optional) | Fall back to an inline chat-based review/approval step |
| **PR / issue-tracker ops** | Native CLIs (`gh`, etc.) | Equivalent MCP tools, if `gh` isn't installed in the cloud |

### Environment-agnostic rules (apply EVERYWHERE)
The Hard Rules, Change Tiering, the workflow phases, brand tokens, and
"never push to the default branch directly" apply in both environments
without exception. Only the *machinery* changes, never the contract.

---

## Change Tiering: Classify Before Starting

Not every change deserves full ceremony. Before Phase 1, classify the ticket
into a tier. The tier decides which parts of the workflow apply.

**Anti-bias rule.** Trivial work feels faster to start, so the natural pull
is to under-classify. Resist that. Classify by what the change actually
touches — files, layers, data, surfaces — not by how the ticket title
sounds. If between two tiers, pick the higher one. State the tier and
reasoning in one line before doing anything else.

### Considered classification (do this out loud)
Before picking, name:
1. Which files will this touch? One file is likely Trivial or Standard.
   Many files is Heavy.
2. Which layers? Copy only is Trivial. UI plus state is Standard. UI plus
   state plus database or access-control is Heavy.
3. User-facing or internal? User-facing plus data means escalate.
4. Reversible if wrong? If no, the answer is Heavy.

If you cannot answer any of these without reading code first, the answer is
at least Standard. Trivial requires confidence, not optimism.

### Tier 1: Trivial
**Examples:** copy edit, colour tweak to an existing token, link fix, prop
rename inside one component, dependency bump with no API change.

**Workflow modifications:**
- Phase 1: ticket only. No clarifying-question pause. If it is ambiguous, it
  is not Trivial.
- Phase 2: SKIP. No plan file.
- Phase 2.5: if user-visible, a lightweight inline check-in before code; a
  pure copy edit needs no design review. Non-visible changes: SKIP.
- Phase 3: SKIP TDD if there is no logic change. The "test-first" rule
  applies to logic changes only, not to copy or style.
- Phase 4: one-stage review. Does it match the ticket?
- Phase 5: open PR, self-merge on green required checks per Auto-Merge Rules
  below.

**Always keep:** ticket reference, brand/design tokens, mobile-first (if
applicable), any localization requirement.

### Tier 2: Standard
**Examples:** new component, new data-fetching hook, form change,
single-feature UI, refactor inside one file.

**Workflow modifications:**
- Phase 1: full Phase 1.
- Phase 2: light plan — three bullets in the ticket: what changes, where it
  mounts, what could break. Skip the standalone plan file.
- Phase 2.5: lightweight design check-in for a single user-visible surface.
  A new page or redesign escalates to Tier 3.
- Phase 3: full TDD as written.
- Phase 4: full two-stage review.
- Phase 5: full Phase 5. Self-merge on green required checks per Auto-Merge
  Rules, unless the change hits a carve-out (then human-merge).

### Tier 3: Heavy
**Examples:** new page, schema change, any access-control-touching query,
auth flow, cross-cutting refactor, anything customer-data-adjacent,
serverless/edge functions.

**Workflow modifications:** none. Run the full methodology as written below.
This is the default if you cannot confidently classify lower.

### Mid-work escalation (mandatory)
If during a Trivial or Standard task you discover the change touches
access control, auth, schema, or expands beyond the planned scope, STOP and
re-classify upward before continuing. Pause, write a plan file, then resume.

---

## Workflow (MANDATORY per tier)
Follow all phases applicable to your tier (see Change Tiering above). Do not
skip phases your tier requires.

### Phase 1 — Brainstorm
Before writing any code:
- **If no ticket exists for this work, create one first**, via your issue
  tracker's API/MCP tool if available. Use this format:
  - Title: imperative, scoped (e.g., "Fix analytics-export 401 from API key
    mismatch")
  - Description must include: **Current state** (what's broken/missing
    today, with file:line refs), **Why it matters** (user impact, blast
    radius), **Proposed change** (before/after diff or bullet list),
    **Acceptance criteria** (testable outcomes)
  - Capture the returned ticket ID — it's used in every commit, branch,
    file path, and PR title from here on
- If a ticket already exists, read it fully first
- Move the ticket to "In Progress"
- Identify all files that will be touched, edge cases, constraints, and any
  schema changes needed
- **Write the Acceptance Criteria in plain English a non-engineer could
  verify** — outcome language ("user sees an error instead of a blank
  screen"), not implementation language. This list is the contract for any
  goal-confirm step and the final review.
- **Do not pause mid-build to ask clarifying questions** beyond the genuine
  blockers in *Handling Ambiguity* below. Make the best reasonable choice,
  log the assumption, and keep going. Bank open questions and surface them
  at the next human checkpoint (design approval or final review), not by
  interrupting mid-task.

### Phase 2 — Plan
Write a plan to `{{PLAN_DOCS_PATH}}/{{TICKET_PREFIX}}-XXX.md`:
- Exact file paths to create or modify
- Acceptance criteria per file
- Test cases that must pass
- Migration files required (if any)
Keep tasks small — 2 to 5 minutes of execution each.
Commit the plan file before starting implementation.

**Mirror the plan to your issue tracker**, if it supports comments — post
the full plan as a comment so the ticket always reflects the current plan.
If the plan changes mid-implementation, update the file and post the diff
as a follow-up comment.

Do not proceed to Phase 3 until the plan is committed AND mirrored.

### Phase 2.5 — Design (conditional, user-visible surfaces only)
Run between Plan and TDD whenever the change touches a user-visible surface
(pixel, layout, label, empty/loading/error state, new screen or component).
**Presumption:** any diff touching a UI file or component directory is
user-visible even if the ticket says "fix" or "refactor" — only an explicit
human waiver skips this. Backend, data, logic, types, tooling, and
migration-only work skip this phase.

- For a single surface: produce a lightweight wireframe or mockup (even a
  quick sketch or a described layout) and get a pick before writing code.
- For a new page, new major component, or redesign: produce multiple
  options and route them through whatever design-review tooling you have —
  or, absent that, ask the human directly which option to build.
- Order: structure first (no copy commitment), then real copy in the
  target language(s) — mandatory whenever any new or changed string ships —
  then one cheap layout touch-up if a string breaks it, then freeze.

**Approval gate.** Do not start Phase 3 on a user-visible surface until a
human has explicitly approved both the design option and the final copy.
A blanket "go build it" before any design exists does not authorize
skipping this. Mirror the chosen option and copy decisions to the ticket,
quoting the approval, before Phase 3.

### Phase 3 — TDD Implementation
Iron law: no production code without a failing test first.
Red, then Green, then Refactor, for every task.
Compact your context / take stock after each completed task before starting
the next.

### Phase 4 — Review
Two-stage review before opening the PR:
1. Spec compliance — does it match the plan?
2. Code quality — naming, error handling, edge cases, no dead code
3. Design gate: if the diff touches a user-visible surface, confirm the
   ticket has a Phase 2.5 design comment with explicit approval. If not,
   the gate was skipped — stop and run Phase 2.5.

### Phase 5 — Finish
- Run the full test suite — all tests must pass
- Commit: `{{TICKET_PREFIX}}-XXX: description`
- Open PR: title starts with `{{TICKET_PREFIX}}-XXX`, body closes the ticket
- Self-merge once all required checks pass, per Auto-Merge Rules — unless
  the change hits a carve-out, in which case leave it for a human and say
  so on the PR
- Comment on the ticket linking to the PR
- Write a handoff doc (see *Context Management* below) if the session is
  ending mid-work
- If scope changed during implementation, update the ticket description so
  it reflects what actually shipped

---

## Auto-Merge Rules
**Default: the agent self-merges once all REQUIRED CI checks pass.** The
agent merges its own PR the moment CI goes green on every required check —
it does not wait for a human to click merge. This is the default for every
PR EXCEPT the carve-outs below.

"Required checks" means the required CI jobs only. A red or pending
**non-required / advisory** job (e.g. a known-flaky visual job) does not
block the merge — but never merge while any *required* check is red,
pending, or still running.

**Green is the trigger — do not pause for confirmation.** When the agent is
actively driving a PR to merge, a green required-check result is *itself*
the signal to merge. Do not end the turn, go quiet, or wait for a human
"go ahead" once checks are green.

**No block on merging.** A green required-check result is the only gate.
Draft status, the absence of a human "go ahead", or the agent's own
hesitation are **never** blocks.

**Carve-outs — define these explicitly for your project, never self-merge:**
List the highest-blast-radius categories where a mistake is either
unrecoverable or silently harmful. The canonical examples are:
- **Access-control / permission policies** (e.g. database row-level
  security) — a wrong policy can silently expose or hide user data
- **Payment or billing logic** — money movement can be unrecoverable if
  wrong

Add your own project-specific carve-outs here. Everything not on this list
self-merges on green — including auth/session handling, backend functions,
schema/new-table migrations, multi-file shared-state refactors, and
security-tagged tickets, unless you choose to add them explicitly.

When in doubt about whether a change touches a carve-out category: do NOT
self-merge — leave it for a human and say so on the PR.

---

## Rollup Tickets (optional pattern)

A rollup ticket bundles N findings under one parent ticket. Two valid
shapes — pick the right one based on the work:

**Mechanical rollup** (e.g., 15 instances of the same missing pattern).
N instances of the same fix.
- One PR per instance. PR title format: `{{TICKET_PREFIX}}-XXX: <action>
  <target>`.
- Close the rollup only when ALL instance PRs merged AND any prevention
  check (e.g. a lint rule or CI script) is in place.

**Heterogeneous rollup** (e.g., 4 different improvements sharing a root
cause). N distinct fixes.
- Each sub-item gets its own labelled sub-section in the description.
- Each sub-item resolves to exactly ONE of:
  - (a) An attached PR (sub-item identifier in the PR title or body)
  - (b) A comment on the rollup explicitly cancelling or deferring the
    sub-item with rationale
  - (c) A spun-off follow-up ticket linked in a comment

**Never close a rollup with sub-items that have no resolution path visible
from the ticket page.** "All shipped, trust me" closes are banned. If a
reader can't tell from the ticket alone whether a sub-item shipped, the
rollup is not done.

---

## Audit / Architecture Cadence (optional pattern)

When an audit sweep ships an unusually large number of tickets in a single
day, the next available work day is an architecture day: no new audit-tag
tickets are filed or worked until at least one architecture-tag ticket has
landed.

Rationale: audit-style tickets close fast and compound — without
architecture work in the mix, a codebase accumulates surface-level fixes
without addressing root causes.

---

## Context Management (Automatic)
Do not wait to be asked. Manage context proactively throughout every
session.

### Automatic compaction triggers
Compact context automatically after each of these events:
- A plan is written and committed
- Each individual implementation task is complete
- Review is complete
- Any time you estimate you are past ~60% of your context window

### Automatic handoff triggers
Write a handoff doc to `{{HANDOFF_DOCS_PATH}}/{{TICKET_PREFIX}}-XXX.md`
AND post it to the ticket automatically if:
- You estimate you are past ~80% of your context window
- The session has been running unusually long
- You are about to start a new major phase and the session is already long
- You receive any indication the session may be interrupted

The ticket comment is the recovery surface — a fresh session resuming from
the ticket alone (no repo context) must be able to continue the work.

Handoff format:
```markdown
---
ticket: {{TICKET_PREFIX}}-XXX
status: in-progress | complete
restored: false
---

## Goal
One sentence summary of the ticket

## Current state
What has been completed so far

## Key decisions
Architectural choices made and why

## Modified files
List of every file changed with a one-line description of what changed

## Next steps
Exactly what to do next — specific enough that a fresh session can continue
without any other context

## Blockers
Anything requiring human input before work can continue
```

### Resuming from a handoff
When starting a new session with an existing handoff:
1. Read the handoff doc
2. Set `restored: true` in the front matter
3. Move the file to an archive location
4. Commit the archive move
5. Continue from Next Steps

---

## Rate Limit / Interruption Protection (Critical)
Interruptions mid-task leave a repo in an inconsistent state. These steps
ensure any interruption is recoverable.

### Before starting Phase 3 — confirm all three are done:
1. Plan file is written AND committed
2. Test file stubs are created AND committed (even if empty)
3. Handoff doc skeleton is written AND committed

### During Phase 3 — after each task:
1. Compact context
2. Commit all changes — never leave uncommitted work
3. Run the test suite — confirm tests still pass before moving to the next
   task
4. If any task takes unusually long, write an interim handoff before
   continuing

### Commit discipline
Never batch more than one task into a single commit.
Each commit should be a passing, deployable state.
This means an interruption after any commit leaves the repo working.

---

## Parallel-Agent Safety — Worktree Isolation (optional, local machine only)

If a bot or multiple agent sessions can push to the same local checkout
concurrently, working directly in the top-level tree risks two failure
modes: silent data loss (uncommitted edits vanish when another agent's
checkout touches the same working tree) and non-deterministic branch
contents (a freshly created branch unexpectedly contains commits from an
unrelated task).

**The rule (local machine only):** every session working on this repo
locally operates in its own worktree checkout, never in the shared
top-level working tree.

```bash
git fetch origin {{DEFAULT_BRANCH}}
git worktree add .worktrees/{{TICKET_PREFIX}}-XXX-slug -b {{BRANCH_PREFIX}}/{{TICKET_PREFIX}}-XXX-slug origin/{{DEFAULT_BRANCH}}
cd .worktrees/{{TICKET_PREFIX}}-XXX-slug
```

**Does NOT apply in a managed cloud session** — there the repo is cloned
fresh into an isolated container, so there is no shared working tree to
collide with. Work directly in the cloned checkout.

If you discover you forgot and already started editing in the top-level
checkout: stop, `git stash` what you have, create the worktree, `cd` into
it, `git stash pop` there, and continue. Do NOT keep going in the
top-level tree just because work is already in flight.

---

## Data-Layer Rules (customize per your backend)
1. Never run destructive/write operations against production directly from
   an ad-hoc tool call — use a dev branch/environment, or a migration file
   reviewed like any other code change
2. Every new table/collection needs its access-control policy defined
   explicitly — no exceptions
3. Schema changes via migration files only — never via a dashboard
4. Functions/endpoints touching auth or payments need human review before
   deploy
5. Never log user PII

---

## Hard Rules
1. No production code without a failing test first
2. Every commit must contain the ticket ID
3. Every PR title must start with the ticket ID
4. Never push directly to the default branch
5. Never expose API keys or secrets in code or PRs
6. Agent self-merges once all REQUIRED CI checks pass — no block on
   merging (draft status or a missing human "go" is never a block). The
   only carve-outs that require human merge are the ones you define above
   (canonically: access-control policies and payments/billing logic).
   Non-required/advisory checks do not block.
7. Commit after every completed task — never leave uncommitted work
8. Write and commit a handoff doc before ending any incomplete session
9. Do not modify files outside the current ticket scope
10. If you hit a build error in a file you did not touch — wait briefly and
    retry before assuming it's caused by your change
11. On a local machine, every session works inside its own worktree if
    concurrent agents/bots share the repo (see *Parallel-Agent Safety*). In
    a managed cloud session, the container is already an isolated clone —
    skip the worktree dance and work in the checkout directly
12. No code on a user-visible surface without explicit human sign-off on
    the design AND the final copy (Phase 2.5). Design-to-build is never
    automatic

---

## Parallel Heavy Work (optional, local machine only)

If you run local orchestration tooling that spawns multiple agents across
worktrees for Standard/Heavy tier work, document it here: how to launch it,
where worktrees land, and any project-specific rules crewmates must follow
(self-merge policy, push gate, branch naming, issue-tracker mirroring,
worktree teardown safety). This section is entirely optional — most teams
using this template will rely on in-session subagents instead, launched
directly from a chat session rather than a separate orchestrator.

## Overnight Batch Work (optional, local machine only)

If you run an unattended batch tool to process a queue of Trivial-tier
tickets overnight, document its invocation and guardrails here (Trivial
tier only; review all generated PRs before merging in the morning).

---

## Handling Ambiguity — Make the Best Choice
Do not stop and ask for clarification on ambiguous requirements.
Make the best reasonable decision, document it, and continue.

When you encounter ambiguity:
1. Choose the most logical interpretation based on existing codebase
   patterns
2. Document your decision in the plan file under a `## Decisions` section
3. Add a comment in the relevant code explaining the assumption
4. Note it in the handoff doc and the PR description so a human can review
   it

Example PR description addition:
"Assumption: X was interpreted as Y based on existing pattern in
`path/to/file`. If this is wrong, change [specific line] to [alternative]."

### Only stop and ask for these (genuine blockers):
- A schema change would delete or corrupt existing user data
- Two valid interpretations would produce completely incompatible
  implementations
- A security-sensitive flow has a requirement that could expose user data
  either way
- The task would require pushing directly to the default branch
