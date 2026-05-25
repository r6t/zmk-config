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
| Row 4 / Row 5 | OUT_BLE OUT_USB C_PP `&trans` C_MUTE BSPC `[inner: &trans]` | `[inner: &trans]` BSPC `&mo RAISE` `&trans` `&trans` `&trans` `&trans` |

### Layer 2 — RAISE

The right side is a numpad. The left side differs between 40% and 50%.

**Right side (identical on all boards):**

| Row | Right 6 keys |
|-----|--------------|
| Row 1 | `&trans` x5, DEL (50%) — or — N6 N7 N8 N9 N0 DEL (40% number sub) |
| Row 2 | SLASH N7 N8 N9 MINUS `&trans` |
| Row 3 | ASTRK N4 N5 N6 PLUS `&trans` |
| Row 4 | `&trans` N1 N2 N3 CARET `&trans` |  
| Row 5 / bottom | `[inner: &trans]` DEL `&trans` N0 DOT `&trans` EQUAL |

**Left side:**

| Row | 50% | 40% |
|-----|-----|-----|
| Row 1 (bonus) | BT_SEL 0–4, BT_CLR | *(no row)* |
| Row 2 / Row 1 (40%) | BT_DISC 0–4, `&trans` | `&mo LOWER` `&trans` x5 |
| Row 3 / Row 2 (40%) | `&mo LOWER` `&trans` x5 | BT_SEL 0–4, BT_CLR |
| Row 4 / Row 3 (40%) | `&trans` x6 | `&trans` x6 |
| Row 5 / Row 4 (40%) | `&trans` x5, DEL `[inner: &trans]` | `&trans` x5, DEL `[inner: &trans]` |

### Layer 3 — ADJUST (conditional: LOWER + RAISE held)

| Row | All 12 keys |
|-----|-------------|
| 50% Row 1 (bonus) | `&trans` x12 |
| Row 1 / Row 2 | F1 F2 F3 F4 F5 F6 — F7 F8 F9 F10 F11 F12 |
| Rows 2–4 | `&trans` throughout (including inner column on split) |

---

## 3. Parity Audit Checklist

1. **Includes:** `bt.h` and `outputs.h` present.
2. **Lower bottom row left:** `OUT_BLE OUT_USB C_PP &trans C_MUTE BSPC`.
3. **Lower bottom row right:** `BSPC &mo RAISE &trans &trans &trans &trans`.
4. **Raise row 1 right:** ends with `DEL`.
5. **Raise numpad rows:** SLASH/N7/N8/N9/MINUS on row 2, ASTRK/N4–N6/PLUS on row 3.
6. **Adjust row 1 (40% only):** F1–F12 across the full row.
7. **40% raise row 3 left:** BT_SEL 0–4 + BT_CLR (not `&mo LOWER` — that lives on row 2).

---

## 4. Adding a New Board

1. **`build.yaml`** — append the target. Split wireless example:
   ```yaml
   - board: nice_nano_v2
     shield: <name>_left
   - board: nice_nano_v2
     shield: <name>_right
   ```

2. **`config/<name>.conf`** — standard wireless template:
   ```properties
   CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
   CONFIG_USB_HID_POLL_INTERVAL_MS=1
   ```

3. **`config/<name>.keymap`** — start from:
   - 50% split: copy `config/helix.keymap`
   - 40% split: copy `config/helix_4row.keymap`
   - 40% one-piece: copy `config/planck.keymap`
   - 50% one-piece: copy `config/preonic.keymap`

4. **Custom shield** (only if the upstream ZMK shield doesn't exist or the matrix differs):
   Create `boards/shields/<name>/` with `Kconfig.shield`, `<name>.dtsi`, and `<name>_left.overlay` / `<name>_right.overlay`. See `boards/shields/helix_4row/` as the reference implementation.
