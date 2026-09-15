# Friends Lobby pack - the doorway, deliberately almost empty

packwiz pack, **Minecraft 26.1.2 / Fabric loader 0.19.3**. This is the pack for the world
named by `API_VERIFY_INSTANCE`: the server a player joins for thirty seconds to prove they
own their Minecraft name (`api/app/verification.py`).

**NOTHING HERE MAY REQUIRE A CLIENT MOD.** Verification is the step before anybody has a
pack, so the client joining is a stock one. The lobby used to borrow normal-survival's
pack, and Fish of Thieves' registry entries got vanilla clients kicked at login with
"This server requires Fabric Loader and Fabric API installed on your client" - which asked
players to install a modpack in order to become able to install modpacks.

| Mod | Slug | Side | Why it's here |
|---|---|---|---|
| Fabric API | `fabric-api` | both | the loader's base; ViaFabric needs it |
| ViaFabric | `viafabric` | both | protocol bridge, so a client that is not exactly 26.1.2 still gets in |

Adding anything that registers a block, item, entity or dimension breaks this world's only
job. If a mod is wanted for the lobby, check it against a vanilla client first.
