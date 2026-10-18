---
title: Overlay
---

# Overlay

`overlay.dll` draws a console, a settings editor and other plugins' windows inside the game.
It is a plugin, not part of the loader: drawing inside a game means hooking how it presents
frames, which differs for every graphics API and is risky in each. A game that needs no
overlay does not carry one, and a different overlay can replace this one, because other
plugins only talk to the `overlay` service, never to the drawing library behind it.

## Using it

Drop `overlay.dll` into `plugins/`. In game:

| Hotkey | Opens |
| --- | --- |
| key under Escape, or `ctrl+shift+c` | the console |
| `f11` | the settings editor: every plugin's settings and hotkeys |
| `ctrl+shift+m` | gives the mouse back to the game with the windows left open, or takes it again |

The console command `overlay.toggle console|settings|mouse` does the same. All three are
ordinary hotkeys and can be rebound (see [console](console.html#hotkeys)).

While a window is open the overlay has the keyboard and mouse, so typing in the console does
not walk the player forward, and other plugins' hotkeys stay quiet. Where you left the
windows, their sizes and opacity are saved to `uniloader/config/overlay-windows.toml`.

## Which games

| Graphics API | State |
| --- | --- |
| Direct3D 11, native or through DXVK | drawn |
| Direct3D 10 | hooked, not drawn |
| Direct3D 12, native DXGI | hooked, not drawn yet |
| Direct3D 12 under vkd3d-proton | not hooked |
| Vulkan | not yet |
| OpenGL | not yet |

On Direct3D the hook does not depend on the name uniloader was installed under, or on
whether the game created its swapchain before the overlay loaded. In a game the overlay
cannot draw into, it logs why and stays out of the way; nothing else changes.

**VR:** VR games hand their eye images to OpenXR or OpenVR separately from the desktop
window, so the overlay shows on the desktop mirror but not in the headset.

## Drawing from a plugin

Panels, toasts and dialogs from your own plugin go through the overlay service. In Rust,
depend on the client crate and declare the overlay as an optional dependency, so your
plugin still runs without it:

```toml
[dependencies]
uniloader-overlay = { git = "https://github.com/uniloaders/uniloader" }
```

```rust
use uniloader_overlay::{Overlay, UiLevel};

// Any plugin providing the service will do: this one, or a game's native UI.
const DEPENDENCIES: &'static [Dependency] =
    &[Dependency::service(uniloader_overlay::SERVICE).optional()];

// in start:
if let Some(overlay) = Overlay::connect(&host) {
    overlay.panel("status", "My plugin", false, |ui| {
        ui.heading("Status");
        ui.setting("speed"); // the editor control for one of your settings
        if ui.button("Reset") { /* ... */ }
    })?;
    overlay.toast(UiLevel::SUCCESS, "My plugin", "Ready.")?;
}
```

| Call | Does |
| --- | --- |
| `panel(id, title, open, fn)` | A window whose contents `fn(&mut Ui)` builds every frame while it is open. |
| `set_open`, `toggle`, `is_open`, `remove_panel` | Control a panel. |
| `toast(level, title, text)` | A small card in the corner that fades out by itself. |
| `notice(level, title, text, seconds)` | Large text in the middle of the screen, for the few things that must be seen. |
| `dialog(kind, level, title, text, default, fn)` | A message, a confirm/cancel question or a line to type into; `fn` gets the answer. `close(id)` withdraws it. |
| `on_present(fn)` | Called once per presented frame with the API, the swapchain and its size. |
| `is_capturing` | Whether the overlay has the keyboard and mouse right now. |

Levels are `INFO`, `SUCCESS`, `WARNING` and `ERROR`. `Ui` has `label`, `heading`,
`separator`, `space`, `button`, `checkbox`, `slider`, `slider_int`, `toggles` (a row of
checkboxes over the bits of an integer) and `setting(key)`, which draws the same control the
settings editor uses for one of your settings and writes changes through the loader.

The closures run on the game's render thread. They run under the loader's fault guard: a
plugin that panics or crashes in one is disabled, and the overlay drops everything that
plugin registered. The kitchen sink example
([`examples/kitchen-sink/src/overlay.rs`](https://github.com/uniloaders/uniloader/blob/main/examples/kitchen-sink/src/overlay.rs))
uses every part of the service.

From another language, look up the service `overlay`, version 1, and call its
[`OverlayApi`](https://github.com/uniloaders/uniloader/blob/main/crates/uniloader-overlay/src/abi.rs)
table directly (see [the C ABI](abi.html#services)).
