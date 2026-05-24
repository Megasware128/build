# U-Boot v2023.01 Patches for ODROID-C5

## Overview

This directory (`patch/u-boot/v2023.01/odroidc5/`) holds U-Boot patches for Hardkernel's ODROID-C5 (Amlogic S905X5M / S7D).

**Versioned patch directory layout:**
- Source: Hardkernel `u-boot` repository, branch `odroidc5-v2023.01`
- Target: Armbian build framework expects `BOOTPATCHDIR='v2023.01/odroidc5'`
- Apply order: numeric prefix (`0000-`, `0001-`, etc.) sorted lexically
- CLI workflow: Use `./compile.sh uboot-patch BOARD=odroidc5 BRANCH=current` to edit, commit, and export patches

See `.github/skills/armbian-board-patches/SKILL.md` § 5 for CLI usage and override rules.

---

## Phase 1 Standalone Build Findings

Container validation of Hardkernel `u-boot` branch `odroidc5-v2023.01` shows that the GitHub repository is a firmware superproject. The buildable BL33 U-Boot tree is the `bl33/v2023` submodule; firmware/FIP inputs are other submodules.

### Phase 1 Validation Answers

1. **defconfig discovery**
   - `odroidc5_defconfig` does **not** exist.
   - The ODROID-C5 defconfig is `s7d_odroidc5_defconfig`, stored at `bl33/v2023/configs/amlogic/s7d_odroidc5_defconfig`.
   - Configure from `bl33/v2023` with `make s7d_odroidc5_defconfig` (not `make amlogic/s7d_odroidc5_defconfig`; this tree prepends `configs/amlogic/`).

2. **Final boot artifact format**
   - Standalone BL33 compile succeeds and emits `bl33/v2023/build/u-boot.bin` (1,805,712 bytes in the validated container run).
   - This is **not** the flashable SD/eMMC bootloader; it is BL33 input to Amlogic FIP packaging.
   - Hardkernel's FIP flow is expected to copy the final flashable image to top-level `build/u-boot.bin` after `./mk s7d_odroidc5 ...` completes.

3. **Required blobs & tools**
   - Required blobs/tools are vendored as submodules, not LibreELEC `amlogic-boot-fip`: `bl2/bin/s7d/s905x5m/*`, `bl31/bl31_2.7/bin/s7d/s905x5m/*`, `bl32/bl32_3.18/bin/s7d/s905x5m/*`, `bl40/bin/s7d/s905x5m/*`, `soc/templates/s7d/s905x5m/*`, and `fip/s7d/aml_encrypt_s7d`/`fip/s7d/binary-tool/acpu-imagetool`.
   - Full `./mk s7d_odroidc5 --disable-bl33z` packaging remains partial: it reaches BL30, then fails because BL30 is built from the RTOS SDK and needs extra setup (`cmake`, RTOS SDK compiler environment via `scripts/env.sh`, and likely a RISC-V toolchain) or a matching prebuilt `bl30.bin`.

4. **Boot flow compatibility**
   - Not proven by compile-only validation. The U-Boot tree includes `cmd/pxe.o` and `cmd/sysboot.o` in the successful BL33 build, so extlinux support appears compiled in, but hardware boot/UART validation is still required.

5. **Bootloader write offset**
   - Not proven by compile-only validation. Keep sector-1 as the working assumption until verified against a Hardkernel-flashed image with `fdisk`/`dd`.

### Phase 1 Output Deliverables

- ✅ **Standalone BL33 U-Boot build** of `s7d_odroidc5_defconfig` on Ubuntu 24.04 container succeeds.
- ⏳ **Full FIP flashable image packaging** is partial/blocked at BL30 RTOS SDK setup.
- ⏳ **Boot log capture** from flashed U-Boot on hardware is still required.
- ⏳ **Hardkernel vendor-baseline capture** remains required for `fdisk`, DTB filename, and compatibility details.

---

## Placeholder Files

| File | Purpose |
|---|---|
| `0001-PLACEHOLDER-armbian-extlinux-support.patch.todo` | Phase 4: Stub for extlinux boot flow support (if Phase 1 shows standard extlinux incompatible) |
| `NOTES-bootloader-offset.md` | Bootloader sector offset discovery notes |

---

## Related Files

- **Draft FIP hook:** `config/sources/families/include/meson-s7d-fip-hook.inc.draft` — DRAFT post-process hook (not yet sourced) for Amlogic FIP packaging
- **Family config:** `config/sources/families/meson-s7d.conf` — S7D family declaration (sourcing this hook pending Phase 1 validation)
- **Board config:** `config/boards/odroidc5.wip` — ODROID-C5 board declaration

---

## Key Dependencies

- Phase 1: U-Boot standalone validation (`uboot-standalone-build`, `uboot-flash-test`)
- Phase 4: U-Boot integration into Armbian board flow + extlinux support
- U-Boot source: [hardkernel/u-boot](https://github.com/hardkernel/u-boot) branch `odroidc5-v2023.01`

---

## TBD Markers

- `<TBD:phase-1-uboot-validation>` — Unresolved FIP flow + extlinux compatibility questions
