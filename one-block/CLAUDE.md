# One Block pack — mod manifest and how the world is actually made

packwiz pack, **Minecraft 26.1.2 / Fabric loader 0.19.3** (pinned in `pack.toml`) — the same
versions as [normal-survival](../normal-survival/CLAUDE.md), for the same reason: 26.1.2 is the
newest release Effortless Building has a Fabric build for. General workflow and
instance-recreation steps: [../../CLAUDE.md](../../CLAUDE.md).

Design goal: the classic OneBlock challenge — you stand on a single regenerating block in an
empty void and mine your way through 21 phases — with the three mods Jon asked for on top of
it (ReplayMod, proximity voice, Effortless Building).

## Mods (`mods/*.pw.toml`, all from Modrinth)

| Mod | Slug | Side | Why it's here |
|---|---|---|---|
| Fabric API | `fabric-api` | both | base API |
| Simply OneBlock | `simply-oneblock` | **server** | the world itself — see below |
| Effortless Building | `effortless-building` | both | building QoL: mirrors, arrays, build modes — works in survival |
| Simple Voice Chat | `simple-voice-chat` | both | in-game proximity voice |
| ReplayMod | `replaymod` | client | auto-records every session as tiny .mcpr files |

**Effortless Building 4.3 needs no Cloth Config.** `normal-survival` carries
`cloth-config.pw.toml` because its 4.1 build depended on it; 4.3 declares no dependencies at
all (checked against the Modrinth API, not assumed — `packwiz ... -y` skips dependency prompts,
so a missing dep is silent until first boot).

## The world is a datapack, not a world type

`simply-oneblock`'s jar contains **no code** — it is a datapack wrapped as a Fabric mod
(`fabric.mod.json` declaring one dependency, `fabric-resource-loader-v0`, plus a `data/` tree).
What it does is replace `data/minecraft/dimension/overworld.json` with:

```json
{ "type": "minecraft:overworld",
  "generator": { "type": "minecraft:noise",
                 "biome_source": { "type": "minecraft:fixed", "biome": "minecraft:the_void" },
                 "settings": "minecraft:overworld" } }
```

so the overworld is void everywhere, and its `load`/`tick` mcfunctions place the starter block,
regenerate it on break, and drive the phases.

Three consequences worth keeping, because they are all counter-intuitive:

- **`level-type` does not matter.** The dimension override is applied from the datapack on every
  load, so it beats whatever world preset `server.properties` names. The instance is generated
  with `world_type: normal` and that is correct — *not* a mistake to "fix" by hunting for a void
  preset. Do **not** use the generator's `single_biome` here either: that writes an
  `mcworldgen:single_biome` world preset and points `level-type` at it, which this datapack
  would simply override, leaving a misleading config behind.
- **No seed constraints are meaningful.** Every biome is `the_void`, so `/locate biome` finds
  nothing and `/locate structure` has nothing biome-viable to find. A `spawn_near` block in the
  worldgen config would burn the whole 8-attempt seed search and fail honestly but pointlessly.
  The instance is therefore generated with no `spawn_near` at all.
- **The mod must be present before first generation**, which the jar form gives for free. The
  same project also publishes a raw datapack `.zip` that goes in `world/datapacks/` — that form
  was **rejected on purpose**: it needs the world folder to exist before it can be installed
  (chicken-and-egg on a fresh instance), and it would be a file edited inside an instance,
  which this project forbids precisely because it drifts from the pack.

## Why `side = "server"`

Modrinth marks it client-optional / server-required, and packwiz's auto-detection wrote
`side = "both"`; it was changed by hand to `server`. A datapack is server-side state — over the
network a client gets the world from the server, so the jar does nothing on a client except
cost every friend a 9.7 MB download. Same call as `floodgate` in `normal-survival`.

The one thing this gives up: a friend who makes a **singleplayer** world from this Prism tile
gets an ordinary world, not a one-block one. That is not what the pack is for (the tile
auto-joins Jon's server, and voice chat and ReplayMod are both multiplayer features), but if
singleplayer one-block is ever wanted, flip that one line to `side = "both"`, `packwiz refresh`,
commit and push — nothing else changes.

## Changing mods

Never edit an instance's `mods\` — edit here, `packwiz refresh`, commit and push this repo,
then stop/sync/start the server. Clients resync themselves at next Prism launch.
