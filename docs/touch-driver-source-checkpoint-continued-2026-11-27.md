# Touch-driver source checkpoint — 2026-11-27 (continued)

## Current exact firmware target
- Device: Samsung Galaxy A34 5G, SM-A346E
- Build: A346EXXSGFZJ4
- Android: 16 / API 36; One UI 8.5
- Samsung update history lists this build as released 2026-10-19, patch 2026-10-05.
- Firmware metadata lists changelist 33505653 and kernel string 6.6.92-android15-8-abA346EXXSGFZJ4-4k.

Official update history:
https://doc.samsungmobile.com/SM-A346E/026590230406/eng.html

## Source acquisition status
A targeted web search for the exact model/build/changelist did not surface a verifiable Samsung Open Source kernel package for FZJ4. Search results repeatedly lead to older or different model source releases. This is a search outcome, not proof that Samsung has not published the source.

The public repository https://github.com/BS1388/Kernel_Samsung_a346E is a useful A346E research tree, but GitHub code-search queries for the exact FZJ4 string and touchscreen-compatible bindings returned no matches through the connected search interface. That interface may not index all files; absence of a search hit is not evidence of absence in the repository.

A separate A346B kernel release is explicitly tagged for SM-A346B and must not be used as exact A346E/FZJ4 driver proof:
https://github.com/Fede2782/kernel_samsung_a34x/releases

## What is and is not proven
**Not proven:** active touch IC, exact device-tree binding, firmware filename, stock touch tuning, or writable runtime controls for this exact firmware.

**Proven:** the target build identity; A346E community source contains Samsung sec_input and multiple MTK touch-driver candidates; these candidates require matching through device tree / module / firmware / probe data before attribution.

## Next source-audit route
1. Obtain a Samsung OSS source archive matching SM-A346E and preferably the FZJ4/changelist, or find a source tag with demonstrable matching config and device-specific files.
2. Inspect the device-specific DTS/DTBO and kernel module metadata for the actual touchscreen node, compatible string, I2C/SPI address, interrupt GPIO, regulator/power sequence, and selected driver.
3. Trace that compatible string to the driver's probe and firmware/config load path.
4. Then inventory source-defined parameters and classify each:
   - runtime exposed and writable (document exact interface and permission)
   - runtime exposed read-only
   - compile-time/configuration only
   - firmware/config blob controlled
   - hardware fixed / unsupported
5. Only after identification, discuss safe, measurable latency experiments. Do not fabricate sysfs paths or issue generic tuning commands.

## User constraints
No bootloader unlock, flashing, root modification, or data wipe in this research phase. Preserve official OTA compatibility as a priority. Do not generate an installable kernel/boot image.
