# Conventions: <PRD title>

> **Lifespan:** ephemeral, scoped to this PRD — not a long-lived standard.
> Entries flagged **Promote to KB** should be moved into `docs/knowledge/` before
> this doc is retired with the PRD (distil to current-truth — what's true now and
> how to work with it — not a decision log).
> **For implementing agents:** the knowledge base (`docs/knowledge/`) is
> authoritative for repo-wide conventions; this doc covers only this PRD's
> in-flight greenfield decisions not yet in the KB. Follow both; where they
> overlap, the KB wins. If you hit a greenfield decision neither covers, stop and
> add it here (and flag the user) rather than guessing.

## Existing patterns to follow

<Areas where the repo already has prior art. Agents copy these — not decisions
made here, just pointers. If a pattern is already a KB concept, link it instead
of re-describing it, so this table can't drift from the catalogue.>

| Area | Follow this pattern | Example / KB concept |
|---|---|---|
| <e.g. env validation> | <e.g. per-app `env.ts` via `@t3-oss/env`> | `<path>` |
| <e.g. data-access shape> | <already catalogued — link the concept> | `docs/knowledge/conventions/data-layer/data-access-shape.md` |

## Greenfield conventions

<One subsection per area with no repo pattern, as agreed in the interview.
Each is informed by current official docs — cite them.>

### <Area — e.g. Test runner & harness>

- **Decision:** <what was agreed>
- **Rationale:** <short why>
- **Alternatives considered:** <only if this was a fork — the option(s) not taken
  and why, incl. any trade-off accepted (e.g. a 2nd driver alongside an existing
  one). Omit if there was only ever one sensible approach.>
- **Docs:** <official doc URL(s) the decision was informed by>
- **Promote to KB:** yes (durable repo-wide convention → `docs/knowledge/`) /
  no (PRD-local one-off — dies with this PRD)

## Open questions

<Anything still unresolved that an implementing agent must escalate rather than
guess. Empty is good.>
