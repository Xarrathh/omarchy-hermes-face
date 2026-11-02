# Hermes Face — Omarchy bar widget

A bar slot for the [Hermes face](https://github.com/Xarrathh/hermes-face): one glyph that tells
you what the holographic agent head is doing, with the controls you actually reach for.

```
○  idle        ◐  thinking        ◉  speaking        ✕  failed or not running
```

- **Left click** — go to workspace 1 and bring the face window forward, starting it if it
  is not running.
- **Right click** — stop the live turn and its speech.
- **Middle click** — restart the face bridge and wait until it answers again.

The tooltip carries the current state, the mood while it speaks, an error if the bridge
reported one, and whether theme sync is on.

## Requirements

This widget is a remote control, not the face itself. It needs the Hermes face bridge
answering on `http://127.0.0.1:9120/state` — install that first:

```bash
git clone https://github.com/Xarrathh/hermes-face.git
cd hermes-face && ./install.sh --no-autostart
```

Without the bridge the glyph shows `✕` and clicking it tries to start the window anyway.

## Install

```bash
omarchy plugin add https://github.com/Xarrathh/omarchy-hermes-face.git --enable
omarchy restart shell          # a new bar widget is not instantiated until the shell restarts
```

Or by hand: copy this folder to `~/.config/omarchy/plugins/pierregoosen.hermes-face/`, add
`{"id": "pierregoosen.hermes-face"}` to a `bar.layout` section in `~/.config/omarchy/shell.json`,
then `omarchy restart shell`.

## Optional extras

`bin/hermes-face-stop` is the global stop key — it interrupts the live turn and silences the
voice, and refuses when nothing is running:

```bash
install -Dm755 bin/hermes-face-stop ~/.local/bin/hermes-face-stop
# then, in ~/.config/hypr/bindings.lua:
#   o.bind("SUPER + SHIFT + H", "Stop Hermes face", "hermes-face-stop")
```

`hooks/theme-set.d/hermes-face-theme.hook` pulls the face's colours toward the active theme —
hue and saturation only, never the geometry — and switches itself off the moment you pick
colours by hand. Install it with:

```bash
mkdir -p ~/.config/omarchy/hooks/theme-set.d
cp hooks/theme-set.d/hermes-face-theme.hook ~/.config/omarchy/hooks/theme-set.d/
install -Dm755 bin/hermes-face-theme-sync ~/.local/bin/hermes-face-theme-sync
hermes-face-theme-sync on      # or: off
```

## Notes

- The glyph is a single character on purpose: Omarchy's `BarIconButton` renders one optical
  glyph in a fixed slot, so anything longer is clipped.
- `bin/*.sh` resolve Hyprland's socket name themselves. A bar-launched process can arrive
  without `HYPRLAND_INSTANCE_SIGNATURE`, and `hyprctl` then fails silently with exit 0.
- On this Hyprland, `hyprctl dispatch workspace 1` is a parse error and the comma form is
  ignored; the only form that switches workspace is
  `hyprctl dispatch 'hl.dsp.focus({ workspace = "1" })'`.

## Licence

MIT — see `LICENSE`.
