---
title: Rust plugins
---

# Writing a plugin in Rust

A plugin is a DLL that exports three functions. The `uniloader-sdk` crate writes them for
you from a trait implementation, and turns the loader's C interface into safe calls.

## A complete plugin

```toml
# Cargo.toml
[package]
name = "hello-plugin"
version = "0.1.0"
edition = "2021"

[lib]
name = "hello"                 # the DLL is hello.dll
crate-type = ["cdylib"]

[dependencies]
uniloader-sdk = { git = "https://github.com/uniloaders/uniloader" }
log = "0.4"
```

```rust
// src/lib.rs
use std::sync::atomic::{AtomicU32, Ordering};

use uniloader_sdk::{export_plugin, Host, Plugin, PluginError, PluginRef, Setting};

pub struct Hello {
    greetings: AtomicU32,
}

impl Hello {
    fn greet(&self, host: &Host, name: &str) {
        let greeting = host.get("greeting").unwrap_or_default();
        let n = self.greetings.fetch_add(1, Ordering::Relaxed) + 1;
        host.console_print(&format!("{greeting}, {name}! (#{n})"));
    }
}

impl Plugin for Hello {
    const NAME: &'static str = "hello";
    const TITLE: &'static str = "Hello";
    const VERSION: &'static str = env!("CARGO_PKG_VERSION");
    const DESCRIPTION: &'static str = "Greets on a hotkey.";

    fn start(host: Host) -> Result<Self, PluginError> {
        host.setting(
            &Setting::text("greeting", "Greeting", "Hello")
                .with_description("What the greet hotkey and command print."),
        )?;
        host.hotkey("greet", "Print the greeting", "f9")?;
        host.command("greet", "Print the greeting.", "[name]")?;
        host.subscribe("hello.ping")?;
        log::info!("hello {} started", Self::VERSION);
        Ok(Hello { greetings: AtomicU32::new(0) })
    }

    fn on_hotkey(&self, host: &Host, _action: &str) {
        self.greet(host, "player");
    }

    fn on_command(&self, host: &Host, _name: &str, args: &str) -> Result<(), PluginError> {
        self.greet(host, if args.is_empty() { "console" } else { args });
        Ok(())
    }

    fn on_event(&self, host: &Host, _from: PluginRef, _topic: &str, payload: &[u8]) {
        self.greet(host, &String::from_utf8_lossy(payload));
    }
}

export_plugin!(Hello);
```

Build it for Windows (`cargo build --release --target x86_64-pc-windows-msvc`, or
`cargo xwin build` from Linux) and put `hello.dll` into the game's `plugins/` folder. In the
console, `hello.greet Bob` and `set hello.greeting Hi` now work, and `F9` greets.
`log::info!` and friends go to the loader log under the plugin's name.

## The lifecycle

1. The loader reads `uniloader_plugin_describe`: name, version, dependencies. A plugin whose
   dependencies are missing waits; `plugins` lists why.
2. `start` runs once, in dependency order. **Registration only works here**: settings,
   hotkeys, commands, subscriptions and services. Afterwards they fail with "wrong phase".
3. The callbacks (`on_hotkey`, `on_setting_changed`, `on_command`, `on_message`, `on_event`)
   arrive from the loader's thread, or from whichever thread published an event. Keep the
   plugin `Sync` and do real work elsewhere.
4. `plugin disable` pauses callbacks (`on_disable`/`on_enable`); `stop` runs once with a
   reason when the game closes or someone stops the plugin. The loader then removes every
   hook the plugin installed.

Every callback runs under a guard. A panic or an access violation inside one marks the
plugin faulted and disables it; the game and the other plugins keep running.

## What `Host` offers

| Area | Calls |
| --- | --- |
| Where things are | `game_dir`, `exe_path`, `plugin_dir`, `config_dir`, `module_base`, `loader_version` |
| Process | `command_line`, `args`, `env` |
| Settings | `setting(&Setting)`, `get`, `get_as::<T>`, `set`, `settings` (all of them, every plugin) |
| Hotkeys | `hotkey(action, title, default)`, `hotkeys`, `rebind`, `hotkey_press`, `capture_input` |
| Console | `command`, `console_print`, `console_exec`, `console_capture`, `console_read`, `console_names` |
| Plugins | `find_plugin`, `plugin_state`, `modules`, `message` (direct, with a reply) |
| Events | `subscribe`, `publish` |
| Services | `provide` (offer a `#[repr(C)]` table under a name), `service` (look one up) |
| Hooks | `patch` (bytes), `hook_vtable`, `detour`, `remove_hook` |
| Safety | `guarded(owner, fn)`: run code on another plugin's behalf under the fault guard |

Settings come in five kinds: `Setting::bool`, `int` (with a range), `float` (with a range),
`choice` and `text`. Add `with_description`, and mark them `restart_required`, `transient`
(never saved) or `advanced` (hidden in the editor until asked for). The loader validates,
saves and loads them; the plugin only sees `on_setting_changed`.

Dependencies are declared in `DEPENDENCIES`:

```rust
const DEPENDENCIES: &'static [Dependency] = &[
    Dependency::plugin("unreal", ">=0.1"),          // starts first, or this waits
    Dependency::library("openxr_loader.dll"),       // must be findable
    Dependency::file("Content/Paks/pakchunk0.pak"), // must exist
    Dependency::plugin("jump", "").optional(),      // only orders the start
    Dependency::service("overlay").optional(),      // after whoever provides it
];
```

A plugin that registers a service names it in `PROVIDES` (`&["overlay"]`), so plugins
depending on the service start after it. When several plugins provide the same service,
each one only orders the start, because any of them is enough.

Hooks are owned by the loader: it refuses a second hook on the same address, lists them all
under `hooks`, and takes them out when the plugin stops, so a stopped plugin leaves no code
pointing into it.

The loader publishes `loader.plugin_state` whenever a plugin starts, pauses, stops or faults.
A plugin that offers a service subscribes to it to forget what a stopped consumer registered.

## Examples

- [`examples/hello-plugin`](https://github.com/uniloaders/uniloader/tree/main/examples/hello-plugin):
  the plugin above.
- [`examples/kitchen-sink`](https://github.com/uniloaders/uniloader/tree/main/examples/kitchen-sink):
  uses every host function and the overlay service, and runs a self-test
  of 70 checks at start (`kitchen.selftest` runs it again).
- [`plugins/lua-bridge`](https://github.com/uniloaders/uniloader/tree/main/plugins/lua-bridge) and
  [`plugins/overlay`](https://github.com/uniloaders/uniloader/tree/main/plugins/overlay): real plugins.
