---
name: zmk-keymap-builder
description: Layer-by-layer specs and workflows for ZMK keyboard configs in this repo. Covers keymap parity rules, 40% vs 50% layout mapping, custom shield structure, and the decision rule for when to use a separate ZMK config repo for a board.
license: MIT
compatibility: claude
metadata:
  target: zmk-config
  layout_type: ortholinear-split-and-one-piece
---

## What I do
- Provide exact per-layer, per-key binding specs for the boards in this repo.
- Guide creation of new keyboard profiles (keymap, conf, custom shield if needed).
- Audit and fix parity divergences between boards.
- Decide whether a new board belongs in this repo or a separate one.

## When to use me
- Creating or porting a keymap to a new board.
- Editing any layer in any `config/*.keymap`.
- Adding a new build target or custom shield.
- Auditing parity between 40% and 50% or split and one-piece boards.
- Evaluating whether an external ZMK module board can be added here.

---

## 0. Scope: this repo vs separate repos

This repo (`zmk-config`) builds for boards that work cleanly against ZMK `main` (Zephyr 4.1 / HWMv2). The Blank Slate is **excluded** and lives in a sister repo at `~/git/blank-slate-zmk-config` because the upstream `blank-slate-zmk-module` HWMv2 migration is unmerged, and only the ZMK `v0.3` branch produces firmware that flashes correctly to the hardware. A west manifest can only pin one ZMK revision per repo, so the two boards split across two repos by necessity.

**Decision rule for a new external module board:**
- If the module is HWMv2/ZMK-main compatible and proven to flash correctly → add to this repo.
- If the module is only validated against an older ZMK release, or its HWMv2 migration is unverified → put it in a separate ZMK config repo pinned to the appropriate ZMK release.

Do not attempt to re-add the Blank Slate to this repo until upstream's HWMv2 migration merges *and* the resulting firmware is confirmed to flash and run.

---

## 1. Core Rules

- **Reference:** `config/helix.keymap` (5-row split 50%) is the gold standard.
- **4-row parity (the central rule):** The bottom 4 rows are byte-identical in function across ALL boards on ALL layers — same keys, same `&trans` placements, same numpad ordering. The only difference between 50% and 40% is whether the bonus row exists at all. Don't try to remap or shift anything between board sizes; copy helix's lower-4-rows verbatim. **This applies to the sister blank_slate repo too** — keep its keymap aligned.
- **Bindings:** `&kp`, `&mo`, `&bt`, `&out`, `&soft_off` only. No hold-tap, combos, or macros unless explicitly requested.
- **Includes:** Every wireless keymap must include `<dt-bindings/zmk/bt.h>` and `<dt-bindings/zmk/outputs.h>`.
- **No stuck layers:** all layer activation uses `&mo`. Never use `&to`, `&tog`, or `&sl`.
- **ADJUST access:** hold LOWER + RAISE simultaneously (`zmk,conditional-layers`). Order-independent.
- **Reset / bootloader:** all keyboards have a physically accessible reset button (single tap = reset, double tap = bootloader). No keymap bindings for reset or bootloader.
- **soft_off:** all wireless boards have `&soft_off` on ADJUST bottom row col 0 with `hold-time-ms = <500>`. Any keypress wakes from sleep/soft-off. Planck is USB-only — no soft-off.
- **soft_off wakeup sources:** wireless boards in this repo (helix, helix_4row) define `zmk,soft-off-wakeup-sources` with `&kscan0` in their keymap root. The Blank Slate in the sister repo uses `&kscan` (board DTS label differs).

---

## 2. Layer Specs

### Layer 0 — DEFAULT

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | GRAVE N1 N2 N3 N4 N5 | N6 N7 N8 N9 N0 BSPC |
| Row 1 (40%) / Row 2 (50%) | TAB Q W E R T | Y U I O P BSLH |
| Row 2 (40%) / Row 3 (50%) | `&mo LOWER` A S D F G | H J K L SEMI SQT |
| Row 3 (40%) / Row 4 (50%) | LSHFT Z X C V B `[inner: &trans]` | `[inner: &trans]` N M COMMA PERIOD SLASH RSHFT |
| Row 4 (40%) / Row 5 (50%) | LCTRL ESC LALT EQUAL LCMD SPACE `[inner: &trans]` | `[inner: &trans]` SPACE `&mo RAISE` MINUS LBKT RBKT RET |

### Layer 1 — LOWER

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | F1 F2 F3 F4 F5 F6 | F7 F8 F9 F10 F11 F12 |
| Row 1 / Row 2 | TILDE EXCL AT HASH DLLR PRCNT | CARET AMPS ASTRK LPAR RPAR `&trans` |
| Row 2 / Row 3 | `&trans` `&trans` C_NEXT C_BRI_UP C_VOL_UP `&trans` | LEFT DOWN UP RIGHT `&trans` `&trans` |
| Row 3 / Row 4 | `&trans` `&trans` C_PREV C_BRI_DN C_VOL_DN `&trans` | HOME PG_DN PG_UP END `&trans` `&trans` |
| Row 4 / Row 5 | `&trans` `&trans` C_PP `&trans` C_MUTE BSPC `[inner: &trans]` | `[inner: &trans]` BSPC `&mo RAISE` `&trans` `&trans` `&trans` `&trans` |

### Layer 2 — RAISE

Right side is a numpad arranged like a calculator: **789 / 456 / 123 / bottom-extras**, top to bottom. The numpad occupies the four lower rows on every board, identical across 40% and 50%. The 50% bonus row is unused (`&trans`).

**No number-row substitute on 40% boards.** Numbers come exclusively from the numpad.

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% bonus row (50% only) | `&trans` x6 | `&trans` x5, DEL |
| Lower-4 row 1 (LOWER row) | `&mo LOWER` `&trans` x5 | SLASH N7 N8 N9 MINUS `&trans` |
| Lower-4 row 2 | `&trans` x6 | ASTRK N4 N5 N6 PLUS `&trans` |
| Lower-4 row 3 (ZXCV row) | `&trans` x6 `[inner: &trans]` | `[inner: &trans]` N0 N1 N2 N3 CARET `&trans` |
| Lower-4 row 4 (bottom row) | `&trans` x5, DEL `[inner: &trans]` | `[inner: &trans]` DEL `&trans` N0 DOT `&trans` EQUAL |

The lower-4 rows are **identical across all four boards** (helix, helix_4row, planck, blank_slate). 50% boards just add the bonus row on top. This is the universal rule: 50% and 40% layouts only differ by whether the bonus row exists at all.

### Layer 3 — ADJUST (conditional: hold LOWER + RAISE)

Identical on all boards. 50% has empty bonus row at top.

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | `&trans` x6 | `&trans` x6 |
| Row 1 (40%) / Row 2 (50%) | F1 F2 F3 F4 F5 F6 | F7 F8 F9 F10 F11 F12 |
| Row 2 (40%) / Row 3 (50%) — `&mo LOWER` row | `&trans` x6 | `&trans` x6 |
| Row 3 (40%) / Row 4 (50%) — ZXCV row | BT_SEL 0-4, BT_CLR `[inner: &trans]` | `[inner: &trans]` BT_DISC 0-4, `&trans` |
| Row 4 (40%) / Row 5 (50%) — bottom row | `&soft_off`, OUT_BLE, OUT_USB, `&trans` x3 `[inner: &trans]` | `[inner: &trans]` `&trans` x6 |

**Layer key positions on ADJUST:** `&mo LOWER` and `&mo RAISE` positions must be `&trans` — physically held to reach ADJUST, so their binding never fires.

---

## 3. Parity Audit Checklist

1. **Includes:** `bt.h` and `outputs.h` in every wireless keymap.
2. **No BT/OUT on LOWER or RAISE:** all `&bt` and `&out` live on ADJUST only.
3. **LOWER bottom row left:** `&trans &trans C_PP &trans C_MUTE BSPC`.
4. **LOWER bottom row right:** `BSPC &mo RAISE &trans &trans &trans &trans`.
5. **RAISE left:** `&mo LOWER` at home-row col-0; everything else `&trans`.
6. **RAISE numpad order (right side, top to bottom):** SLASH/7/8/9/MINUS → ASTRK/4/5/6/PLUS → N0/1/2/3/CARET → bottom-extras. Identical on all boards; on 50% just shift down one row to leave the bonus row `&trans`.
7. **ADJUST F-key row:** F1–F12 across row 1 (40%) / row 2 (50%).
8. **ADJUST ZXCV row:** BT_SEL 0-4 + BT_CLR left, BT_DISC 0-4 + `&trans` right.
9. **ADJUST bottom row:** `&soft_off`, OUT_BLE, OUT_USB, then `&trans`. (`planck` uses `&trans` at col 0 — USB only, no soft-off.)
10. **soft_off config (all wireless boards):** `&soft_off { hold-time-ms = <500>; }` outside root node.
11. **soft_off wakeup (this repo):** `zmk,soft-off-wakeup-sources` with `wakeup-sources = <&kscan0>` inside root.
12. **No reset/bootloader bindings anywhere** — physical reset button on every board handles this.
13. **Sister repo parity:** when changing layer logic that affects the universal 4-row design, also port the change to `~/git/blank-slate-zmk-config/config/lpgalaxy_blank_slate.keymap` (HWMv1 board name there; its kscan label is `&kscan`, not `&kscan0`).

---

## 4. Adding a New Board

First, decide if the board belongs in this repo or a separate one (see Section 0). The steps below assume this repo.

1. **`build.yaml`** — append the target. ZMK Zephyr 4.1 board ID syntax:
   - `nice_nano//zmk` for all nice!nano v2 (used exclusively in this repo)
   - `planck//zmk` for planck
   - For other upstream ZMK boards: append `//zmk` to the board name

2. **`config/<name>.conf`** — standard wireless template:
   ```properties
   CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
   CONFIG_USB_HID_POLL_INTERVAL_MS=1
   CONFIG_ZMK_SLEEP=y
   CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=10800000
   CONFIG_ZMK_PM_SOFT_OFF=y
   ```

3. **`config/<name>.keymap`** — start from:
   - 50% split: copy `config/helix.keymap`
   - 40% split: copy `config/helix_4row.keymap`
   - 40% one-piece USB-only: copy `config/planck.keymap`

4. **Custom shield** (only if matrix differs from upstream): create `boards/shields/<name>/` with `Kconfig.shield`, `<name>.dtsi`, overlays. See `boards/shields/helix_4row/` as reference.

5. **External module boards** — before adding, verify the module is validated against ZMK `main`. If not, build in a separate ZMK config repo pinned to a working release. See Section 0.
