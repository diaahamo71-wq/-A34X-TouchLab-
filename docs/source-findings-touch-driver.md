# Source findings: driver families and configuration baseline

Date: 2026-11-27

## Important finding
I inspected the public A346E community kernel source tree and its touchscreen module/configuration directories. It contains Samsung's input integration and multiple MediaTek touch-driver families. This is a valuable source map, but **it does not yet prove which controller/driver is bound on the user's exact stock firmware A346EXXSGFZJ4**.

## Files and paths located in the community source
- Samsung input integration:
  `kernel/kernel_device_modules-6.6/drivers/input/sec_input/`
  - `sec_input.h`
  - `sec_common_fn.c`
  - `sec_cmd.c`
  - `sec_secure_touch.c`
  - `sec_tclm_v2.c`
- Vendor touch-driver source families:
  `kernel/kernel_device_modules-6.6/drivers/input/touchscreen/`
  - `GT9895`, `GT9886`, `GT9896S`, `GT1151`
  - `NT36672C_I2C`, `NT36532`
  - `ILITEK`, `focaltech_touch`
  - `synaptics_tcm2_sec`, `synaptics_dsx_96789`, `synaptics_dsx_96851`
  - `goodix` and other generic candidates
- The checked-in kernel config has:
  `CONFIG_TOUCHSCREEN_MTK_TOUCH="GT9895 GT9886 GT9896S GT1151 NT36672C_I2C legacy_gt9896s"`
- The config's external-module list references additional candidate modules: `FT3518U`, `GT9886`, `GT9916`, `NT36672C`, `ST61Y`, and `synaptics_tcm`.

## What this means (and what it does not)
The config/source is not a runtime report from the user's handset. It can list alternatives, modules, or build-time candidates. It does **not** identify the installed touch IC, selected firmware blob, or active driver by itself. Do not conclude "your panel is Goodix/FocalTech/Synaptics" from folder names.

The exact controller identification must be cross-linked from the stock FZJ4 Samsung OSS package:
1. device-tree node and its `compatible` string;
2. module/driver match table and probe binding;
3. controller firmware/config filename and panel/vendor ID;
4. ideally, independent runtime driver identity evidence.

## Deep baseline fields to extract from the active driver
| Area | Fields to capture | Why it matters |
|---|---|---|
| Hardware binding | compatible, bus/address, panel ID, firmware/config | proves which IC and panel variant |
| Acquisition | controller scan/report-rate modes, report packet, IRQ behavior | possible real input-rate constraints |
| Signal processing | filter, smoothing, debounce, jitter, edge/palm rejection | can alter perceived responsiveness or accuracy |
| Samsung integration | sec_input command/sysfs, factory controls, TCLM/calibration, charger/cover modes | OEM tuning and service settings |
| Performance path | input booster/QoS, IRQ/thread priority, CPU/bus constraints | software latency, not sensor frequency |
| Power/thermal | controller power states, low-power/gesture mode | whether tuning changes stability or drain |

For every parameter record: source path + line, stock/default value, unit, range, read/write interface, reset/revert behavior, and classification:
- runtime adjustable,
- build-time only,
- firmware/controller owned,
- fixed hardware limit,
- unknown pending evidence.

## Next action
Get the exact Samsung Open Source Center source package matching SM-A346E and build/changelist 33505653 / FZJ4. Then inspect the device-specific DT and corresponding vendor touch module/config. If the exact source archive is not accessible through the repository or official portal, record that blocker rather than substitute an older A346B/E source.

No handset changes, bootloader unlock, root, flashing, or speculative sysfs writes are part of this audit. This preserves the user's current data and stock OTA state during research.
