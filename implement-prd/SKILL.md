---
name: implement-prd
description: Implement the next incomplete milestone from a PRD produced by create-prd-from-plan — one milestone per session, commit when green, no auto-push. Use when picking up PRD-driven implementation work, in a loop, or as a sub-agent.
disable-model-invocation: true
---

# Implement PRD

**Usage:** `/implement-prd [path-to-prd] [--no-commit]`

Pick up the **next incomplete milestone** from a PRD (the kind produced by
`create-prd-from-plan`), implement exactly that one, commit it, and stop.

**Flags** (anywhere in `$ARGUMENTS`, in any order):

- `--no-commit` — do everything as normal, but **stop before committing** so you
  can review the work by hand. Tick the verified boxes and leave the changes
  staged; don't run the `commit` skill.

## Operating contract

- Your inputs are the PRD, its `conventions.md`, the knowledge base
  (`docs/knowledge/`), and the repo. You do not have the planning conversation —
  everything you need is in those docs. The KB is authoritative for repo-wide
  conventions; `conventions.md` covers only this PRD's in-flight decisions; where
  they overlap, the KB wins. If something you need genuinely isn't in any of them,
  that's a gap to surface, not to fill from imagination.
- **One milestone per session.** Commit when it's green. Do **not** push, open a
  PR, or start the next milestone — a fresh session (or the orchestrator/loop)
  picks that up.

## Process

### 1. Locate the PRD

- First strip any flags (e.g. `--no-commit`) out of `$ARGUMENTS`; the remainder
  is the path. Read that path if given; otherwise find a PRD under
  `docs/prds/`. If there are several and it's ambiguous, ask which.
- Read in full and treat as binding: the PRD **header** (Outcome, standing
  rules, pick-up protocol, Manual ops), `conventions.md`, `AGENTS.md` /
  `CLAUDE.md`, and the knowledge base (`docs/knowledge/index.md`, plus the
  concept docs relevant to this PRD).

### 2. Select the milestone

- The next milestone is the first one whose `Depends on` milestones are **all
  complete** (every checkbox ticked) and whose own checkboxes are **not** all
  ticked.
- If a milestone is partially ticked (e.g. an earlier session crashed), resume
  it — and re-verify any ticked box you can't trust.
- If every milestone is complete, the PRD is done — go to **§7 Retire the PRD**
  instead of starting new work.
- Announce which milestone you're taking before starting.

### 3. Load its context

- Read the milestone in full: `Depends on`, `Preconditions`, `Context pointers`,
  `Tasks`, `Definition of done`.
- Open and read **every** pattern file cited in `Context pointers` — you will
  mirror it — the named `conventions.md` sections, and any KB concept docs they
  reference. When working inside a package that has its own `AGENTS.md`
  (e.g. `apps/human-plus/db/AGENTS.md`), read it — it maps code areas to KB
  concepts.
- Confirm the `Preconditions` actually hold in the repo. If one is false, stop
  and report (a dependency may not really be done).

### 4. Implement — enforce the contract

- **Prefer the KB and existing repo patterns.** Follow the KB concepts and any
  pattern the milestone cites; don't reinvent them.
- **For greenfield, follow `conventions.md` exactly** — versions, shapes,
  snippets.
- **Escalate, don't guess.** If you hit a decision that neither the KB, an
  existing pattern, nor `conventions.md` covers, **stop**, surface it to the
  user, and — once decided — record it in `conventions.md` (flag it **Promote to
  KB** if it's a durable repo-wide convention) before continuing. Never improvise
  a new pattern silently.
- **TDD where the milestone says so** (e.g. "write the spec first (red), then
  implement (green)") — follow the `tdd` skill's discipline: one test → one
  implementation, vertical slices, never refactor while red.
- Keep changes scoped to this milestone. Don't pull in anything under
  **Out of scope**.

### 5. Verify against the Definition of done

- Run **every** Definition-of-done check. Tick a `- [x]` box (tasks *and*
  Definition of done) **only** when its check genuinely passes.
- If a check fails: fix it; if you can't, report the failure honestly with the
  output and stop. **Never tick a box for an unmet criterion.**
- **Surface, don't perform, Manual ops.** Where a Manual op (external service /
  secret) blocks a live check, note exactly what the user must do, and use the
  documented hatch (e.g. `SKIP_ENV_VALIDATION`) for typecheck/build only — don't
  fake the check.

### 6. Commit and stop

- When every checkbox in the milestone is ticked:
  - **Default:** commit the milestone's changes **plus the updated PRD** (the
    ticked boxes) using the repo's commit conventions — run the `commit` skill.
  - **If `--no-commit` was passed:** do **not** commit. Stage the milestone's
    changes plus the updated PRD (`git add`) and leave them for the user to
    review. Tell them the work is staged and uncommitted, and that they can run
    `/commit` once they're happy.
  - Either way, do **not** push or open a PR unless asked.
- **Stop.** Do not begin the next milestone.
- Report: the milestone done, what changed, any **Manual ops** the user must
  action, and which milestone comes next.

### 7. Retire the PRD (only when every milestone is complete)

When the last milestone is done, close the PRD out instead of leaving it to rot:

- **Promote durable conventions.** For each entry under the PRD's "Durable
  conventions to promote" (and each `conventions.md` entry flagged **Promote to
  KB**), ensure a concept doc exists under `docs/knowledge/` — create it if not.
  Distil to **current-truth**: what's true now and how to work with it, plus any
  load-bearing constraint — **not** a decision log. Link it from
  `docs/knowledge/index.md`.
- **Repoint references.** Update any code comment or doc pointing at this PRD's
  `conventions.md` to the KB concept (stable path); leave no dangling refs.
- **Verify.** Run `pnpm check:knowledge` — it must pass before any deletion.
- **Propose deletion.** Once promotion is verified, propose removing the retired
  `docs/prds/<slug>/` (and any spent source plan). Don't delete on your own
  judgement — confirm with the user, and surface any open **Manual ops** the PRD
  still tracks.
