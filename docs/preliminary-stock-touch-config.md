# Preliminary stock touchscreen configuration (source-level)

> Status: strong source match for the A34X 1080×2340 variant, but not yet proven byte-for-byte against the owner's exact A346EXXSGFZJ4 kernel/DTB. Keep the word **preliminary** until the exact firmware DTB/source is matched.

## Controller identification
A public A346E kernel source branch contains this device-tree configuration:

`kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/mediatek/k6877ttv1_64_titan_black_tiger/cust_mt6877_touch_gt9895_1080_2340.dtsi`

It declares:
- Device node: `touchscreen@5d`
- Bus: I²C controller `i2c0`
- Address: `0x5d`
- Compatible: `goodix,berlin`
- Samsung controller type: `sec,ic_type = <2>`, driver maps 2 to **GT9895**
- Driver family: Goodix Berlin (Samsung-sec integrated)
- Screen coordinates: `1080 × 2340`
- Firmware/config name: `A34_GT9895_03040002_Combined_20220902_updated.BIN`
- Device-tree enables `sec,refresh_rate_enable` and `sec,enable_esd_check`
- IRQ is falling-edge + one-shot; GPIO 14 in this source variant.

The source tree also contains a matching named firmware blob:
`firmware/tsp_goodix/gt9895_a34x.bin`

## Existing driver control we found
In `drivers/input/touchscreen/goodix/berlin/goodix_ts_cmd.c`, Samsung's `refresh_rate_mode` command accepts:
- `0`: normal (60 Hz)
- `1`: adaptive
- `2`: always high refresh (90/120 Hz according to display mode)

The driver sends command `0x9D` with the selected mode. This is a **display/refresh operating mode control**, not proof that the controller can be forced to a higher-than-supported touch sampling rate. Do not confuse 120 Hz display refresh with touch report rate.

The controller's firmware reports its supported `active_scan_rate` list in IC information. Those actual supported values need to be extracted from the exact stock firmware/config or a read-only runtime report before choosing a change.

## Stock baseline to preserve
- Exact DT values above (record source path and revision).
- GT9895 firmware/config and panel ID.
- Firmware-defined scan-rate list and controller limits.
- Filtering, sensitivity, edge/palm/grip and noise/ESD configuration.
- IRQ wiring/trigger and bus settings.
- All vendor defaults; do not replace with generic Goodix values.

## Candidate research directions (not modifications yet)
1. Trace the GT9895 firmware's advertised scan rates and which mode selects each.
2. Trace whether stock kernel sets touch boost on input events and whether scheduling/IRQ latency, rather than controller scan, is the user's perceived bottleneck.
3. Compare report timestamp cadence under actual movement and separate it from frame rate.
4. Only then prepare a reversible source patch. No live device writes, firmware flashing, kernel flashing, bootloader unlock, or OTA-affecting installation in this phase.
