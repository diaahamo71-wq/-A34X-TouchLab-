# GT9895 touch pipeline and performance modification plan — 2026-11-27

Target reported by user:
- Samsung SM-A346E, firmware A346EXXSGFZJ4, Android 16
- kernel release 6.6.92-android15-8_abA346EXXSGFZJ4-4K
- live I2C device 51-005d, driver goodix_i2c, module goodix_ts_berlin
- source candidate: BS1388/Kernel_Samsung_a346E branch A346E
- cross-reference source candidate: Fede2782/android_kernel_samsung_a34x commit 19383cb991675100010b338fc55aa1c64fc08d8b

## GT9895 identification refinement
The A34x Device Tree source `input_berlin.dtsi` sets `sec,ic_type = <2>`; the Goodix core source maps ic_type 2 to GT9895 and logs "ic_type is GT9895". The DTS also identifies `tsp_goodix/gt9895_a34x.bin`, max coordinates 1080x2340, 17 drive channels and 35 sense channels. This is a strong model identification for the A34x source configuration, but the live FZJ4 DT property values have not yet been read directly.

Goodix's upstream Berlin porting guide classifies GT9895 under the Nottingham family. This is consistent with the Samsung source's explicit GT9895 mapping; do not confuse the family name with another chip model.

## Verified controller scan-rate control
In `goodix_ts_cmd.c`:
- `refresh_rate_mode` accepts 0..2.
- Source comment: 0 normal (60Hz), 1 adaptive, 2 always (90Hz/120Hz).
- `set_refresh_rate_mode` sends command `0x9D`, with data byte 0 equal to the selected mode.
- The command is rejected if the Device Tree boolean `sec,refresh_rate_enable` is absent/false.
- Core parses `sec,refresh_rate_enable` with `of_property_read_bool`.
- The driver log says "set scan rate".

Engineering interpretation: this is a touch-controller scan-mode request, not simply Android display refresh rate. The source comments do not establish that the controller is sampling at 120 Hz in every mode or state. Exact effective scan rate and interaction with display mode/firmware require runtime measurement. No blind write to the factory command node is appropriate on the current non-root device.

## Game mode control
In `goodix_ts_cmd.c`, `set_game_mode`:
- accepts 0 or 1 only;
- sends command `0xC2` with data byte 0 set to 0/1;
- reports OK if the hardware send operation succeeds.
The source does not describe the internal firmware behavior or guarantee reduced latency, higher scan rate, altered filtering, or better sensitivity. Treat it as a vendor-defined game-mode switch whose effect must be measured, not assumed.

## Interrupt/event pipeline
In `goodix_ts_core.c`:
- The driver registers a threaded IRQ using `devm_request_threaded_irq`, with `goodix_ts_threadirq_func` as the threaded handler.
- The thread increments an IRQ trigger counter, handles suspend/LPM cases, then calls `hw_ops->event_handler(core_data, ts_event)` to read touch data from the controller.
- It handles an empty event by releasing all fingers and clears the event type.
- The core's IRQ handler itself does not directly show a generic coordinate filter or call Android input reporting in the inspected excerpt; lower hardware/event and Samsung sec_input layers need inspection before modifying filtering or report timing.

Engineering implication: blindly raising IRQ priority, bypassing suspend/LPM handling, or removing synchronization is not justified. A safe performance experiment must first measure interrupt-to-input latency, event rate, missed/empty frames, and CPU scheduling under load.

## Edge/grip configuration
The source sends command `0x68` for edge-handler and normal grip configuration. It serializes edge direction and Y-range or edge/dead-zone values from Samsung platform grip data. This is an edge rejection / grip configuration path, not a global touch sampling frequency control. Do not remove edge rejection globally: it may cause accidental side touches during normal holding.

## Recommended sequence
1. Establish exact FZJ4 source compatibility and DT/DTBO variant.
2. Identify the exact firmware revision/checksum via matching `chip_info` or source package; do not update firmware.
3. Build a read-only diagnostic interface (or a narrow SELinux rule for one dedicated diagnostic label) while keeping SELinux enforcing.
4. Measure baseline touch latency and effective scan mode.
5. Compare normal/adaptive/always scan modes only through a supported authorized service path on a recoverable test build; record power, temperature, dropped frames, and latency.
6. Test game mode independently and keep only a change that demonstrates a repeatable improvement.
7. Only then consider a kernel patch. Preserve the existing interrupt, ESD, suspend/resume and input reporting paths unless measurements demonstrate a specific bottleneck.

## Explicit non-actions
- No sysfs permission bypass, SELinux permissive mode, blind register writes, firmware flashing, or driver unbind/rebind.
- No kernel/DTBO installation until source version, config, module ABI, AVB state, backups and recovery route are verified.
