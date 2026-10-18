---
title: Installing with RaiPal
---

# Installing with RaiPal

[RaiPal](https://github.com/Raicuparta/rai-pal) finds your games (Steam, Epic, GOG, Xbox,
itch, Heroic and folders you add), installs mod loaders into them, remembers what it installed where, and
uninstalls it again. It runs on Windows and Linux, where it also sets the Wine override for
you. uniloader is in a mod list RaiPal can read:

```text
https://uniloaders.github.io/raipal/mods.json
```

Add it in RaiPal's mod sources, or open
[this link](rai-pal://add-mod-source?url=https%3A%2F%2Funiloaders.github.io%2Fraipal%2Fmods.json)
with RaiPal installed. The list has three mods:

| Mod | What it installs |
| --- | --- |
| uniloader | `dwmapi.dll` next to the game executable, and `uniloader-root.txt` pointing at RaiPal's folder for that game, where `plugins/`, `config/` and `logs/` go |
| uniloader overlay | `overlay.dll` in that `plugins/` folder (needs uniloader) |
| uniloader Lua bridge | `lua_bridge.dll` in that `plugins/` folder (needs uniloader) |

Everything except `dwmapi.dll` and `uniloader-root.txt` stays out of the game's folder: open
the game's mods folder in RaiPal to find `uniloader/plugins/` for your other plugins and Lua
scripts. See [Configuration](configuration.html#uniloader-roottxt) for how the root file
works.

## Things to know

- **UE4SS** from RaiPal uses `dwmapi.dll` as well. Install one of the two. To run UE4SS with
  uniloader, install uniloader and chain-load UE4SS: see
  [`.asi` mods and chain-loading](asi.html).
- **Uninstalling** removes what RaiPal installed: the DLL, the root file and the `plugins/`
  folder in RaiPal's game folder, including plugins you put there yourself. `config/` and
  `logs/` stay. A `dwmapi.dll` that another mod had placed before is not brought back.
