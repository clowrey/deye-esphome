# deye-esphome

ESPHome packages for Deye-family hybrid inverters (Deye, Sunsynk, Sol-Ark) read over RS485/RS232 Modbus with an
ESP32. A site's config holds only what is particular to it (device name, secrets, pins, extra sensors) and pulls
the rest from here, pinned to a tag.

Started from [slipx06/Sunsynk-Home-Assistant-Dash](https://github.com/slipx06/Sunsynk-Home-Assistant-Dash)
(`esphome-1p-sunsynk-deye.yaml`), extended and fixed over several installs.

| File | What |
|---|---|
| `common/rs485-modbus.yaml` | UART, Modbus, the polling controller (3 s, 50 ms between commands), Home Assistant time, hourly inverter clock sync |
| `common/diagnostics.yaml` | Wi-Fi signal and uptime |
| `models/12k-us/registers.yaml` | 12K US split-phase: about 185 sensors, switches, numbers, selects and text sensors |
| `examples/12k-us.yaml` | A minimal site config using the three files above |

## Using it

```yaml
packages:
  deye:
    url: https://github.com/clowrey/deye-esphome
    ref: v1.0.0
    files:
      - common/rs485-modbus.yaml
      - common/diagnostics.yaml
      - models/12k-us/registers.yaml
```

Substitutions, each with a default in the package and overridable in the site's `substitutions:`:

| Substitution | Default | Meaning |
|---|---|---|
| `tx_pin`, `rx_pin` | `6`, `5` | ESP GPIOs to the RS485/RS232 transceiver |
| `modbus_address` | `0x01` | the inverter's Modbus ID (set each inverter differently when several share a bus) |
| `modbus_controller_id` | `inverter` | id of the controller the register entities use |
| `settings_skipped_updates` | `30` | settings and text sensors are read once every N polls |
| `battery_current_multiply` | `0.1` | register 191 scale: 0.1 on most 12K US firmware, 0.01 on some |

## Keeping history when changing these files

Home Assistant keys an ESPHome entity's history on the device and the entity's name. Renaming an entity here makes
a new entity at every site and its old history stops. So:

- Never rename an existing entity. Add new ones instead.
- Before tagging a release, expand each site's config with `esphome config` before and after the change and compare
  the entity list: nothing may disappear.
- Sites move to a new version by changing `ref:`, one site at a time.

## Changes

- **v1.0.0** (2026-11-19): first release. Number settings use `mode: box` (typed values, no accidental slider drags).
  `Overall Status` uses `modbus::helpers::word_from_hex_str` (ESPHome 2026 moved it from `modbus_controller::`).
  Wi-Fi signal and uptime added.
