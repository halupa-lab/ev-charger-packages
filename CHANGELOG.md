# Changelog

## v0.4.12

- `ev_night_timer` also sends the timer (and sets the charger clock) daily at 22:30, not only at 12:00, so the night starts and ends on the minute: the charger clock runs ~15–20 s a day fast (on 15.11 it stopped at 06:59:54 by HA time). A one-off charge running at 22:30 is left alone, as at 12:00.

## v0.4.11

- `ev_safety_grid_back` stops at once when the grid goes again while it is putting the timer back (it waits for the charger and tries up to 5 times, so this can take minutes). Before, it went on sending the timer; every timer command clears the stop, so inside the timer window the car charged from the battery for a few seconds each time until `grid_lost` stopped it again. The pause flag is set again and the kept timer stays.

## v0.4.10

- "Hold home battery" off now checks the inverter really took the saved program SOC values back. Right after a write HA shows the written value at once; the real read-back comes with the next poll of these registers (every 300 s). HA waits 6 minutes, compares, rewrites what differs once, waits again, and forgets the saved values only when all 6 match. On 13.11 program 2 read back 32 after HA wrote 30, and the next hold saved 32 as the original: now the saved originals stay until the read-back matches, so a hold in between uses them. Push if they still differ.
- `ev_battery_hold_release` runs in `restart` mode and stops when the hold is turned on again while it waits.

## v0.4.9

- Grid back and the charge does not resume by itself (a one-off charge in the day, or outside the timer hours): a push to the phone, not only the HA notification. HA does not start it again; the owner decides.

## v0.4.8

- Removed `ev_charge_now.yaml` ("Зарядити зараз" script, `input_boolean.ev_charge_now`, `ev_charge_now_done`): the charger's own start is a one-off charge now (v0.4.7). Needs card v0.3.0 (its "Почати зарядку" is that start); card v0.2.x calls the removed script.
- `input_boolean.ev_night_mode` is shown as "Авто: заряджати за таймером" (was "… лише вночі"); the entity ID stays.

## v0.4.7

- A one-off charge works with night mode on. "Start" (card or the charger's web page) is a one-off override of the timer in the charger: it charges at once and the timer reads as off. `ev_day_charge_guard` now stops a day charge only when the timer is in the charger (a real timer failure); in a one-off charge it holds the home battery instead. `ev_night_timer` leaves a one-off charge alone in the day.
- The override does not stay: unplugging does not end it in the charger (the next plug-in would charge at once). When the one-off charge is over (not charging for 1 minute) `ev_night_timer` sends the timer again; at 23:01 a one-off charge still running is put back on the timer, so it stops at 07:00.

## v0.4.6

- `ev_safety_grid_back` waits until the charger answers, puts the kept timer back and checks it is really there (up to 5 tries); only then it forgets the kept timer. On 14.11 the charger restarted a minute after the grid came back, the timer command sent during the restart was lost, the kept timer was erased anyway, and after the restart the charger had neither the timer nor the stop, so it charged. Now an urgent push if the timer does not come back.

## v0.4.5

- No grid inside the timer window: the stop now holds. A stop alone did not: the stored timer came back and the charger charged again at once (owner's test 14.11), so the battery would be drained in a stop-start loop. `ev_safety_grid_lost` now takes the timer out of the charger, keeps it in the new `input_text.ev_timer_before_no_grid`, then stops.
- `ev_safety_grid_back` puts the kept timer back: inside the window the charger charges again and stops at the timer end; outside it waits for the window. Also checked 3 minutes after an HA restart, so the timer is not lost when HA restarts during an outage.
- `ev_night_timer` does nothing while charging is paused for no grid.

## v0.4.4

- `ev_safety_grid_back` resumes with a timer command instead of a start. A start is a one-off override in the charger: it ignores the timer, so a charge resumed at night would not stop at 07:00. Now, when the grid is back and the charger's own timer allows charging at this moment, HA sends the same timer again: that clears the stop, the charger charges and stops at the timer end by itself.
- The resume follows the charger's timer, not fixed 23:00–07:00 in HA, so it can be tested in the day with any timer window. A push when the charge did not resume.

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
