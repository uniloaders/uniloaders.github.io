---
title: Docs
---

# Documentation

## Playing with it

- [Install](../) — the main page: which name to give the DLL, where it goes, Proton.
- [Configuration](configuration.html) — `loader.toml`, command line flags, the files the
  loader writes.
- [Console and hotkeys](console.html) — built-in commands, settings from the console, the
  key grammar and rebinding.
- [Overlay](overlay.html) — the in-game console and settings editor.
- [`.asi` mods and chain-loading](asi.html) — Ultimate ASI Loader mods, UE4SS and other DLLs
  uniloader displaces.
- [Games](games.html) — game-specific plugins.

## Writing plugins

- [Rust plugins](plugins.html) — the SDK: settings, hotkeys, commands, events, messages,
  services, hooks.
- [The C ABI](abi.html) — for plugins in C, C++, Zig or anything else that exports C
  functions.
- [Lua scripts](lua.html) — the `uniloader` table the Lua bridge gives scripts.
- [Overlay service](overlay.html#drawing-from-a-plugin) — panels, toasts and dialogs from a
  plugin.

## How it works

uniloader is a DLL that pretends to be a system library. Its export table is generated
from lists of the real library's exports (`crates/uniloader/exports/`). Each export is a
one-instruction jump through a table that `DllMain` fills from the real DLL in `System32`, so
the game cannot tell the difference. Exports a list does not cover answer "not implemented".

Once loaded, it starts a thread rather than doing work under the loader lock. The thread
reads the config, loads every `plugins/*.dll` that exports the uniloader entry points, and
orders them by their declared dependencies. It then calls each plugin's `start` with a table
of host functions, and finally loads `.asi` files and the chain-load list. Every call into a
plugin goes through a guard that catches panics and hardware faults: a faulting plugin is
disabled, and the game and the other plugins carry on.

Everything game-specific lives in plugins. Adapter plugins teach the others about one
engine (Enfusion for DayZ, Unreal Engine for Train Sim World) and offer that knowledge as
services.
