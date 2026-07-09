# PRD: <title>

> **Source plan:** <relative link to the plan this PRD is derived from>
> **Knowledge base:** `docs/knowledge/` (start at `index.md`) — repo-wide durable conventions; authoritative over conventions.md.
> **Conventions:** [./conventions.md](./conventions.md) — this PRD's in-flight greenfield decisions; read before implementing.
> **Status:** living document. Implementing agents tick checkboxes as they complete work.

<!--
Fill every placeholder. Remember the fresh-session rule: an implementing agent
has only this file, conventions.md, and the repo — never the planning
conversation. Capture decisions, don't reference a chat.
-->

## Outcome & context

<2–4 sentences: what this delivers and why. Pull in the decisions and overrides
from the plan and the planning conversation — this is the only place they
survive.>

## How to implement (standing rules)

- **Consult the knowledge base first** (`docs/knowledge/`, start at `index.md`):
  it is authoritative for repo-wide conventions. `conventions.md` holds only this
  PRD's in-flight greenfield decisions not yet promoted; where the two overlap,
  the KB wins.
- **Prefer existing repo patterns.** Where one exists it's cited in the milestone
  or in conventions.md — follow it; don't reinvent it.
- **For greenfield work, follow [./conventions.md](./conventions.md).** Do not
  invent a new pattern. If you hit a greenfield decision neither the KB nor
  conventions.md covers, stop, add it to conventions.md, and flag it to the user
  before proceeding.
- Keep changes scoped to the current milestone. Respect `AGENTS.md` / `CLAUDE.md`.

## How to pick up work (fresh session / orchestrator)

1. The **next milestone** is the first one whose `Depends on` milestones are all
   complete and whose checkboxes are not all ticked.
2. Read this header, that milestone in full, and conventions.md before starting.
3. Implement the tasks. Tick each `- [x]` once it's done and holds.
4. A milestone is **complete** only when every checkbox in it — tasks and
   Definition of done — is ticked.

Concurrency and claiming across parallel agents is the orchestrator's
responsibility; it is not tracked in this file.

## Manual ops (human-only)

<!-- Include this section only if the work has steps a coding agent cannot
perform — provisioning external services, secrets, dashboards (Neon, Doppler,
Vercel, GitHub secrets, etc.). Omit it entirely if there are none. Any milestone
that depends on one of these should say so and point here. -->

These touch external services an implementing agent cannot configure. An agent
must **surface** them to the user, not attempt them, and may proceed for
build/typecheck using a documented hatch (e.g. `SKIP_ENV_VALIDATION`).

- <service> — <the manual step, stated precisely>

## Milestones

<!-- Repeat this block per milestone. Prefer vertical slices. If a milestone is
a thin/layered fallback because the work won't slice vertically, say so in
"Delivers". -->

### Milestone 1 — <name>

- **Depends on:** none <!-- or: M<n>, M<n> -->
- **Delivers:** <one line: the slice of functionality this produces. Note here if
  it's a layered/infra fallback rather than a true vertical slice, and why.>

**Preconditions (repo state assumed):**

- <a fact about the repo an agent can rely on, delivered by prior milestones —
  e.g. "`@human-plus/db` exists and exports `createDb`". "none" for the first.>

**Context pointers:**

- Follow existing pattern: `<path>` — <what to copy from it>
- Conventions: §<section> of [./conventions.md](./conventions.md)

**Tasks:**

- [ ] <outcome-oriented task — what's true when done, not keystrokes>
- [ ] <...>

**Definition of done:**

- [ ] <runnable acceptance criterion, e.g. `pnpm --filter <pkg> typecheck` passes>
- [ ] <...>

## Out of scope

- <explicit exclusions, so an agent doesn't pull adjacent work into a milestone>

## Durable conventions to promote

<!-- conventions.md is ephemeral. List the entries it flags as "Promote to KB"
here, so they're moved into docs/knowledge/ (distilled to current-truth, not a
decision log) before this PRD is retired. Omit if none. -->

- <durable convention> — see §<section> of [./conventions.md](./conventions.md);
  promote to `docs/knowledge/conventions/<area>/<id>.md`
