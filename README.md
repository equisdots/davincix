# davincix

Wallpaper engine kernel for the [equisdots](https://github.com/equisdots)
desktop. Headless CLI that applies, thumbnails, searches and rotates
wallpapers — no shell dependency; the Quickshell picker UI lives in the
dottes shell and talks to this CLI.

Version: **0.1.0** · License: MIT

## Requirements

| Role | Tool |
|---|---|
| Apply still images | `xwww` (xwww-daemon) |
| Apply videos | `mpvpaper` |
| Thumbnails / webp / color helpers | ImageMagick (`magick`) |
| Video posters | `ffmpeg` + `ffprobe` |
| Search downloads | `curl` |
| Search scraper | `python3` (stdlib only) |
| Trash on delete | `gio` (optional; falls back to `rm`) |
| Notifications | `notify-send` (optional) |

## Install

```bash
git clone https://github.com/equisdots/davincix ~/davincix
```

The UI resolves the CLI in this order:

1. `$DAVINCIX_CLI` — explicit path to `davincix.sh`.
2. Sibling `../kernel/davincix.sh` next to the UI.

So on the dottes shell the recommended wiring is a symlink:

```bash
ln -s ~/davincix ~/.config/hypr/scripts/quickshell/davincix/kernel
```

Any other frontend only needs the CLI path.

## CLI

```bash
davincix.sh set <file|url> [--video] [--monitors all|A,B] \
             [--transition name] [--thumb <poster>] [--notify] [--dry-run]
davincix.sh fetch --name <n> --map <f> --dest <f> \
             [--thumb-in <f>] [--thumb-out <f>] [--monitors ...] [--transition ...]
davincix.sh current [--thumb-name]
davincix.sh thumbs
davincix.sh search <query> [--source ddg|wallhaven]
davincix.sh search --continue <query>   # next page (keeps the cache)
davincix.sh search --clear              # stop + drop the cache
davincix.sh stop
davincix.sh rm <file>
davincix.sh import <paths…>
davincix.sh slideshow start|stop|status [interval]
davincix.sh paths
davincix.sh --version
```

## Environment overrides

| Variable | Default | Use |
|---|---|---|
| `DAVINCIX_WALLPAPER_DIR` | `$WALLPAPER_DIR` or `~/.config/hypr/wallpapers` | source directory |
| `DAVINCIX_CACHE_DIR` | `~/.cache/quickshell/wallpaper_picker` | thumbs, current, search |
| `DAVINCIX_STATE_DIR` | `~/.local/state/quickshell/wallpaper_picker` | persistent flags |
| `DAVINCIX_RUN_DIR` | `$XDG_RUNTIME_DIR/quickshell/wallpaper_picker` | control, locks, PIDs |
| `DAVINCIX_LOG_DIR` | `$XDG_RUNTIME_DIR/quickshell/logs` | logs |
| `DAVINCIX_CLI` | — | consumed by the UI (CLI location) |

## Files that are contracts

- `current_wallpaper.png` — current wallpaper cache (lock screens, theme tools).
- `ddg_search_control` — `run|pause|stop`; written by the UI.
- `search_cursors/<source>` — per-provider pagination cursor (`search --continue`).
- `search_source` — active search provider (persisted per fresh search).
- `thumbs/.manifest` + `thumbs/.source_dir` — thumbnail cache index.
- `search_map.txt` — `name|url` for search results.
- `slideshow.pid` + `slideshow_enabled` — slideshow daemon state.

## Slideshow

`slideshow start [interval]` runs a detached daemon that rotates still images
(`--transition random`) and never repeats the previous one. State lives in the
run/state dirs; resuming after a reboot needs an autostart entry in the host
(e.g. `davincix.sh slideshow start` when the enabled flag exists).

## Testing

```bash
bash davincix.sh --version
bash davincix.sh paths
bash davincix.sh current
bash davincix.sh set ~/.config/hypr/wallpapers/7.png --dry-run   # no aplica
bash davincix.sh thumbs
```

## Search notes

Search providers live in `providers/` (one script per source) and are plain
"thumb|full" emitters consumed by `search.sh`:

- `ddg.py` — DuckDuckGo (stdlib only; JSON endpoint + VQD token dance; DDG can
  change it at any time).
- `wallhaven.py` — Wallhaven public API (no key needed for SFW), native
  resolution filter and page-number pagination.

Results are filtered to >= 1920x1080 and validated (`content-type` + mime)
before being kept.

## License

MIT — see `LICENSE`.
