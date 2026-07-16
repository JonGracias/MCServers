# Setting up this pack on a new computer

**Client setup (most machines): don't follow this file.** Follow the repo-root
[../CLAUDE.md](../CLAUDE.md) — it is the installer manual (asks for username first,
checks what's already installed, creates the Prism instance). Pack-specific facts and
known quirks live in [CLAUDE.md](CLAUDE.md).

The rest of this file covers the one thing the root manual doesn't: building a **server**
for this pack.

## Server build (only if this machine should host its own world)

1. Java 21+ required system-wide: `winget install EclipseAdoptium.Temurin.21.JDK` (ask the user first).
2. Create an instance folder (path without spaces), inside it download:
   - `fabric-server.jar` from `https://meta.fabricmc.net/v2/versions/loader/<mc>/<loader>/<latest-installer>/server/jar`
     — mc + loader versions from this pack's `pack.toml`, latest installer version from
     `https://meta.fabricmc.net/v2/versions/installer`
   - `packwiz-installer-bootstrap.jar` from
     `https://github.com/packwiz/packwiz-installer-bootstrap/releases/latest/download/packwiz-installer-bootstrap.jar`
3. Sync server-side mods: `java -jar packwiz-installer-bootstrap.jar -g -s server <path-to-this-folder>\pack.toml`
4. Dry-boot: `java -Xmx3G -jar fabric-server.jar nogui` — must reach the EULA error with
   NO mod-resolution errors (this catches missing dependencies loudly).
5. EULA: requires the machine owner's explicit agreement to https://aka.ms/MinecraftEULA —
   only then set `eula=true` in `eula.txt`.
6. `server.properties`: `white-list=true`, `enforce-whitelist=true`, set `motd`.
7. `whitelist.json` / `ops.json`: UUIDs from `https://api.mojang.com/users/profiles/minecraft/<name>`,
   dash-formatted (8-4-4-4-12). Verify username spelling against the launcher, never memory.
8. Boot and verify the log shows `Done` and `[voicechat] Voice chat server started at port 24454`.
9. For remote players: TCP 25565 + UDP 24454 must be reachable (firewall/router/tunnel).

## Sanity checklist after any setup

- [ ] Prism instance launches, console shows packwiz downloading 5 files (4 mods + addon zip)
- [ ] In-game: red RECORDING indicator top-left (ReplayMod)
- [ ] In-game: voice chat mic icon (press V for the voice menu)
- [ ] `/give @s lucky:lucky_block 1` works if opped (or craft: 8 gold ingots around a dropper)
- [ ] Lucky block item shows its texture (if magenta/black checker, the addon zip lost
      its item-model patch — see CLAUDE.md)
- [ ] Breaking lucky blocks gives varied outcomes, not only TNT bats (if only bats, the
      weights patch is missing — see CLAUDE.md)
