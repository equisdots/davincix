# Interactive scenes

`davincix` treats an interactive wallpaper ("scene") as one more entry of the
wallpaper collection: it is detected, thumbnailed, applied with an entry
transition and reported as the current wallpaper like any image or video.

A scene is a directory inside `DAVINCIX_WALLPAPER_DIR` containing `scene.js`
(the `xwww` scene engine format). See `equisdots/background` for the collection
contract and the available scenes.

## Detection and scanning

The thumbnail preparation (`thumbs.sh`) walks the wallpaper directory
recursively:

- A directory containing `scene.js` is a scene; its children are not scanned
  as regular files.
- Nested media files are flattened into their thumb name with `__` as the
  separator (`sub/dir/pic.jpg` becomes `sub__dir__pic.jpg`).
- Scene entries are named `scn_<name>.jpg`; videos keep the `000_` prefix.

## Thumbnails

The scene thumbnail is rendered from the cover image inside the scene
directory, in this order:

1. `base.jpg`, `base.jpeg`, `base.png` or `base.webp`;
2. otherwise the first image found in the directory;
3. otherwise a placeholder with the text `scene.js`.

For the scenes in `equisdots/background` this is the original artwork, so the
picker shows what the wallpaper looks like at a glance.

## Applying

`davincix set <path>` auto-detects a directory with `scene.js` and starts the
scene instead of the image pipeline:

```sh
davincix set ~/.config/hypr/wallpapers/astro-palette
davincix set ~/.config/hypr/wallpapers/ascii-astro --transition decrypt
```

- The transition is resolved like for images (`--transition` accepts the
  `xwww` set plus `random`; empty means random). It is used as the entry
  transition of the first frame; later frames are instant and delivered only
  when the scene canvas changes.
- Applying an image or a video stops the running scene first.
- The first frame is cached to `current_wallpaper.png` (lock screen and SDDM).

## State and restore

- The active scene directory is written to
  `$DAVINCIX_STATE_DIR/current_scene` and removed when another wallpaper is
  applied.
- `davincix current` prints `<dir>/scene.js` while a scene runs;
  `davincix current --thumb-name` prints `scn_<name>.jpg`.
- `init.sh` in `equisdots/hyprland` re-applies `current_scene` on session
  start.

## Slideshow and removal

- The slideshow only rotates still images and skips the base image of a scene
  directory.
- `davincix rm <name>` accepts scenes (the whole directory is moved to the
  trash) and flattened nested paths (`sub__dir__pic.jpg`), and removes their
  thumbnails and manifest entries.

## Palette

Scenes started by the kernel use `--palette equisdots`, so they follow the
active bar palette (`settings.json` -> `bar.palette`) within about a second.
