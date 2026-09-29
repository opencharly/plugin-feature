# AGENTS.md — plugin-feature

Standalone plugin repo owning the externalized `charly feature` command
(`command:feature`, compiled-in). The plugin is a Go module at
`candy/plugin-feature/` (module path
`github.com/opencharly/plugin-feature/candy/plugin-feature`); the root
`charly.yml` only declares `discover: candy` so the repo is a project and its
candy is scanned (it also carries the `feature-skill:` entity).

Canonical files:

- `candy/plugin-feature/charly.yml` — the `plugin-feature:` candy entity
  (`plugin:` block, `plan:` check).
- `candy/plugin-feature/plugin.go` / `provider.go` — `NewProvider()` /
  `NewMeta()` / `CliMain` and the `Invoke(OpRun)` surface.
- `candy/plugin-feature/command.go` — the `charly feature` CLI + the plugin-side
  project enumeration.
- `candy/plugin-feature/schema/feature.cue` — the self-contained plugin schema.
- `candy/plugin-feature/params/cue_types_gen.go` — generated params (do not
  hand-edit).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-feature:feature` — the `charly feature list|pending|validate`
  reference (the plugin's own `skill:` entity, `feature-skill`, in `charly.yml`).
- `/charly-internals:plugin` — the plugin authoring reference: the `plugin:`
  block, the `command` provider class, the per-plugin CUE-schema contract.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `go build ./...` in `candy/plugin-feature/` — compile the plugin module.
- `go test ./...` in `candy/plugin-feature/` — the plugin's Go tests (the command,
  the `flattenFeatures` transform, the schema-serve seams).
- `charly box validate` at the repo root — the structural check (the candy +
  `plugin:` block, CUE schema).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate.
- The R10 witness is the disposable `check-feature-local` bed (in
  `opencharly/charly`): `charly feature list candy` exits 0 and lists the
  project's candies.

## Modify this repo

- Edit the `plugin-feature:` candy entity, the Go source, and `schema/feature.cue`
  **together** — the schema is the served declaration surface.
- Keep feature **compiled-in**: its `Invoke(OpRun)` needs the in-proc reverse
  channel for the plugin-side loader enumeration. The out-of-process `CliMain`
  path has no reverse channel and errors.
- The `Feature RUN` verbs (`charly box feature run` / `charly check feature run`)
  are NOT this plugin — they stay children of `box`/`check` in the core binary.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
