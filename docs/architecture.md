# Architecture

```text
 GTK main loop (widgets, signals) ◀── async-channel ── tokio runtime ── HTTPS /api/v1 ──▶ starbooru-server
```

## Crates

| Crate | From | Role |
|---|---|---|
| `gtk4` 0.11, `libadwaita` 0.9 | crates.io (gtk-rs) | Widgets, styles, animations |
| `starbooru-core`, `starbooru-client` | starbooru-server, by tag | API types and the typed client |
| `tokio` | crates.io | The runtime behind `starbooru-client` |
| `async-channel` | crates.io | Results back to the main loop |
| `oo7` | crates.io | The token in the Secret Service, or through the portal inside Flatpak |
| `gettext-rs` | crates.io | UI text |

## The main loop and the network

GTK runs on the main thread. One multi-threaded tokio runtime starts beside it. A widget that needs data spawns the request on the runtime and awaits the answer with `glib::spawn_future_local`, so the main loop stays free while it waits. Widgets are only touched from the main thread.

## Layout

```text
src/
  main.rs         the AdwApplication, the runtime, resources
  application.rs  session, upload queue, preferences (GSettings)
  window.rs       AdwApplicationWindow with the navigation
  pages/          one widget per page, each a subclass with a Blueprint template
  widgets/        tiles, tag chips, the tag box, the viewer stage
  logic/          pure Rust with unit tests: tile layout, zoom, file names
data/
  ui/*.blp        Blueprint files, compiled to .ui at build time
  eu.filipnone.starbooru.gschema.xml, .desktop, .metainfo.xml, icons
po/               gettext catalogues
build-aux/        the Flatpak manifest
meson.build       builds the resources and runs Cargo
```

## Widgets to start from

| Need | Widget |
|---|---|
| Window and pages | `AdwApplicationWindow`, `AdwNavigationView`, `AdwHeaderBar` |
| Navigation, at most five places | `AdwViewStack` with `AdwViewSwitcher`, and `AdwViewSwitcherBar` when narrow |
| Gallery | `GtkGridView` over a `GListModel`, which only builds the rows in view |
| Viewer | `GtkPicture` in a `GtkScrolledWindow` for zoom and pan, `GtkVideo` (GStreamer) for video |
| Notices | `AdwToastOverlay` |
| Settings | `AdwPreferencesDialog` |
| Motion | `AdwSpringAnimation`, `AdwTimedAnimation` |
| Narrow windows | `AdwBreakpoint` |

## Media

Thumbnails and pictures come from the signed `/media` URLs into `GdkTexture`s. A disk cache under `$XDG_CACHE_HOME/starbooru`, keyed by each file's SHA-256, keeps them when the URLs change every 6 hours, as the reference's `cache.rs` did, with the same size limit setting.

## Packaging

Flatpak on the GNOME runtime is the package. It runs on any distribution, which the reference's `.deb` didn't.
