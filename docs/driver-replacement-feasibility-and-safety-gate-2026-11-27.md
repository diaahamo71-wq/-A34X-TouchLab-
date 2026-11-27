# Driver replacement request — feasibility and safety gate (2026-11-27)

## User's current goal
The user now wants to replace the stock touchscreen driver on their Samsung Galaxy A34 5G (SM-A346E, Android 16, build A346EXXSGFZJ4, kernel 6.6.92-android15-8-abA346EXXSGFZJ4-4k) with a modified driver, rather than only adjusting display refresh rate or ordinary touch settings.

## Technical reality
A touchscreen driver is normally built into the device kernel or supplied as a kernel module. It is not a standalone Android app and cannot safely be swapped with an unrelated driver. The replacement must match:
- exact touch controller silicon and panel variant
- bus/protocol and device-tree binding
- kernel ABI/configuration and vendor integration
- firmware/config blob expected by the controller
- Samsung sec_input framework and interrupt/power/reset wiring

The public A346E kernel tree contains candidate MTK touch families and Samsung sec_input, but this does not prove the active controller or compatibility with FZJ4. The exact active IC and exact FZJ4 source are still unconfirmed.

## Repository research lead
The A34x proprietary vendor repository exposes an SM-A346E_OJM branch/tree for firmware binaries, but the README does not establish that its contents correspond to FZJ4. GitHub code search did not surface exact FZJ4 or touch-firmware matches in that repository. Treat this as a lead for further inspection, not as a validated source baseline.
Repository: https://github.com/Quantum-A-TR/proprietary_vendor_samsung_a34x

## No installation performed
No driver was installed or changed on the user's phone. No kernel/boot image was built, flashed, or provided. No bootloader/root operation was initiated.

## Required gate before any installation plan
The user's previously stated constraints were: preserve official OTA, avoid bootloader unlock, avoid data wipe, and do not flash during research. A real kernel-driver replacement may conflict with these goals; Samsung bootloader unlock can trigger a factory reset and permanently trip Knox, while a modified boot/kernel may break OTA or boot verification.

Before moving from source research to any device modification, explicitly confirm whether the user is changing those constraints and accepts the associated risks. Until then, continue with non-destructive source identification and a driver patch/build compatibility study only.

## Next technical work
1. Search vendor tree/device-specific firmware metadata for controller identifier and touch firmware/config names.
2. Match the identified controller to exact source and device-tree node.
3. Select a narrow, source-supported change (e.g. a real driver parameter) rather than replacing the entire driver unnecessarily.
4. Produce a reviewable patch and compatibility report; do not package or flash it before the user explicitly approves the changed risk constraints.
