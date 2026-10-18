---
title: Games
---

# Games

uniloader itself knows nothing about any game. Game support comes in two layers of plugins:

- An **adapter plugin** knows one engine: where its objects are, how its functions are
  called, when a level loads. It offers that as a [service](abi.html#services) and as
  broadcast events.
- **Feature plugins** depend on the adapter and do the actual modding.

| Game | Engine | Adapter | Plugins | Site |
| --- | --- | --- | --- | --- |
| DayZ | Enfusion | [dayz-adapter](https://github.com/dayz-plugins/dayz-adapter) | [game log](https://github.com/dayz-plugins/dayz-log-plugin), [engine patches](https://github.com/dayz-plugins/dayz-patches-plugin), [debug socket](https://github.com/dayz-plugins/dayz-debug-plugin) | [dayz-plugins.github.io](https://dayz-plugins.github.io/) |
| Train Sim World 7 | Unreal Engine 4 | [tsw-sdk](https://tsw-plugins.github.io/plugins/tsw-sdk) (`tsw.dll`) | [jump](https://tsw-plugins.github.io/plugins/tsw-jump), [co-op](https://tsw-plugins.github.io/plugins/tsw-coop/) | [tsw-plugins.github.io](https://tsw-plugins.github.io/) |

### DayZ

Install uniloader as `dxgi.dll` next to `DayZ_x64.exe`, then put `overlay.dll`, `dayz.dll`,
the `dayz-data/` database folder and the plugins you want into `plugins/`. Start the game with
`-nobe`: BattlEye rejects modified clients, so this is for offline play and servers without it.
The adapter's README lists its console commands (`dayz.symbols`, `dayz.game`, `dayz.read`,
`dayz.connect`, ...).

### Train Sim World 7

Install uniloader as `dwmapi.dll` next to `TrainSimWorld.exe` (in `Binaries/Win64`; under
Proton the launch option `WINEDLLOVERRIDES="dwmapi=n,b" %command%` is needed), then `tsw.dll`,
`tsw/puzzle.png` and the plugins you want into `plugins/`. The adapter scans the engine once,
hooks the game tick and hands both to the other plugins; it also skips the start-up screens and
puts a puzzle piece on the main menu that lists uniloader, its plugins and UE4SS. UE4SS can run
alongside: `load = ["ue4ss/UE4SS.dll"]` in `loader.toml`. The overlay does not draw there yet
(the game runs on Direct3D 12 through vkd3d-proton under Proton).
[Install guide](https://tsw-plugins.github.io/install).

Plugins that are not tied to a game work everywhere: the [overlay](overlay.html), the
[Lua bridge](lua.html), and `.asi` mods.

## Adding a game

Start with the [Rust guide](plugins.html) and the kitchen sink example. Write the adapter
first, give it a service with a small, versioned table, and keep game knowledge out of the
feature plugins, so a game update means fixing one plugin.
