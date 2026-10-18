---
title: Console and hotkeys
---

# Console and hotkeys

The console is how you talk to the loader and the plugins while the game runs. Reach it
through the [overlay](overlay.html) (the key under Escape, or `ctrl+shift+c`), or start the
game with `--loader-console` for a separate window. Plugins can run console lines too, and
the Lua bridge exposes it as `uniloader.exec`.

## Built-in commands

| Command | Does |
| --- | --- |
| `help [name]` | List everything, or explain one setting or command. |
| `list [prefix]` | List settings and commands, optionally filtered. |
| `plugins` | Loaded plugins and their state, waiting plugins, `.asi` mods and chain-loaded DLLs, services. |
| `plugin deps [name]` | What plugins declared they need. |
| `plugin load <name>` | Load and start `plugins/<name>.dll`, or a plugin waiting for a dependency. |
| `plugin stop <name>` | Stop a plugin. Its hooks are removed; the DLL stays in the process. |
| `plugin enable <name>`, `plugin disable <name>` | Resume or pause a plugin's callbacks. |
| `hotkeys` | Every hotkey action and what it is bound to. |
| `hooks` | Every patch, vtable hook and detour plugins installed. |
| `get <plugin.key>` | Print a setting. |
| `set <plugin.key> <value>` | Change a setting; it is checked, saved and the plugin told. |
| `<plugin.key> [value]` | Shorthand for `get` and `set`. |
| `<plugin.command> [args]` | Run a plugin's command. |
| `quit` | Stop every plugin, then end the game process. |

Tab completes commands, settings, hotkey actions and plugin names; up and down recall
earlier lines.

## Hotkeys

Plugins register named actions with a default binding. You can rebind any of them in the
overlay's settings editor, or in `uniloader/config/hotkeys.toml`:

```toml
"overlay.console" = "sc29, ctrl+shift+c"
"hello.greet" = "f9"
"kitchen.poke" = "none"
```

Hotkeys only fire while the game window has focus. While the overlay has the keyboard (a
window is open), only the overlay's own hotkeys fire, so typing does not trigger other
plugins.

### Key grammar

A binding is one or more chords separated by commas; any of them fires the action. A chord
is optional modifiers and one key:

```text
f11            ctrl+shift+r        alt+enter        sc29        none
```

- Case does not matter. Modifiers: `ctrl` (`control`), `alt`, `shift`.
- Keys: `a`–`z`, `0`–`9`, `f1`–`f24`, `numpad0`–`numpad9`, and by name: `backspace`, `tab`,
  `enter` (`return`), `pause`, `capslock`, `escape` (`esc`), `space`, `pageup`, `pagedown`,
  `end`, `home`, `left`, `up`, `right`, `down`, `printscreen`, `insert`, `delete`, `numlock`,
  `scrolllock`, `multiply`, `add`, `subtract`, `decimal`, `divide`, `semicolon`, `plus`,
  `comma`, `minus`, `period`, `slash`, `grave` (`tilde`), `lbracket`, `backslash` (`caret`),
  `rbracket`, `quote`, `oem102`.
- `0x<hex>` is a Windows virtual key code, for keys without a name.
- `sc<hex>` names a key by its position instead of its label, for example `sc29` for the key
  under Escape on every layout. Position bindings need the overlay, which reports key presses
  to the loader.
- `none` or an empty string unbinds.
