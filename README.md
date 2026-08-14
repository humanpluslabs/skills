# Agent Skills

A collection of agent skills that enhance the development workflow.

## In-House

```bash
pnpx skills@latest add humanpluslabs/skills --agent claude-code
```

### Planning → implementation pipeline

Four skills chain a design conversation all the way to committed code. Paired
with `/grill-me` (see [Matt Pocock](#matt-pocock) below), the flow is:

```
/grill-me                         design + decisions (the interview)
      ↓
/create-plans-from-conversation   capture + split into self-contained plans
      ↓  (one plan at a time)
/create-prd-from-plan             plan → prd.md + conventions.md
      ↓  (loop until milestones done)
/implement-prd                    next milestone → code, one per session
```

| Skill                            | What it does                                                                                                                                                                                      |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `create-plans-from-conversation` | Turns the design decisions in a conversation into one or more self-contained plan files under `docs/plans/`, each shaped to feed `/create-prd-from-plan`. Produces plans only — no PRDs, no code. |
| `create-prd-from-plan`           | Turns a plan into an agent-executable PRD (`prd.md`) plus a per-PRD `conventions.md`, breaking work into self-contained milestones a fresh session can implement. Documents only.                 |
| `implement-prd`                  | Implements the next incomplete milestone from a PRD — one milestone per session, commit when green, no auto-push.                                                                                 |

### Utilities

| Skill                 | What it does                                                                                                                                                                                              |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `commit`              | Creates a conventional commit message for staged changes.                                                                                                                                                 |
| `create-asana-ticket` | Creates well-formed Asana tickets using a vertical-slice, outcome-focused template and a confirmation-first workflow.                                                                                     |
| `explain`             | Read-only mode: answers questions about the project without changing anything.                                                                                                                            |
| `init-monorepo`       | Bootstraps a fresh TypeScript pnpm monorepo with the house defaults (Biome, cspell, lefthook, Turbo, CI, and more).                                                                                       |
| `pr-review-comments`  | Works through a pasted CodeRabbit review on the current PR: verifies each finding, gates on a verdict table, implements, syncs the knowledge base, then resolves/replies on threads and commits + pushes. |
| `prime`               | Familiarises the agent with the project codebase at the start of a new conversation.                                                                                                                      |

## 3rd Party

Skills from other authors that I find useful.

### Matt Pocock

Variety of skills to aid in the agentic development process

```
pnpm dlx skills@latest add https://github.com/mattpocock/skills --agent claude-code pi
```

### Drizzle

```
pnpm dlx skills@latest add https://github.com/bobmatnyc/claude-mpm-skills --agent claude-code pi
```

### Mastra

```
pnpm dlx skills@latest add https://github.com/mastra-ai/skills --agent claude-code pi
```

### openrouter-typescript-sdk

```
pnpm dlx skills@latest add https://github.com/openrouterteam/agent-skills --agent claude-code pi
```

### Composio

```
pnpm dlx skills@latest add composiohq/skills --agent claude-code pi
```

### Nextjs

```
pnpm dlx skills@latest add https://github.com/vercel-labs/vercel-plugin --agent claude-code pi
```

### Clerk

```
pnpm dlx skills@latest add clerk/skills --agent claude-code pi
```

### Railway

Operate Railway infrastructure — projects, services, databases, deploys, domains, environments and variables, plus querying Railway's docs

```
pnpm dlx skills@latest add https://github.com/railwayapp/railway-skills --agent claude-code pi
```

Or just the `use-railway` skill, without the rest of the repo:

```
pnpm dlx skills@latest add https://github.com/railwayapp/railway-skills --skill use-railway --agent claude-code pi
```

### Cmux

```
pnpm dlx skills add https://github.com/manaflow-ai/cmux --agent claude-code pi
```

### ADHD

Parallel divergent ideation — fans thoughts out under different cognitive frames, scores them, prunes the traps, deepens the survivors

```
pnpm dlx skills@latest add https://github.com/UditAkhourii/adhd --agent claude-code pi
```

### advise-project-approach

Researches comparable projects, tradeoffs, costs and failure conditions before it gives build advice — so a recommendation rests on prior art rather than vibes

```
pnpm dlx skills@latest add https://github.com/AaravKashyap12/advise-project-approach --agent claude-code pi
```

### NeuroArxiv

Checks real arXiv prior art before designing a new architecture — papers fetched over HTTP, read in isolation so no source anchors another, converged into one cited recommendation

```
pnpm dlx skills@latest add https://github.com/UditAkhourii/neuroarxiv --agent claude-code pi
```
