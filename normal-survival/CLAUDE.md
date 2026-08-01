# Normal Survival pack — mod manifest and Bedrock bridge

packwiz pack, **Minecraft 26.1.2 / Fabric loader 0.19.3** (pinned in `pack.toml`). Version
was chosen because it's the newest release Effortless Building has a Fabric build for —
Mojang is on year.month versioning now (current latest is 26.2), and Effortless Building
lags one release behind. General workflow and instance-recreation steps:
[../../CLAUDE.md](../../CLAUDE.md).

Design goal: a normal survival server. Spawn is deliberately unremarkable; the interesting
terrain (village, dark forest + mansion, pillager outpost, mountains) is nearby but not
visible from spawn. It originally shipped with an ambient "Herobrine" horror mod running
quietly in the background — **that was removed on 2026-08-01 at Jon's request**; see
"Removed: the Herobrine mod" below before adding anything like it back.

## Mods (`mods/*.pw.toml`, all from Modrinth)

| Mod | Slug | Side | Why it's here |
|---|---|---|---|
| Fabric API | `fabric-api` | both | base API |
| Effortless Building | `effortless-building` | both | building QoL: mirrors, arrays, build modes — works in survival |
| Simple Voice Chat | `simple-voice-chat` | both | in-game proximity voice |
| ReplayMod | `replaymod` | client | auto-records every session as tiny .mcpr files |
| Floodgate | `floodgate` | server | lets Bedrock players join without a linked Java/Xbox account |
| ViaFabric | `viafabric` | both | protocol bridge — see Bedrock bridge below |

## Bedrock bridge (Geyser)

Geyser-Fabric only ever supports the single latest Minecraft release (26.2), so it can't
run as a mod on this 26.1.2 server. Instead:

```
Bedrock client -> Geyser Standalone (emulates a 26.2 Java client, UDP 19132)
               -> ViaFabric on the server (translates 26.2 -> 26.1.2)
               -> Fabric server (26.1.2)
```

`Floodgate` (server mod) + a matching Floodgate key copied into Geyser Standalone's
config lets Bedrock players join without owning Java Edition. Geyser Standalone itself
lives in `instances\normal-survival\geyser\` — it's infra, not a packwiz mod, since it
runs as a separate process alongside the server (see that folder's notes / `start-geyser.ps1`).

## Removed: the Herobrine mod

`server-side-horror` (ambient sightings, footstep/mining sounds when alone, torches going
out, fake joiners, random signs and heads) was **removed on 2026-08-01 at Jon's request**,
along with `deimos`, its only dependent-free hard dependency — nothing else in the pack
declares it. Both were server-side, so no client change was needed and the existing world
is unaffected: everything the mod placed was vanilla blocks (signs, heads, torches), which
survive its removal as ordinary blocks.

The instance's tuning file `instances\normal-survival\config\serversidehorror.json` was
deliberately left in place — it is inert with the mod gone, and it is the only record of
the per-event frequencies if the mod is ever re-added. To restore: `packwiz modrinth add
server-side-horror` in this folder (it pulls Deimos back in), commit, re-sync the server.

This pack's docs, the root `CLAUDE.md`, and `packs\CLAUDE.md` all used to describe the
horror as the point of this server. They have been updated — if a stale reference turns
up, the removal is the current state, not a mistake.

## Seed

`level-seed=6673490222833072774`. Verified LIVE against this pack's actual 26.1.2 worldgen
via RCON `/locate` (not just trusted from a seed-list site — the first candidate tried,
`-8908401537643281118`, looked great on paper but its mansion turned out to be 2191
blocks from spawn once actually generated; always re-verify, worldgen shifts between
versions). Distances from spawn (0,0) on this seed:

| Feature | Distance | Notes |
|---|---|---|
| Woodland mansion | 124 blocks | at the base of the mountain |
| Pillager outpost | 102 blocks | |
| Dark forest | 96 blocks | same area as the mansion, low relative to the peak above it |
| Jagged Peaks mountains | 0 blocks | spawn itself sits at the foot of them |
| Village (plains) | 481 blocks | |

Re-verify after any worldgen-affecting update: `tools\rcon.ps1 -Port 25578 -PasswordFile
instances\normal-survival\rcon-password.txt -Commands 'locate structure minecraft:mansion', ...`
(see root `CLAUDE.md` for the full command list used). To try a different seed:
`tools\mcserver.ps1 seed normal-survival <seed>` (backs up the old world first).
