# Progress update — exact firmware and driver research

Date: 2026-11-27

## Confirmed release metadata
Samsung's device update page lists build `A346EXXSGFZJ4` for SM-A346E, Android 16, released 2026-10-19, security patch 2026-10-05.
Firmware metadata sources report:
- Changelist: 33505653
- Kernel: 6.6.92-android15-8-abA346EXXSGFZJ4-4k
- Device codename: a34x
- SoC: MT6877 / Dimensity 1080
- Bootloader binary: G / revision 16

Official Samsung update history:
https://doc.samsungmobile.com/SM-A346E/026626230407/eng.html

## Source inspection completed
The public community source tree `BS1388/Kernel_Samsung_a346E` was inspected at a high level. It contains Samsung `sec_input` integration and multiple MTK/vendor touchscreen families. Its config and candidate module directories do not prove which controller is bound on this stock build.

Searches in the connected GitHub code-search integration did not return exact file matches for the initial terms. This is a tooling/search limitation, not evidence that the driver files do not exist.

## Current answer: what is the touchscreen IC?
**Not conclusively identified yet.** Do not claim Goodix, FocalTech, Novatek, Ilitek, or Synaptics as the user's exact IC without an exact stock device-tree compatible string, driver match/probe, module/firmware association, or runtime binding evidence.

## What needs to happen next
1. Locate the official Samsung Open Source release package for SM-A346E changelist 33505653 / build FZJ4 (or a source package whose build/configuration demonstrably matches).
2. Inspect device-specific DT/DTBO and vendor touch modules/firmware to extract:
   - touchscreen compatible string and bus/address
   - selected driver and firmware/config name
   - stock panel tuning/profile
3. Read the selected driver and Samsung sec_input layer to build a source-linked inventory of:
   - scan/report rate tables and supported modes
   - filters, smoothing, debounce, jitter/edge/palm settings
   - IRQ/input boost/QoS and scheduling behavior
   - factory/sysfs/command interfaces and their permissions
   - default values, units, valid ranges, and runtime vs build-time vs firmware-owned classification
4. Only after this baseline is recorded, decide which parameter is technically useful for perceived latency and can be changed without fabricating an interface.

## Constraints
The user prioritizes official OTA, keeping data, and no bootloader unlock or flashing during research. Therefore this repository remains a source audit/measurement project. Do not generate or recommend an installable modified boot/kernel image at this stage.
