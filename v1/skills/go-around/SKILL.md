---
name: go-around
description: Go around from an ILS or other approach in the Zibo 737-800X. Use when approach guidance fails, the approach is unstable by 1000 ft AGL, the required visual references are missing at the minima, or autoland does not arm or flare.
---

# Go-around

Decide early. If guidance fails, the approach becomes unstable, or the visual references are absent at the applicable minima, go around. Never continue descending simply because a command was accepted: establish confirmed guidance or take control.

1. `to-ga-left` in `cockpit_engines_and_thrust`. Verify thrust and the FMA (`fma-pitch`, `fma-roll`, `fma-autothrottle`).
2. Read both CMD lights; TO/GA guidance is not an engaged autopilot. If both are off, verify `joystick-override` on and fly the flight director with small inputs. Read pitch, pitch trend and airspeed immediately after each change; ease the input before the predicted attitude at the next command reaches the target. Zero elevator does not instantly stop a rising nose. Keep this scan ahead of chart or checklist work.
3. Follow the published missed approach: lateral path, altitude and configuration schedule. Prepare it before final from the chart (`list_documents` with kind `chart`) or route skill; do not invent the altitude. Before selecting HDG SEL or re-engaging CMD, set and read back the required heading and altitude and verify the climb pitch mode; do not reuse a departure heading left in the MCP.
4. Retract the gear after a positive rate of climb and the flaps on the scheduled speeds. Set the missed approach altitude on the MCP.
5. When stable and following the verified guidance, use the neutral-controls CMD handover in `takeoff-straight-ahead`: zero and read back pitch, roll and rudder, engage only an off channel, verify CMD and FMA, then release `joystick-override` and check the next flight trend. Stop writing manual controls while CMD is on. The flight phase returns to `climb`. Read the situation again with `assess-flight-state`, then prepare another approach or tell the user you cannot continue safely.

Issue each mode action once, observe its effect, and decide the next action from the resulting airplane state.

A `set-stabilizer-trim` call can block for tens of seconds. Do not leave corrective pitch or roll inputs unattended through it during an unstable recovery: establish control and a stable trend first, then verify the actual trim and flight state after the call, including a timeout.
