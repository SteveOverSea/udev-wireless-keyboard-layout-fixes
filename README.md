# udev-wireless-keyboard-layout-fixes

Fixes for Logitech keyboards that reach Linux with a **macOS-style keymap**,
plus the udev/hwdb fallback for units that stay that way.

## Read this first: it is usually the keyboard's layout mode, not Linux

Logitech's cross-platform keyboards (MX Keys, MX Mechanical, Craft, K380/K480/K780,
K375s, MK850, …) keep a **layout mode in their own firmware**. In macOS mode the
*hardware* swaps the HID usages, so the wrong keys arrive at the kernel and every
layer above them is wrong — compositor, console, games, everything. No layout
setting can fix that, because the characters are already wrong before XKB runs.

| physical key     | macOS mode            | PC mode              |
| ---------------- | --------------------- | -------------------- |
| left Alt         | `0xE3` Left GUI       | `0xE2` Left Alt      |
| Windows/Super    | `0xE2` Left Alt       | `0xE3` Left GUI      |
| AltGr            | `0xE7` Right GUI      | `0xE6` Right Alt     |
| `^` (left of 1)  | `0x64` NonUSBackslash | `0x35` Grave         |
| `<` (left of Y)  | `0x35` Grave          | `0x64` NonUSBackslash|

Symptoms: Alt acts as Super, AltGr as right Super, and on ISO layouts `^` and `<`
are swapped.

Switch the keyboard to PC layout — hold the keys for about 3 seconds until an
Easy-Switch LED (1/2/3) blinks:

- **Fn + P** → PC / Windows layout
- **Fn + O** → macOS layout

That is stored in the keyboard, survives reboots and affects every host slot, so
it is the whole fix. No root, no udev rule, no reboot.

Verify with `sudo evtest` or `libinput debug-events`: press the keys and watch the
reported *code* (not the character). ALT should show `KEY_LEFTALT`, the Windows
key `KEY_LEFTMETA`.

## When the rule is still needed

Some units cannot be switched (or flip back, e.g. another host running Logi
Options+ with "always keep the keyboard in Mac layout"). The udev rule remaps the
HID usages at kernel level, so it also covers the text console, games and
anything that bypasses the compositor — an XKB option such as
`altwin:swap_alt_win` does not.

**Only apply this in macOS mode.** In PC layout the two `KEYBOARD_KEY` lines below
would *create* the `^`/`<` swap, and they never fix Alt↔Super — if Alt is your
symptom, you are in macOS mode.

### Find out your vendor_id and product_id

- Use `lsusb`
- Find your device
- The vendor_id and product_id are placed right before the textual name of the
  device in the format `vendor_id:product_id` (example: `046d:c548`)

### Edit the .hwdb file

- Download the `.hwdb` file from this repo
- Fill in your vendor_id and product_id (do not place the `:` in between,
  example: `046dc548`)
- In macOS mode with broken modifiers, add:

```
    KEYBOARD_KEY_700e2=leftmeta
    KEYBOARD_KEY_700e3=leftalt
    KEYBOARD_KEY_700e7=rightalt
```

### Apply the .hwdb file

- Move to your `udev/hwdb.d/` folder (for me `/etc/udev/hwdb.d/`)
- `sudo systemd-hwdb update`
- `sudo udevadm trigger`

Done! `^` and `<` now correspond to the correct hardware keys.

**Attention:** the match is on the receiver's vendor/product ID, so this does not
work for the same keyboard connected over Bluetooth — use the Bluetooth product ID
there instead, or just fix the layout mode with Fn + P.
