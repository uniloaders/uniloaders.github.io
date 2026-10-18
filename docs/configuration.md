---
title: Configuration
---

# Configuration

## Files

Everything uniloader writes lives in one folder next to the game executable. Only `plugins/`
is meant for you to put files into.

```text
<game exe dir>/
  plugins/                    plugin DLLs, .asi mods, lua/ scripts
  uniloader/
    config/
      loader.toml             the loader's own settings (you write this; optional)
      hotkeys.toml            hotkey bindings you changed (written by the loader)
      <plugin>.toml           each plugin's settings (written by the loader)
      overlay-windows.toml    where the overlay's windows were left
    logs/
      loader.log              this run
      loader.prev.log         the run before
```

The per-plugin files and `hotkeys.toml` only hold what differs from the defaults, so deleting
one resets that plugin. They are plain `key = value` TOML and can be edited while the game is
closed.

## `loader.toml`

Every key is optional.

```toml
enabled = true                  # false: forward the system DLL, load nothing
disabled_plugins = ["jump"]     # file stems that are never loaded
log_level = "info"              # error, warn, info, debug, trace
asi = true                      # load plugins/*.asi
load = ["ue4ss/UE4SS.dll"]      # other DLLs to load, relative to the game executable
```

`load` is for DLLs uniloader displaced. If the game already had a `dwmapi.dll` mod and you
installed uniloader under that name, rename the other one and list it here; see
[chain-loading](asi.html).

## Command line

Add these to the game's own command line (in Steam: launch options, after `%command%`):

| Flag | Effect |
| --- | --- |
| `--loader-console` | Open a console window that shows the log and reads commands. |
| `--noplugins` | Load nothing; forward only. For checking whether a problem is a plugin's. |
| `--loader-log=debug` | Override `log_level` for this run. |

`-flag` and `/flag` work as well, and so does `--flag value` for the log level.

Plugins can read the whole command line and the environment through the host API, so a
plugin may document flags of its own.

## Settings

Settings belong to plugins. Change them in the overlay's settings editor, from the console
(`set overlay.<key> <value>`, see [console](console.html)), or in `<plugin>.toml` while the
game is closed. A setting marked "restart required" only takes effect on the next start.
