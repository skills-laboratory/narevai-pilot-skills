---
name: climb-and-cruise
description: Climb the Zibo 737-800X after takeoff and hold cruise with LNAV, VNAV, autothrottle and CMD A: 400 ft handover, gear and flap retraction, MCP altitude management, route progress and fuel checks. Use when airborne below or at the cruise altitude.
---

# Climb and cruise

Use `fly-the-airplane` for how the tools work. Read the `climb_cruise` checklist section (`read_checklist` with `category`) when you reach it, and mark items as you confirm them.

## Climb

- Use a fresh `flight-instruments.height-agl` reading for the **400-foot** autopilot gate, not the flight phase or a positive vertical speed. Below it, or while unstable, continue the manual takeoff scan in `takeoff-straight-ahead`.
- At or above 400 feet AGL, once stable, trimmed and following flight director guidance, select `lnav` when the route geometry permits, otherwise `hdg-sel`. Select VNAV after the flaps and slats are retracted. If `fma-pitch` stays blank or VNAV is only in `fma-pitch-armed`, use `lvl-chg` and confirm active `MCP SPD`, even with a route loaded. Verify a safe selected speed and the cleared altitude before selecting the pitch mode; an armed mode is not controlling pitch.
- Engage `cmd-a` in a separate call, then read the command light and active FMA modes. Once CMD is on, release `joystick-override` and stop writing yoke inputs. Check CMD and the climb again on the next scan: a verified button press is not proof that engagement persists. If CMD drops, resume manual control with verified joystick override; stabilize before another handover attempt, rather than repeatedly pressing VNAV or CMD during a sink.
- `pitch-control` is the elevator position, held and sensitive (see `fly-the-airplane` and `takeoff-straight-ahead`): make small changes, read `flight-instruments.pitch` and its `trend` at once and decide the next input from what the airplane did. With falling airspeed or a stall warning, restore required thrust, smoothly reduce angle of attack and keep the wings level; do not chase altitude with full back pressure. Recover speed and a stable climb before mode changes or flap cleanup.
- Retract the gear after a positive rate of climb (`gear` value 0). Retract flaps only at the scheduled speeds with airspeed recovering and the climb stable; 400 feet is not permission to retract at any speed. Hold the current flap setting during a stall warning or decaying-speed recovery. Verify both N1 values against the required climb thrust; autothrottle `ARM` alone does not control thrust.
- Manage the MCP altitude through the climb so that it permits the cleared climb and keeps the restrictions. If the airplane levels at the MCP altitude, selecting a higher one may need the appropriate VNAV resumption. Confirm the resulting mode before continuing.
- Set standard pressure at the transition altitude: `cockpit_altimeter` `std`, then read `altimeter.std-selected`.

## Cruise

- Maintain LNAV and VNAV. Check the route progress with `latitude` / `longitude` against the chart or the FMC PROG page, the fuel remaining, and the weather at the destination (`read_group` on `weather`).
- The flight phase becomes `cruise` when the airplane is level at altitude. A task that ends at cruise stops by itself: do not continue past what it asks for.
- Plan the descent before the top of descent: continue with `descent-and-approach`.
