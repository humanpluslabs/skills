---
name: pr-review-comments
description: "Work through a pasted CodeRabbit review on the current PR: verify each finding, get sign-off on a verdict table, implement and sync the knowledge base, pause for review, then resolve/reply on the PR threads and commit + push. Usage: /pr-review-comments <pasted CodeRabbit prompt>"
disable-model-invocation: true
---

# CodeRabbit review response

**Usage:** `/pr-review-comments <pasted CodeRabbit prompt>`

The argument (`$ARGUMENTS`) is the CodeRabbit review text — the dump of inline
comments and nitpicks, each tied to a `file:line`. Work through it against the
**current branch's open PR**, with two human gates and a strict order of
operations.

## Operating contract — non-negotiable

This skill performs **outward-facing actions** (posts PR comments, resolves
threads, pushes). It must never run past a gate without explicit user go-ahead.

The flow has exactly **two STOP points**:

1. **After the verdict table** — present it and wait for permission before
   touching any code.
2. **After implementation + validation** — pause for the user to review. Only
   once you and the user are *aligned* do you resolve threads, reply, commit and
   push.

Honour the order: **align → resolve & reply → `/commit` → push.** Never resolve
threads or reply before the user has approved the implementation, and never
commit before the threads are reconciled.

## Process

### 1. Locate the PR and load the live review threads

- `$ARGUMENTS` is the source of *findings*; the live PR is the source of
  *threads to resolve/reply to*. You need both.
- Derive the PR and repo from the current branch:
  ```bash
  PR=$(gh pr view --json number -q .number)
  OWNER=$(gh repo view --json owner -q .owner.login)
  REPO=$(gh repo view --json name -q .name)
  ```
  If `gh pr view` finds no PR for the branch, stop and say so.
- Pull every review thread with its resolution state and comment ids (you reply
  to the **first comment's `databaseId`** in a thread, and resolve via the
  thread's node `id`):
  ```bash
  gh api graphql -f query='
  query($owner:String!,$repo:String!,$pr:Int!){
    repository(owner:$owner,name:$repo){
      pullRequest(number:$pr){
        reviewThreads(first:100){ nodes{
          id isResolved isOutdated path line originalLine
          comments(first:10){ nodes{ databaseId author{login} body } }
        }}
      }
    }
  }' -F owner="$OWNER" -F repo="$REPO" -F pr="$PR"
  ```
- Match each pasted finding to a live thread by **path + line** (CodeRabbit's
  bot login is `coderabbitai`). A finding with no matching live thread is still
  worth acting on, but note it has no thread to resolve. A thread CodeRabbit has
  already auto-resolved still deserves a check that the code truly addresses it.

### 2. Verify each finding against the current code

- **Read the actual code at each `file:line` — do not trust the finding.** Some
  will already be fixed, outdated, or simply wrong. Confirm the issue is real and
  still present before deciding to act.
- Keep verdicts honest: a real-but-non-minimal or product-decision finding is a
  legitimate **Skip (defer)**, not an automatic Fix.

### 3. Present the verdict table — **GATE 1, then STOP**

Output one row per finding:

| # | Finding (`file:Lline`) | Summary | Verdict | Reason |
|---|---|---|---|---|

- **Verdict** is one of: `Fix`, `Skip — already addressed`, `Skip — not valid`,
  `Skip — deferred (your call)`.
- For every Skip, give a one-line reason. For a deferred item, say what the real
  fix would involve and why it's not a minimal in-place change.
- End by explicitly asking permission to proceed with the `Fix` rows. **Stop and
  wait.** Do not edit anything yet.

### 4. Implement the approved fixes, sync the knowledge base, then validate — **GATE 2, then STOP**

- Apply only the `Fix` rows the user approved (respect any verdict the user
  overrode). Keep changes minimal and match surrounding code style.
- **Sync the knowledge base.** Once the fixes are in, check whether they make any
  concept in `docs/knowledge/` stale or incomplete — a changed invariant, contract,
  convention, or behaviour the catalogue documents. Start at
  `docs/knowledge/index.md` and open only the concept(s) the changed files relate
  to. If one is affected, update it to current-truth following the house rules in
  `docs/knowledge/AGENTS.md` (what's true now + how to work with it; point at code
  for volatile detail; don't mirror implementation). Be honest about scope: a
  tightened detail the concept already delegates to source needs no change; a
  changed durable fact does. **If nothing is affected, say so explicitly** rather
  than skipping the check silently. Any KB edit rides in the same commit as the fix.
- Run the repo's checks on the changed code and report results honestly:
  `pnpm check` (Biome), `pnpm typecheck`, `pnpm test`, `pnpm spellcheck`, and —
  whenever a `docs/knowledge/` doc changed — `pnpm check:knowledge` (scope with
  `--filter` where it's faster). Fix any failures you introduced.
- Summarise what changed — code **and** any knowledge-base concept (or an explicit
  "no KB change needed") — then **pause for the user to review.** Do not proceed to
  threads/commit until you and the user are aligned. Iterate here if they want
  changes.

### 5. Reconcile the PR threads (only after alignment)

Apply this policy per thread, then reply (and resolve where stated):

| Situation | Reply | Resolve? |
|---|---|---|
| Fix applied | Summarise what changed | **Resolve** |
| Skipped — already addressed / not valid | Brief reason | **Resolve** |
| Skipped — deferred (valid concern, not acted on) | Explain the reasoning and offer to do it on request | **Leave open** |
| Already resolved but the shipped fix diverged from the suggestion | Clarify what actually shipped | Leave as-is |

- For multi-line replies with backticks, write the body to a JSON file to avoid
  shell-escaping pain, then post it to the thread's first comment:
  ```bash
  # body.json = {"body": "..."}
  gh api --method POST "repos/$OWNER/$REPO/pulls/$PR/comments/$COMMENT_DB_ID/replies" --input body.json
  ```
- Resolve a thread by its node id:
  ```bash
  gh api graphql -f query='mutation($id:ID!){ resolveReviewThread(input:{threadId:$id}){ thread{ isResolved } } }' -F id="$THREAD_ID"
  ```
- **Never silently close a deferred concern** — leaving it open is the signal
  that it's a real call for the human to make. Re-query the threads afterwards
  and confirm the resolved/open split is exactly what you intended.

### 6. Commit and push

- Stage the changed files (`git add …`), then run the **`commit`** skill
  (`/commit`) so the message follows repo conventions and the hooks run. Do not
  hand-roll the commit or bypass hooks.
- Push to the PR branch (`git push`). This is the final action — the PR now
  reflects the fixes that the resolved threads describe.
- Report: the verdict outcomes, threads resolved vs left open (with why), check
  results, and the pushed commit.

## Notes

- If the working tree has unrelated changes, only stage what this review touched.
- If a check fails or a hook blocks the commit, report it as-is and stop — let
  the user decide. Don't `--no-verify`.
- Branch hygiene: this runs on the PR's own branch; never commit straight to
  `main`.
