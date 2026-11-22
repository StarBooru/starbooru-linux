# Design

The archived [StarBooru](https://github.com/StarBooru/StarBooru) app is the visual reference: how each page looks, what it holds and how it moves. It is read-only. Clone it to look, to run it, or to take files from it.

| In the reference | For |
|---|---|
| `docs/showcase/` | Screenshots of the main pages, light and dark, desktop and phone |
| `apps/app/assets/main.css` | The colour roles from one seed hue, radii, easing and durations |
| `apps/app/src/pages/` | What each page holds and does |
| `apps/app/src/motion.rs`, `gesture.rs` | How things move: a post grows out of its tile, touchpad swipes follow the fingers 1:1 |
| `apps/app/src/icons.rs` | The icons' path data, to turn into symbolic SVG icons |
| `apps/app/locales/` | The UI text in English and Italian |
| `docs/branding.md`, `docs/brand/` | The logo, its two cuts and its colours |
| `docs/roadmap.md` › Desktop app | What the desktop build did when the apps split |

## What carries over

- The pages and what is on them, and the order of the navigation.
- The accent: GNOME's accent colour by default, or the user's seed hue, set through libadwaita's accent variables.
- Light, dark and system themes, through `AdwStyleManager`.
- The motion: a post grows out of its tile and shrinks back, previous and next slide, springs on the way in. Off when the system's animations are off.
- The keyboard shortcuts of the viewer (`←`, `→`, `F`, `E`, `+`, `-`, `0`, `Esc`).
- The icons and their attribution rows (MingCute and the others in the reference's `ATTRIBUTION.md`).

## What changes

The window looks like a GNOME app: libadwaita's header bar, boxed lists, toasts and preferences, its typography and its rounded corners. The reference's floating glass capsules don't exist in libadwaita; floating controls over media use the `osd` style instead. Anything that looks different from the reference on purpose goes past the owner first.
