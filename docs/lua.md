---
title: Lua scripts
---

# Lua scripts

With `lua_bridge.dll` in `plugins/`, every `plugins/lua/*.lua` runs once at start, in file
name order. Scripts register what they need at their top level, just as a native plugin does
in `start`. Everything they register lives under the `lua.` prefix: a hotkey `greet` becomes
`lua.greet`, a setting `speed` becomes `lua.speed`.

A script that fails to load is logged and skipped, and an error inside a callback is logged
without stopping the bridge, so one broken script cannot take the others down.

## Example

```lua
-- plugins/lua/hello.lua
uniloader.setting("greeting", "Greeting", "Hello")

uniloader.hotkey("greet", "Print the greeting", "f8", function()
  print(uniloader.get("greeting") .. " from Lua")
end)

uniloader.command("echo", "Print the arguments back.", "<text>", function(args)
  print(args)
end)

uniloader.subscribe("hello.ping", function(payload, topic)
  uniloader.log("got " .. topic .. ": " .. payload)
end)
```

## The `uniloader` table

| Function | Does |
| --- | --- |
| `uniloader.log(msg)`, `.warn(msg)`, `.error(msg)` | Write to the loader log. |
| `print(...)` | Print to the console, tab separated like stock Lua. |
| `uniloader.exec(line)` | Run a console line; returns `ok, {lines}`. |
| `uniloader.get(name)`, `.set(name, value)` | Read or write a setting as text: a key of the scripts' own, or any plugin's as `<plugin>.<key>`. |
| `uniloader.setting(key, title, default [, fn])` | Register a setting. Its type follows `default` (boolean, integer, number or string); `fn(value)` runs when it changes. |
| `uniloader.hotkey(action, title, binding, fn)` | Register a hotkey; `fn()` runs on press. |
| `uniloader.command(name, help, usage, fn)` | Register `lua.<name>`; `fn(args)` runs it. |
| `uniloader.subscribe(topic, fn)` | `fn(payload, topic)` for every broadcast on `topic`. |
| `uniloader.publish(topic, payload)` | Broadcast to every subscriber, native plugins included. |
| `uniloader.game_dir`, `uniloader.version` | The game directory and the loader version. |

Registration only works while the scripts first run; afterwards those calls raise a "wrong
phase" error. Errors from the host are Lua errors, so `pcall` catches them.

Publishing and subscribing is how scripts and native plugins talk: a plugin can publish
game state on a topic, and a script reacts to it.
