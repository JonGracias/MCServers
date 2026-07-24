# Normal Survival pack — mod manifest and Bedrock bridge

packwiz pack, **Minecraft 26.1.2 / Fabric loader 0.19.3** (pinned in `pack.toml`). Version
was chosen because it's the newest release Effortless Building has a Fabric build for —
Mojang is on year.month versioning now (current latest is 26.2), and Effortless Building
lags one release behind. General workflow and instance-recreation steps:
[../../CLAUDE.md](../../CLAUDE.md).

Design goal: looks and plays like a completely normal survival server — until it doesn't.
Spawn is deliberately unremarkable; the interesting terrain (village, dark forest +
mansion, pillager outpost, mountains) is nearby but not visible from spawn, and
Server-Side Horror runs quietly in the background from day one.

## Mods (`mods/*.pw.toml`, all from Modrinth)

| Mod | Slug | Side | Why it's here |
|---|---|---|---|
| Fabric API | `fabric-api` | both | base API |
| Effortless Building | `effortless-building` | both | building QoL: mirrors, arrays, build modes — works in survival |
| Simple Voice Chat | `simple-voice-chat` | both | in-game proximity voice |
| ReplayMod | `replaymod` | client | auto-records every session as tiny .mcpr files |
| Server-Side Horror | `server-side-horror` | server | the "Herobrine" content — ambient sightings/sounds/tampering, entirely server-side so no client mod is needed to experience it |
| Deimos | `deimos` | both | hard dependency of Server-Side Horror (packwiz caught this automatically, same pattern as Architectury for Modern Lucky Block in the lucky-blocks pack) |
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

## The Herobrine mod

`server-side-horror` ships sane ambient-horror defaults out of the box (occasional
sightings, footstep/mining sounds when alone, torches going out, one particle jumpscare)
— chosen deliberately over hand-scripted set-pieces so it needs zero maintenance. Config
lives in the synced `config/` folder if tuning (frequency/intensity) is ever wanted.
Entirely server-side — client players need no extra mod to see/hear it.

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
