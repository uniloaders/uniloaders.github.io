---
title: .asi mods and chain-loading
---

# `.asi` mods and chain-loading

## `.asi` mods

uniloader loads `plugins/*.asi` the way the Ultimate ASI Loader does: alphabetically, as
plain DLLs that start themselves from `DllMain`, calling `InitializeASI` once if they export
it. Move your `.asi` files into `plugins/` and remove the ASI loader DLL, or both loaders load
them twice. Set `asi = false` in [`loader.toml`](configuration.html#loadertoml) to leave them
to another loader.

`.asi` mods do not get the host API: no settings, hotkeys or console of their own. They are
listed in `plugins` with their file version, so you can see everything that went into the
game.

## Chain-loading what uniloader displaced

Some mods are themselves proxy DLLs. UE4SS ships as `dwmapi.dll`, ReShade as `dxgi.dll`, and
only one DLL can have each name. Either install uniloader under a different name (see the
[install table](../#install)), or rename the other mod and have uniloader load it:

```toml
# uniloader/config/loader.toml
load = ["ue4ss/UE4SS.dll"]
```

Paths are relative to the game executable. uniloader loads them after its own plugins
start. A DLL that is a proxy for the same system library as uniloader is fine to chain-load
as long as it was renamed: it forwards to the real library on its own.

### UE4SS

UE4SS's own `dwmapi.dll` is only a small proxy that loads `ue4ss/UE4SS.dll`. With uniloader
installed as `dwmapi.dll`, delete or rename UE4SS's proxy and add the line above; UE4SS and
its Lua mods then run as before, and uniloader plugins run beside them.
