# Lucky Blocks pack — mod manifest and local patches

packwiz pack, **Minecraft 1.21.5 / Fabric loader 0.19.3** (pinned in `pack.toml`).
Version was chosen as the newest where all mods below overlap on Fabric.
General workflow and instance-recreation steps: [../../CLAUDE.md](../../CLAUDE.md).

## Mods (`mods/*.pw.toml`, all from Modrinth)

| Mod | Slug | Side | Why it's here |
|---|---|---|---|
| Fabric API | `fabric-api` | both | base API almost every Fabric mod needs |
| Architectury API | `architectury-api` | both | hard dependency of Modern Lucky Block (packwiz `-y` didn't auto-add it; the dry-boot caught it) |
| Modern Lucky Block | `modern-lucky-block` | both | the lucky block ENGINE only — ships zero content; content comes from the addon below |
| Simple Voice Chat | `simple-voice-chat` | both | in-game proximity voice; server listens on UDP 24454 |
| ReplayMod | `replaymod` | client | auto-records every session as tiny .mcpr files for cinematic re-filming (Fabric-only mod — this is why the pack is Fabric, not NeoForge) |

## Bundled addon (`addons/modern-lucky-block-0.2.1+1.21.4.zip`)

"Modern Lucky Block Addon" by creoii (Modrinth slug `modern-lucky-block-addon`) — the
classic/original Lucky Block content: 100+ outcomes, textures, crafting recipe. Modern
Lucky Block loads it from the game dir's `addons/` folder (NOT `mods/`, NOT world
datapacks). It's stored in the pack as a plain file override so packwiz-installer
delivers it to server and clients alike. The 0.2.1+1.21.4 build covers MC 1.21.4-1.21.8.

### ⚠ This zip carries three LOCAL PATCHES — do not re-download without re-applying

1. **Recipe format fix** (`data/lucky/recipe/lucky_block.json`): upstream uses the
   pre-1.21.5 `{"item": "..."}` key format which 1.21.5 rejects ("Couldn't parse data
   file" in server log, recipe silently missing). Patched to plain-string keys:
   `"X": "minecraft:gold_ingot", "D": "minecraft:dropper"` (8 gold around a dropper).

2. **Outcome weight fix** (upstream bug [jack-zisa/lucky_block#16](https://github.com/jack-zisa/lucky_block/issues/16),
   still open as of 2026-07-15): `tnt_on_bat.json` had `"chance": 10000` and
   `ender_pearls.json` / `launched_tnt.json` had `"chance": 1000` while the other 92
   outcomes default to ~1 — so ~99% of rolls were TNT bats/pearls. Patched by deleting
   the `chance` field from those three files.

3. **Item model definition** (`assets/lucky/items/lucky_block.json`, ADDED): MC 1.21.4+
   requires item model definitions in `assets/<ns>/items/`; the addon only ships the old
   `models/item/` file, so the item icon rendered as the magenta/black missing texture.
   Added `{"model": {"type": "minecraft:model", "model": "lucky:block/lucky_block"}}`.

All patches are separate commits in this repo's git history. Before updating the addon,
check whether upstream fixed both; otherwise re-apply (edit JSON inside the zip, then
`packwiz refresh` + commit).

## Facts that save debugging time

- Item/block ID: `lucky:lucky_block` (`/give @s lucky:lucky_block 64` — needs op).
- Lucky blocks do NOT generate naturally in terrain with this addon — crafting or /give only.
  (If natural spawning is ever wanted: add a small worldgen datapack to the pack.)
- `No data fixer registered for lucky:lucky_block` in the server log is cosmetic — ignore.
- Addons load at game start: after changing the addon zip, restart server AND clients.
  Outcome logic runs server-side; client copy only supplies textures/lang.
- Simple Voice Chat needs its client mod — vanilla or Bedrock(-bridged) players can't use it.
