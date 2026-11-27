# A34X TouchLab

A research-only project investigating touchscreen input latency and touch-controller capabilities on the Samsung Galaxy A34 5G (SM-A346E).

## Current device baseline
- Model: SM-A346E / a34x
- Firmware: A346EXXSGFZJ4
- Android: 16 (SDK 36)
- Kernel reported by device: 6.6.92-android15-8_abA346EXXSGFZJ4-4K
- SoC: MediaTek Dimensity 1080 (MT6877)
- Bootloader binary: G / revision 16

## Goal
Determine, from matching source code and documented hardware behavior, whether the touch controller and driver support a higher touch report rate or lower input latency than the current configuration.

## Safety and scope
- This repository is for investigation and documentation first.
- No firmware, boot image, kernel, or device settings are modified by this project.
- No claim is made that software can exceed the touch controller's physical supported rate.
- A custom kernel/boot image may require bootloader unlocking and flashing, can erase data, affect Knox, interfere with OTA updates, or make the device unbootable. Do not install anything based only on these notes.
- Preserve stock firmware and OTA compatibility as the user's priority.

## Research stages
1. Identify the exact stock kernel source/build corresponding to the firmware.
2. Locate touchscreen controller, firmware, and input driver configuration.
3. Determine the controller's supported report rates and current driver limits.
4. Compare source evidence with measured touch latency, if safe measurement methods are available.
5. Only document feasible options and their trade-offs; no flashing instructions until exact source and risk are understood.

See [device baseline](docs/device-baseline.md), [research plan](docs/research-plan.md), [deep touch-driver audit](docs/deep-touch-driver-audit-2026-11-27.md), and [source findings](docs/source-findings-touch-driver.md).
