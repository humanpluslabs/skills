# Plan <NN> — <title>

<!--
Fill every section. Remember the downstream-session rule: a create-prd-from-plan
session has only THIS file (+ the overview, if any) + the repo + the knowledge
base — never the planning conversation. Every decision must live here.
Keep the milestone section a SKETCH — create-prd-from-plan writes the real
milestones. UK British English.
-->

> Part of the **<effort name>** — **read [00-overview.md](./00-overview.md) first**
> for the full design, decision log, and conventions. <!-- omit this line if this is a single-plan effort with no overview -->
> **Pipeline:** `/create-prd-from-plan docs/plans/<slug>/<NN-this-file>.md` →
> `docs/prds/<plan-slug>/` → `/implement-prd`.
> **Position:** Plan <NN> of <MM>. **Depends on:** <plans, or "nothing">.
> **Followed by:** <next plan, or "—">.

## Outcome

<2–4 sentences: the slice of functionality this plan delivers and why. State that
it leaves the codebase in a working/releasable state when done.>

## Scope

**In scope**

- <what this plan does>

**Out of scope (other plans / later)**

- <what is deliberately excluded, and which plan owns it — so create-prd doesn't
  pull adjacent work in>

## Decisions already settled (do not re-litigate)

<Every decision from the conversation relevant to this plan, each as
decision → why → what it rules out. create-prd-from-plan treats these as given and
will NOT re-interview them.>

- **<decision>** — <why>. *Rules out:* <the alternative(s) not taken>.

## Open decision forks for the create-prd grilling

<Genuinely unresolved choices this plan leaves for create-prd-from-plan to
research (step 3) and grill (step 4). Give your recommendation for each so the
downstream interview is short. Empty is fine if nothing is open.>

- **<fork>** — <options + trade-offs>. *Recommend:* <your pick + why>.

## Verified findings so far

<Only facts ALREADY established in this conversation — versions, APIs, constraints
confirmed during grilling — with any source. Do NOT do fresh deep docs research
here; leave that to create-prd-from-plan.>

- <finding> — <source/where confirmed>.

## Current state — repo facts to build on

<Pointers an implementing effort relies on: files, existing patterns, prior art,
KB concept IDs. Link KB concepts rather than restating them.>

- <path or KB concept> — <what's there / what to mirror>.

## Suggested milestone shape (create-prd refines — do NOT pre-write milestones)

<A sketch of vertical-slice milestones to inform create-prd-from-plan's breakdown.
Each should leave the code green. This is guidance, not the final milestone list.>

1. <milestone sketch — the slice, left green>
2. <…>

## Manual ops (human-only)

<External services/secrets a coding agent cannot configure — Neon, Doppler,
Vercel, WorkOS, DNS, GitHub secrets, etc. Omit if none.>

- <service> — <the manual step, stated precisely>

## Risks & watch-outs

- <risk> — <how it's mitigated / what to watch>
