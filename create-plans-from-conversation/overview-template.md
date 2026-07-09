# <Effort name> — Design Record & Plan Index

<!--
The durable capture of the design. Written only when an effort splits into MORE
THAN ONE plan. Every plan in this folder links here and says "read the overview
first". Keep it current-truth about the DESIGN + the decisions that produced it
(the decision log is deliberately kept here, unlike the KB). UK British English.
-->

> **Status:** Designed, **not implemented**. Produced from a design/grilling
> session on <date>.
>
> **How to use this folder:** each `NN-*.md` file is a self-contained *plan*. Feed
> it to `/create-prd-from-plan docs/plans/<slug>/NN-*.md` → `docs/prds/<plan-slug>/`
> → `/implement-prd`. Do the plans **in order** — each depends on the previous.
> **Read this overview first when running any plan.**

## 1. Context & goals

<Where things are today, and what this effort is for — the goals and the hard
constraints that shape everything.>

## 2. The core approach

<The one-line thesis of the design, then the shape it takes. If the design
reframes the original ask, say so and why.>

## 3. Decision log

<Every decision, as decision → why → what it rules out. This is the faithful
capture of the interview; plans reference these and must not re-litigate them.>

1. **<decision>** — <why>. *Rules out:* <alternatives not taken>.

## 4. Design / architecture

<The target design: components/layers/modules, how they fit, key data shapes, and
(if relevant) the repo/package layout.>

## 5. Cross-cutting conventions (every plan honours these)

<Rules that apply across all plans — link KB concepts where they exist rather than
restating them. New greenfield conventions this effort introduces are captured in
each plan's decisions and promoted to the KB when its PRD retires.>

## 6. Plan sequence

| # | Plan (slug) | Delivers | Depends on | Order/phase |
| --- | --- | --- | --- | --- |
| 01 | `<slug>` | <one line> | — | <phase> |
| … | | | | |

Dependency chain: **<e.g. 01 → 02 → 03 → {04, 05}>**. Recurring templates
(`A-…`) apply once their prerequisites exist.

## 7. Global risks & open questions

- <risk / open question that spans plans>

## 8. Out of scope / not changing

- <explicit exclusions and settled constraints, so no plan drifts into them>

## 9. Reference — findings

<Facts established during the session that plans draw on (versions, APIs,
constraints), with sources. The single place they're recorded in full; plans
repeat only the subset they need.>
