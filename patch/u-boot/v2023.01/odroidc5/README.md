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

## ⚠️ Research-Blocked: Phase 1 Validation Required

**DO NOT author patches until Phase 1 standalone U-Boot validation is complete.** The Amlogic multi-stage firmware (FIP) flow — BL2/BL30/BL31/BL32/BL33 + `aml_encrypt` + LibreELEC `amlogic-boot-fip` blobs — is not yet proven to integrate cleanly with Armbian's standard `post_uboot_custom_postprocess` hook flow.

### Phase 1 Validation Checklist

**Before any patch authoring, Phase 1 must answer:**

1. **defconfig discovery**
   - Does `odroidc5_defconfig` exist in `hardkernel/u-boot` `odroidc5-v2023.01` branch?
   - If not, which defconfig is the base (e.g., `meson64_defconfig`)?
   - Can Armbian's `BOOTCONFIG` variable point to it, or does it require a custom override patch?

2. **Final boot artifact format**
   - What is the **actual final U-Boot binary name** that gets written to sector 1 of the SD/eMMC?
   - Examples observed in Amlogic: `u-boot.bin.sd.bin`, `fip/build/u-boot.bin`, custom post-processed FIP package?
   - This name must match Armbian's `UBOOT_TARGET_MAP` string (e.g., `u-boot.bin.sd.bin:u-boot.bin`).

3. **Required blobs & tools**
   - Does the Hardkernel build require **LibreELEC's `amlogic-boot-fip`** prebuilt package (BL2/BL30 bins)?
   - Does it require **`aml_encrypt` binary** (Amlogic proprietary encryption tool)?
   - Can these be sourced as part of the U-Boot source tree, or are they external?
   - Does Armbian's build pipeline have precedent for integrating them?

4. **Boot flow compatibility**
   - Does `odroidc5-v2023.01` expect to load a **custom `boot.scr`** (Hardkernel proprietary), or can Armbian's standard **extlinux + `/boot/extlinux/extlinux.conf`** boot flow work?
   - If extlinux does NOT work, what custom bootscript is required, and can it be authored as a patch?

5. **Bootloader write offset**
   - On official ODROID-C5 images, at what **sector** does BL2 (SPL) start?
   - Expected: **sector 1** (not 8192). Verify with `fdisk -l /dev/mmcblk0` on a Hardkernel-flashed image.
   - This is critical for `dd` operations during CI/CD and affects `UBOOT_TARGET_MAP`.

### Phase 1 Output Deliverables

Phase 1 must produce:
- ✅ **Standalone U-Boot build** of `odroidc5-v2023.01` on Ubuntu container (outside Armbian) — prove it compiles and boots.
- ✅ **Boot log capture** from flashed U-Boot on hardware — verify prompt + `printenv` output.
- ✅ **Hardkernel vendor-baseline capture** — full UART boot log, `fdisk -l`, DTB filename, `compatible` string.
- ✅ **Decision memo** answering all 5 questions above, with evidence (URLs, code snippets, test results).

Only after Phase 1 completes can patches here be authored with confidence.

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
