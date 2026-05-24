# ODROID-C5 Bootloader Offset & Write Location

## Summary

Amlogic platforms (including ODROID-C5 with S905X5M) write the flashable bootloader image to **sector 1**, NOT sector 8192.

Confirmed evidence:
- CoreELEC ODROID-C5 image has Amlogic boot markers in the first sectors (`@AMLBOOT` in sector 0, `@ML` at sector 1).
- Official ODROID-C5 Yocto WIC metadata (`mdrjr/meta-odroid-aml`, `wic/odroid-c5.wks`) uses `part --source rawcopy --sourceparams="file=u-boot.bin.signed" --offset 1S`.

This is **critical** for:
- `UBOOT_TARGET_MAP` offset calculation
- SD card / eMMC write operations in CI/CD (`dd` invocations)
- Debugging partition table issues

---

## Discovery Method

1. **Vendor baseline capture** (Phase 0): Obtain an official Hardkernel ODROID-C5 Ubuntu 24.04 image (pre-flashed eMMC or SD card).

2. **Inspect partition table:**
   ```bash
   sudo fdisk -l /dev/mmcblk0
   ```
   Look at the "Start" column for the first partition (usually `/boot` or `/`). If it starts at sector **2048**, then sector 1 is free for bootloader.

3. **Confirm BL2 location:**
   ```bash
   sudo dd if=/dev/mmcblk0 of=bl2.bin bs=512 skip=1 count=1
   file bl2.bin
   # Should show ELF or Amlogic FIP magic
   ```

4. **Record in Phase 1 findings:**
   - Sector offset observed (expected: **1**)
   - Partition alignment (expected: sector 2048, leaving sectors 1–2047 for firmware)
   - `UBOOT_TARGET_MAP` format required (e.g., `u-boot.bin.sd.bin:u-boot.bin` or `fip/build/u-boot.bin:u-boot.bin`)

---

## Implication for Armbian Integration

When defining `UBOOT_TARGET_MAP` in `config/boards/odroidc5.wip`:

```bash
# Example (pending Phase 1 validation):
UBOOT_TARGET_MAP="u-boot.bin.sd.bin:u-boot.bin"
# or
UBOOT_TARGET_MAP="fip/build/u-boot.bin:u-boot.bin"
```

The Armbian build framework will invoke `dd` approximately as:
```bash
dd if=${UBOOT_TARGET_MAP%:*} of=${uboot_name}/u-boot.bin bs=1 status=none
# Then during image creation:
dd if=${uboot_name}/u-boot.bin of=IMAGE.img bs=512 seek=1 status=none
```

**Key offset:** `seek=1` (sector 1, not 8192 as in some ARM platforms). This is now confirmed by CoreELEC ODROID-C5 image inspection and official ODROID-C5 Yocto WIC metadata.

---

## Reference: Rubber-Duck Finding

In the comprehensive ODROID-C5 research report, the vendor baseline capture step (Phase 0) is critical for confirming this offset without guessing. Amlogic datasheets are not always available publicly, so direct observation is the most reliable method.

---

## Vendor Baseline Status

- `vendor-baseline done` — Official Hardkernel Ubuntu image URLs remain blocked/unlisted, but a CoreELEC ODROID-C5 image and official ODROID-C5 Yocto WIC metadata confirm the sector-1 bootloader write offset. See `research/vendor-baseline-c5.md` for hashes, partition table, DTB, and bootargs.
