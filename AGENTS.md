# AGENTS.md: guide for AI agents and contributors

starbooru-linux is the StarBooru app for the Linux desktop, built for GNOME: Rust with [gtk4-rs](https://gtk-rs.org) and [libadwaita](https://gnome.pages.gitlab.gnome.org/libadwaita/). It talks to [starbooru-server](https://github.com/StarBooru/starbooru-server) over its JSON API and nothing else.

There is no code yet, only these docs. Read this file first, then `docs/`.

## 1. Orientation

| Read | Why |
|---|---|
| `docs/architecture.md` | Stack, layout, the main loop and the network |
| `docs/development.md` | Toolchain, running against a local server, tests, UI tests in a nested compositor, Flatpak |
| `docs/design.md` | How to use the archived app as the visual reference |
| `docs/roadmap.md` | What to build, in order |

The API is in starbooru-server's `docs/api.md`. The owner's backlog is its `docs/checklist.md`.

## 2. Related repositories

| Repository | Role |
|---|---|
| [starbooru-server](https://github.com/StarBooru/starbooru-server) | The server, plus `starbooru-core` (API types, search parser, validators) and `starbooru-client` (typed HTTP client) |
| [StarBooru/StarBooru](https://github.com/StarBooru/StarBooru) | The first app, in Dioxus, which ran on Linux in WebKitGTK. Archived and read-only: the visual reference. Never push to it. |
| `starbooru-web`, `starbooru-android`, `starbooru-windows` | The other apps |

## 3. Golden rules

### Never crash

- Non-test code never panics: no `unwrap()`, `expect()`, `panic!`, `unreachable!`, `todo!` or `unimplemented!`. Copy the workspace lints and `clippy.toml` from starbooru-server. The gtk-rs book uses `expect` freely; don't copy that.
- No `unsafe`: set `unsafe_code = "forbid"`. gtk-rs subclassing and templates need none.
- A failed request leaves the window usable and says what went wrong, in a toast or the page itself.

### Project rules

1. **The API is the contract.** Use `starbooru-core` and `starbooru-client` from starbooru-server, pinned to a tag. Don't redefine API types here. A missing endpoint is added in starbooru-server first.
2. **Keep the main loop free.** Network and disk work run on the tokio runtime; results come back to the GTK main context. Nothing blocks a frame.
3. **GNOME first.** Follow the GNOME Human Interface Guidelines and use libadwaita's widgets and styles. StarBooru's accent, layout and motion go on top.
4. **Verify UI changes visually**: light and dark, a wide window and a narrow one. Before committing anything that changes how the app looks, show the owner a preview and commit only once they confirm.
5. **UI tests never touch the owner's desktop session.** The owner runs KDE Plasma. Anything that needs a compositor, `xdotool` or `import -window root` runs inside `tools/nested-kwin.sh`, taken from the reference repository.
   - Never start `kwin_wayland` directly, not even with `--virtual`. A nested KWin on the desktop's D-Bus session broke Alt+Tab and Super+Tab everywhere when it exited.
   - Never stop processes by name (`pkill kwin_wayland`), and never restart or signal the host compositor.
   - Don't change the desktop's shortcuts (`kglobalaccel` calls) as part of a test.
   - If a nested KWin is already running on the desktop's bus, don't stop it: its exit is what does the damage. Tell the owner.
6. **Tests are the gate.** Logic worth a branch lives in plain functions with unit tests. Widgets get tests that run under the nested compositor.
7. **Privacy.** Never log tokens, passwords or file contents. Test images are generated or public domain, never personal photos.
8. **Assets.** Third-party assets must be permissively licensed and get a row in `ATTRIBUTION.md`. Never commit font files.
9. **Text** goes through gettext, as in other GNOME apps: English in the code and the Blueprint files, Italian in `po/it.po`, both taken from the reference's Fluent catalogues.
10. **Time goes through `chrono`.**

## 4. Before you finish a task

```sh
cargo fmt --all
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
meson compile -C _build
```

Then append a terse entry to `log/devlog.md`, which stays local (`/log` goes in `.gitignore` with the first code): what landed, what you verified, what's still open. Update `docs/roadmap.md` when an item changes state.

## 5. Notes

- License: AGPL-3.0-or-later, like the server. Add `LICENSE` with the first code.
- The product name is written StarBooru in user-facing text. Machine names stay lowercase: the app id is `eu.filipnone.starbooru`.
