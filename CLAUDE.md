# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A [ZMK firmware](https://zmk.dev) **user config** for the Totem split keyboard, built around two Seeed Studio XIAO nRF52840 controllers. No firmware source lives here — only keymap, Kconfig, and the west manifest. ZMK itself is pulled in as a dependency at build time.

## Build & flash

There is no local build command. Builds run in CI:

- **CI build**: `.github/workflows/build.yml` delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`. Any push or PR triggers builds for both halves. Download the resulting `firmware.zip` artifact from the Actions run, extract, and drag the `.uf2` files onto each XIAO's bootloader drive.
- **What gets built** is defined by `build.yaml` (`include:` list): `xiao_ble//zmk` board × `totem_left` / `totem_right` shields. The left half also uses the `studio-rpc-usb-uart` Zephyr snippet so ZMK Studio can talk to it over USB.
- **Local builds** would require a full Zephyr/west toolchain (`west init -l config && west update && west build …`); not set up here.

`CONFIG_ZMK_STUDIO=y` with `CONFIG_ZMK_STUDIO_LOCKING=n` (in `config/totem.conf`) means once flashed, the keymap can be edited live via [ZMK Studio](https://zmk.studio) over USB without re-flashing — useful for iterating on bindings.

## Keymap architecture (`config/totem.keymap`)

The Totem is a 3×(5+5) split with a 3-key thumb cluster per half, plus outer-column extras on the bottom row. Bindings are devicetree (`.keymap` is a `.dtsi`-style include).

Defined layers (index → name):

- `0` **base** — QWERTY. Left thumbs: `LGUI`, `&mo 2` (sym_func), `&mo 1` (nav_num). Right thumbs: `SPACE`, `R_SHIFT`, `R_GUI`. The `F` key is `&lt 4 F` (hold → layer 4).
- `1` **nav_num** — arrows + word-jump on the left, numpad on the right.
- `2` **sym_func** — brackets/parens on the left, F-keys + clipboard macros on the right.
- `3` **device** — Bluetooth profile select (`&bt BT_SEL 0..4`), media, brightness, `&bt BT_CLR`. **No binding activates this directly**; it's a [conditional layer](https://zmk.dev/docs/keymaps/conditional-layers) — holding layer 1 + layer 2 thumbs simultaneously activates it (`if-layers = <1 2>; then-layer = <3>;`).
- `4` **ff** — reached via `&lt 4 F` on the base layer's `F` key.
- `5`, `6` — reserved placeholders (`status = "reserved"`).

When changing layer indices, update both the binding references (`&mo N`, `&lt N`, `&to N`) and the `conditional_layers` block.

## Files

- `build.yaml` — CI matrix of board/shield combinations to build.
- `config/west.yml` — west manifest pulling ZMK from `zmkfirmware/zmk@main`.
- `config/totem.keymap` — keymap and conditional layers.
- `config/totem.conf` — Kconfig overrides (currently just Studio settings).
- `.github/workflows/build.yml` — calls the upstream reusable build workflow.
