# TOTEM: QMK Colemak-DH migration

Based on `handwired/dactyl_manuform/5x6_5_ergohaven` / `home_row_3x5_colemak_dh_mode`.
The original 62-position QMK export is saved in `reference/qmk-keymap.json`.
Run `python3 scripts/convert_qmk.py` after editing the converter or reference.
Hardware pin assignments and matrix transform are retained from the TOTEM author.
ZMK and its build workflow are pinned to v0.3.0 instead of the moving main branch.

## Physical order (viewed from above, both halves left to right)

```text
       Q W F P B       J L U Y ;
       A R S T G       M N E I O
 BT+   Z X C D V       K H , . /   Studio
          Esc Space Tab   Backspace Enter Delete
```

Home row holds: A=Left Ctrl, R=Left Alt, S=Left Shift, T=Left GUI;
N=Right GUI, E=Right Shift, I=Right Alt, O=Right Ctrl.
Thumb holds: Esc=FUN(3), Space=NAV(1), Tab=NUM(2), Enter=MOUSE(4).
All 36 original active positions are retained on all five layers, including blocked
positions (`KC_NO` -> `&none`, not transparent).

## Two extra TOTEM keys

- Outer left: next Bluetooth profile. On FUN (hold Esc), clear the current profile.
- Outer right: unlock ZMK Studio. On FUN, toggle USB/Bluetooth output.
- Studio remains locked by default. The layout diagram is a logical 38-key diagram,
  not a mechanically accurate drawing of the case.

## Portability differences — this is not a byte-for-byte QMK behavior clone

- Normal letters, digits, punctuation and standalone Tab use Auto Shift at 175 ms.
  Hold past this threshold to send the shifted key, including host key repeat.
- Home-row mods use 175 ms hold-taps with retro-tap. A solo hold/release produces
  the unshifted letter. QMK Retro Shift (uppercase on solo release between 175 and
  500 ms) is NOT reproduced on these eight keys. ZMK retro-tap also defers the
  modifier until another keyboard key is pressed: modifier + physical mouse click
  is not equivalent to QMK's >500 ms hold. Use the dedicated modifiers on NAV/NUM/FUN
  for mouse-click combinations, or tune/replace this behavior after testing.
- Same-hand non-mod keys and opposite-hand keys can trigger home-row holds.
  Same-hand home-row-mod rolls favor taps. QMK PERMISSIVE_HOLD's nested same-hand
  mod-tap sequences are not exactly equivalent to ZMK positional hold-tap.
- Thumb hold-taps have the same 175 ms threshold and layer numbers, with retro-tap.
  The Tab layer-tap does not auto-shift into Shift+Tab when held alone.
- ZMK Auto Shift is a hold-tap implementation. QMK's modifier suppression and
  repeat/re-tap rules are not exactly reproduced; test shortcuts and fast rolls.
- Mouse motion, wheel directions, clicks and shortcuts are preserved.
  QMK MS_ACL0/MS_ACL2 become held slow/fast overlays (200/1200 units per second;
  default 600). These are implementation layers 5/6, not additional user layers.
  Speed switching applies to subsequent direction presses; release a direction,
  hold the speed key, then press the direction again. QMK KINETIC_SPEED's movement
  curve and in-flight speed changes are not reproduced exactly.
- Device-specific QMK settings such as EE_HANDS, matrix row numbers and AVR
  bootloader configuration are replaced by the TOTEM ZMK hardware definitions.

## Build

Push this branch to GitHub, open Actions, and wait for all three builds to pass.
Download the `firmware` artifact. It contains `totem-left.uf2`, `totem-right.uf2`
and `totem-settings-reset.uf2`. The reset image is recovery-only; do not flash it
as the normal keyboard firmware.

## Flash and verify

1. Connect the left half over a data-capable USB cable. Double-tap Reset.
2. Copy `totem-left.uf2` to the mounted bootloader drive. Wait for automatic reboot.
3. Repeat for the right half with `totem-right.uf2`.
4. Power both halves and test the left half over USB first.
5. Verify every tap, each of the four thumb layers, home-row shortcuts, Auto Shift,
   all mouse directions/wheel/buttons and speed keys.
6. Because mouse HID capabilities change from older firmware, re-pair Bluetooth
   if needed: forget TOTEM on the Mac, hold Esc and press outer-left to clear the
   active keyboard profile, then pair again.
7. Open ZMK Studio and press outer-right to unlock. Future source keymap updates
   may require Studio's Restore Stock Settings if a layout was saved in Studio.

Only use the settings-reset image to troubleshoot pairing that cannot otherwise
be recovered; it erases stored bonds/settings. Flash it to both halves and then
reinstall their respective normal firmware, following the ZMK troubleshooting guide.

## Sources

- https://keeb.supply/products/geist-totem
- https://github.com/GEIGEIGEIST/TOTEM#firmware
- https://github.com/GEIGEIGEIST/zmk-config-totem#how-to-use
- https://zmk.dev/docs/user-setup#flash-uf2-files
- https://zmk.dev/docs/keymaps/behaviors/hold-tap
- https://zmk.dev/docs/keymaps/behaviors/mouse-emulation
- https://zmk.dev/docs/features/studio
