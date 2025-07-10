# udev-wireless-keyboard-layout-fixes

**Attention: This won't work for keyboards connected via bluetooth!**

As non-US keyboard layout user it often happens that the "^" and "<" keys are falsely mapped when using USB wireless receiver (I guess they default to some US standard). No layout settings fixed that for me, only this udev rule!

## How to use

### Find out your vendor_id and product_id

- Use `lsusb`
- Find your device
- The vendor_id and product_id are placed right before the textual name of the device in the format `vendor_id:product_id` (example: 046d:c548)

### Edit the .hwdb file

- Download the `.hwdb` file from this repo
- Fill in your vendor_id and product_id (do not place the `:` in between, example: 046dc548)

### Apply .hwdb file

- Move to your `udev/hwdb.d/` folder (for me `/etc/udev/hwdb.d/`)
- `sudo systemd-hwdb update`
- `sudo udevadm trigger`

Done! The "^" and "<" should be swapped now and correspond to the correct hardware keys.

