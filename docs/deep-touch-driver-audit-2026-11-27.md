# Deep touch-driver audit — 2026-11-27

## User target
The owner wants the deepest practical investigation of touchscreen behavior on SM-A346E, beyond display refresh rate and ordinary Android toggles. Preserve stock OTA/locked-bootloader status and user data during research; do not flash or change the handset in this phase.

## Exact reported device baseline
- Model: SM-A346E / a34x
- Firmware: A346EXXSGFZJ4
- Android: 16, API 36
- Kernel release: 6.6.92-android15-8_abA346EXXSGFZJ4-4K
- SoC: MediaTek MT6877 / Dimensity 1080

## What was inspected in the public community kernel tree
Repository: https://github.com/BS1388/Kernel_Samsung_a346E
Branch: A346E

The source contains Samsung input framework files under:
`kernel/kernel_device_modules-6.6/drivers/input/sec_input/`
including `sec_input.h`, `sec_common_fn.c`, `sec_cmd.c`, and secure-touch support.

The source also contains several candidate MTK/vendor touchscreen families under:
`kernel/kernel_device_modules-6.6/drivers/input/touchscreen/`
including GT-family, ILITEK, FocalTech, Novatek/NT36672C, and Synaptics TCM2 variants.

The checked-in kernel config lists a generic MTK touch framework selection:
`CONFIG_TOUCHSCREEN_MTK_TOUCH="GT9895 GT9886 GT9896S GT1151 NT36672C_I2C legacy_gt9896s"`
and mentions external module source paths including FT3518U, GT9886, GT9916, NT36672C, ST61Y and Synaptics TCM.

**This is not sufficient to identify the controller installed in this exact phone.** The config is a community-source build configuration, not proven to be the active stock FZJ4 device-tree/module selection. Multiple driver families in one source tree are candidates, not evidence that the handset uses all or any one of them.

## Current conclusion
- The SoC and platform are identified; the actual touchscreen IC / panel variant and active bound driver are **not yet confirmed**.
- Do not state a specific controller brand/model until the exact FZJ4 device tree, vendor module metadata/firmware, or reliable runtime binding evidence is obtained.
- The correct next source is Samsung Open Source package matching SM-A346E and the exact FZJ4 build/kernel, plus its device-specific DT/vendor touch modules. The public community repository is a useful navigation/reference tree but cannot by itself certify exact stock defaults.

## Deep driver baseline to extract once exact source is matched
Create a source-linked table with:
1. Controller identification: compatible string, bus (I2C/SPI), address, panel/vendor ID, driver probe/match entry.
2. Current stock configuration: device-tree properties, driver compile-time defaults, controller firmware/config blob, panel tuning profile.
3. Touch acquisition: scan/report-rate tables, available controller modes, IRQ/poll path, packet/report format. Record maximums only where source or vendor documentation explicitly proves them.
4. Input processing: filtering, smoothing, debounce, jitter rejection, coordinate interpolation, palm/edge rejection, glove/wet mode.
5. Latency/performance: input boost/QoS, IRQ/thread scheduling, CPU affinity or frequency requests if present, bus and power constraints.
6. Samsung integration: sec_input sysfs/commands, factory test controls, charger/cover/low-power modes, secure touch and calibration/TCLM paths.
7. Each parameter's interface, units, valid range, stock value, evidence location, and whether runtime-writable, build-time-only, firmware-owned, or hardware-fixed.

## Development method
- Preserve an untouched upstream snapshot and record commit SHA.
- Work only in a separate research branch; make a reversible patch after the stock baseline is documented.
- Do not fabricate sysfs names or write values based on generic guides.
- Do not change the phone, unlock the bootloader, flash, or create a supposedly installable image during this audit.
- OTA preservation is a requirement; kernel/boot modifications would need separate explicit discussion because they can affect OTA, Knox, boot integrity, and data.

## Immediate next milestone
Obtain and inspect the exact Samsung Open Source FZJ4 package and its A346E device-specific touch DT/module files. Then identify the controller conclusively and populate the baseline table before proposing any tuning.
