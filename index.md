---
title: Install
---

# uniloader

A plugin loader for any Windows game, written in Rust. You drop **one DLL** into the game
folder, under the name of a system library the game already loads. It passes every call on
to the real library and loads the plugins in `plugins/`: native plugins, Lua scripts and
`.asi` mods. It runs the same under Windows and under Proton or Wine on Linux.

## Install

1. **Download** `uniloader.dll` and the plugins you want from the
   [latest release](https://github.com/uniloaders/uniloader/releases/latest).
2. **Find the game's executable folder.** For Unreal Engine games this is usually
   `<Game>/Binaries/Win64`, not the folder with the launcher.
3. **Pick a name** the game imports from the system, and rename `uniloader.dll` to it. Most
   games work with the first one in this list that they load:

   | Name | Works for |
   | --- | --- |
   | `dxgi.dll` | almost every Direct3D 10, 11 or 12 game |
   | `version.dll` | most games, whatever they render with |
   | `winmm.dll` | games that play sound through WinMM |
   | `dwmapi.dll` | Unreal Engine games |
   | `dinput8.dll` | games that read input through DirectInput |
   | `d3d11.dll`, `d3d12.dll`, `opengl32.dll`, `winhttp.dll`, `xinput1_3.dll`, `xinput1_4.dll` | when the above are taken |

   If another mod already uses that name (ReShade often is `dxgi.dll`), pick a different one,
   or rename the other mod and let uniloader chain-load it (see [`.asi` and
   chain-loading](docs/asi.html)).
4. **Copy** it next to the game executable, and the plugins into a `plugins` folder there:

   ```text
   <game exe dir>/
     version.dll           uniloader.dll, renamed
     plugins/
       overlay.dll         the in-game overlay, if wanted
       lua_bridge.dll      Lua scripting, if wanted
       lua/*.lua           scripts for it
       something.asi       .asi mods
   ```

5. **On Linux (Proton or Wine)**, tell Wine to use the DLL from the game folder. In Steam,
   set the game's launch options to, for example:

   ```text
   WINEDLLOVERRIDES="version=n,b" %command%
   ```

   using the name you picked. Windows needs nothing more.

## Check that it works

Start the game. uniloader writes a log to `uniloader/logs/` next to the executable. The
first lines name the DLL it stands in for and how many functions it forwards. Further down,
it lists each plugin it starts and how many are running:

```text
[INFO] uniloader::win: uniloader 0.1.0 in C:\Games\Example
[INFO] uniloader::win::proxy: proxy for version.dll: 17 of 17 listed exports forwarded
[INFO] uniloader::win::plugins: starting overlay 0.1.0 from overlay.dll
[INFO] uniloader::win: 2 of 2 plugins running
```

With the overlay plugin installed, press the key under Escape (or `ctrl+shift+c`) for the
console and `F11` for the settings of every plugin. Without it, start the game with
`--loader-console` for a console window.

No log at all means the game never loaded the DLL under that name. Try the next name in the
list, or check the Wine override on Linux.

## Uninstall

Delete the renamed DLL. The `plugins` and `uniloader` folders do nothing without it; delete
them too to remove settings and logs.

## What next

- [Configuration](docs/configuration.html): `loader.toml`, command line, where files go.
- [Console and hotkeys](docs/console.html): built-in commands, the key grammar.
- [Overlay](docs/overlay.html): the in-game console, the settings editor, plugin panels.
- [Writing plugins](docs/plugins.html) in Rust, or in any language over the
  [C ABI](docs/abi.html), or as [Lua scripts](docs/lua.html).
- [Games](docs/games.html): what game-specific plugins exist.
