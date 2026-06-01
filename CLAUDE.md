# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal ZMK firmware configuration for a Corne split keyboard (6×3 + 2 thumb keys per half) running on Nice!Nano v2 microcontrollers with Bluetooth connectivity.

## Building

Firmware is built via GitHub Actions — push to `main` triggers the build automatically using the official ZMK workflow (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@v0.3`). There is no local build command; all compilation happens in CI.

The build targets defined in `build.yaml`:
- `corne_left` — left half firmware
- `corne_right` — right half firmware
- `settings_reset` — clears stored BLE pairings (flash to both halves when re-pairing)

## Architecture

### Key files

- `config/corne.keymap` — all layers, behaviors, macros, and tap-dance definitions (device tree syntax)
- `config/corne.conf` — Zephyr/ZMK Kconfig flags (sleep, BLE power, HID settings)
- `config/west.yml` — pins ZMK to `v0.3`; imports ZMK's own `west.yml`
- `build.yaml` — board/shield matrix passed to the ZMK CI workflow
- `boards/shields/corne/` — device tree overlays enabling wakeup on keypress for both halves
- `zephyr/module.yml` — sets `board_root: .` so Zephyr finds custom board definitions

### Keymap layers (in order)

| Index | Name | Purpose |
|-------|------|---------|
| 0 | `en_layer` | QWERTY base (English) |
| 1 | `ru_layer` | Russian/Cyrillic layout |
| 2 | `sym_layer` | Symbols and punctuation |
| 3 | `nav_num_layer` | Numbers, arrows, nav shortcuts |
| 4 | `bt_layer` | Bluetooth device selection |

### Custom behaviors in `corne.keymap`

- **Macros**: `switch_to_en` (sends F17, activates layer 0) / `switch_to_ru` (sends F18, activates layer 1) — host OS maps F17/F18 to input source switching
- **Tap-dance**: dual-function punctuation keys (e.g. tap `,` vs. shifted `,`); defined as `td_*` nodes
- **Hold-tap**: `ht_tab_layer` (tap = Tab, hold = BT layer), `ht_bt_clear` (tap = BT profile select, hold = BT clear)

### Language switching

Layer 1 (`ru_layer`) is not a standalone Cyrillic firmware layer — it mirrors the physical key positions while the OS handles the actual Cyrillic input. `switch_to_ru` / `switch_to_en` macros signal the OS to change the input source and simultaneously switch the active layer so tap-dance punctuation behavior stays correct for each language.
