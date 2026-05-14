---
name: prime
description: "Familiarise the agent with the project codebase at the start of a new conversation. Reads conventional documentation, the package manifest, and recent git activity to load context. Usage: /prime"
disable-model-invocation: true
---

# Prime

Load context about the current project so you are ready to assist with tasks in this repo. Read the canonical sources of project knowledge, then post a brief summary of your understanding.

## Steps

1. **Discover what exists.** Run in parallel:
   - `ls -la` on the repo root
   - `ls` on `docs/`, `.github/`, `.cursor/` (ignore errors for missing dirs)
   - `git status` for current branch
   - `git log --oneline -20` for recent activity

2. **Read in full** — fire all reads for existing files in a single parallel batch:
   - `CLAUDE.md` at the root, plus any nested `CLAUDE.md` files surfaced during discovery
   - `AGENTS.md`
   - `README.md`
   - `CONTRIBUTING.md`
   - `ARCHITECTURE.md`, `DESIGN.md`, `DEVELOPMENT.md` (whichever exist)
   - One overview file inside `docs/` if present (e.g. `docs/README.md`, `docs/index.md`, `docs/getting-started.md`)
   - AI-agent config: `.cursorrules`, files in `.cursor/rules/`, `.github/copilot-instructions.md`
   - Any other clearly-named project documentation in the root (e.g. `ROADMAP.md`, `SECURITY.md`, `DECISIONS.md`)

3. **Read for signal** — extract scripts, runtime, and dependencies; do not dump contents into the summary:
   - Package manifest: `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` / `Gemfile` / `requirements.txt`
   - Task runner: `Makefile`, `justfile`, `Taskfile.yml`
   - Runtime version: `.nvmrc`, `.tool-versions`, `.python-version`
   - Containers: `Dockerfile`, `docker-compose.yml`
   - CI: filenames in `.github/workflows/` (do not read contents unless small)

4. **Note existence only** — mention in summary if present, but do not read:
   - Linters/formatters: `.editorconfig`, `.prettierrc*`, `.eslintrc*`, `ruff.toml`, `biome.json`
   - `LICENSE`

5. **Tier 2 — deeper reads, capped at ~10 files total.** Prioritise:
   - Entry points (e.g. `src/index.ts`, `main.py`, `app/page.tsx`)
   - Test setup files (test config + one example test)
   - One representative file per major top-level directory

   If after the steps above you already feel oriented (e.g. `CLAUDE.md` was comprehensive), skip Tier 2.

6. **Monorepo handling.** If you detect a workspace (`pnpm-workspace.yaml`, root `package.json` with `workspaces`, `turbo.json`, `nx.json`, Cargo workspace, etc.):
   - Read the root manifest in full
   - List each package by name plus a one-line purpose pulled from its README's first line (if present)
   - Do not recursively prime every package — that is a follow-up the user can request

7. **Exclusions.** Never read or descend into:
   - `node_modules/`, `vendor/`, `dist/`, `build/`, `.next/`, `target/`, `.git/`
   - Lockfiles: `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Cargo.lock`
   - Minified or generated files: `*.min.js`, `*.bundle.js`

8. **Maximise parallelism.** Independent file reads should fire in a single message with multiple tool calls.

## Output

Post a brief summary of your understanding of the project. Keep it scannable. No fixed template, no recommended next steps, no file paths or line numbers. The point of the summary is to signal that you are oriented — the real value of this skill is the context you have loaded for the rest of the conversation.

## Rules

- Do not write, edit, or create any files during priming. If `CLAUDE.md` is missing, do not create one — that is `/init`'s job.
- Do not run any commands with side effects (no installs, no git mutations, no builds).
- Do not invoke other skills as part of priming.
