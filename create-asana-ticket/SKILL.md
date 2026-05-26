---
name: create-asana-ticket
description: "Create well-formed Asana tickets in any project using a vertical-slice, outcome-focused template. Use whenever the user wants to create, draft, or scope Asana tasks — including phrases like 'create an Asana ticket', 'open a task in Asana', 'turn this into Asana tickets', or 'scope this work into Asana'. Drives a confirmation-first workflow, enforces a strict five-section ticket structure, and handles the Asana `html_notes` XML rules that silently reject malformed input. Usage: /create-asana-ticket [optional context]"
disable-model-invocation: false
---

# Create Asana Ticket

Create one or more Asana tickets that are scoped as vertical slices and described in terms of outcomes, not implementation. Tickets follow a fixed five-section template and use Asana's `html_notes` field, which has strict XML rules — the API silently rejects malformed input, so the mechanics below matter as much as the content.

## When to use

Trigger on any request to create, draft, or scope Asana work — for example:

- "Create an Asana ticket for the migration."
- "Turn this discussion into a couple of Asana tasks."
- "Scope this initiative into tickets for the Foo project."
- "Open a follow-up ticket in Asana when you're done."

If the user asks to *update* an existing ticket, this skill is the wrong tool — edit the task directly.

## Workflow

Work through the steps in order. Do not create anything until Step 2 is confirmed.

### 1. Confirm the destination and ownership

Ask the user (in a single AskUserQuestion call if more than one is unknown):

- Which Asana **project** the ticket(s) belong to.
- Which **section** within the project. Default to `Todo` (or whichever section is the equivalent intake column — `Backlog`, `Inbox`, `Up Next`). Confirm before assuming.
- Who the **assignee** is. Default to the user themselves unless they have indicated otherwise.

If the user has already supplied any of these in their request, do not re-ask — just confirm them back as part of Step 2.

### 2. Propose the slate before creating anything

If creating **more than one ticket**, draft the list as bullet points — each line is `Ticket title — one-sentence outcome` — and ask the user to approve, edit, or reorder before any task is created. Surface intended dependencies in the same message (e.g. "Ticket B is blocked by Ticket A"). This is the cheapest place to course-correct; once tasks exist, every change is a round-trip.

For a **single ticket**, draft the title and a one-line outcome, then proceed to draft the full description (Step 3) and show it to the user before creating.

### 3. Draft each ticket against the template

Each ticket's description must contain exactly these five `<h2>` headings, in this order:

1. **Task Description / Context** — what the work is and why, in 2–4 sentences. Frame it as the outcome the system will deliver, not the steps to get there. May include an `<hr/>` separator between the situational context and a closing framing sentence.
2. **Acceptance Criteria** — bullet list of observable conditions the system must satisfy when the ticket lands. Each item should be checkable, not procedural — describe the *state* after the work, not the *actions* taken.
3. **Definition of Done** — bullet list of process-level gates that are distinct from acceptance criteria: tests pass, docs updated, code review approved, runbook committed, feature flag flipped, etc.
4. **How to Test** — bullet list of steps an operator or reviewer can take to confirm the work behaves as intended end-to-end. Written so someone unfamiliar with the implementation can follow them.
5. **References** — docs, source files, or external sources that informed the ticket. Use in-repo relative paths wrapped in `<code>` tags. Name external docs by reference (e.g. "Asana API: html_notes formatting") and **omit the URL unless you actually consulted the page** — fabricated URLs are worse than no URL.

### 4. Resolve section GIDs

Call the Asana `get_project` tool with `include_sections: true` to fetch the project's sections. Match the user's chosen section by name (case-insensitive) and capture its GID for use as `section_id` in Step 5.

### 5. Create all tickets in a single call

Use `create_tasks` with:

- `default_project` set to the project GID.
- `default_assignee` set to the assignee GID.
- A `tasks` array where each entry carries its `name`, `section_id` (from Step 4), and `html_notes` (from Step 3).

Batching matters: a single call is one network round-trip, atomic from the user's perspective, and easier to roll back if something goes wrong. Avoid per-ticket calls in a loop.

### 6. Wire up dependencies, if any

If the slate has blocking relationships, call `update_tasks` in a second pass with `add_dependencies` populated per task. The semantics are:

> `add_dependencies: [Y]` on task X means **X is blocked by Y**.

Dependencies must be added after creation because both task GIDs must already exist. Do this even if it means two API calls — there is no way to declare dependencies inline in `create_tasks`.

### 7. Return permalinks

Report back the permalink of each created ticket so the user can open, review, and edit if needed. Group related tickets together and note any dependency edges in plain prose.

## Ticket template

Use this as the literal skeleton for `html_notes`. Substitute the bracketed sections; keep the structure exact.

```xml
<body>
<h2>Task Description / Context</h2>
[2–4 sentences describing the outcome and why it matters. Plain text, no <p> tags.]
<hr/>
[Optional single framing sentence — e.g. "This ticket delivers the operator-visible slice; downstream automation is tracked separately."]

<h2>Acceptance Criteria</h2>
<ul>
  <li>[Observable condition 1 — describes state, not steps.]</li>
  <li>[Observable condition 2.]</li>
  <li>[…]</li>
</ul>

<h2>Definition of Done</h2>
<ul>
  <li>[Process gate 1 — e.g. "Tests pass in CI."]</li>
  <li>[Process gate 2 — e.g. "Runbook committed under <code>docs/runbooks/</code>."]</li>
  <li>[…]</li>
</ul>

<h2>How to Test</h2>
<ul>
  <li>[Step 1 a reviewer can follow end-to-end.]</li>
  <li>[Step 2.]</li>
  <li>[…]</li>
</ul>

<h2>References</h2>
<ul>
  <li><code>path/to/relevant/file.ts</code> — one-line reason it's relevant.</li>
  <li>External source name (no URL unless actually consulted).</li>
</ul>
</body>
```

## Style rules

These are the rules that distinguish a good ticket from a noisy one. Apply them to every draft before showing it to the user.

- **No implementation details in the four prose sections.** No library names, no function signatures, no file paths in the body. File paths belong only in `References`. If a criterion only makes sense by referring to a specific function, you are probably scoping at the wrong level — re-state it as an outcome.
- **Vertical slices.** Each ticket should be independently shippable and leave the app in a working state with some user- or operator-visible improvement. If a ticket can only be merged after another ticket lands, that is a dependency, not a problem — but if a ticket can only be *understood* in the context of another, the slice is wrong.
- **Outcome framing.** "Operators can revoke a session from the admin panel" beats "Add a revoke button to AdminSessionsPage". The first is checkable by behaviour; the second leaks implementation.
- **Acceptance vs. Definition of Done.** Acceptance criteria describe the system's externally observable state. Definition of Done describes the process required to call the work merged. Tests passing is a process gate, not an acceptance criterion. "Failed logins are rate-limited to 5/minute per IP" is an acceptance criterion, not a process gate.
- **UK British English** throughout: "behaviour", "authorise", "initialise", "organisation", "recognise". Apply this to titles, descriptions, and bullet text.
- Titles should be **imperative and specific**: "Gate access on Google Group membership" — not "Google Group gating" or "Add Google Group check".

## Asana `html_notes` mechanics

The Asana API parses `html_notes` as XML and silently rejects malformed input with a `bad_request: XML is invalid` error that does **not** point to the offending tag. Get these right the first time.

- **`<p>` is not allowed.** Asana rejects it. Render prose as bare text between block elements (e.g. between `<h2>` and the next `<h2>`/`<hr/>`/`<ul>`). Use `<hr/>` to separate logical paragraphs.
- **Single root.** The whole payload must be wrapped in a single `<body>...</body>` element.
- **Escape `&` as `&amp;`** in XML content. "Identity & access" must be written as "Identity &amp; access" in the payload — the most common silent rejection comes from a stray ampersand in a heading or list item.
- **Allowed block elements** in practice: `<body>`, `<h1>` through `<h2>`, `<ul>`/`<ol>`/`<li>`, `<hr/>`, `<code>`, `<a href="…">`, `<strong>`, `<em>`. Stick to these unless you have a reason.
- **Self-close void elements** as `<hr/>`, not `<hr>`. The XML parser is strict.

If a `create_tasks` call returns `XML is invalid`, the body almost certainly contains a `<p>`, an unescaped `&`, or an unclosed element. Re-validate the payload before retrying; do not blindly retry the same input.

## Multi-ticket sequences with dependencies

When scoping a piece of work into several tickets, think of the slate as a small directed graph:

- Nodes are vertical slices, each independently shippable.
- Edges are *blocks* relationships — A → B means "B cannot start until A lands".

Surface this graph to the user in Step 2 before any tickets are created. After creation, wire the edges in Step 6.

Common patterns to expect:

- A **recovery / rotation** ticket usually blocks the **net-new feature** ticket that depends on the rotated credentials.
- A **gating / access-control** ticket usually blocks **feature rollout** tickets that should only ship once the gate is in place.
- A **migration** ticket usually blocks any ticket whose acceptance criteria assume the new schema.

For a worked multi-ticket example, see the sequence created in the "Human Plus Agents" Asana project on 2026-05-26 covering Composio breach recovery, Arcade integration, and Google Group access gating — five tickets, dependency-wired, drafted against this exact template.

## Rules

- Never create a ticket without explicit user confirmation of the slate (Step 2).
- Never fabricate URLs in `References`. Name the source and omit the URL if you did not actually consult it.
- Never use `<p>` inside `html_notes`. Asana rejects it.
- Always escape `&` as `&amp;` in `html_notes` content.
- Always create tickets first, then wire dependencies in a separate `update_tasks` pass.
- Do not embed file paths or function names in the four prose sections — they belong only in `References`.
- Return permalinks for every created ticket so the user can review and edit.
