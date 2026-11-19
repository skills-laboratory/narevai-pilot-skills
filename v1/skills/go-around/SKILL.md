---
name: go-around
description: Go around from an ILS or other approach in the Zibo 737-800X. Use when approach guidance fails, the approach is unstable by 1000 ft AGL, the required visual references are missing at the minima, or autoland does not arm or flare.
---

# Go-around

Decide early. If guidance fails, the approach becomes unstable, or the visual references are absent at the applicable minima, go around. Never continue descending simply because a command was accepted: establish confirmed guidance or take control.

1. `to-ga-left` in `cockpit_engines_and_thrust`. Verify thrust, the FMA (`fma-pitch`, `fma-roll`, `fma-autothrottle`) **and both CMD lights** immediately. TO/GA, HDG SEL or ALT HOLD displayed with both CMD lights off is flight-director guidance only; the airplane will not automatically climb, turn or level at the MCP altitude.
2. If CMD is off, take manual control immediately using `fly-the-airplane`: confirm joystick override on and actual input feedback, keep wings near level and initially aim for about 15° nose-up, then adjust pitch to maintain the weight/configuration-appropriate go-around speed. Use small held-input changes with an instrument read after each; ease toward neutral as pitch approaches the target or its trend is already correcting. Do not wait for an uncoupled TO/GA label or use sustained full elevator to chase vertical speed.
3. Follow the published missed approach: lateral path, altitude and configuration schedule. Brief it before final from the current chart or route skill; do not invent the altitude or stop the attitude/speed scan to research it during recovery. Set the missed approach MCP altitude, but actively control level-off unless sustained CMD engagement is verified. If already above it, stabilise safely before correcting altitude rather than continue an uncontrolled climb or dive.
4. Apply the published go-around flap schedule for the current configuration, retract the gear after a confirmed positive rate and retract flaps further only at their scheduled speeds. Monitor actual speed, pitch and climb through configuration changes; do not select a clean speed while leaving an intermediate flap configuration indefinitely.
5. Once stable, use the **Manual control to CMD handoff** in `fly-the-airplane` if returning to automation: neutralise held inputs while override is on, verify override off, then engage CMD once and verify it remains engaged on the next read. If it does not, resume verified manual control rather than treating the FMA as an engaged autopilot.
6. Only after attitude, speed and the missed-approach path/level-off are under control, reassess with `assess-flight-state` and set up another approach with `descent-and-approach`, or report a specific safety blocker. A phase change to `climb` alone is not evidence of a successful recovery.

Issue each mode action once, observe its effect, and decide the next action from the resulting airplane state.
