# Touch-controller investigation — SM-A346E

## Confirmed device baseline
- Model: SM-A346E (Galaxy A34 5G)
- Firmware: A346EXXSGFZJ4
- Android: 16 / API 36
- Kernel: 6.6.92-android15-8-abA346EXXSGFZJ4-4k
- SoC: MediaTek MT6877 / Dimensity 1080
- Device tree codename: a34x

## Important distinction
The SoC is known; the touchscreen controller/IC is **not yet proven** for this exact handset and firmware. The panel vendor can vary by production batch or replacement screen. Do not label a controller (Goodix, FocalTech, Samsung, etc.) based only on another A34 device or generic driver folders.

## Evidence needed to identify it
1. Exact firmware's device-tree / vendor kernel source: touchscreen node, compatible string, I2C/SPI address, GPIO and regulator wiring.
2. Runtime device identification from the active touchscreen driver's probe/logs or sysfs, if safely accessible.
3. Cross-check driver name and firmware/config filenames against the device-tree node.
4. Record the controller revision and panel vendor where available.

## Driver settings: establish baseline before editing
We need capture the current stock driver's configuration and defaults, not blindly reset it:
- Driver and controller name / compatible string
- Firmware/config file name and panel ID
- Supported report rate / scan mode, if explicitly exposed
- Input event path and IRQ configuration
- Existing vendor tuning parameters (filter, sensitivity, edge/palm handling)
- Device-tree values and driver defaults, with source file + line references

## Proposed change discipline
- First document stock values.
- Make one isolated change at a time in a separate experimental branch.
- Compare source and measured touch latency/consistency.
- Do not flash or alter the phone in this phase.
- Keep a stock-source restoration patch/commit so any experimental change can be reverted exactly.

## OTA and bootloader constraint
The owner wants to retain official OTA updates and avoid unlocking/flashing. Therefore source research and analysis are the current scope. A custom kernel/driver installation is not an OTA-preserving path on a locked stock device.
