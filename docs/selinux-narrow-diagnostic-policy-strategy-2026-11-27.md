# SELinux policy strategy for Goodix chip_info — 2026-11-27

## New repository finding
The public community kernel repository `BS1388/Kernel_Samsung_a346E`, branch `A346E`, contains:
- `Permissive/README.md`
- `Permissive/selinux-make-permissive.patch`

The README says this patch is optional for custom-kernel workflows and that no-root builds remain enforcing by default. The patch changes `selinux_enforcing_boot` to 0 and removes the normal enforcement implementation from `selinuxfs.c`.

## Decision
Do NOT apply this patch to the user's daily-driver build. It disables SELinux enforcement globally, not just access to Goodix diagnostics. It is not a suitable fix for one read-only sysfs node and creates an unnecessarily broad security regression.

## Preferred engineering design
For a future, fully matched custom build:
1. Keep SELinux enforcing.
2. Give the diagnostic node a dedicated SELinux type instead of the generic `sysfs` label, if the platform's labeling architecture permits it.
3. Grant read-only access only to a narrowly scoped, trusted diagnostic domain; do not grant broad access to `shell` or generic sysfs.
4. Keep write interfaces (`cmd`, reset, register/config interfaces) inaccessible to the diagnostic domain.
5. Verify the policy against the exact shipping vendor policy and build, and inspect AVC/audit evidence during validation.

Android's official SELinux implementation guidance recommends device-specific policy additions and labels rather than editing platform policy broadly:
https://source.android.com/docs/security/features/selinux/implement

## Compatibility gate
The user's installed build is `A346EXXSGFZJ4`, Android 16, kernel `6.6.92-android15-8_abA346EXXSGFZJ4-4K`. Samsung's device update page confirms the firmware build, but the inspected community kernel source is not yet proven byte-for-byte/source-release matched to this exact build. Do not build or flash from the community branch until source compatibility, config, DT/DTBO, vendor modules and recovery path are established.

## Current practical status
No-root ADB shell cannot read `chip_info` despite mode 0444. `dmesg` access is also denied, so no AVC confirmation is available. The next work item is exact FZJ4 source/vendor-policy identification and non-invasive firmware/source inspection—not weakening SELinux on the live device.
