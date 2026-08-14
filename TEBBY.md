# TEBBY.md - pickle_restaurants_assets

## Overview

`pickle_restaurants_assets` is a pure-asset FiveM resource that ships all custom 3-D
models (`.ydr`) and the accompanying type-definition file (`.ytyp`) required by the
`pickle_restaurants` gameplay resource. It contains no Lua logic, no client/server
scripts, and no NUI. Its only job is to stream custom props and register their types
with the GTA 5 engine so that `pickle_restaurants` can spawn them in-world.

## Code design

This resource has no code. Its architecture is entirely declarative:

- **`fxmanifest.lua`** — the FiveM resource manifest. It declares `fx_version
  'cerulean'`, targets `gta5`, enables Lua 5.4 (though no Lua is used), and lists
  `stream/pickle_restaurants.ytyp` in both the `files` block (so the file is served
  to clients) and as a `DLC_ITYP_REQUEST` data file (so the engine registers the
  prop types at resource start).
- **`stream/`** — contains every custom model. All files are prefixed `pickle_` and
  follow a `<category>_<state/variant>_<index>.ydr` naming convention (e.g.
  `pickle_patty_beef_01.ydr`, `pickle_patty_beef_02.ydr`, `pickle_patty_beef_03.ydr`
  represent raw → cooking → cooked states). The `.ytyp` file at
  `stream/pickle_restaurants.ytyp` is the archetype definition that maps model names
  to engine metadata.

There is no client/server split, no NUI, and no configuration. The resource is
started before `pickle_restaurants` so its models are available when that resource
needs them.

## Important files

| Path | Why it matters |
|---|---|
| `fxmanifest.lua` | Resource manifest — declares the resource to FiveM, registers the `.ytyp`, and serves assets to clients. The only file to edit for resource-level changes. |
| `stream/pickle_restaurants.ytyp` | Archetype type definition — registers all custom prop model names with the GTA 5 engine. Without it, spawned props appear as missing/invisible. |
| `stream/pickle_*.ydr` | Individual prop models (≈ 100 files) — every cooking station state, food item, packaging object, and kitchen equipment variant used by `pickle_restaurants`. |

## Intended behaviors

- **No Lua entry point is intentional.** The manifest declares `lua54 'yes'` but
  there are no scripts. This is correct — the resource is assets-only. A future
  triage agent should not flag the absence of client/server scripts as a bug.
- **The `.ytyp` is listed twice in `fxmanifest.lua`** — once under `files {}` and
  once as a `data_file 'DLC_ITYP_REQUEST'`. Both entries are required by FiveM: the
  `files` entry delivers it to clients, and the `data_file` entry tells the engine to
  load it as a type registry. Removing either entry breaks prop spawning.
- **All model filenames are prefixed `pickle_`** — this is a deliberate namespace
  convention that prevents collisions with GTA base assets and other resources.
  Do not rename or un-prefix models.
- **Multi-stage models use a numeric suffix** (`_01`, `_02`, `_03`) to represent
  cooking or assembly states (e.g. raw/cooking/cooked for patties, drumsticks, fries,
  doughnuts; open/closed for pizza box; dough application stages). These variants are
  swapped programmatically by `pickle_restaurants` — all variants must stay present.

## Common issues & fixes

| Issue | Likely cause | Fix |
|---|---|---|
| Props are invisible or show as missing models in-game | `pickle_restaurants_assets` is not started before `pickle_restaurants`, or the `.ytyp` is not loading. | Ensure `ensure pickle_restaurants_assets` appears in `server.cfg` **before** `ensure pickle_restaurants`. |
| `DLC_ITYP_REQUEST` error in server console | The `.ytyp` path in `fxmanifest.lua` does not match the actual file location. | Confirm `stream/pickle_restaurants.ytyp` exists and matches exactly — the path is case-sensitive on Linux. |
| Locale / UI text issues in `pickle_restaurants` | Wrong resource folder name. | This is a `pickle_restaurants` issue, not an assets issue. The assets resource name (`pickle_restaurants_assets`) must not be confused with the main resource (`pickle_restaurants`). |
| A specific prop variant is missing (e.g. cooked state) | A `.ydr` file was accidentally deleted or the `.ytyp` was not updated after adding new models. | Restore the missing `.ydr` from version control and, if a new model was added, re-export the `.ytyp` to include it. |

## Conventions

- **File naming:** all assets follow `pickle_<noun>_<descriptor>_<two-digit-index>.ydr`
  (e.g. `pickle_cake_mix_pour_03.ydr`). New models must follow this pattern.
- **No scripts:** do not add Lua files to this resource. Any new logic belongs in
  `pickle_restaurants`, not here.
- **Manifest edits:** if a new `.ytyp` file is ever introduced, it must be added to
  both the `files {}` block and a `data_file 'DLC_ITYP_REQUEST'` entry.
- **Stream folder only:** all binary assets live flat inside `stream/` — no
  sub-directories. FiveM's streamer expects a flat layout for `.ydr` files.
- **Version field:** the `version` field in `fxmanifest.lua` is managed automatically
  by Tebby Code. Do not edit it manually in a fix PR.
