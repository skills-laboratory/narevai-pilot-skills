---
name: landing
description: Land the Zibo 737-800X with a proper dual-autopilot ILS autoland. Use when setting up before intercepting the ILS, capturing LOC and G/S, engaging the second autopilot, checking that autoland is really armed, flying flare and rollout, or deciding between autoland, a single-autopilot ILS and a go-around.
---

# Landing — ILS autoland

Tools: autopilot buttons are `cockpit_mcp_and_autopilot`, radios are `cockpit_radios_and_ils`, gear/flaps/autobrake/speedbrake are `cockpit_brakes_flaps_and_gear`, go-around is `to-ga-left` in `cockpit_engines_and_thrust`. Readings come from `read_reading` / `read_group`. The client may prefix tool names with the server name. What not to press once on final is in `ils-final-hands-off`; this skill is the setup and the sequence. The simulator tracks the flight phase automatically; use actual aircraft state for the next step, not a phase-setting call.

## Autoland is not an ILS with one autopilot

A coupled ILS with a single autopilot (CMD A only) tracks LOC and G/S down to minima but does not flare or roll out. You land it manually, so tell the user that the landing needs manual input. Do not call it an autoland. A proper autoland needs both autopilots engaged and the autoland armed (checks below).

## Before intercepting the ILS

Set up and read back each item:

1. **Nav 1 and Nav 2** on the same ILS frequency: `set-nav-1-standby-frequency` / `set-nav-2-standby-frequency`, then `nav-1-swap` / `nav-2-swap`. Frequency, identifier and course come from the chart (`list_documents` / `read_document`), never from memory. Verify `radios-and-ils.nav-1-ident` and `nav-2-ident` match the chart.
2. **Course 1 and Course 2** on the published inbound ILS course: `set-nav-1-course` and `set-nav-2-course`; read `nav-1-course` and `nav-2-course`.
3. **MCP altitude**: normally the missed-approach altitude once established on the glideslope, according to the procedure you fly (`set-altitude`). Do not set it above you earlier than the procedure allows, and see `ils-final-hands-off` for when it is safe.
4. **A/T ARM** on and **flight directors** on. `a-t-arm-toggle` and `flight-director-toggle` flip each press: read `get_function_states` first.
5. **Configured and slowing** for the approach: flaps to schedule and selected speed appropriate to the current flap setting (`set-speed`); use the alive/capture sequence below for final configuration.

You may still be flying LNAV/VNAV, HDG SEL or LVL CHG. The ILS takes over when you arm and capture APP.

## Intercept and capture

Read the `landing` checklist before changing configuration; it may return separate parts for glideslope alive and capture. Keep each part's identifiers and mark only its independently confirmed items at the applicable condition, rather than postponing all marks until final configuration.

1. Get onto an intercept heading, preferably 30° or less from the localizer course.
2. Read the active and armed FMA first. Press **`app`** once only if not already armed/captured. Verify the resulting modes even if the button reports `no change`: `fma-roll-armed` `VOR/LOC` and `fma-pitch-armed` `G/S`. Do not press CMD B yet.
3. **Localizer captures**: `fma-roll` becomes `VOR/LOC` and the airplane turns onto the centreline; G/S stays armed.
4. **Glideslope alive**, moving toward centre from below: follow the checklist's intermediate configuration before advancing to landing flaps. Select gear DOWN, verify nose/left/right gear safe with no transit indications, select and verify flaps 15, set both engine start switches CONT and verify them, and arm the speedbrake. The SPEEDBRAKE ARMED lamp is a separate check from the lever detent: use available documented lamp feedback, or report it unavailable and leave only that item unconfirmed.
5. **Glideslope captures from below**: `fma-pitch` becomes `G/S`, the deviation is near centre and a sustained descent begins along the charted path. The mode label alone is not capture evidence. Only then do you have a coupled ILS, not yet an autoland. Verify the selected landing flap configuration and target speed, then set and read back the published missed-approach altitude when permitted; mark these in the capture part of the checklist.

If taking over already at landing flaps, do not retract them just to fill a missed intermediate item. Leave that item unconfirmed unless its condition and required configuration were actually observed.

While still below the beam outside the expected capture distance, ALT HOLD with G/S armed is normal: hold the permitted intercept altitude, configure and monitor, rather than descend or reset the radios. Do not intercept from above or press APP repeatedly. If verified capture fails as the path reaches the aircraft, guidance is lost or the approach becomes unstable, go around rather than experiment.

## Second autopilot

With CMD A flying the coupled approach and APP armed, use **`cmd-b-hold` once** (press and hold; if B is flying, use `cmd-a-hold` instead). A short `cmd-b` press only changes over from A to B and never gives two channels. Both must be engaged **before 800 ft radio altitude**; below that the second autopilot is locked out. In practice do it right after G/S capture and well above 1,500 ft. Read `cmd-a` and `cmd-b` (both on). Never press either again (see `ils-final-hands-off`).

## Checks at about 1,500 ft radio altitude

Read `flight-instruments.height-agl` and the group `mcp-and-autopilot`. This is the autoland verification point; the second autopilot's pitch channel becomes active and the dual-channel checks run:

- `fma-roll` `VOR/LOC` and `fma-pitch` `G/S` (captured).
- `fma-pitch-armed` **`FLARE`** and `fma-roll-armed` **`ROLLOUT`**.
- `cmd-a` and `cmd-b` both on.

On the real PFD **SINGLE CH** disappears and **LAND 3** (or FLARE armed) appears. The MCP server has **no reading for that annunciation**, so you cannot see it: rely on the FMA readings and the CMD lights. Do not continue expecting an autoland if FLARE is not armed or one CMD is off. **FLARE must be armed by 350 ft radio altitude**; if it is not, treat it as an autoland failure and go around.

## 1,000 to 500 ft

Do not touch the MCP modes: no `lnav`, `vnav`, `hdg-sel`, `v-s`, `lvl-chg`. Leave the autopilot in `VOR/LOC` + `G/S`. Finish configuring normally:

- Gear down (`gear` with value 2) and landing flaps (`flaps` with value 30 or 40).
- Speed at VREF plus the appropriate correction (one `set-speed`).
- `speedbrake` with value 1 (arm).
- Autobrake selected (`autobrake` with value 2, 3 or 4 for settings 1, 2 or 3); read `autobrake-position`.
- Autothrottle still engaged (`fma-autothrottle`).

## Flare and thrust retard (hands off)

Hands off the controls, not the instrument scan: keep reading the FMA, height, speed and descent until touchdown is confirmed, then perform the stopping sequence. You must **not** try to flare or pull the throttles yourself.

1. About **50 ft**: the AFDS flares automatically. `fma-pitch` changes from `G/S` to `FLARE` and the flight director bars retract. Do not disconnect the autopilot.
2. About **27 ft**: the autothrottle retards the thrust levers toward idle during the flare.
3. **Touchdown.** The autopilot stays engaged for the immediate landing and `fma-roll` `ROLLOUT` tracks the centreline.

Monitor only: `airspeed`, `vertical-speed` (negative), `height-agl`, FMA, `nav-1-localizer` and `nav-1-glideslope` dots. If FLARE and ROLLOUT do not appear when they should, treat it as a failed autoland.

## After touchdown

Autoland keeps the airplane on the centreline but is not a taxi system. You own the stopping sequence:

1. **Speedbrakes**: verify they deployed (`brakes-flaps-and-gear.speedbrake-lever` close to fully up; the armed lever deploys on touchdown).
2. **Reverse thrust**: `cockpit_engines_and_thrust` action `reverse-thrust` holds maximum reverse for 5 seconds, then releases to idle. Repeat while `flight-instruments.ground-speed` is above about 60 kt and watch `engines-and-thrust.reverser-left` and `reverser-right`; stop reversing at about 60 kt. Do not call it in the air.
3. **Autobrake** slows the airplane (`autobrake-position` shows the setting). Do not use `brakes` straight away, because manual braking can disarm it. Use `brakes` (3 s regular) or `brakes-max` in `cockpit_brakes_flaps_and_gear` only if the airplane is not decelerating enough.
4. **Disconnect the autopilot** with `ap-disconnect` in `cockpit_mcp_and_autopilot` once the airplane is slow and straight on the centreline. Do not use it on approach; to give up an approach use `to-ga-left`.
5. Read `ground-speed`, `fma-roll` and the CMD lights between steps.

## After-landing verification

Once safely stopped or clear of the runway, read the `after_landing` checklist and verify each required setting. Decide exterior lights and transponder from the actual runway/taxi state and the published procedure; keep runway-occupancy settings until clear and report any missing assignment needed for further changes. Verify weather-radar operating mode OFF separately from the EFIS WXR overlay; an unlit overlay button does not establish radar OFF. If operating-mode feedback is unavailable, report it and leave that item unconfirmed. Mark separately verified items and read the section back for omissions.

Before declaring the flight complete, read `fire-and-emergency`, the relevant system groups and available autopilot-disconnect indications (discover them with `list_readings` if needed). A stopped airplane and quiet configuration/stall alarms do not prove all warnings are clear.

- Identify each lit system annunciator from its individual system indications and required configuration. With probe heat deliberately OFF on the ground, PITOT OFF indications can be expected; distinguish that switch-selected condition from a heat-system fault before acknowledging the caution.
- Correct faults through the applicable procedure. For an identified ground-configuration caution whose acknowledgement is permitted, press `master-caution-captain` once in `cockpit_fire_and_emergency`, then read `master-caution` and the six-pack lamps again. A `six-pack-captain` press is not a substitute for the Master Caution acknowledgement.
- Do not cycle unrelated window-heat switches or reverse a required probe-heat OFF selection merely to extinguish ICE. Do not blindly reset cautions or re-engage CMD to silence a disconnect warning. If the caution returns or indications remain lit, investigate and report them; an acknowledgement attempt does not prove warnings clear.

## Vacate the runway

Steer on the ground with the rudder control: turn `flight-controls.joystick-override` on first (toggle: read the state), then `set-rudder-control` / `rudder-control` with a small ratio (the sign moves the nose left or right: check `heading` after the first input and correct), and return it to 0 to go straight. Control speed with `throttle` and `brakes` (`ground-speed` should be slow, about 10 to 20 kt, before any turn). Follow the chart for the exit, keep to the centreline until you turn, and stop with the parking brake (`parking-brake-toggle`, read the state first) clear of the runway. Before reporting completion, verify `on-ground`, ground speed near zero and position at the destination, and review the `after_landing` checklist; do not set the phase yourself.

## Go around

Go around with `to-ga-left` and follow the published missed approach if any of these happens:

- a CMD drops, LOC or G/S is lost, or the FMA disagrees with the lights;
- FLARE is not armed by 350 ft radio altitude (or ROLLOUT is not armed) and you cannot land manually with confidence;
- the approach is unstable, or the required visual references are absent at minima for the approach you are flying.
