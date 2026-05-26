---
name: zmk-keymap-builder
description: Layer-by-layer specs and workflows for ZMK keyboard configs in this repo. Covers keymap parity rules, 40% vs 50% layout mapping, custom shield structure, and step-by-step process for adding new boards.
license: MIT
compatibility: claude
metadata:
  target: zmk-config
  layout_type: ortholinear-split-and-one-piece
---

## What I do
- Provide exact per-layer, per-key binding specs for all boards in this repo.
- Guide creation of new keyboard profiles (keymap, conf, custom shield if needed).
- Audit and fix parity divergences between boards.

## When to use me
- Creating or porting a keymap to a new board.
- Editing any layer in any `config/*.keymap`.
- Adding a new build target or custom shield.
- Auditing parity between 40% and 50% or split and one-piece boards.

---

## 1. Core Rules

- **Reference:** `config/helix.keymap` (5-row split 50%) is the gold standard.
- **4-row parity:** The bottom 4 rows are physically identical in key position and function across all boards. The 50% top row is a bonus — its content is remapped into layers on 40% boards, not dropped.
- **Bindings:** `&kp`, `&mo`, `&bt`, `&out`, `&soft_off` only. No hold-tap, combos, or macros unless explicitly requested.
- **Includes:** Every keymap must include `<dt-bindings/zmk/bt.h>` and `<dt-bindings/zmk/outputs.h>`.
- **No stuck layers:** all layer activation uses `&mo`. Never use `&to`, `&tog`, or `&sl`.
- **ADJUST access:** hold LOWER + RAISE simultaneously (`zmk,conditional-layers`). Order-independent.
- **Reset / bootloader:** all keyboards have a physically accessible reset button (single tap = reset, double tap = bootloader). No keymap bindings for reset or bootloader on any board.
- **soft_off:** all wireless boards have `&soft_off` on ADJUST bottom row col 0 with `hold-time-ms = <500>`. Any keypress wakes from sleep/soft-off. Planck is USB-only — no soft-off.
- **soft_off wakeup sources:** helix and helix_4row define `zmk,soft-off-wakeup-sources` with `&kscan0` in their keymap root. `blank_slate` board DTS owns its own wakeup sources — do not add this node to `blank_slate.keymap`.

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

Right side is a numpad. Left side has `&mo LOWER` at home-row col-0 (enables ADJUST), rest `&trans`. No BT bindings.

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | `&trans` x6 | `&trans` x5, DEL |
| Row 1 (40%) / Row 2 (50%) | `&mo LOWER` `&trans` x5 | N6 N7 N8 N9 N0 DEL (40%) / SLASH N7 N8 N9 MINUS `&trans` (50%) |
| Row 2 (40%) / Row 3 (50%) | `&trans` x6 | SLASH N7 N8 N9 MINUS `&trans` (40%) / ASTRK N4 N5 N6 PLUS `&trans` (50%) |
| Row 3 (40%) / Row 4 (50%) | `&trans` x6 | ASTRK N4 N5 N6 PLUS `&trans` (40%) / `&trans` N1 N2 N3 CARET `&trans` (50%) |
| Row 4 (40%) / Row 5 (50%) | `&trans` x5, DEL `[inner: &trans]` | `[inner: &trans]` DEL `&trans` N0 DOT `&trans` EQUAL |

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

1. **Includes:** `bt.h` and `outputs.h` in every keymap.
2. **No BT/OUT on LOWER or RAISE:** all `&bt` and `&out` live on ADJUST only.
3. **LOWER bottom row left:** `&trans &trans C_PP &trans C_MUTE BSPC`.
4. **LOWER bottom row right:** `BSPC &mo RAISE &trans &trans &trans &trans`.
5. **RAISE left:** `&mo LOWER` at home-row col-0; everything else `&trans`.
6. **RAISE numpad order:** SLASH/7/8/9/MINUS → ASTRK/4/5/6/PLUS → trans/1/2/3/CARET.
7. **ADJUST F-key row:** F1–F12 across row 1 (40%) / row 2 (50%).
8. **ADJUST ZXCV row:** BT_SEL 0-4 + BT_CLR left, BT_DISC 0-4 + `&trans` right.
9. **ADJUST bottom row:** `&soft_off`, OUT_BLE, OUT_USB, then `&trans`. (`planck` uses `&trans` at col 0 — USB only, no soft-off.)
10. **soft_off config (all wireless boards):** `&soft_off { hold-time-ms = <500>; }` outside root node.
11. **soft_off wakeup (all wireless boards):** `zmk,soft-off-wakeup-sources` with `wakeup-sources = <&kscan_label>` inside root. Use `&kscan0` for helix/helix_4row; use `&kscan` for blank_slate (board's kscan label differs). This overrides the board DTS wakeup source so any key wakes the board, not just the dedicated hardware button.
12. **No reset/bootloader bindings anywhere** — physical reset button on every board handles this.

---

## 4. Adding a New Board

1. **`build.yaml`** — append the target. ZMK Zephyr 4.1 board ID syntax:
   - `nice_nano//zmk` for all nice!nano v2 (used exclusively in this repo)
   - `planck//zmk` for planck
   - `blank_slate` (no suffix — external module board, no `zmk` variant)

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
   - 40% one-piece wireless: copy `config/blank_slate.keymap`
   - 40% one-piece USB-only: copy `config/planck.keymap`

4. **External module boards:** add the remote and project to `config/west.yml`.

5. **Custom shield** (only if matrix differs from upstream): create `boards/shields/<name>/` with `Kconfig.shield`, `<name>.dtsi`, overlays. See `boards/shields/helix_4row/` as reference.
