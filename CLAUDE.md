# MCServers — Minecraft pack installer manual for Claude

This repo (https://github.com/JonGracias/MCServers) holds Minecraft modpacks, one folder
per pack. Each pack folder is a packwiz pack with its own `CLAUDE.md` (what's in it, why,
known quirks). You — Claude, on the computer this repo was cloned to — are the installer.

**Standard location on a receiving computer: `C:\MCServers`** (this folder = the clone).
Work inside this folder and inside `%APPDATA%\PrismLauncher\` only.

## Rules — read before doing anything

1. **FIRST, before any other action, ask the user two things:**
   - *Which pack do you want?* (list the pack folders in this repo)
   - *What is your exact Minecraft username?* — tell them to read it from their launcher,
     not from memory. Report the username back to Jon so it can be whitelisted on his
     server (a misspelled name silently whitelists a stranger's account — this has
     actually happened).
2. **Assume the user already owns Minecraft Java Edition and is (or will be) signed in.**
   Do not manage Microsoft accounts. If Prism ends up with no account signed in, just
   tell the user: Prism → Settings → Accounts → Add Microsoft.
3. **Never do work that's already done.** Run the state check below first, every time,
   and skip completed steps. Report what you found already in place.
4. Downloads need the user's OK (Prism ~100MB from winget, bootstrap jar ~100KB from GitHub).

## State check (run this first, act only on what's missing)

| Check | How | If missing |
|---|---|---|
| repo up to date | `git -C C:\MCServers pull` | — (always pull) |
| Prism installed | `Test-Path "$env:LOCALAPPDATA\Programs\PrismLauncher\prismlauncher.exe"` or `winget list PrismLauncher` | step 1 below |
| instance exists | `Test-Path "$env:APPDATA\PrismLauncher\instances\<PackName>\instance.cfg"` | steps 2-4 below |
| bootstrap jar | `Test-Path "...instances\<PackName>\.minecraft\packwiz-installer-bootstrap.jar"` | step 3 below |
| account signed in | `%APPDATA%\PrismLauncher\accounts.json` exists and non-empty | tell user (rule 2) |
| mods synced | `.minecraft\mods\` non-empty after first launch | launches on Play; no action |

A `.setup-state.json` may exist next to this file on the receiving machine (it is
gitignored). After finishing, write/update it: `{ "packs": { "<PackName>": { "instance": true,
"username": "<name>", "whitelistReported": true, "date": "<iso>" } } }` — future sessions
read it as a shortcut but should still trust the checks above over the file.

## Install steps (skip any the state check passed)

1. **Prism Launcher**: `winget install PrismLauncher.PrismLauncher --silent --accept-package-agreements --accept-source-agreements`

2. **Instance folder**: `%APPDATA%\PrismLauncher\instances\<PackName>\` (use the pack's
   display name, e.g. "Lucky Blocks" for `lucky-blocks`). Two files:

   `mmc-pack.json` — versions come from the pack's `pack.toml` `[versions]` section:
   ```json
   {
       "components": [
           { "important": true, "uid": "net.minecraft", "version": "<minecraft>" },
           { "uid": "net.fabricmc.fabric-loader", "version": "<fabric>" }
       ],
       "formatVersion": 1
   }
   ```

   `instance.cfg` — ⚠ NO quotes in PreLaunchCommand (Prism's INI parser corrupts them),
   forward slashes only, path must have no spaces:
   ```ini
   [General]
   ConfigVersion=1.2
   InstanceType=OneSix
   iconKey=default
   name=<PackName>
   OverrideCommands=true
   PreLaunchCommand=$INST_JAVA -jar packwiz-installer-bootstrap.jar C:/MCServers/<pack-folder>/pack.toml
   ```

3. **Bootstrap jar** into `<instance>\.minecraft\`:
   `https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar`

4. **If Prism was open during any of this**: user must fully quit and reopen Prism
   (it caches instance configs in memory and writes stale ones back).

5. **Auto-join instances (optional but recommended).** A pack can have multiple tiles,
   one per server, that connect automatically on launch. Create an additional instance
   folder (e.g. "Lucky Block PVP") exactly as above, plus two lines in `instance.cfg`:
   ```ini
   JoinServerOnLaunch=true
   JoinServerOnLaunchAddress=10.0.0.172:25566
   ```
   For the `lucky-blocks` pack, create BOTH tiles: **"Lucky Blocks"** (auto-join
   `10.0.0.172:25565`) and **"Lucky Block PVP"** (auto-join `10.0.0.172:25566`).
   For the `better-build` pack, create one tile: **"Better Build"** (auto-join
   `10.0.0.172:25567`).
   They share the pack, so mods stay identical; each keeps its own settings and
   ReplayMod recordings. If one instance already exists, copy its `.minecraft\options.txt`
   into the new one so the user's keybinds/settings carry over.

6. **Done.** User presses Play → pre-launch hook downloads all mods from the local clone.
   Tell the user: after Jon announces any pack change, run `git -C C:\MCServers pull`
   (or ask you to) before launching.

## Joining Jon's servers

Jon's laptop hosts on `10.0.0.172` (home Wi-Fi, DHCP — if unreachable, ask Jon to
re-check his IP and that the server is running). Three servers, two packs:

| Server | Address | Pack | What it is |
|---|---|---|---|
| Lucky Blocks (Survival) | `10.0.0.172:25565` | `lucky-blocks` | survival world with lucky blocks |
| Lucky Block PVP Arena | `10.0.0.172:25566` | `lucky-blocks` | Lucky Block Walls arena: open your team's wall of lucky blocks for gear, fight in the middle |
| Better Build (Survival) | `10.0.0.172:25567` | `better-build` | near-vanilla survival with Effortless Building (mirrors, arrays, build modes) |

- The user's username must be whitelisted (per server) first — rule 1 covers collecting it.
- Voice chat (proximity, press V) and ReplayMod (auto-records, red dot top-left) are in
  the `lucky-blocks` pack and work automatically on both its servers. The `better-build`
  pack has neither — it's near-vanilla.

## Packs in this repo

| Folder | Display name(s) | MC / loader | Docs |
|---|---|---|---|
| `lucky-blocks` | Lucky Blocks + Lucky Block PVP (two instances, one pack) | 1.21.5 / Fabric 0.19.3 | [lucky-blocks/CLAUDE.md](lucky-blocks/CLAUDE.md) |
| `better-build` | Better Build | 1.21.11 / Fabric 0.19.3 | [better-build/CLAUDE.md](better-build/CLAUDE.md) |

Adding a new pack (done on Jon's machine): create `<folder>\` with packwiz
(`packwiz init`), add mods, add a `CLAUDE.md`, extend `.packwizignore` pattern
(each pack ignores its own docs), add a row to this table, commit, push.
