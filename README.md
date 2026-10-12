# Ibara Kuro – Show Time patch

Builds the **Show Time** ROMs from your own dump of *Ibara Kuro Black Label*.
This repository contains **no ROM data** – not even patch bytes. The script
makes every change from data that is already in your dump, then checks the
result against known checksums.

## What it changes

| Black Label song   | Plays instead | Title in game |
|--------------------|---------------|---------------|
| Bonds of Steel     | Show Time     | Show Time     |
| Air Pollution      | Sky High      | Sky High      |
| The Family's Work  | Robot No28    | Robot No28    |

Boot screen: `2006/xx/xx. MASTER VER.` → `2026/10/12 SHOW TIME`.
Each slot keeps Black Label's own volume.

## Usage

Requires Python 3.8+ (standard library only).

```sh
python showtime_patch.py path/to/ibarablk.zip                 # MAME romset or any zip
python showtime_patch.py path/to/ibarablk_folder              # or a folder
python showtime_patch.py ibarablk.zip --zip ibarablk_new.zip  # also write a complete romset
```

Patched `ibarablk_u4` and `u23` go to `./output` (change with `-o`). They keep
the original names, so they drop straight into a romset. With `--zip`, the
other romset files (`ibarablk_u2`, `u24`) are copied in unchanged. The script
won't write over your originals.

Repacking the game program takes about a minute.

### Options

| Option | What it does |
|---|---|
| `-o DIR` | output folder (default `./output`) |
| `--zip FILE` | also write a complete romset zip |
| `--ips DIR` | also write standard IPS patches for Flips / Lunar IPS (made from your dump; don't share them, the `ibarablk_u4` one contains packed game code) |
| `--list` | show the songs and slots |
| `--swap 'OLD=NEW'` | your own swap, e.g. `--swap 'Rose Garden=Endress Train'` (repeatable; replaces the default swaps) |
| `--boot-text TEXT` | your own boot text (up to 26 characters) |
| `--keep-boot-text` | leave the boot text alone |
| `--force` | patch even if an input checksum doesn't match |

The default build is checked against the output checksums below. Custom
builds can't be compared against anything, so they're labelled "custom build".
They're still checked by unpacking the repacked program and comparing it.

## Required input files

| File          | Size      | CRC32      | SHA-1                                      |
|---------------|-----------|------------|--------------------------------------------|
| `ibarablk_u4` | 2,097,152 | `ee1f1f77` | `ac276f3955aa4dde2544af4912819a7ae6bcf8dd` |
| `u23`         | 4,194,304 | `a436bb22` | `0556e771cc02638bf8814315ba671c2d442594f1` |

Files are found by name, or by checksum if they were renamed.

## Output files (default build)

| File          | Size      | CRC32      | SHA-1                                      |
|---------------|-----------|------------|--------------------------------------------|
| `ibarablk_u4` | 2,097,152 | `07243591` | `82403646d72070ea3207d464a25fa8c2ff90f943` |
| `u23`         | 4,194,304 | `69403f24` | `51f6536b0845fc26f266761ae344992fb38ffee8` |

## How it works

Changing a song takes three edits:

1. **`u23` – sound-chip sequence table.** The game starts a music slot by
   running a short script on the YMZ770 sound chip. The script picks the audio,
   waits a fixed time, then switches the channel off. The slot is pointed at the
   new song's script, so it plays for the right length.
2. **`ibarablk_u4` – play-time tables.** The sound driver moves from the intro
   to the loop, and restarts the loop, on a timer for each slot (in milliseconds,
   two copies). The new song's times are copied in.
3. **`ibarablk_u4` – song title.** This is padded to fit the existing space and
   shortened if it's too long.

`ibarablk_u4` is stored packed (LZSS). The boot loader unpacks it from ROM
`0x51000` into RAM `0x0C051000`. The script unpacks it, makes the edits, and
repacks it with a deterministic optimal-parse compressor, so the default build
is byte-identical every time. The boot-text change also disables the two calls
that print the board clock's month and day over the text.
