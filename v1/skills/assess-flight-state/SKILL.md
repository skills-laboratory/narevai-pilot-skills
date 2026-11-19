---
name: assess-flight-state
description: Work out what state the airplane is in and which skill to use next. Use at the start of every task, after any interruption, and whenever you are unsure whether the airplane is cold and dark, ready on the runway, climbing, cruising, descending or on final.
---

# Assess the flight state

The task tells you where you start and what to achieve. It does not tell you what the airplane looks like; read it. Change nothing until you have read the state.

## Read

1. `get_status`, then `get_run` (the flight phase and, if the run has one, where it ends). If `get_run` returns no run, tell the user; you cannot start one.
2. `read_flight_deck_indications` (flight instruments with `on-ground`, `height-agl`, `airspeed`, `altitude`, `vertical-speed`, `heading`, position; engines; flaps, gear and speedbrake; the FMA, CMD lights and selected altitude, speed and heading; nav 1 and 2).
3. `read_group` on `electrical`, `fmc-cdu` (is there a route?), `takeoff-data` (fuel, weight, V-speeds) and `weather`, which it does not cover.

The flight phase is worked out from the simulator, so you do not set it. Phases in order: `preflight` (stationary on the ground), `taxi`, `takeoff` (roll above about 40 kt), `climb`, `cruise`, `descent`, `approach` (within about 20 nm of the destination, descending), `landing` (gear down, below about 500 ft), `arrived` (stopped at the destination). A go-around returns the phase to `climb`. A phase changes only after its conditions have held for a few seconds, so it can lag the airplane.

## Which skill next

| What you read | State | Skill |
| --- | --- | --- |
| Nothing powered, engines off, parking brake set | Cold and dark | `cold-and-dark-start` |
| Powered, engines off or running, the FMC has no route or no performance data | Not set up | `fmc-setup`, then `before-takeoff` |
| On the ground, engines running, FMC set, not rolling | Ready to go | `before-takeoff`, then `takeoff-straight-ahead` |
| Rolling on the runway | Takeoff | `takeoff-straight-ahead` |
| Airborne, gear down or up, below the cleared altitude | Climb | `climb-and-cruise` |
| Airborne, level at altitude, far from the destination | Cruise | `climb-and-cruise`, then `descent-and-approach` |
| Airborne, descending or near the destination, ILS not captured | Descent or approach | `descent-and-approach` |
| LOC and G/S captured (`fma-roll` VOR/LOC, `fma-pitch` G/S) | Final | `ils-final-hands-off`, `landing` |
| Approach unstable or guidance lost | Go-around | `go-around` |
| On the ground after touchdown | Rollout and vacate | `landing` |

Load the selected phase skill and `fly-the-airplane` now, then use `routes/` skills for the route facts you need. Read the current checklist section (`read_checklist` with `category`) and mark items as their conditions and required evidence are confirmed; do not replay earlier ground procedures in flight.

## Take over, do not redo

When you start in the air or with the engines running, the airplane has been prepared by the simulator and not by you. Do not repeat start-up steps, and do not press toggles whose state you have not read. Anything the task needs that was not prepared (altimeter, radios, lights, minimums, autobrake, the FMC route) is yours: check each one before relying on it.

## Continue to the task endpoint

Reading the state, a skill or a chart is preparation, not completion. After a document read, refresh `read_flight_deck_indications` and carry out the next required phase step, verifying each change before continuing. Do not finish with a promise to load a skill or begin the approach: load it and continue the read–act–verify loop while the task remains unfinished.

Stop only when the requested endpoint is verified, the run ends, or a specific blocker prevents further safe progress. For a land-and-stop task, verify `on-ground`, ground speed near zero and position at the destination; review the current checklist, mark only confirmed items and report anything unresolved. A phase label such as `approach` is not completion. Some tasks end earlier, for example at cruise: respect that endpoint and do not fly past it.
