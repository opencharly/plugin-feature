# plugin-feature

The `charly feature` command, externalized into a compiled-in command plugin
(`command:feature`). It owns the plan-shaped-description inspection surface — the
Agent Driven Evaluation (ADE) view over a project's entities.

## What it provides

| Capability | Surface |
|---|---|
| `command:feature` | the `charly feature` CLI — `list` / `pending` / `validate` |

`charly feature` enumerates a project's plan-shaped entity descriptions and
flattens each entity's plan into plain data (kind/name/description/plan). The
plugin owns the subcommand grammar, the output formatting, **and** the project
enumeration — it loads the project plugin-side over the reverse channel via
`loaderkit`, so no core host-build seam is involved.

The plugin is **compiled-in** (`compiled_plugins:`): its `Invoke(OpRun)` needs the
in-process reverse channel that `dispatchInProcCommand` threads to reach the host
loader legs. The out-of-process `CliMain` path has no reverse channel and errors,
so the canonical placement is compiled-in.

The `Feature RUN` verbs — `charly box feature run` / `charly check feature run` —
are **not** part of this plugin; they remain children of `box`/`check` in the core
binary.

## How to use it

```bash
charly feature list              # every feature entity
charly feature list candy        # filtered by kind
charly feature pending <entity>  # entities whose plan has not been graded
charly feature validate <entity> # validate an entity's plan/description shape
```

## Layout

- `candy/plugin-feature/` — the plugin module: `plugin.go` (provider + meta),
  `command.go` (the CLI + the plugin-side enumeration), `provider.go` (the
  `Invoke(OpRun)` surface), `schema/feature.cue`, `params/cue_types_gen.go`,
  `cmd/serve/main.go`.
- `charly.yml` — the root project manifest (`discover: candy`) + the
  `feature-skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Related

- Owning skill: `/charly-feature:feature` — `charly feature list|pending|validate`.
- `/charly-check:check` — the ADE run + grading flow.
- `/charly-internals:strict-policy` — the RDD/ADE/SDD discipline.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI.
