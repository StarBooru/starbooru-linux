# Development

## Toolchain

- Rust 1.88 or newer.
- GTK 4 and libadwaita development files, `blueprint-compiler`, Meson and Ninja. On Debian or Ubuntu: `libgtk-4-dev libadwaita-1-dev blueprint-compiler meson`.
- GStreamer plugins for video.
- `flatpak-builder` and the GNOME SDK for packaging.
- A local starbooru-server (its `docs/development.md` › Run the server).

## Build and run

```sh
meson setup _build
meson compile -C _build
meson devenv -C _build ./src/starbooru
```

`STARBOORU_SERVER` prefills the server address at sign-in.

The Flatpak:

```sh
flatpak-builder --user --install --force-clean _flatpak build-aux/eu.filipnone.starbooru.json
flatpak run eu.filipnone.starbooru
```

## Tests

- `cargo test --workspace` runs the logic.
- Widget tests need a display, so they run inside the nested compositor below, never on the desktop.
- Behaviour against a real server is checked by hand and written in the dev log.

## Nested compositor

The owner's desktop is KDE Plasma. A compositor started the wrong way joins its D-Bus session and breaks its shortcuts when it exits. Take `tools/nested-kwin.sh` from the reference repository and run every UI test, screenshot and `xdotool` script through it:

```sh
tools/nested-kwin.sh --x-display :77=1280x800 -- ./my-test.sh
```

Inside, start the app with `GDK_BACKEND=x11` and screenshot with `import -window root`. A test script refuses to run outside the harness:

```sh
[ "${STARBOORU_NESTED_KWIN:-}" = 1 ] || { echo "run this inside tools/nested-kwin.sh" >&2; exit 2; }
```

The harness's rules, and what to check if you change it, are in the reference's `docs/development.md` › Nested KWin.

## Visual checks

Screenshot sign-in, the gallery, a post, upload and settings, in light and dark, at 1280 px and at 400 px wide, inside the nested compositor. Look at them next to the reference's (`docs/design.md`), then show the owner before committing.
