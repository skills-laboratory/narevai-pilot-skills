---
name: climb-and-cruise
description: Climb the Zibo 737-800X after takeoff and hold cruise with LNAV, VNAV, autothrottle and CMD A: 400 ft handover, gear and flap retraction, MCP altitude management, route progress and fuel checks. Use when airborne below or at the cruise altitude.
---

# Climb and cruise

Use `fly-the-airplane` for how the tools work. Read the `climb_cruise` checklist section (`read_checklist` with `category`) when you reach it, and mark items as you confirm them.

## Climb

- At or above **400 feet height AGL**, once stable, trimmed and following flight director guidance, complete the CMD handover in `takeoff-straight-ahead` without waiting for flap retraction. Use `hdg-sel` and `lvl-chg` with confirmed HDG SEL / MCP SPD for the straight-ahead climb; engage `cmd-a` only if off and verify CMD and modes immediately. Select `lnav` when route geometry permits and `vnav` after flaps/slats are retracted and FMC performance is valid; verify the resulting FMA, not just armed modes. Never press an already-engaged CMD again.
- `pitch-control` is the elevator position, held and sensitive (see `fly-the-airplane` and `takeoff-straight-ahead`): make small changes (any value from -1 to 1 is accepted), read `flight-instruments.pitch` and its `trend` at once and decide the next input from what the airplane did, and stop writing it once `cmd-a` is on.
- Retract the gear after a positive rate of climb (`gear` value 0) and the flaps on the scheduled speeds. Verify climb thrust.
- Manage the MCP altitude through the climb so that it permits the cleared climb and keeps the restrictions. If the airplane levels at the MCP altitude, selecting a higher one may need the appropriate VNAV resumption. Confirm the resulting mode before continuing.
- Set standard pressure at the transition altitude: `cockpit_altimeter` `std`, then read `altimeter.std-selected`.

## Cruise

- Maintain LNAV and VNAV. Check the route progress with `latitude` / `longitude` against the chart or the FMC PROG page, the fuel remaining, and the weather at the destination (`read_group` on `weather`).
- The flight phase becomes `cruise` when the airplane is level at altitude. A task that ends at cruise stops by itself: do not continue past what it asks for.
- Plan the descent before the top of descent: continue with `descent-and-approach`.
