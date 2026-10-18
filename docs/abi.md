---
title: The C ABI
---

# The C ABI

Plugins in any language talk to the loader through plain C structs and function pointers.
The Rust definitions in
[`crates/uniloader-api`](https://github.com/uniloaders/uniloader/tree/main/crates/uniloader-api/src)
are the reference; mirror them in your language. The current version is **1**.

## Conventions

- **Strings** are `Str { const uint8_t *ptr; size_t len; }`: UTF-8, not null-terminated,
  valid only during the call that hands them over. Nobody frees them; copy what you keep.
- **Growing structs** start with `size_t struct_size`. Fields are only ever appended, so
  both sides can tell which fields the other knows. Set it to the size you compiled with.
- **Every call returns a `Status`** (`uint32_t`): `OK = 0`, `ERROR`, `UNSUPPORTED`,
  `NOT_FOUND`, `INVALID_ARGUMENT`, `ALREADY_EXISTS`, `WRONG_PHASE`, `BUFFER_TOO_SMALL`.
  Nothing may unwind or throw across the boundary in either direction.
- **Enums** are `uint32_t` with named constants, so unknown values are not undefined
  behaviour.
- Everything is the C calling convention of x86-64 Windows.

## The three exports

```c
const PluginInfo *uniloader_plugin_describe(void);
Status uniloader_plugin_start(const HostApi *host, PluginHandle me, PluginCallbacks *callbacks);
void   uniloader_plugin_stop(void *ctx, StopReason reason);
```

`describe` returns a struct that stays valid while the DLL is loaded:

```c
typedef struct {
    size_t struct_size;
    uint32_t api_version;          /* must be 1 */
    Str name;                      /* [a-z0-9_-]+, the settings namespace */
    Str title, version, authors, description;
    const Dependency *dependencies;
    size_t dependency_count;
    const Str *provides;           /* service names this plugin registers; appended, */
    size_t provides_count;         /* so read them only if struct_size covers them   */
} PluginInfo;
```

A `Dependency` names a plugin (`kind` 0, with a version requirement), a library (1), a file
(2) or a service (3). A service dependency starts the plugin after every plugin whose
`provides` lists that service; with no provider at all a mandatory one waits like a missing
plugin.

`start` may keep `host` for the plugin's lifetime. It fills in the callbacks it wants and
returns `OK`; any other status means the plugin failed to start.

```c
typedef struct {
    size_t struct_size;
    void *ctx;                                         /* handed back to every callback */
    void   (*on_hotkey)(void *ctx, Str action);
    void   (*on_setting_changed)(void *ctx, Str key, Str value);
    Status (*on_command)(void *ctx, Str name, Str args);
    Status (*on_message)(void *ctx, PluginHandle from, Str topic, Bytes payload,
                         ReplyFn reply, void *reply_ctx);
    void   (*on_event)(void *ctx, PluginHandle from, Str topic, Bytes payload);
    void   (*on_enable)(void *ctx);
    void   (*on_disable)(void *ctx);
} PluginCallbacks;
```

Every pointer may be null. `stop` receives `ctx` once at the end.

## The host table

`HostApi` starts with `struct_size`, `api_version` and an opaque `host` pointer that is the
first argument of every function in it. In order:

| Group | Functions |
| --- | --- |
| Paths and process | `loader_version`, `game_dir`, `exe_path`, `plugin_dir`, `config_dir`, `module_base`, `command_line`, `args`/`arg_count`, `env`/`env_count` (fields, not calls) |
| Logging | `log` |
| Settings | `setting_register`, `setting_get`, `setting_set`, `setting_enumerate` |
| Hotkeys | `hotkey_register`, `hotkey_enumerate`, `hotkey_rebind`, `hotkey_press`, `input_capture` |
| Console | `command_register`, `console_print`, `console_exec`, `console_capture`, `console_read`, `console_names` |
| Plugins | `plugin_find`, `plugin_state`, `plugin_message`, `plugin_enumerate` |
| Events | `event_subscribe`, `event_publish` |
| Hooks | `hook_patch`, `hook_vtable`, `hook_detour`, `hook_remove` |
| Services | `service_register`, `service_get` |
| Guard | `guarded_call` |

Functions that return lists or text call a callback you pass once per item, with a context
pointer, instead of filling buffers. The doc comment on each field in
[`host.rs`](https://github.com/uniloaders/uniloader/blob/main/crates/uniloader-api/src/host.rs)
gives its exact signature and rules.

## Services

A service is a `#[repr(C)]` table of function pointers that one plugin registers under a
name and a version, and any other plugin looks up. It is how plugins offer each other more
than messages: the overlay's drawing API, an engine adapter's object lookups. The provider
keeps the table alive; consumers check the version before using fields a later version
added, and should subscribe to `loader.plugin_state` (payload: handle and state as
little-endian `u32`, then the name) to notice the provider going away.
