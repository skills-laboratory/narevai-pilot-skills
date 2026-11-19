---
name: fly-the-airplane
description: Fly the simulator aircraft through the flightsim-pilot (v2) MCP server. Use when starting, taxiing, taking off, flying, approaching, or landing.
---

# Fly the airplane

Work a closed loop. Do not skip the read after a control change.

Call the MCP server `flightsim-pilot`. It is the cockpit (MCP v2). The client may prefix tool names with the server name; this skill uses the bare tool names. Read each tool schema before calling it.

## How the pilot server is organised

- **Readings** are named values: `read_flight_deck_indications`, `list_readings`, `read_reading`, `read_group`. `read_flight_deck_indications` reads the whole airplane in one call, all values taken together: flight instruments, altimeter, engines, FMA and MCP selections, flaps, gear, speedbrake, autobrake, and nav 1 and 2 with localizer, glideslope and DME. It returns `read_ms`, `window` (seconds) and `groups`; each reading has `trend` (change per second over `window`) and `ago` (the value `window` seconds earlier). Start every cycle with it instead of several `read_group` calls, and use `read_group` or `read_reading` only for what it does not list.
  Reading ids look like `flight-instruments.airspeed` or `mcp-and-autopilot.cmd-a`. Groups: `v-speeds`, `takeoff-data`, `fmc-cdu`, `flight-instruments`, `altimeter`, `weather`, `radios-and-ils`, `runway-and-approach`, `engines-and-thrust`, `brakes-flaps-and-gear`, `flight-controls`, `mcp-and-autopilot`, `lights-and-transponder`, `electrical`, `apu-and-fuel`, `hydraulics-and-air`, `anti-ice-and-wipers`, `engine-start`, `lights-and-signs`, `efis-captain`, `efis-first-officer`, `irs`, `display-panels`, `radio-panel-and-audio`, `tests-and-service`, `fire-and-emergency`.
- **Cockpit actions** are one tool per group (`cockpit_<group>`, with dashes turned into underscores, for example `cockpit_engine_start`, `cockpit_electrical`, `cockpit_efis_captain`). Each takes `action` (an enum listed in the tool description with its kind, unit and range or positions) and, depending on the kind, `value` or `text`:
  - `press`: no value. `toggle`: flips the switch.
  - `setpoint` and knobs: `value` in the listed unit. The server turns the knob until it matches, so one call is enough.
  - `select`: `value` is a position listed in the tool description (for example `flaps` 5, `gear` 2 = down, `transponder-mode` 5 = TA/RA, `autobrake` 3 = setting 2). One call moves the switch to that position and reports whether it got there.
  - `text` (`cdu-type` in `cockpit_fmc_cdu`): `text` is the characters to type, in one call.
- **States**: `get_function_states` returns on/off for every action that has an indicator (lights, switches, flap detents, MCP modes).
- **Toggles flip.** A `toggle` action (landing-lights, beacon, taxi-light, joystick-override) reads its own state and flips it. Cockpit buttons such as `flight-director-toggle`, `a-t-arm-toggle`, `parking-brake-toggle`, `cmd-a`, and `cmd-b` flip every time you press them: read the state first with `get_function_states`, press once, read again.
- **Read the result.** A `cockpit_*` call returns `ok`, `verified`, `before`, `after` and `detail`. `verified: true` means the server observed the change (`detail` says what moved, for example `off → on`). `verified: false` with `no change` or `no state feedback` means the command was sent but the change was not seen: read the related reading before assuming it worked.
- **Spring-loaded switches** return to centre when released, so they cannot hold a position (the fuel flow switch `used`, the air valve manual `open`, the test switches in `tests-and-service`). Pressing them is a momentary action; check the effect, not the switch.
- **Some switches need a moment.** The server holds the command for you; a slow switch (APU, GPU, generators) can take a couple of seconds to report.
- Raw access (`read_dataref`, `write_dataref`, `run_command`, `search_datarefs`) is a last resort and may be switched off, in which case those tools are not listed. Almost every value you need has a reading: call `list_groups`, `list_readings` and `read_group` before you search for a dataref. Never write altitude, position, or engagement datarefs to imitate flying.
- **You do not control the simulation.** There is no pause, resume or speed tool. If the readings stay frozen (for example the airspeed, altitude and `flight-instruments` values never change between reads), say so and stop instead of looking for a workaround.
- **Stick and pedal input:** `pitch-control`, `roll-control` and `rudder-control` are held override setpoints, not attitude targets. Read the `flight-controls` group and the actual `*-input` readings available in the schema after commanding them; the full flight-deck read does not include those inputs. A verified setpoint alone does not prove control authority, especially with joystick override off. Confirm override state and the actual attitude response before increasing a correction; report missing input feedback rather than assume a control-interface fault.
- **Extra readings:** `flight-instruments.elevation-msl` (feet above sea level), `brakes-flaps-and-gear.gear-handle-down`, `mcp-and-autopilot.cmd-a` and `cmd-b` (CMD mode requires one of these lights; CWS can be engaged separately, so also check `cws-a` / `cws-b` through `get_function_states` after a mode change; the FD master light is not autopilot engagement), and `takeoff-data.takeoff-flaps` (the takeoff flaps entered in the FMC).
- **Runway and approach:** `read_group` on `runway-and-approach` gives the FMC reference airport and runway (`reference-airport`, `reference-runway-course`, `reference-runway-length` in metres, start and end latitude and longitude), `ils-pointers-shown`, and the HGS flags `on-runway-hgs` and `near-runway-hgs`. For ILS deviation use `radios-and-ils` (`nav-1-localizer`, `nav-1-glideslope`, `nav-1-dme`).
- **CDU/FMC:** you can read the display (`read_group` on `fmc-cdu`: `line-0-large` ... `line-6-large`, `small`, `label`, `scratchpad`, `execute-light`) and move between pages with `cockpit_fmc_cdu` (`legs`, `rte`, `init-ref`, `n1-lim`, `dep-arr`, `prev-page`, `next-page`, line select keys `lsk-1l` ... `lsk-6r`, `exec`, `clr`, `del`). To enter data, type it with `cdu-type` (letters, digits, space, `.`, `-`, `/`), then press the line select key next to the field and `exec`. Read the scratchpad and the page after each step. The flight may be loaded with the FMC already set up: read it before you change anything.
- **Weather and altimeter:** `weather` has temperature, dew point, wind, visibility, QNH (hPa and inHg), cloud base and precipitation at the aircraft. `altimeter` has the current setting and STD; set it with `cockpit_altimeter` `set-altimeter`. Clearance (ATC) is not available from any tool: ask the user. The flight plan (route, SID, approach, cruise altitude, performance defaults) is a document: `list_documents` with category `flight` and kind `plan`.
- **Radios and ILS:** `radios-and-ils` has Nav and Com frequencies, courses, the Nav identifiers, localizer and glideslope deviation, and DME. Tune with `cockpit_radios_and_ils` (`set-nav-1-standby-frequency`, `nav-1-swap`, `set-nav-1-course`).
- **Takeoff data:** `read_group` on `takeoff-data` gives gross weight, fuel, the FMC CG and the actual CG, takeoff trim (set and calculated) and V1/VR/V2 (set and calculated).

## 1. Establish the world

1. `get_status` (connection state and aircraft), then `list_groups` / `read_group`. For the manual, `list_documents` with category `aircraft` and kind `manual`, then `read_document` or `list_document_chapters` / `read_document_chapter`.
3. `list_checklist_categories`, `activate_checklist`, and `read_checklist` for the relevant section. If `read_checklist` returns `user_id: null`, the pilot server knows no active run: you can read definitions but cannot record progress.

## Cold and dark, in order

Use the groups in this order when the cockpit is cold and dark. After each step read the result and the related reading before moving on.

1. `electrical`: `battery` 1, `standby-power` 1 (auto). Nothing else on the electrical panel responds without the battery.
2. `apu-and-fuel`: fuel pumps on for the tanks with fuel, then `apu` 1 and `apu-start`. The APU takes about a minute; wait for it before using the APU generators. If there is no fuel, the APU will not start: report it, you cannot load fuel.
3. `electrical`: `apu-generator-1` 1 and `apu-generator-2` 1 (or ground power if it is connected). Read the bus readings.
4. `hydraulics-and-air`: bleed, packs, isolation valve, yaw damper.
5. `irs` and `efis-captain` / `efis-first-officer` as the checklist requires.
6. `engine-start`: start source, fuel levers and `engine-1-start` / `engine-2-start`.

Guarded and irreversible functions are in `fire-and-emergency`. Do not use them unless a checklist for that failure calls for them.

## 2. Before touching the airplane

1. Read the `cockpit_*` tool description for the group you are about to use (it lists the actions and ranges) and `get_function_states` for its current state.
2. Read the readings that will tell you whether the action worked (`read_group` or `read_reading`).
3. If a checklist item is open, `mark_checklist` as soon as the airplane state matches (see Checklist below).

## 3. Fly

1. One action at a time with the `cockpit_*` tool.
2. Recheck the primary flight instruments and the group you changed.
3. If the airplane does not match what you expected, re-read the group and `get_function_states`. There is no action history to consult.
4. Keep scan: attitude, airspeed, altitude, heading, engine, then navigation.

### Selections, modes and response time

- A window value is a target, not the current aircraft response. Check the selected value, active FMA, CMD lights and actual flight trend separately. `trend` is change per second and `ago` is the earlier value, not a new selection; heading wraps through 360°/0°, so compare turn direction and angular error rather than a raw heading trend.
- In HDG SEL, select one planned heading and watch bank, turn direction and shrinking heading error over successive reads. A turn takes time: do not reverse or step through targets every few seconds while the aircraft is still converging. If the remaining intercept margin will not allow the turn to finish, discontinue the join rather than chase the beam. A button returning `no change` while its mode is already active is not evidence of a steering fault.
- Outside a captured final, deliberately select `v-s`, then set the desired rate and read back both the window and FMA. If airspeed, terrain and altitude margins remain safe and the response is converging, keep scanning for several cycles without changing the target again. Pitch, vertical speed and bank do not reach new targets instantly; continuing toward the target is not an autopilot fault. If the response diverges or safety margins are threatened, take the appropriate recovery action.
- `alt-hld` levels at the current altitude; it does not fly to the MCP altitude. For a permitted climb/descent, select the target altitude and a suitable pitch mode (LVL CHG or V/S), then verify capture. Do not press ALT HOLD to reach a lower selected altitude, or cancel a working descent because its first readings lag the request.

### CMD to manual takeover

- Keep one source of attitude control. For a deliberate manual takeover, use `ap-disconnect`, verify CMD and CWS disengagement, then switch joystick override on only if off. Confirm small pitch and roll inputs produce the intended attitude response before relying on them; do not test override inputs against an engaged autopilot.
- `cws-a` / `cws-b` are control-wheel-steering modes, not substitutes for `cmd-a` / `cmd-b`. CWS attitude holding does not establish MCP heading, vertical-speed or altitude capture. If CWS was selected unintentionally, take verified manual control and stabilise before attempting the CMD handoff below.

### Manual control to CMD handoff

- Stabilise attitude and speed before handing over. While joystick override is still on, return pitch, roll and rudder setpoints to 0 and verify the available actual-input readings; a nonzero held input can upset the handoff or disconnect CMD. Neutral input is not a wings-level command: keep scanning bank and pitch.
- Read the override state, switch it off only if on, and verify off. Then, with suitable lateral/pitch modes and targets selected, read the CMD lights and engage one channel once if neither is on. Check both CMD lights, FMA and flight trend on the next read; a momentary button acknowledgement is not sustained engagement.
- If both CMD lights remain off, no commanded-mode autopilot has been verified. Check for unintended CWS; with CMD and CWS off, the FMA is flight-director guidance only. Resume the verified manual takeover above rather than wait for the selected heading/altitude to fly itself. Do not alternate CMD and override toggles to troubleshoot an unstable airplane.

## 4. If you are lost

- Charts and airport procedures are `flightsim-pilot` documents. `list_documents` with kind `chart` or `sop` and the airport ICAO, then `read_document`.
- Position is `flight-instruments.latitude` / `longitude`; ground speed is `flight-instruments.ground-speed`.
- There is no tool to start, restart, or reload the simulation. If the session is wedged or the flight is lost, tell the user.

## Known simulator quirks (learned in flight)

- **`pitch-control` is the elevator position (-1 to 1 of full travel), held until you change it.** It is not a pitch angle or a rate, and its effect grows with airspeed, so the same value does far more at high speed. Seen in flight: one large input over-rotated to ~35° within seconds and bled the speed to ~98 kt, and the large correction in the other direction drove a -16° dive into the ground; a modest input at 195 kt gave +10,000 ft/min and a stall. Make small changes (any value from -1 to 1 is accepted), one change per read, read `flight-instruments.pitch` and its `trend` right after each one, base the next input on what you just read and not on the previous input, and let the input go back to 0 when the pitch is right. Your calls take seconds, so a manual pitch loop lags: do not chase a reading. On takeoff, hand over to `cmd-a` at 400 ft AGL (HDG SEL / LNAV and the pitch mode set); this is not a CMD re-engagement instruction during a manual landing. A fresh run can retain the previous attempt's control/FMA state, so read and zero `flight-controls.pitch-control` in preflight.
- **VNAV may not start the descent.** With `NAV DATA OUT OF DATE` on the CDU, VNAV PTH can hold cruise even after a lower MCP altitude. Watch the PROG `TO T/D`; if the aircraft does not descend, take `lvl-chg` (annunciates `MCP SPD`) or `v-s` deliberately and manage drag with `speedbrake` value 2.
- **A radio course may not set.** `set-nav-1-course` / `set-nav-2-course` can be accepted while the reading stays 0; verify the read-back and do not assume APP will track the published course. `nav-1-dme` did not track distance here — prefer the FMC PROG `DTG` and `latitude`/`longitude` for position.
- **Autoland may not flare.** Both CMD lights alone do not prove autoland: verify FLARE and ROLLOUT arming as specified in `landing`. On a planned autoland, if FLARE is not armed by 350 ft radio altitude, go around; do not accept a firm touchdown or improvise a late manual flare. A planned manual landing uses the early verified handoff in `landing`.
- **Rollout:** the autopilot disconnects on touchdown (both CMD lights off). Take the rudder immediately. The first `reverse-thrust` may not engage (N1 stayed ~60% and the speed rose); if so set `throttle` 0 and use `brakes`, and reverse only when `reverser-*` actually spools. Without pedal input the aircraft drifts off a narrow runway centreline.
- **Transient errors:** `get_function_states` and some `read_group` calls intermittently answer `Simulator is unavailable`; retry the same call.
- **PDF charts may be unreadable** (`read_document` on a chart returns a PDF resource some models cannot read): fall back to the numbers in the relevant skill and confirm with the user.

## Checklist (the flight is scored on it)

The flight is scored on the checklist items you mark with `mark_checklist`, so a done item that is not marked counts as missed.

- Read a section with `read_checklist` (`category`) when you reach that part of the flight. Reuse the returned execution identifiers and the exact section/item ids from the checklist, not guessed category names or challenge text. If activation is required (`active_section_id` is null and automatic activation is off), use `activate_checklist` for that returned section according to its schema before marking.
- When you have done an item and seen the required confirmation (a reading, `verified: true`, or the FMA), mark it at once with `mark_checklist` using its returned identifiers and required evidence. Read the checklist back to verify the mark persisted. Do not wait or batch it for later; an unresolved VREF or minimums item does not prevent marking separately verified radios or autobrake.
- If you did something before you read its section, read the section and mark what you already did and confirmed.
- An `As needed` item requires a deliberate decision and a read-back of the appropriate configuration; an already-correct setting can be confirmed without moving its switch. Do not treat every such item as inapplicable: establish what the actual phase and published procedure require, and report any genuinely missing assignment or feedback. Distinguish these decisions from an unmet conditional item: record not-applicable only if the checklist supports it, otherwise report the condition and leave it unmarked rather than claim an action you did not perform.
- Before you move on to the next part of the flight, read the section you are finishing and mark every item you did and confirmed. Leave an item unmarked only if you did not do it or could not confirm it, and say so.
- Never mark an item you did not do, and never mark a whole section in one go.
- Marks record your acknowledgement, not the simulator state, so keep reading the airplane as usual.

## Related skills

`assess-flight-state` to decide what to do next, `cold-and-dark-start`, `fmc-setup`, `before-takeoff`, `takeoff-straight-ahead`, `climb-and-cruise`, `descent-and-approach`, `landing`, `go-around`, `ils-final-hands-off` for what not to press once LOC and G/S are captured, and the route skill (for example `yssy-yscb`) for chart numbers and fixes.
