# Touchscreen latency research plan

## Questions to answer
- What touchscreen controller is fitted in this specific SM-A346E hardware revision?
- Which driver and configuration are used by the exact A346EXXSGFZJ4 stock kernel?
- Does the controller expose selectable scan/report rates, and what rates are documented by its datasheet or driver?
- Is the current rate limited by controller firmware, device-tree configuration, driver code, display mode, or another system layer?
- Does changing the rate improve actual end-to-end touch latency, or merely increase event frequency / CPU load?

## Evidence requirements
For every conclusion, record:
- Source URL and exact branch/commit
- File path and relevant symbol/configuration
- Whether the source matches firmware build and device variant
- Confidence: confirmed / probable / unknown

## Safe order of work
1. Examine public source repositories and Samsung's published source package metadata.
2. Cross-check device variant, kernel base, and firmware revision.
3. Read driver and device-tree code; do not alter it yet.
4. Establish hardware-supported limits from reliable documentation.
5. Separate software-adjustable settings from physical controller limits.
6. Assess OTA, bootloader, data-wipe, and Knox consequences before discussing any modification.

## Explicitly out of scope for now
- Flashing kernels, boot images, or recovery images
- Bootloader unlocking
- Disabling OTA protections
- Promising a higher polling rate without source/hardware evidence
