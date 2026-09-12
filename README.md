# r2_clips

LED clips for the dome's WS2812B displays, pulled by the `r2_domeplayer` firmware with
`leds pull`. Nothing is pushed to the droid: it fetches this repo's `manifest.txt` and the
clips it lists, installs the ones that changed into `/data/clips`, and never deletes one.

## What is here

Each file is one clip in the `.r2lc` format, named `<name>.<target>.r2lc`. The target is the
grid it plays on:

| target | surface | grid |
| --- | --- | --- |
| `fld` | the two front logic boards as one | 8 x 10 |
| `rld` | the rear logic | 24 x 4 |
| `psi` | either PSI | 5 x 7 |

`leds run <surface> <name>` plays the clip of that name for the surface's target, so one name
usually has a file per target. The format is specified in r2_domeplayer's
`components/ws2812b/docs/effects_plan.md`.

`manifest.txt` lists every clip with its size and CRC-32. It is written from the files, never
by hand.

## Adding or changing a clip

With r2_domeplayer checked out beside this repo:

```sh
PY=~/.espressif/tools/python/v6.1/venv/bin/python
T=../r2_domeplayer/tools/led_clip.py

$PY $T make pulse -t rld -o pulse.rld.r2lc                      # a generated clip
$PY $T convert rec.dat -t psi -o rec.psi.r2lc                   # a Glediator recording
$PY $T convert top.dat bottom.dat -t fld --fps 30 -o rec.fld.r2lc   # both front boards
$PY $T preview rec.psi.r2lc                                     # watch it through the maps
$PY $T info *.r2lc                                              # check what the board would say
$PY $T manifest .                                               # then rewrite manifest.txt
```

Commit the clips and the manifest together, then tag the set:

```sh
git tag v2 && git push origin main v2
```

## Pulling onto the droid

Pin a tag rather than a branch. A tag makes a pull reproducible, and it avoids
`raw.githubusercontent.com` serving a half-updated set from its cache of a branch:

```
esp32s3> leds pull -n https://raw.githubusercontent.com/daveismith/r2_clips/v1/
esp32s3> leds pull https://raw.githubusercontent.com/daveismith/r2_clips/v1/
```

`-n` reports what would change and downloads nothing. A pull reports every clip as new,
updated, same, not in manifest (kept) or failed. `leds clip rm <name>` is the only way a clip
leaves the board.

For development, serve a folder from the Mac over the USB link instead:

```sh
python3 -m http.server 8000 --bind 10.0.0.1      # in the folder
```

```
esp32s3> leds pull http://10.0.0.1:8000/
```

## Clips here

- `flash`: the run's colour on, then off, 200 ms each.
- `pulse`: the run's colour up and down in 32 steps each way, 40 ms a step.

Both are tinted: they take the colour given to `leds run ... -c rrggbb`, and a blue when none
is given. The firmware carries the same two built in; a pulled copy replaces the built-in one.
