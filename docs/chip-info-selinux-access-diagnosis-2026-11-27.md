# chip_info access diagnosis — 2026-11-27

## Device observation
The user reports:
- `/sys/devices/platform/goodix_ts.0/chip_info` exists.
- `ls -lZ` reports `-r--r--r-- 1 root root ... u:object_r:sysfs:s0`.
- Parent directory is `drwxr-xr-xr-x root root u:object_r:sysfs:s0`.
- Reading with the ADB shell identity (`uid=2000(shell)`, domain `u:r:shell:s0`) returns `Permission denied`.

## Interpretation
The Unix mode bits grant read access to all users, so ordinary DAC permissions alone do not explain the denial. SELinux is the leading explanation, but this is not yet proven without a matching AVC denial/audit record. The generic `sysfs` label is relevant; do not assume the policy rule from the community kernel repository matches the shipping Samsung vendor policy.

## Community source inspected
Repository: https://github.com/BS1388/Kernel_Samsung_a346E
Branch: `A346E`
File: `kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_core.c`

The source defines:
- `static DEVICE_ATTR(chip_info, 0444, goodix_ts_chip_info_show, NULL);`
- Registers it in `sysfs_attrs[]`, then creates the group with `sysfs_create_group(&core_data->pdev->dev.kobj, &sysfs_group)`.
- The show function calls `hw_ops->read_version()` and `hw_ops->get_ic_info()`, and prints ROM PID/VID, patch PID/VID, sensor ID, config ID/version, and firmware checksum.

This is strong evidence for how this community source implements the file, but not proof that its source exactly matches the user's FZJ4 shipping kernel.

## Safe next steps
1. Do not use chmod/chown, SELinux permissive mode, policy bypasses, exploit chains, or writes to Goodix sysfs command/reset/config nodes.
2. If available through normal Android diagnostics, inspect kernel/audit logs for an AVC denial involving this exact path and `scontext=u:r:shell:s0`. Absence of a visible AVC to shell is not proof no denial occurred.
3. Identify the exact FZJ4 kernel/vendor policy source and compare the Goodix sysfs node labeling and access rules.
4. If a custom build becomes feasible, prefer a dedicated SELinux type for this read-only diagnostic node and the narrowest intended diagnostic-domain read permission, rather than broadening access to generic `sysfs`. This still requires a fully matched build and recovery plan; it is not an on-device no-root tweak.
5. In parallel, continue identifying the actual controller from firmware/source artifacts; do not infer an exact IC model from the `goodix_ts_berlin` driver family alone.

## Current conclusion
The file exists and is mode 0444, but ADB shell cannot read it. SELinux is the leading cause; confirmation awaits a relevant AVC or matching shipping policy. No device settings have been changed.
