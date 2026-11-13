# Changelog

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
