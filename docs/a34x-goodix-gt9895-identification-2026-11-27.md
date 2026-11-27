# A34x Goodix controller identification — 2026-11-27

## Strongest identification found
The device reports:
- I2C device `51-005d`
- bound driver `goodix_i2c`
- module `goodix_ts_berlin`
- Samsung sysfs path `/sys/devices/platform/goodix_ts.0`
- input touchscreen dimensions 1080 x 2340
- 10 multitouch slots
- Device Tree node `touchscreen@5d`

The A34x kernel source repository `Fede2782/android_kernel_samsung_a34x`, commit `19383cb991675100010b338fc55aa1c64fc08d8b`, contains A34x-specific DT sources:
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/input_berlin.dtsi`
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/a34x/a34x_eur_open_w00_r05.dts`
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/a34x/a34x_eur_open_w00_r00.dts`

The shared `input_berlin.dtsi` defines:
- node `touchscreen@5d`, `compatible = "goodix,berlin"`, `reg = <0x5d>`
- `sec,max_coords = <1080 2340>`
- `sec,firmware_name = "tsp_goodix/gt9895_a34x.bin"`
- `sec,ic_type = <2>; //GT9895`
- `sec,max_drv_num = <17>`
- `sec,max_sen_num = <35>`
- `sec,refresh_rate_enable`
- command/register configuration for the A34x panel.

This matches the user's observed address, platform sysfs node, dimensions, Goodix Berlin driver family, and exposed Device Tree property names. Therefore **GT9895 is now a high-confidence candidate / likely controller identity for this A34x configuration**, much stronger than the previous generic Berlin-family identification. It is not yet a direct read from the user's live DT because property reads are blocked, and the exact installed FZJ4 DTB/DTBO has not been extracted and compared.

## Independent source cross-check
The Goodix Berlin source files in:
- `BS1388/Kernel_Samsung_a346E`, branch `A346E`
- `Fede2782/android_kernel_samsung_a34x`, commit `19383cb991675100010b338fc55aa1c64fc08d8b`

have identical Git blob hashes:
- `goodix_ts_core.c`: `f93425ea073435810bda62436ef2009591de8237`
- `goodix_ts_cmd.c`: `aed1423710bce4f567a66396a7b0b354fc805ac0`

This independently confirms that both community repositories contain the same relevant Goodix implementation. It does NOT prove that either source is the exact source release for installed FZJ4 firmware.

## Verified capabilities from source
- `chip_info` is declared mode `0444` and reads ROM/patch PID/VID, sensor ID, config ID/version, and firmware checksum.
- `refresh_rate_mode` is gated by DT boolean `sec,refresh_rate_enable`; code sends Goodix command `0x9D` with selected mode (0 normal 60 Hz, 1 adaptive, 2 always 90/120 Hz as described by source comments). This is a touch-controller scan-mode command, not merely Android display refresh setting.
- `set_game_mode` sends command `0xC2`; source does not document internal behavior, so do not claim a specific latency/sampling improvement without measurement.
- `low_sensitivity_mode_enable` is low-sensitivity mode, not a sensitivity boost.

## SELinux conclusion
The user's `chip_info` has mode 0444 and generic label `u:object_r:sysfs:s0`, but ADB shell domain `u:r:shell:s0` receives Permission denied. SELinux is the leading explanation, not yet proven without an AVC record. Do not make SELinux permissive globally. Any future custom policy must narrowly expose a dedicated read-only diagnostic type/domain and must remain enforcing.

## Compatibility and installation gate
The installed firmware is `A346EXXSGFZJ4`, Android 16, kernel `6.6.92-android15-8_abA346EXXSGFZJ4-4K`, changelist `33505653`. Public firmware metadata identifies these values, but the inspected community source commit has not been proven to be the exact source tag/commit for this FZJ4 release.

Do not build or flash a modified kernel/DTBO until exact source compatibility, config, module ABI, DTB/DTBO selection, vendor module set, AVB/bootloader unlock state, backup, and recovery path are established.

## Next engineering tasks
1. Inspect A34x DT source revisions r00/r05 and confirm which one the target FZJ4 device tree corresponds to.
2. Identify exact firmware binary `tsp_goodix/gt9895_a34x.bin` and compare its version/checksum only from a matching source package; do not download huge firmware bundles unnecessarily.
3. Analyze scan-rate and game-mode command flow and available runtime measurement methods.
4. Identify exact Samsung vendor SELinux policy source and labels.
5. Build a read-only diagnostic change first; keep SELinux enforcing and leave all write controls unchanged.
