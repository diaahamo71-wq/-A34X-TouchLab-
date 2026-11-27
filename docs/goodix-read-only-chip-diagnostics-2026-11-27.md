# Read-only Goodix chip diagnostics — 2026-11-27

## Device facts already confirmed from the user's SM-A346E
- Input node: `sec_touchscreen`
- I2C device: bus 51, address `0x5D`
- Bound driver: `goodix_i2c`
- Module: `goodix_ts_berlin`
- Device Tree node: `touchscreen@5d`
- Exact controller product ID is not yet confirmed.

## Source audit (community kernel, branch A346E)
Repository: https://github.com/BS1388/Kernel_Samsung_a346E
File: `kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_core.c`

The source defines a read-only sysfs attribute:
```c
static DEVICE_ATTR(chip_info, 0444, goodix_ts_chip_info_show, NULL);
```
and includes it in `sysfs_attrs[]`. The sysfs group is created on `core_data->pdev->dev.kobj`, i.e. the Goodix platform device, not necessarily the I2C device directory.

The show function calls:
- `hw_ops->read_version()`: ROM PID/VID, patch PID/VID, sensor ID.
- `hw_ops->get_ic_info()`: config ID, config version, firmware checksum.

The companion `driver_info` attribute is also read-only (0444).

The factory command `get_chip_name` formats `GT%s` using `core_data->fw_version.patch_pid`; `get_fw_ver_ic` performs a version and IC-info read. Both require writing to Samsung's factory command endpoint, which the user's ADB shell cannot access. Do not attempt to bypass its permissions.

## Critical limitation
This is a community source tree and has not been proven byte-for-byte or configuration-equivalent to firmware `A346EXXSGFZJ4` (Android 16, kernel 6.6.92...). Attribute presence in this source does NOT prove it exists in the running kernel. Do not flash this kernel or module on the device on the basis of this finding.

## Safe next check on the running device
Search only for the existing read-only attribute; do not write to sysfs:
```sh
find /sys/devices/platform -type f -name chip_info 2>/dev/null
```
If a path is returned, read that exact path:
```sh
cat /the/exact/path/returned/chip_info
```
If no path is returned, stop; it means this particular source attribute was not exposed at that location (or access/traversal is restricted). Do not create the file manually or change permissions.

## Proposed future source patch (not yet applied)
If the matching Samsung source is obtained, a narrowly scoped diagnostic patch may expose a dedicated read-only attribute (0444) that reports only the fields above. It should:
- use the existing Goodix `hw_ops->read_version()` and `hw_ops->get_ic_info()` paths;
- return an error if either read fails, rather than presenting stale/uninitialized values;
- avoid raw register read/write interfaces, firmware update, reset, or any tuning command;
- avoid changing existing `cmd` ownership/mode or SELinux policy;
- be built only against the exact matching kernel tree/config and tested with a recoverable boot path.

No patch or device modification has been made by this report.
