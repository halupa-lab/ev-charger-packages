# Changelog

## v0.4.3

- `ev_night_timer` runs in `restart` mode and waits 3 s after a mode change, so a quick off-on of "charge only at night" counts once, as the last state. Before, a double toggle queued runs, one of them cleared the timer, and the charger started charging (a timer command also clears a stop in the charger).
- Timer commands are sent only when needed: on a mode change; with night mode on, daily at 12:00 and when the timer or clock is wrong, never during 23:00–07:00; with night mode off, only when a timer is still in the charger.
- No tries and no push while the charger is unavailable; it is checked a minute after it comes back. The push is only for a charger that is online but does not confirm.

## v0.4.2

- `ev_day_charge_guard` acts only 07:05–22:55. The charger clock runs a little fast, so the timer start at the charger's 23:00 could come a few seconds before 23:00 by HA time and be stopped as a daytime start.

## v0.4.1

- The charging current is the one set on the card. `ev_night_current` (always 20 A at night) is replaced by `ev_current_apply`: when charging starts and the charger has a different current stored, HA sends the card current. HA no longer picks a current itself (the 22 A safety ceiling stays).

## v0.4.0

- New package `ev_charge_now.yaml`: script `script.ev_charge_now` ("Авто: зарядити зараз", the card button) for a one-off charge while "charge only at night" keeps the charger timer on. In the day it holds the home battery, turns night mode off (the timer is cleared), waits for the charger to start by itself and sends start only if it does not. If nothing starts: night mode back on at once and a push.
- New automation `ev_charge_now_done`: when the one-off charge is over (charging stopped for 10 min or the car unplugged), "charge only at night" is turned back on.
- New helper `input_boolean.ev_charge_now` ("Авто: разова зарядка зараз") marks a one-off charge.

## v0.3.0

Needs the ElectroS integration fork v1.8.0 and the charger entities renamed to its English IDs (in HA: entity settings → ↻ "regenerate entity ID").

- All charger entity IDs changed to the fork's fixed English IDs: `switch.electros_charging`, `number.electros_charging_current`, `sensor.electros_set_current`, `sensor.electros_power`, `sensor.electros_timer`, `sensor.electros_clock_offset`. No change in behaviour.

## v0.2.0

Needs the ElectroS integration fork [electros-ev-local-ha](https://github.com/halupa-lab/electros-ev-local-ha) v1.7.0 or newer.

- Night mode now uses the charger's own timer: "Авто: заряджати лише вночі" on sets 23:00–07:00 in the charger, off clears it. The charger then starts and stops by itself, and a car plugged in during the day waits without a start-then-stop. New automation `ev_night_timer` also keeps the charger clock right (daily at 12:00 and when it drifts over 60 s) and sends a push when the charger does not confirm the timer.
- New automation `ev_night_current`: when night charging starts, set 20 A (the charger starts with the current left from the day).
- `ev_day_charge_guard`: with night mode on, a daytime charge means the timer failed, so it stops charging and sends a push.
- Removed `ev_night_start`, `ev_night_stop`, `ev_night_keep_off` (the timer does their job) and `ev_safety_missed_night_start` (the charger starts at 23:00 by itself, also when HA restarts).
- `ev_safety_grid_lost`: the stop is retried 3 times; if the charger still charges, an urgent push.

## v0.1.0

- The four packages of the proof of concept (`ev_accounting`, `ev_battery_hold`, `ev_night`, `ev_safety`). All entity IDs and `unique_id`s are the same.
- CI: yamllint and the Home Assistant config check.
