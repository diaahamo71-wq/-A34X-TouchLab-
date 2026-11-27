# Goodix Berlin command-path audit — 2026-11-27

## Device-side facts supplied by the owner
- Galaxy A34 5G SM-A346E, Android 16 / API 36, firmware build A346EXXSGFZJ4.
- The active I2C device is `51-005d`, address `0x5D`.
- Its bound driver is `goodix_i2c`; its module link is `goodix_ts_berlin`.
- The Samsung input device is `sec_touchscreen`.
- The Samsung factory command list includes `refresh_rate_mode`, `set_game_mode`, `set_sip_mode`, and `low_sensitivity_mode_enable`.
- Device Tree node listing includes `sec,refresh_rate_enable`, but the shell UID cannot read property contents. Presence alone is not proof of the active runtime value on this firmware.

## Public source examined
Repository: https://github.com/BS1388/Kernel_Samsung_a346E
Branch: `A346E`
Branch commit inspected: `40393b598fb81868eed6107152d0d1dfc887b520`
Relevant files:
- `kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_cmd.c`
- `kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_core.c`
- `kernel/kernel_device_modules-6.6/drivers/input/sec_input/sec_cmd.c`

This is a public community kernel source for A346E, not yet proven to be byte-for-byte or commit-for-commit the source used by firmware A346EXXSGFZJ4. Treat conclusions as source-backed leads until exact firmware/source matching is established.

## Confirmed source behavior

### `refresh_rate_mode`
In `goodix_ts_cmd.c` around lines 3269–3322:
- The accepted parameter range is 0 through 2.
- The source comment defines:
  - 0 = normal (60 Hz)
  - 1 = adaptive
  - 2 = always (90 Hz / 120 Hz)
- It stores the chosen value in `core_data->refresh_rate`.
- `set_refresh_rate_mode()` sends Goodix command byte `0x9D` with that value in data byte 0.
- The source logs this as “set scan rate”, including success/failure.

This is strong evidence that the command controls the controller's scan-rate mode, not merely Android display refresh rate. It does NOT prove a fixed 120 Hz touch sampling rate; mode 2 is described as 90/120 Hz and the exact behavior depends on panel/display state and firmware.

The command is guarded by `core_data->refresh_rate_enable`. In `goodix_ts_core.c` around line 1137, this flag is populated from the Device Tree boolean `sec,refresh_rate_enable`.

### `set_game_mode`
In `goodix_ts_cmd.c` around lines 2364–2405:
- Accepts only 0 or 1.
- Sends Goodix command byte `0xC2`, data byte 0 = 0/1.
- Reports OK if `hw_ops->send_cmd()` succeeds.

The source does not document the internal algorithm or prove that this changes scan rate, latency, filtering, or sensitivity. It is a controller game-mode toggle; further firmware documentation or controlled measurements are required.

### `set_sip_mode`
Around lines 2446–2485:
- Accepts only 0 or 1.
- Sends command byte `0x73`, data byte 0 = 0/1.
- The source labels it SIP mode but does not define the acronym or explain its performance effect. Do not assume what SIP stands for.

### `low_sensitivity_mode_enable`
Around lines 2120–2160:
- Accepts values 0–3, not just 0/1.
- Comment maps 0 to exit LSM, 1 to enter LSM, 2 and 3 to debug.
- Writes the value through `GOODIX_LS_MODE_ADDR` while holding `modechange_mutex`.
- It fails if the IC is powered off or the controller command fails.

This is not a “boost sensitivity” command. It is explicitly a low-sensitivity-mode control. Do not change it experimentally without understanding device policy and preserving the original state.

### Factory aggregate result files
In Samsung `sec_cmd.c`, `cmd_result_all` is built from aggregate factory test output; the reader emits the item count followed by named result entries. The Goodix `factory_cmd_result_all` handler clears the buffer, sets the aggregate state RUNNING, runs a sequence of hardware tests, then sets it OK and logs the aggregate data.

Therefore, the device's `cmd_status_all = WAITING` and `cmd_result_all = 0` are consistent with no aggregate factory test currently running and an empty/zero-item aggregate result. They are not evidence of a fault and are unrelated to the prior single-command result `set_sip_mode,1:OK`.

## Safety / next steps
1. Do not write to `/sys/class/sec/tsp/cmd` yet. The sysfs interface executes commands on the live touchscreen controller.
2. First establish exact command registration and runtime support on this firmware. The DT property listing contains `sec,refresh_rate_enable`, but the value could not be read as shell.
3. Before any live mode change, obtain the current mode/state and define a rollback to the stock mode. Do not assume a successful command result means the mode improves latency.
4. Measure touch latency/scan behavior before and after under repeatable conditions; Android display refresh and controller scan rate are separate quantities.
5. Match the public source against the exact FZJ4 firmware/kernel build before compiling or flashing anything. No driver replacement or firmware write is justified by this audit alone.

## Source links
- Goodix command implementation: https://github.com/BS1388/Kernel_Samsung_a346E/blob/A346E/kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_cmd.c
- Goodix core / Device Tree flag: https://github.com/BS1388/Kernel_Samsung_a346E/blob/A346E/kernel/kernel_device_modules-6.6/drivers/input/touchscreen/goodix/berlin/goodix_ts_core.c
- Samsung factory command framework: https://github.com/BS1388/Kernel_Samsung_a346E/blob/A346E/kernel/kernel_device_modules-6.6/drivers/input/sec_input/sec_cmd.c
