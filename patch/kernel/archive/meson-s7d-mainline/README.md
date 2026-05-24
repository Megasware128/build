# Mainline Kernel Patches: Amlogic S7D / ODROID-C5

## Purpose

This directory holds device-tree and driver patches intended for **upstream mainline Linux kernel** (`linux.git` torvalds).

As of mainline v6.16, S7D (Amlogic S905X5M) support is minimal:
- ✅ SoC stub: `arch/arm64/boot/dts/amlogic/amlogic-s7d.dtsi` — CPUs, GIC-400, PSCI, UART
- ✅ Reference board: `arch/arm64/boot/dts/amlogic/amlogic-s7d-s905x5m-bm202.dts` (Amlogic BM202, not ODROID-C5)
- ❌ No HDMI, USB, eMMC, Ethernet, GPU support in mainline yet

This port brings:
1. **ODROID-C5 board-specific DTS** — device tree for the Hardkernel board
2. **SoC peripheral expansion** — clock, reset, MMC, USB, GbE MAC, PHY nodes in SoC DTS
3. **Driver deltas** — S7D-specific tweaks to existing mainline drivers (meson-mmc, dwc2/dwc3, stmmac, etc.)

## Patch Strategy

Patches follow standard Linux kernel submission format (`git format-patch`):
- One logical change per file
- `[PATCH]` subject line with subsystem prefix (e.g., `[PATCH] arm64: dts: amlogic:`)
- `Signed-off-by:` trailer (or `Co-authored-by:` if Copilot-assisted)
- Descriptive commit body explaining the _why_, not just the _what_

## Phases and TBD Dependencies

### Phase 3a — ODROID-C5 Board DTS (skeleton)
**File:** `0001-arm64-dts-amlogic-add-odroid-c5-board.patch`

Initial board DTS with UART and memory only. Baseline values currently use Hardkernel vendor sources because the official image download was blocked by Cloudflare during image-only capture:
- BSP top-level `compatible`: `"s7d_s905x5m_bm201"`
- BSP DT memory reservation: `linux,usable-memory = <0x0 0x0 0x0 0xf0000000>`
- SoC stub (`amlogic-s7d.dtsi`) already in mainline

### Phase 3b — SoC Peripheral Expansion
**File:** `0002-arm64-dts-amlogic-s7d-add-soc-peripherals.patch.todo`

Extends `amlogic-s7d.dtsi` with:
- Clock controller node (CLKC) — inherited/ported from S905X3 or BSP
- Reset controller node (RESETC)
- Pin control (PINCTRL) nodes — GPIO groups for UART, MMC, USB, Eth, etc.
- MMC controller nodes (eMMC + SD slot)
- USB host (dwc2/dwc3) and PHY nodes
- Gigabit Ethernet MAC (stmmac) node
- Power domains / regulators (if SoC-level)

Depends on: Phase 3a UART working, vendor-baseline clock tree documented.

### Phase 3c — Board-Level Peripherals
**File:** `0003-arm64-dts-odroid-c5-add-board-peripherals.patch.todo`

Extends the board DTS with:
- eMMC and SD card slot nodes (phandles to SoC MMC controller)
- USB hub and device nodes (GPIO power enable, PHY config)
- Gigabit PHY address, interrupt, LED config
- GPIO LEDs (power, activity, if any)
- Audio codec (if on-board; C5 has no audio codec, but included for completeness)
- Thermal sensor / throttling config (SoC has thermal sensor)

Depends on: Phase 3b SoC nodes; driver support in mainline for those controllers.

## Submission and Rebase

1. **Per-patch series:** Each logical feature (e.g., "arm64: dts: amlogic: add MMC support") is a separate series submitted to `linux-amlogic@lists.infradead.org`.
   - Example: `git send-email --compose --to=linux-amlogic@lists.infradead.org 0001-*.patch`

2. **Rebase:** As mainline kernel versions advance (e.g., v6.17, v6.18), patches are rebased against the new tag:
   - `git rebase --onto <new-tag> <old-tag>`
   - Run `./compile.sh` with `KERNELBRANCH='tag:<new-tag>'` to validate

3. **Upstream merge:** Once patches are accepted by linux-amlogic maintainer, they appear in next mainline merge window. At that point:
   - Patches can be removed from this directory
   - `KERNELBRANCH` in `meson-s7d-mainline.conf` is bumped to the kernel version containing the merged patch

## BSP Reference

BSP DTS files (source of ported nodes) are located in:
- **Vendor BSP (Amlogic):** `DannieDarko/odroid-c5_linux_common_drivers` (mirror of proprietary Amlogic BSP)
  - S7D SoC DTS: `arch/arm64/boot/dts/amlogic/meson-s7d.dtsi`
  - ODROID-C5 board DTS: `arch/arm64/boot/dts/amlogic/s7d_s905x5m_odroidc5.dts`
  - Device tree overlays: `arch/arm64/boot/dts/amlogic/overlays/odroidc5/`

When porting DTS nodes:
1. Check if mainline already has a similar node (e.g., `amlogic-g12-common.dtsi` for other SoCs)
2. Use mainline naming conventions (e.g., `compatible = "amlogic,s7d-mmc"` not `meson_mmc`)
3. Reference mainline binding documentation: `Documentation/devicetree/bindings/mmc/`, etc.

## Driver Deltas

If mainline drivers require S7D-specific tweaks (e.g., `drivers/mmc/host/meson.c`, `drivers/net/ethernet/stmicro/stmmac/`):
- Create a patch file: `0010-drivers-mmc-meson-add-s7d-support.patch` (separate from DTS patches)
- Number in sequence (after DTS patches)
- Include:
  - The exact register offsets / bits that differ from S905X3
  - Conditional logic: `if (IS_ENABLED(CONFIG_MESON_S7D)) { ... }`
  - or: `if (of_device_is_compatible(node, "amlogic,s7d-mmc")) { ... }`

## Testing

Each patch is tested via:
1. **Mainline build:** Kernel compiles cleanly with the patch applied
2. **ODROID-C5 image build:** `./compile.sh BOARD=odroidc5-mainline RELEASE=noble BRANCH=current` succeeds
3. **Hardware validation:** Boot on physical ODROID-C5, verify peripheral works (MMC mounts, USB detected, Eth gets IP, etc.)
4. **`make dtbs_check`:** Validate DTS against bindings (optional but recommended)

## Notes for Maintainers / Reviewers

- **Compatibility strings:** All `compatible` entries must match upstream bindings in `Documentation/devicetree/bindings/`. Avoid BSP-specific strings.
- **Clock framework:** Use `CLK_OF_DECLARE` and `CLK_OF_DECLARE_DRIVER` for clock controllers; reference `drivers/clk/meson/` for examples.
- **Reset controller:** Use `RESET_CONTROLLER` framework; see `drivers/reset/` and similar SoCs (S905X3, etc.).
- **Pinctrl:** Reference `drivers/pinctrl/meson/` for pinctrl groups and function names.
- **Licensing:** All new DTS files must include `SPDX-License-Identifier: (GPL-2.0+ OR MIT)` header.

---

## Related Tracks

- **Track A (BSP):** Uses vendor Amlogic GKI kernel v6.6 with full BSP DTS/drivers; faster hardware bring-up, slower to upstream.
- **Track B (Mainline):** This track; slower hardware bring-up (DTS/driver porting), faster path to upstream sustainability.

Both tracks run in parallel. Learnings from Track A (pin mappings, clocking, power domains) inform Track B's DTS authoring.
