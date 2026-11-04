# Sepia app icon

A **fox in profile**, its tail sweeping round the bottom of the tile and a crescent arc behind its
head, with a small cluster of pixel squares on the left. Flat shapes only: no gradients, strokes,
text or embedded images.

Sepia is the internal fork of PhotoCraft (see
[`docs/podlovers/2026-11-04-photocraft-desktop-integration-design.md`](../../docs/podlovers/2026-11-04-photocraft-desktop-integration-design.md)
§12.1). This icon replaces upstream PhotoCraft's kitsune icon; the file names below are unchanged
so the code and packaging that consume them need no edits.

## Palette

Exactly three colours:

| Colour | Hex | Used for |
|---|---|---|
| Ink | `#0d0d0f` | the full-bleed field |
| Amber | `#f2b35d` | the crescent arc, tail accent and one pixel square |
| Cream | `#f7f3ed` | the fox and the other pixel squares |

## Geometry

A 512-unit tile (`viewBox="0 0 512 512"`), rounded square with `rx=112`, no border. The macOS
renders pad it onto Apple's 824/1024 icon grid; the Windows and Linux renders use the whole tile.

## Provenance

Supplied as `sepia_fox_512.svg` on 2026-11-04. Author and licence: see `LICENSE.txt`.

## Files

- `photocraft.svg`: canonical master.
- `photocraft-small.svg`: the hicolor scalable icon. The master is already light (12.5 KB), so this
  is an identical copy.
- `photocraft-1024.png`: 1024 px render on the macOS grid.
- `photocraft.icns`: macOS bundle icon (`CFBundleIconFile`). **Not yet regenerated**: it still holds
  the upstream kitsune, because `iconutil` only runs on macOS. Run `packaging/icons.sh` on a Mac
  before shipping a macOS build.
- `photocraft.ico`: Windows icon, 16–256 px, embedded in the `.exe` by `apps/photocraft/build.rs`.
- `hicolor/<size>/apps/ai.storyteller.photocraft.png` (16–512) and `hicolor/scalable/...svg`:
  Linux icon theme. The `ai.storyteller.photocraft` name is the upstream app id; it changes when
  Sepia's own app id is chosen (open question PC-O8 in the spec).

The app also sets the window icon and Wayland app ID at runtime (`apps/photocraft/src/app_icon.rs`,
`main.rs`). On Windows the taskbar shows the icon of the Start Menu shortcut that launches the
exe, so the MSI's `<Icon Id>` keeps the `.exe` extension (ICE50); `app_icon.rs` documents the
details and tests the `.ico` sizes and the WiX icon references.

## Regenerate

Replace `photocraft.svg` (and `photocraft-small.svg`), then run `packaging/icons.sh`. It needs
`resvg`, plus `iconutil` on macOS; the `.ico` is packed by `cargo xtask ico`.
