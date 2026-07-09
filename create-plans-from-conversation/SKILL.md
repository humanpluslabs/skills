---
name: create-plans-from-conversation
description: Turn the design decisions in a conversation into one or more self-contained plan files under docs/plans/, each an input to /create-prd-from-plan. Use after a design/grilling session to split work into progressive, working-unit plans.
disable-model-invocation: true
---

# Create plans from conversation

**Usage:** `/create-plans-from-conversation [topic-or-slug]`

Turn the design and decisions reached in this conversation into **one or more
self-contained plan files** under `docs/plans/<slug>/`, each shaped to be fed
straight into `/create-prd-from-plan`.

This skill **produces plan documents only.** It does **not** implement, and it
does **not** write PRDs — that is `create-prd-from-plan`'s job, run once per plan.

## Where it sits in the pipeline

```
/grill-me                     design + decisions (the interview)
      ↓
create-plans-from-conversation  capture + split into plans  ← THIS SKILL
      ↓  (one plan file at a time, in order)
/create-prd-from-plan <plan>  plan → prd.md + conventions.md
      ↓
/implement-prd <prd>          next milestone → code, one per session
```

## The two rules that drive everything

1. **Self-contained plans (the downstream-session rule).** A future
   `create-prd-from-plan` session has only *one plan file* + the repo + the
   knowledge base (`docs/knowledge/`) — **never this conversation.** So **every
   decision from this conversation must be written into the plan(s).** Anything
   left only in chat is lost. A plan is self-contained when the set {that plan +
   the overview (if any) + the KB + the repo} is enough to run
   `create-prd-from-plan` against it.

2. **Working-unit decomposition (the cut criterion).** Each plan is a **vertical
   slice that leaves the codebase in a working, releasable state** once its
   milestones are done. Split *at* the points where the system is green, not
   through the middle of a half-migrated state. Size each plan to roughly one PRD
   (a handful of milestones). Add plans only when genuine complexity or a
   "must stay green here" boundary demands it — never to pad.

## Process

Work through these in order.

### 1. Check readiness & resolve inputs

- Re-read the conversation for: the goal, the settled decisions, the rejected
  alternatives, and any still-open forks. These are the raw material.
- **Readiness gate.** If the design is underspecified — key decisions unmade,
  major forks unresolved, the shape still vague — **stop and recommend
  `/grill-me` first.** Do not invent the missing design here; this skill captures
  and splits a design, it does not create one. (It *may* ask a few narrow
  decomposition questions in step 3, but it does not run a full design interview.)
- Derive `<slug>` from `$ARGUMENTS` (a topic/slug) or from the subject of the
  conversation. Output goes to `docs/plans/<slug>/`.
- Read `AGENTS.md` / `CLAUDE.md` and the KB index (`docs/knowledge/index.md`).
  Treat their conventions as given — don't restate them in plans; point to them.

### 2. Scan the repo (light grounding only)

For each area the work touches, capture **current-state pointers** for the plans'
"Current state" sections — the files, existing patterns, and prior art an
implementing effort builds on — and note any relevant KB concept IDs.

Keep this light. **Do not** do deep, per-fork official-docs research — that is
`create-prd-from-plan`'s job (its step 3), and duplicating it here wastes effort
and rots. Capture only findings **already established in this conversation** (e.g.
a version or API confirmed during grilling) under "Verified findings so far," and
leave everything else as an **open fork** for create-prd to research.

### 3. Decompose into plans (the core step)

Apply the working-unit cut criterion (see also "Deciding the number of plans"
below). Produce:

- an ordered list of plans, each a vertical, leaves-it-green slice;
- the **dependency chain** between them (`Depends on`);
- a **recommended count** with a one-line rationale tied to the complexity of what
  was designed;
- any **recurring unit of work** (a shape repeated per instance — e.g. one adapter
  per provider) captured as a single **template plan** (filename prefix `A-`,
  `B-`…) rather than duplicated.

**Present the breakdown as a table and get the user's approval or adjustments
before writing anything.** This is the skill's one interactive step. Columns:
`#`, `plan (slug)`, `delivers`, `depends on`, `order/phase`.

### 4. Write the plans

- Create `docs/plans/<slug>/`.
- If **more than one** plan: write `00-overview.md` from
  [overview-template.md](./overview-template.md) — the durable design record
  (context, the full decision log, the design/architecture, cross-cutting
  conventions, the plan sequence, and findings). Every plan links it and says
  "read the overview first."
- Write each plan from [plan-template.md](./plan-template.md), numbered in order
  (`01-…`, `02-…`; templates as `A-…`). If there is only **one** plan and the work
  is simple, write a single plan file and **skip the overview**.
- Each plan must carry, in full: outcome; in/out of scope; **decisions already
  settled** (so create-prd's grilling doesn't re-litigate them); **open forks**
  (so create-prd knows what to research and grill); verified findings so far;
  current repo state (pointers); a **suggested milestone shape** — a *sketch* to
  inform create-prd, **not** finished milestones (that is create-prd's job);
  manual ops (external services/secrets); and risks.
- Respect `AGENTS.md`/`CLAUDE.md` and UK British English throughout.

### 5. Confirm

Present the file list and the exact commands to run the pipeline, one plan at a
time, in dependency order:

```
/create-prd-from-plan docs/plans/<slug>/01-<name>.md
/implement-prd docs/prds/<01-slug>/        # loop until milestones done
# then 02-…, and so on
```

Do **not** implement and do **not** write PRDs.

## Deciding the number of plans

The count follows the work, not a target. Heuristics:

- **One plan** — a single cohesive feature/change that lands green in a handful of
  milestones. Prefer this; don't split for its own sake.
- **Several plans** — when the work has separable slices, a natural ordering where
  each step leaves the system releasable, or layered/infra work that can't ship in
  one green step. Cut **at** the green boundaries.
- **A template plan** — when a unit of work repeats per instance (per provider, per
  client, per integration). Emit one `A-…` template to copy-and-fill, not N near-
  identical plans.

Watch-outs when cutting:

- **Never split through a broken state.** If step A leaves the build red until step
  B lands, A and B are one plan (or one is a milestone inside the other).
- **Extraction/move-only work is its own plan** — keep behaviour-change and
  pure-move in separate plans so each stays verifiable.
- **Things that must change together are one plan.** If two areas share a hard
  dependency (a schema both must migrate atomically), don't separate them.
- **Order by dependency, and make it explicit** in each plan's `Depends on`.
