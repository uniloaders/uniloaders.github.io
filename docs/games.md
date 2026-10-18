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
| Train Sim World 7 | Unreal Engine 4 | being ported | co-op, jump | [tsw-plugins.github.io](https://tsw-plugins.github.io/) |

### DayZ

Install uniloader as `dxgi.dll` next to `DayZ_x64.exe`, then put `overlay.dll`, `dayz.dll`,
the `dayz-data/` database folder and the plugins you want into `plugins/`. Start the game with
`-nobe`: BattlEye rejects modified clients, so this is for offline play and servers without it.
The adapter's README lists its console commands (`dayz.symbols`, `dayz.game`, `dayz.read`,
`dayz.connect`, ...).

Plugins that are not tied to a game work everywhere: the [overlay](overlay.html), the
[Lua bridge](lua.html), and `.asi` mods.

## Adding a game

Start with the [Rust guide](plugins.html) and the kitchen sink example. Write the adapter
first, give it a service with a small, versioned table, and keep game knowledge out of the
feature plugins, so a game update means fixing one plugin.
