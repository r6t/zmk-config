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
- **Bindings:** `&kp`, `&mo`, `&bt`, `&out` only. No hold-tap, combos, or macros unless explicitly requested.
- **Includes:** Every keymap must include `<dt-bindings/zmk/bt.h>` and `<dt-bindings/zmk/outputs.h>`.
- **No stuck layers:** all layer activation uses `&mo` (momentary). Never use `&to`, `&tog`, or `&sl` unless explicitly asked.
- **ADJUST access:** hold LOWER + RAISE simultaneously. Uses `zmk,conditional-layers`. Order-independent — fires whenever both are held regardless of press order.

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

Right side is a numpad (identical on all boards). Left side has `&mo LOWER` at the home row position to enable ADJUST, rest is `&trans`. No BT bindings on RAISE.

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | `&trans` x6 | `&trans` x5, DEL |
| Row 1 (40%) / Row 2 (50%) | `&mo LOWER` `&trans` x5 | N6 N7 N8 N9 N0 DEL (40%) / SLASH N7 N8 N9 MINUS `&trans` (50%) |
| Row 2 (40%) / Row 3 (50%) | `&trans` x6 | SLASH N7 N8 N9 MINUS `&trans` (40%) / ASTRK N4 N5 N6 PLUS `&trans` (50%) |
| Row 3 (40%) / Row 4 (50%) | `&trans` x6 | ASTRK N4 N5 N6 PLUS `&trans` (40%) / `&trans` N1 N2 N3 CARET `&trans` (50%) |
| Row 4 (40%) / Row 5 (50%) | `&trans` x5, DEL `[inner: &trans]` | `[inner: &trans]` DEL `&trans` N0 DOT `&trans` EQUAL |

Note: on 40%, row 1 of RAISE is the number row substitute (GRAVE N1–N5 left, N6–N0 DEL right), which shifts the numpad down one row vs 50%. The `&mo LOWER` is always at the physical home-row col-0 position.

### Layer 3 — ADJUST (conditional: hold LOWER + RAISE)

Identical functional layout on all boards. 50% boards have an empty bonus row at top.

| Row | Left 6 keys | Right 6 keys |
|-----|-------------|--------------|
| 50% Row 1 (bonus) | `&trans` x6 | `&trans` x6 |
| Row 1 (40%) / Row 2 (50%) — TAB row | F1, F2, F3, F4, F5, F6 | F7, F8, F9, F10, F11, F12 |
| Row 2 (40%) / Row 3 (50%) — `&mo LOWER` row | `&trans` x6 | `&trans` x6 |
| Row 3 (40%) / Row 4 (50%) — ZXCV row | BT_SEL 0-4, BT_CLR `[inner: &trans]` | `[inner: &trans]` BT_DISC 0-4, `&trans` |
| Row 4 (40%) / Row 5 (50%) — bottom row | `&soft_off`, OUT_BLE, OUT_USB, `&trans` x3 `[inner: &trans]` | `[inner: &trans]` `&trans` x6 (`&mo RAISE` pos is `&trans`) |

**Critical:** `&mo LOWER` and `&mo RAISE` key positions on ADJUST must be `&trans` — these keys are physically held to reach ADJUST, so their ADJUST binding is never triggered as a tap. Using `&trans` is correct per ZMK docs; it passes through to the layer below (no-op while held).

---

## 3. Parity Audit Checklist

1. **Includes:** `bt.h` and `outputs.h` present in every keymap.
2. **No BT/OUT on LOWER or RAISE:** all `&bt` and `&out` bindings live exclusively on ADJUST.
3. **LOWER bottom row left:** `&trans &trans C_PP &trans C_MUTE BSPC` (no OUT_BLE/OUT_USB).
4. **LOWER bottom row right:** `BSPC &mo RAISE &trans &trans &trans &trans`.
5. **RAISE left side:** only `&mo LOWER` at the home-row col-0 position; everything else `&trans`.
6. **RAISE right side numpad:** row order is SLASH/7/8/9/MINUS, then ASTRK/4/5/6/PLUS, then trans/1/2/3/CARET.
7. **ADJUST layer key positions are `&trans`:** both `&mo LOWER` and `&mo RAISE` physical key positions on ADJUST must be `&trans`. These keys are held to reach ADJUST; per ZMK docs `&trans` passes through (no-op while held).
8. **ADJUST row 1 (40%) / row 2 (50%):** F1–F12 across the full row.
9. **ADJUST ZXCV row:** BT_SEL 0-4 + BT_CLR left, BT_DISC 0-4 + `&trans` right.
10. **ADJUST bottom row:** `&soft_off`, OUT_BLE, OUT_USB, then `&trans` for the rest.
11. **soft_off config:** `&soft_off { hold-time-ms = <500>; };` declared outside the root node. `zmk,soft-off-wakeup-sources` node listing `&kscan0` in the root so any key wakes.
12. **No `&to`, `&tog`, or `&sl`:** every layer key is `&mo`.
13. **Planck is USB-only:** no BT power boost, no sleep config in `planck.conf`. Planck keymap has no `soft_off_wakers` node and no `&soft_off` binding.

---

## 4. Adding a New Board

1. **`build.yaml`** — append the target. Board IDs require ZMK's Zephyr 4.1 variant suffix (`//zmk`). Split wireless example:
   ```yaml
   - board: nice_nano//zmk
     shield: <name>_left
   - board: nice_nano//zmk
     shield: <name>_right
   ```
   Use `nice_nano@1//zmk` for v1 hardware, `nice_nano//zmk` for v2 (default). Standalone boards: `planck//zmk`.

2. **`config/<name>.conf`** — standard wireless template:
   ```properties
   CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
   CONFIG_USB_HID_POLL_INTERVAL_MS=1
   # Power management (no displays/RGB on these keyboards)
   CONFIG_ZMK_SLEEP=y
   CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=14400000
   CONFIG_ZMK_PM_SOFT_OFF=y
   ```

3. **`config/<name>.keymap`** — start from:
   - 50% split: copy `config/helix.keymap`
   - 40% split: copy `config/helix_4row.keymap`
   - 40% one-piece: copy `config/planck.keymap`

4. **Custom shield** (only if the upstream ZMK shield doesn't exist or the matrix differs):
   Create `boards/shields/<name>/` with `Kconfig.shield`, `<name>.dtsi`, and `<name>_left.overlay` / `<name>_right.overlay`. See `boards/shields/helix_4row/` as the reference implementation.
