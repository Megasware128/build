# Kernel Patches for Amlogic S7D (S905X5M) — Track A — ODROID-C5 BSP
#
# This directory holds kernel patches for Track A (BSP kernel via DannieDarko/odroid-c5_linux_common_drivers).
#
# Patch Workflow
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#
# To add or modify patches, use the armbian-board-patches skill commands:
#
#   ./compile.sh kernel-patch BOARD=odroidc5 BRANCH=current
#
# This opens an interactive session where:
#   1. All existing patches are pre-applied to cache/sources/linux-*/
#   2. You make changes (edit DTS, drivers, etc.)
#   3. git add + git commit per logical change
#   4. Exit the session — patches are exported to output/patch/ and userpatches/
#   5. Move patches to patch/kernel/archive/meson-s7d-6.6/ for upstream inclusion
#
# Naming convention:
#   NNNN-<short-kebab-subject>.patch
#   • NNNN: 4-digit order prefix (0001, 0005, 0010, etc. — leave gaps for future inserts)
#   • Subject: lowercase kebab-case, max ~50 chars
#   • Format: standard git format-patch output
#
# When rebasing on new kernel version:
#   ./compile.sh rewrite-kernel-patches BOARD=odroidc5 BRANCH=current
#   (Resolves merge conflicts, re-exports coherent series)
#
# Expected Patches (Track A BSP)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#
# After Phase 0 vendor-baseline capture, patches will be added for:
#   • DTS overrides: ODROID-C5 board-specific nodes (USB PHY tuning, GPIO pinmux tweaks, etc.)
#   • Driver fixes: S7D-specific register offset/layout differences vs. similar S9x5 variants
#   • Config: Symbols enabled/disabled for BSP stability (if not already in defconfig)
#
# Once Phase 5 first build completes and UART boot confirms what works/fails:
#   • Additional driver patches for failed peripherals (MMC timing, Ethernet PHY init, etc.)
#
# Sources for patches:
#   • Hardkernel BSP repo: DannieDarko/odroid-c5_linux_common_drivers
#   • Existing Armbian meson64/meson-sm1 patches (may apply with minor changes)
#   • Amlogic public driver fixes (amlogic/linux tree)
#
# Disabled/Override Patches
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#
# To disable an upstream patch that conflicts with BSP requirements:
#   • Create an empty file with the same basename (e.g., 0xxx-original-patch-name.patch.disabled)
#   • Add a comment in a sibling README or commit explaining why
#   • Patch manager skips .disabled files
#
# See armbian-board-patches SKILL § 2 (Override / disable rules)
#
