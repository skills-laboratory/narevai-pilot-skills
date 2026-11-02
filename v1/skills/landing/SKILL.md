---
name: landing
description: Land the Zibo 737-800X with a proper dual-autopilot ILS autoland. Use when setting up before intercepting the ILS, capturing LOC and G/S, engaging the second autopilot, checking that autoland is really armed, flying flare and rollout, or deciding between autoland, a single-autopilot ILS and a go-around.
---

# Landing — ILS autoland

Tools: autopilot buttons are `cockpit_mcp_and_autopilot`, radios are `cockpit_radios_and_ils`, gear/flaps/autobrake/speedbrake are `cockpit_brakes_flaps_and_gear`, go-around is `to-ga-left` in `cockpit_engines_and_thrust`. Readings come from `read_reading` / `read_group`. The client may prefix tool names with the server name. What not to press once on final is in `ils-final-hands-off`; this skill is the setup and the sequence. Set the flight phase with `set_run_phase`: `approach` when you start the ILS, `landing` on final, `arrived` once stopped.

## Autoland is not an ILS with one autopilot

A coupled ILS with a single autopilot (CMD A only) tracks LOC and G/S down to minima but does not flare or roll out. For a tool-flown manual fallback, that capability is not permission to stay coupled until near the ground: complete the verified manual handover by **1000 ft height AGL**, or go around. Tell the user that the landing needs manual input; do not call it an autoland. A proper autoland needs both autopilots engaged and the autoland armed (checks below).

Before intercept, read the aircraft manual's manual-landing procedure and establish its flare attitude, thrust sequence and disconnect point if a single-channel fallback may be needed. Do not invent a fixed elevator ratio for the flare. Read the `landing` checklist before intercept and acknowledge the ILS/configuration items as verified, while the workload permits.

## Before intercepting the ILS

Set up and read back each item:

1. **Nav 1 and Nav 2** on the same ILS frequency: `set-nav-1-standby-frequency` / `set-nav-2-standby-frequency`, then `nav-1-swap` / `nav-2-swap`. Frequency, identifier and course come from the chart (`list_documents` / `read_document`), never from memory. Verify `radios-and-ils.nav-1-ident` and `nav-2-ident` match the chart.
2. **Course 1 and Course 2** on the published inbound ILS course: `set-nav-1-course` and `set-nav-2-course`; read `nav-1-course` and `nav-2-course`.
3. **MCP altitude**: normally the missed-approach altitude once established on the glideslope, according to the procedure you fly (`set-altitude`). Do not set it above you earlier than the procedure allows, and see `ils-final-hands-off` for when it is safe.
4. **A/T ARM** on and **flight directors** on. `a-t-arm-toggle` and `flight-director-toggle` flip each press: read `get_function_states` first.
5. **Configured and slowing** for the approach: flaps to schedule, then landing speed set once (`set-speed`).

You may still be flying LNAV/VNAV, HDG SEL or LVL CHG. The ILS takes over when you arm and capture APP.

## Intercept and capture

1. Get onto an intercept heading, preferably 30° or less from the localizer course.
2. Press **`app`** once. Read the FMA: `fma-roll-armed` `VOR/LOC` and `fma-pitch-armed` `G/S`. Do not press CMD B yet.
3. **Localizer captures**: `fma-roll` becomes `VOR/LOC` and the airplane turns onto the centreline; G/S stays armed.
4. **Glideslope captures**, normally from below: `fma-pitch` becomes `G/S` and the descent begins. You now have a coupled ILS, but not yet an autoland.

Do not intercept the glideslope from above and do not press APP repeatedly; if the capture does not happen, go around rather than experiment.

## Second autopilot

With CMD A flying the coupled approach and APP armed, use **`cmd-b-hold` once** (press and hold; if B is flying, use `cmd-a-hold` instead). A short `cmd-b` press only changes over from A to B and never gives two channels. Both must be engaged **before 800 ft radio altitude**; below that the second autopilot is locked out. In practice do it right after G/S capture and well above 1,500 ft. Read `cmd-a` and `cmd-b` (both on). Never press either again (see `ils-final-hands-off`).

If the hold leaves only the other CMD on, it changed channels rather than establishing dual AP. Stop trying alternate CMD holds, classify the approach as single-channel, and report the tool limitation when workload permits. A verified button press is not proof of dual AP; use the lights and FMA to decide on the preplanned manual fallback or go-around.

## Checks at about 1,500 ft radio altitude

Read `flight-instruments.height-agl` and the group `mcp-and-autopilot`. This is the autoland verification point; the second autopilot's pitch channel becomes active and the dual-channel checks run:

- `fma-roll` `VOR/LOC` and `fma-pitch` `G/S` (captured).
- `fma-pitch-armed` **`FLARE`** and `fma-roll-armed` **`ROLLOUT`**.
- `cmd-a` and `cmd-b` both on.

On the real PFD **SINGLE CH** disappears and **LAND 3** (or FLARE armed) appears. The MCP server has **no reading for that annunciation**, so you cannot see it: rely on the FMA readings and the CMD lights. Do not continue expecting an autoland if FLARE is not armed or one CMD is off. **FLARE must be armed by 350 ft radio altitude**; without it, go around unless the preplanned manual fallback below is already established and verified. Do not postpone this decision until flare height.

## Planned single-channel manual landing

- Continue only with a stable approach, required visual references at minima, the manual procedure understood and enough height for control/read-back cycles. By **1000 ft height AGL**, verify both CMD lights off and `flight-controls.joystick-override` on, then complete a small pitch-input/read-back cycle confirming `pitch-input`, pitch/trend, sink rate and the lateral path. Centered LOC/G/S immediately after disconnect is not proof of manual control: actively hold the approach now, not only at flare height. If still coupled or that response cycle is unverified at the gate, **go around immediately**. The 350 ft gate is not permission to defer the handover or test manual control near touchdown.
- Before any wait while single-channel coupled, estimate seconds to the 1000-foot handover gate as `(height-agl - 1000) / descent_ft_per_second`, using the positive descent rate from the faster of height trend or vertical speed divided by 60. Include model/decision delay, control execution and confirming read-back; if the handover cannot finish before that gate, go around instead of waiting. From 1000 ft, keep a continuous primary-flight scan; no waits or unrelated calls that would cross a gate. Apply the same time budget to the 350-foot decision gate and approximately 50-foot flare region; reserve a flare command and confirming pitch/sink-rate read before touchdown. If that cycle cannot fit, go around before the flare region rather than rely on an unverified late pull.
- With both CMD lights off, retained `VOR/LOC` / `G/S` text is not proof the autopilot still tracks the path. Hold the manual approach, then fly the manual's flare with small `pitch-control` changes and immediate pitch/trend and sink-rate read-back, using the predictive input scan in `fly-the-airplane`; a verified control ratio alone does not prove a flare. If a nose-up command leaves pitch falling and sink increasing before the flare region, go around as the next action, not another wait or a larger rescue pull. Coordinate thrust reduction with the flare per the manual, not with AP disconnect: do not cut to idle while still carrying the unchanged G/S descent. If an input balloons the airplane or starts pitch oscillation before the flare region, make `to-ga-left` the next action and follow `go-around`: do not merely neutralize the yoke and retract flaps, or try to regain the landing path.
- The hands-off flare and automatic thrust-retard instructions below apply only to a verified autoland, not this manual branch.

## 1,000 to 500 ft

Do not touch the MCP modes: no `lnav`, `vnav`, `hdg-sel`, `v-s`, `lvl-chg`. **For verified autoland only**, leave the autopilot in `VOR/LOC` + `G/S`; the manual branch must already have completed its verified handover by 1000 ft. A stable single-channel ILS is not an exception to that gate. Verify the landing configuration in either branch:

- Gear down (`gear` with value 2) and landing flaps (`flaps` with value 30 or 40).
- Speed at VREF plus the appropriate correction (one `set-speed`).
- `speedbrake` with value 1 (arm).
- Autobrake selected (`autobrake` with value 2, 3 or 4 for settings 1, 2 or 3); read `autobrake-position`.
- Autothrottle still engaged (`fma-autothrottle`).

## Verified autoland — flare and thrust retard (hands off)

Do nothing with the controls. You must **not** try to flare or pull the throttles yourself.

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

## Vacate the runway

Steer on the ground with the rudder control: turn `flight-controls.joystick-override` on first (toggle: read the state), then `set-rudder-control` / `rudder-control` with a small ratio (the sign moves the nose left or right: check `heading` after the first input and correct), and return it to 0 to go straight. Control speed with `throttle` and `brakes` (`ground-speed` should be slow, about 10 to 20 kt, before any turn). Follow the chart for the exit, keep to the centreline until you turn, and stop with the parking brake (`parking-brake-toggle`, read the state first) clear of the runway. Then report where you are and set the phase to `arrived`.

## Go around

Go around with `to-ga-left` and follow the published missed approach if any of these happens:

- a CMD drops unexpectedly, LOC or G/S is lost, or the FMA disagrees with the lights;
- FLARE is not armed by 350 ft radio altitude (or ROLLOUT is not armed) and the manual handover was not already verified by 1000 ft height AGL;
- the approach is unstable, or the required visual references are absent at minima for the approach you are flying.
