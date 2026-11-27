# A34x Device Tree cross-check: r00 vs r05 — 2026-11-27

Repository: `Fede2782/android_kernel_samsung_a34x`
Commit: `19383cb991675100010b338fc55aa1c64fc08d8b`

Inspected:
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/input_berlin.dtsi`
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/a34x/a34x_eur_open_w00_r05.dts`
- `kernel/kernel_device_modules-6.6/arch/arm64/boot/dts/samsung/a34x/a34x_eur_open_w00_r00.dts`

Both r00 and r05 A34x DTS variants explicitly define:
- touchscreen node `touchscreen@5d`
- `sec,refresh_rate_enable`
- `sec,max_coords = <1080 2340>`
- `sec,firmware_name = "tsp_goodix/gt9895_a34x.bin"`
- `sec,ic_type = <2>; //GT9895`
- `sec,max_drv_num = <17>`
- `sec,max_sen_num = <35>`
- `goodix,edgehandler_direction_max = <5>`

The shared Samsung `input_berlin.dtsi` carries the same core settings. This makes the GT9895 match stronger for the A34x source family. The installed device's actual DTB/DTBO has not yet been extracted/read, so the exact active revision and exact FZJ4 match remain unproven.

The source build manifest `kernel/kernel_device_modules-6.6/.legofile` references the firmware source path:
`kernel/kernel_device_modules-6.6/firmware/tsp_goodix/gt9895_a34x.bin`
and maps it from the Goodix Berlin module firmware directory. This is a source/build-manifest reference, not proof that the firmware binary version is the one currently installed on the user's device.

## Performance implications
The A34x DT variants explicitly enable the driver's refresh-rate-mode support. The Goodix driver source then exposes mode values 0/1/2 and sends command 0x9D. This means the capability is configured in the source-level device tree. It does not prove which mode is currently selected on the live device or the effective physical sampling frequency at runtime.

The same DTS variants set edge-handler direction maximum to 5. Do not infer the meaning of each direction value without the Samsung grip-data caller and controller firmware protocol.

## Next validation gates
1. Match the user's active DTB/DTBO to r00/r05 or another exact source variant.
2. Match the exact FZJ4 source release/commit and firmware version.
3. Establish an authorized diagnostic path to read chip/firmware info and current controller mode.
4. Measure effective scan cadence and touch-to-input latency before changing any behavior.
