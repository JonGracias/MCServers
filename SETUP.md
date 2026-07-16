# Setting up this pack on a new computer

Instructions for Claude Code (or a careful human) on a fresh Windows machine.
Read [CLAUDE.md](CLAUDE.md) first for what's in the pack and why.

There are two scenarios. Most new machines (e.g. a family member's PC joining Jon's
server) only need the **client setup**. Only build a server if this machine should host
its own world.

---

## Scenario A: Client only — join Jon's server (most common)

Prerequisites the user must have: a Microsoft account that **owns Minecraft Java Edition**.

1. **Verify Java is NOT needed system-wide** — Prism manages its own Java. Skip JDK installs.

2. **Install Prism Launcher** (ask the user before downloading):
   ```
   winget install PrismLauncher.PrismLauncher --silent --accept-package-agreements --accept-source-agreements
   ```

3. **Get this pack repo onto the machine** (clone it, or it's already here if you're reading this).
   Note its absolute path; below it's called `<PACK>` (e.g. `C:\Users\kid\lucky-blocks`).

4. **Create the Prism instance** at `%APPDATA%\PrismLauncher\instances\Lucky Blocks\`:

   `mmc-pack.json` (versions must match `pack.toml` `[versions]` — currently MC 1.21.5, Fabric 0.19.3):
   ```json
   {
       "components": [
           { "important": true, "uid": "net.minecraft", "version": "1.21.5" },
           { "uid": "net.fabricmc.fabric-loader", "version": "0.19.3" }
       ],
       "formatVersion": 1
   }
   ```

   `instance.cfg` — ⚠ NO quotes in PreLaunchCommand (Prism's INI parser breaks on them),
   forward slashes, and the pack path must contain NO spaces:
   ```ini
   [General]
   ConfigVersion=1.2
   InstanceType=OneSix
   iconKey=default
   name=Lucky Blocks
   OverrideCommands=true
   PreLaunchCommand=$INST_JAVA -jar packwiz-installer-bootstrap.jar <PACK>/pack.toml
   ```

   `.minecraft\packwiz-installer-bootstrap.jar` — download into the folder:
   ```
   https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar
   ```
   (Ask the user before downloading.)

5. **User signs in**: open Prism → Settings → Accounts → Add Microsoft → their account.
   If Prism was already open during step 4, fully quit and reopen it first.

6. **Launch**: Play on the Lucky Blocks instance. The pre-launch hook downloads all mods
   from the pack automatically (ReplayMod will show a red RECORDING indicator in-game — that's
   normal and wanted; it records replays for Jon's YouTube videos).

7. **Whitelist**: this player's Minecraft username must be whitelisted on Jon's server.
   Get the username from the Prism account dropdown (top-right) — do NOT trust spelling
   from memory; a misspelled name silently resolves to a stranger's account. Then someone
   opped (Jon/Datakin) runs `/whitelist add <name>` in-game, or Claude on Jon's machine
   updates it.

8. **Staying up to date**: the pre-launch hook syncs from the LOCAL clone, so when the
   pack changes upstream, run `git pull` in the clone folder before launching. If mods
   seem out of date or textures break after Jon announces a fix, pull first.

9. **Connect**: Multiplayer → Add Server → address `10.0.0.172` (Jon's laptop on home
   Wi-Fi as of 2026-07-16 — it's DHCP, so if connection fails ask Jon's Claude to
   re-check with `Get-NetIPAddress`). The server must be running on Jon's laptop.
   - If the server is up but unreachable from another machine, Windows Firewall on
     Jon's laptop is likely blocking inbound Java — Jon must allow it (firewall prompt
     or Settings), Claude should not change firewall rules.
   - Voice chat additionally needs UDP 24454 to reach the host; same firewall note.

## Scenario B: Full server + client on this machine

Follow Scenario A, then build the server:

1. Java 21+ required system-wide: `winget install EclipseAdoptium.Temurin.21.JDK` (ask first).
2. Create an instance folder (path without spaces), inside it:
   - `fabric-server.jar` from `https://meta.fabricmc.net/v2/versions/loader/1.21.5/0.19.3/<latest-installer>/server/jar`
     (latest installer version: `https://meta.fabricmc.net/v2/versions/installer`)
   - `packwiz-installer-bootstrap.jar` (same URL as above)
3. Sync server-side mods: `java -jar packwiz-installer-bootstrap.jar -g -s server <PACK>\pack.toml`
4. Dry-boot: `java -Xmx3G -jar fabric-server.jar nogui` — it must reach the EULA error
   with NO mod-resolution errors. (This catches missing dependencies.)
5. EULA: requires the machine owner's explicit agreement to https://aka.ms/MinecraftEULA —
   only then set `eula=true` in `eula.txt`.
6. `server.properties`: `white-list=true`, `enforce-whitelist=true`, set `motd`.
7. `whitelist.json` / `ops.json`: UUIDs from `https://api.mojang.com/users/profiles/minecraft/<name>`,
   dash-formatted (8-4-4-4-12), verify the username spelling against the launcher (see step 7 above).
8. Boot and verify `Done` + `[voicechat] Voice chat server started at port 24454` in the log.

## Sanity checklist after setup

- [ ] Prism instance launches, console shows packwiz downloading 5 files (4 mods + addon zip)
- [ ] In-game: red RECORDING indicator top-left (ReplayMod)
- [ ] In-game: voice chat onboarding popup or mic icon (press V for the voice menu)
- [ ] Joined the server and `/give @s lucky:lucky_block 1` works if opped (or craft: 8 gold ingots around a dropper)
- [ ] Breaking lucky blocks gives varied outcomes, not only TNT bats (if only bats, the
      addon zip lost its local patch — see CLAUDE.md)
