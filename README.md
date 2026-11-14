# EV Charger Packages

Home Assistant YAML packages for EV charging at home: an ElectroS WiFi charger and a Deye hybrid inverter. They count where the car energy comes from (solar / home battery / grid night / grid day) and its grid cost, hold the home battery while the car charges from the grid, charge at the night tariff, and stop charging without grid.

The dashboard card lives in a separate repository: [ev-charger-card](https://github.com/halupa-lab/ev-charger-card).

> These packages are written for one house: entity IDs of the charger and the inverter are in the YAML. Use them as an example, or change the IDs to yours.

## Files

| File | What it does |
|---|---|
| `ev_accounting.yaml` | Car power by source, kWh counters, grid cost in UAH, daily and monthly meters |
| `ev_battery_hold.yaml` | "Hold home battery": raises Deye TOU program SOC while the car charges, restores it after |
| `ev_night.yaml` | "Charge by the timer": sets the timer 23:00–07:00 in the charger and keeps the charger clock right; when charging starts, applies the current set on the card |
| `ev_safety.yaml` | No grid → take the timer out and stop charging (retried, push when it fails); grid back → timer back; current ceiling 22 A |

## Requirements

- Home Assistant with packages enabled in `configuration.yaml`:

  ```yaml
  homeassistant:
    packages: !include_dir_named packages
  ```

  `!include_dir_named` also reads sub-folders, so this repository can live in `/config/packages/ev/`. It skips files and folders whose names start with a dot (`.github`, `.yamllint`).
- Integrations: the fork [electros-ev-local-ha](https://github.com/halupa-lab/electros-ev-local-ha) v1.8.0+ of ElectroS EV Charger (Local) (charger timer actions, fixed entity IDs `switch.electros_charging`, `sensor.electros_timer`, …) and [Solarman](https://github.com/davidrapan/ha-solarman) (`deye_p3` profile), with `number.deye_inverter_program_1_soc` … `_6_soc` enabled.
- A template sensor `sensor.grid_electricity_price` (current price, UAH/kWh).
- The Home Assistant app on a phone for pushes (`notify.mobile_app_oleksiy_iphone`).

## Install

In the Terminal & SSH app:

```bash
cd /config/packages
git clone https://github.com/halupa-lab/ev-charger-packages.git ev
cd ev
git checkout v0.1.0
```

Then Developer tools → YAML → **Check configuration**, and restart Home Assistant.

## Update and rollback

Every version is a git tag (see [CHANGELOG.md](CHANGELOG.md)). To install a version, or to go back to an older one:

```bash
cd /config/packages/ev
git fetch --tags
git checkout vX.Y.Z
```

Then **Check configuration** and reload what the version notes say (automations, scripts, template entities), or restart Home Assistant. Restart only when the car is not charging.

## Checks

GitHub Actions runs on every push: `yamllint` and the official Home Assistant config check (`check_config` in the `home-assistant:stable` container) with these packages in `packages/ev`.

## License

MIT
