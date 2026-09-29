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
so the canonical placement is compiled-in. Nothing extra is needed to use it:
install `charly` and the subcommand is available.

The `Feature RUN` verbs — `charly box feature run` / `charly check feature run` —
are **not** part of this plugin; they remain children of `box`/`check` in the core
binary.

## How to use it

`charly feature` is compiled into the `charly` binary, so it is available as soon
as `charly` is installed — run it from a project directory (one containing a
`charly.yml`):

```bash
charly feature list              # every feature entity
charly feature list candy        # filtered by kind
charly feature pending <entity>  # entities whose plan has not been graded
charly feature validate <entity> # validate an entity's plan/description shape
```

## Platforms

Builds for `linux/amd64`, `linux/arm64` and `linux/arm/v7`. The armv7 target
works because of the sdk's 32-bit fix
([opencharly/sdk#263](https://github.com/opencharly/sdk/pull/263), issue
[#262](https://github.com/opencharly/sdk/issues/262)) — `charly`'s loader
host-builds this plugin with `CGO_ENABLED=0`, so the artifact is a static
binary, which is what an armv7 appliance without a glibc toolchain (such as a
JetKVM's uClibc userland) runs. Nothing extra is needed to use it: install
`charly` and it builds the plugin for the host it runs on.

## Layout

- `candy/plugin-feature/` — the plugin module: `plugin.go` (provider + meta),
  `command.go` (the CLI + the plugin-side enumeration), `provider.go` (the
  `Invoke(OpRun)` surface), `schema/feature.cue`, `params/cue_types_gen.go`,
  `cmd/serve/main.go`.
- `charly.yml` — the root project manifest (`discover: candy`) + the
  `feature-skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Related

- Owning skill: `/charly-feature:feature` — `charly feature list|pending|validate`,
  carried by this repo's own `feature-skill:` entity in `charly.yml`.
- `/charly-check:check` — the ADE run + grading flow.
- `/charly-internals:strict-policy` — the RDD/ADE/SDD discipline.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI.
