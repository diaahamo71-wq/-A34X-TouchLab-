# Touch-driver source matching — research checkpoint (2026-11-27)

## Device target

- Model: Samsung Galaxy SM-A346E
- Firmware: A346EXXSGFZJ4 (Android 16 / One UI 8.5)
- Kernel reported by device: 6.6.92-android15-8_abA346EXXSGFZJ4-4K
- Build changelist: 33505653
- Goal: source-audit the active touch controller and its real configuration path, without changing the phone, unlocking bootloader, rooting, or flashing.

## Findings in this pass

1. The public community source tree `BS1388/Kernel_Samsung_a346E` is a useful code reference, but its public identity alone does **not** prove it is an exact source match for FZJ4. Earlier tree inspection showed Samsung `sec_input` integration and multiple candidate MediaTek touch-driver families. Candidate families must not be treated as the active controller until matched to the target firmware.
2. Public kernel release history found during this pass includes an A346B source release based on firmware `A346BXXSFEZC7`, kernel 6.6.143. This is a different model suffix and a different kernel version from this device's 6.6.92; it cannot establish the A346E/FZJ4 touch driver's exact defaults. See the release listing: https://github.com/Fede2782/kernel_samsung_a34x/releases
3. A TWRP device-tree page points to Samsung's source portal using an older identifier `A346BXXUBEYI7`; that is a discovery lead, not proof of the FZJ4 package. https://github.com/TeamWin/android_device_samsung_a34x
4. Web searches for the exact FZJ4/changelist source package did not return a verifiable official Samsung OSS package or matching source commit in this pass. This is **not** proof that Samsung has not published it; exact package identification remains open.
5. GitHub repository searches scoped to the community source for exact strings around FZJ4, changelist 33505653, and candidate touchscreen compatibles returned no indexed hits. Search-index limitations mean this cannot be interpreted as absence from the source tree.

## Required evidence before calling the controller identified

- Exact source release/build identity matching SM-A346E and FZJ4 (or a documented, auditable source equivalence).
- Device-tree/panel configuration or vendor integration tying the exact board/panel to a controller compatible and driver.
- Controller firmware/configuration data and panel/vendor IDs, where published.
- Traceable driver path from probe/compatible through Samsung sec_input to input reporting.

## Audit targets once exact source is obtained

- Report/scan-rate implementation and supported modes (not merely screen refresh rate).
- Sampling/interrupt path, I2C/SPI details, input event/report path.
- Filtering, smoothing, debounce, jitter, palm/edge rejection and thresholds.
- Input boost, CPU/QoS scheduling and latency hooks.
- Samsung sec_input commands/sysfs/factory interfaces.
- For every setting: default, units, range, runtime writable vs build-time vs firmware/hardware fixed, and evidence file/line.

## Safety and project boundary

No device-side changes, root commands, bootloader unlock, flashing, or generated installable kernel. Preserve the user's stated goal of retaining official OTA and avoiding data loss. Any future experimental change must first be documented with a rollback/OTA impact assessment and explicit user approval.

## Next research action

Find and verify the Samsung OSS source release by model/build/changelist, then inspect its source archive metadata and relevant touchscreen/device-tree files. If only a nearby release is available, document the exact delta and keep all conclusions marked provisional.
