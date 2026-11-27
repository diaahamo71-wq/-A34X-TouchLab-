# What will identify the touchscreen IC?

A marketing model number alone is insufficient because A34 units may use panel/controller variants.

The reliable identifier is the active device-tree node plus the driver that binds to it:
- Device-tree node name (often a vendor-specific touchscreen label)
- `compatible` string
- Bus and address (I2C/SPI)
- Driver's probe/match table
- Firmware/config blob or panel ID

We will search the source for terms such as `touchscreen`, `touchpanel`, `tpd`, `sec_ts`, `goodix`, `focaltech`, `novatek`, `himax`, `ilitek`, and MediaTek touch framework names. A match in a generic kernel directory is not enough; it must be linked to the A34X device tree and preferably the exact A346EXXSGFZJ4 source release.

No driver tuning should be chosen until the stock baseline is documented and the exact controller is established.
