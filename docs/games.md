---
title: Games
---

# Games

uniloader itself knows nothing about any game. Game support comes in two layers of plugins:

- An **adapter plugin** knows one engine: where its objects are, how its functions are
  called, when a level loads. It offers that as a [service](abi.html#services) and as
  broadcast events.
- **Feature plugins** depend on the adapter and do the actual modding.

| Game | Engine | Plugins | Site |
| --- | --- | --- | --- |
| DayZ | Enfusion | debug tools, log, patches, VR | [dayz-plugins.github.io](https://dayz-plugins.github.io/) |
| Train Sim World 7 | Unreal Engine 4 | co-op, jump | [tsw-plugins.github.io](https://tsw-plugins.github.io/) |

Both started on loaders of their own and are moving onto uniloader; each game's site says
which loader its current releases need.

Plugins that are not tied to a game work everywhere: the [overlay](overlay.html), the
[Lua bridge](lua.html), and `.asi` mods.

## Adding a game

Start with the [Rust guide](plugins.html) and the kitchen sink example. Write the adapter
first, give it a service with a small, versioned table, and keep game knowledge out of the
feature plugins, so a game update means fixing one plugin.
