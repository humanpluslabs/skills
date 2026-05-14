---
name: init-monorepo
description: "Bootstrap a fresh TypeScript pnpm monorepo with the house defaults: git, Biome, cspell, lefthook, Turbo, sherif, shared @repo/typescript config, VS Code settings, GitHub Actions CI + weekly audit, and CLAUDE.md/AGENTS.md. Use when the user wants to scaffold a new TypeScript monorepo from scratch, asks to 'init monorepo', 'set up a pnpm workspace', or runs /init-monorepo in an empty (or near-empty) directory. Usage: /init-monorepo"
disable-model-invocation: true
---

# Init Monorepo

Bootstrap a TypeScript pnpm monorepo with the house defaults. Run this in an **empty** directory (or one containing only seed files the user has approved overwriting).

## Prerequisites

Before starting, in parallel:

- `pwd && ls -la` — confirm the directory is empty (or only contains files you're cleared to overwrite). If it isn't, **stop and ask** before continuing.
- `command -v pnpm` — pnpm must be installed.
- `command -v nvm` — nvm needs to be available so Step 7 can query the current Node LTS. If absent, ask the user how they'd like to pin Node before continuing.

The skill assets live alongside this file under `assets/`. Read them with the Read tool and write each to the target path verbatim unless a placeholder substitution is called out.

## Workflow

Work through the 26 steps in order. Mark each complete before moving on — many later steps depend on earlier ones.

### 1. Initialise git

```sh
git init
```

### 2. Generate root `.gitignore`

```sh
pnpm dlx add-gitignore macos node
```

Swap `macos`/`node` for the appropriate templates if the host OS or toolchain differs.

### 3. Append agent-tooling entries to `.gitignore`

Append the contents of `assets/gitignore-tail` to the bottom of `.gitignore` (do **not** overwrite — keep the generated templates above it).

### 4. Install in-house skills for both agents

```sh
pnpm dlx skills@latest add humanpluslabsoss/skills --agent claude-code
pnpm dlx skills@latest add humanpluslabsoss/skills --agent pi
```

The CLI is interactive — at the multi-select prompt, choose `commit` and `prime`. Tell the user this prompt is coming and ask them to make the selection.

### 5. Initialise `package.json`

```sh
pnpm init
```

`pnpm init` does **not** accept `-y` — it's already non-interactive. Do not pass that flag.

### 6. Tidy `package.json`

Edit the freshly generated `package.json`:

- Remove the `"main": "index.js"` line.
- Insert `"private": true` immediately after `"version"`.

### 7. Pin minimum Node version in `engines`

Resolve the current Node LTS via `bash -lc 'source "$NVM_DIR/nvm.sh" && nvm version-remote --lts'`, strip the leading `v`, and call it `<NODE_LTS_VERSION>`. Add an `engines` block to `package.json`:

```json
"engines": {
  "node": ">=<NODE_LTS_VERSION>"
}
```

Use `>=`, not an exact pin. Remember `<NODE_LTS_VERSION>` — Steps 10 and 26 reuse it.

### 8. Add `.npmrc`

Write `assets/npmrc` to `.npmrc` at the repo root.

### 9. Create `pnpm-workspace.yaml` (initial slice)

Write the contents of `assets/pnpm-workspace.yaml` to `pnpm-workspace.yaml`, but **delete the `packages:` and `onlyBuiltDependencies:` blocks for now** — they're added in Steps 20 and 12 respectively. The starting file should contain only `catalogMode`, `cleanupUnusedCatalogs`, `linkWorkspacePackages`, and `minimumReleaseAge`.

### 10. Add a minimal `README.md`

Read `assets/README.tmpl.md`. Substitute:

- `<PROJECT_NAME>` → the value of `name` in `package.json` (i.e. the directory name pnpm chose).
- `<NODE_LTS_VERSION>` → the version from Step 7.

Write the result to `README.md`.

### 11. Install root tooling

```sh
pnpm add -D @biomejs/biome@latest cspell@latest lefthook@latest sherif@latest turbo@latest typescript@latest
```

`catalogMode: strict` means pnpm writes versions into a `catalog:` block in `pnpm-workspace.yaml` and sets each devDependency in `package.json` to `"catalog:"`. Expect a warning about lefthook's blocked postinstall — Step 12 fixes that.

### 12. Approve lefthook's postinstall

Append the `onlyBuiltDependencies:` block (from `assets/pnpm-workspace.yaml`, lines 9–10) to `pnpm-workspace.yaml`. Then:

```sh
pnpm install
```

This re-runs install with lefthook's postinstall now allowlisted — it generates a default `lefthook.yml` (overwritten in Step 19) and installs git hooks. Do **not** use `pnpm approve-builds`; it's interactive and the `--all` flag scope-escalates.

### 13. Add `biome.json`

Read `assets/biome.json`. Resolve the installed Biome version: read the catalog entry for `@biomejs/biome` in `pnpm-workspace.yaml`, or run `pnpm exec biome --version`. Substitute `<INSTALLED_BIOME_VERSION>` in the `$schema` URL with that exact version, then write to `biome.json` at the repo root.

### 14. Add `.vscode/settings.json`

Write `assets/vscode/settings.json` to `.vscode/settings.json`. Keep the 4-space indentation as-is — VS Code's settings file is conventionally 4-space and no formatter rewrites it.

### 15. Add `.vscode/extensions.json`

Write `assets/vscode/extensions.json` to `.vscode/extensions.json`.

### 16. Add `cspell.json`

Write `assets/cspell.json` to `cspell.json` at the repo root.

### 17. Add `turbo.json`

Write `assets/turbo.json` to `turbo.json` at the repo root.

### 18. Replace `package.json` scripts

Replace the entire `"scripts"` block in `package.json` with:

```json
"scripts": {
  "dev": "turbo run dev",
  "test": "turbo run test",
  "check": "biome check",
  "check:fix": "biome check --fix",
  "typecheck": "turbo run typecheck",
  "clean:root": "git clean -xdf .cache .turbo node_modules tsconfig.tsbuildinfo",
  "clean": "turbo run clean && pnpm run clean:root",
  "spellcheck": "cspell '**'"
}
```

This wipes the placeholder `"test"` script `pnpm init` generated in Step 5.

### 19. Author `lefthook.yml` and resync hooks

Overwrite `lefthook.yml` with `assets/lefthook.yml`, then resync:

```sh
pnpm exec lefthook install
```

Always re-run `lefthook install` after editing `lefthook.yml`.

### 20. Declare workspace packages

Append to `pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

This must be present **before** any `turbo run <task>` invocation, otherwise Turbo recurses infinitely on the root's own scripts.

### 21. Add GitHub Actions

Write all three files verbatim:

- `assets/github/actions/setup/action.yml` → `.github/actions/setup/action.yml`
- `assets/github/workflows/ci.yml` → `.github/workflows/ci.yml`
- `assets/github/workflows/audit.yml` → `.github/workflows/audit.yml`

### 22. Add `audit-ci.json`

Write `assets/audit-ci.json` to `audit-ci.json` at the repo root. Without this, the weekly audit workflow from Step 21 fails on every cron tick.

### 23. Add root `CLAUDE.md`

Write `assets/CLAUDE.md` to `CLAUDE.md` at the repo root.

### 24. Add root `AGENTS.md`

Write `assets/AGENTS.md` to `AGENTS.md` at the repo root.

### 25. Create the `@repo/typescript` package

Create `packages/typescript/` and write all four files:

- `assets/packages-typescript/package.json` → `packages/typescript/package.json`
- `assets/packages-typescript/base.json` → `packages/typescript/base.json`
- `assets/packages-typescript/nextjs.json` → `packages/typescript/nextjs.json`
- `assets/packages-typescript/README.md` → `packages/typescript/README.md`

Then re-run install so pnpm registers the new workspace package:

```sh
pnpm install
```

### 26. Add `.nvmrc`

Write the same `<NODE_LTS_VERSION>` from Step 7 to `.nvmrc` (bare version, no `v` prefix, single line).

## Wrap-up

After Step 26, in parallel:

- `pnpm run typecheck` — should pass as a no-op (no packages have a typecheck task yet).
- `pnpm run check` — Biome should report clean.
- `pnpm run spellcheck` — cspell should report clean.
- `git status` — show the user the full set of created files.

Report what was created and remind the user:

- The agent skill install in Step 4 was interactive — confirm they selected `commit` and `prime`.
- `<NODE_LTS_VERSION>` was resolved at replay time; if it differs from what their machine runs, they may want `nvm use`.
- No initial commit has been made — leave that to the user.

## Rules

- Run from the project's working directory. Do **not** change into other directories.
- Do not commit on the user's behalf.
- Do not push to a remote.
- If a step fails, stop and surface the error — do not skip ahead or paper over it.
- The `<NODE_LTS_VERSION>` and `<INSTALLED_BIOME_VERSION>` placeholders in `assets/` are deliberate; never write a literal placeholder string into the project — always substitute.
