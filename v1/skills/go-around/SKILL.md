---
name: go-around
description: Go around from an ILS or other approach in the Zibo 737-800X. Use when approach guidance fails, the approach is unstable by 1000 ft AGL, the required visual references are missing at the minima, or autoland does not arm or flare.
---

# Go-around

Decide early. If guidance fails, the approach becomes unstable, or the visual references are absent at the applicable minima, go around. Abandoning a high or offset intercept to reposition is also a go-around: establish the missed-approach climb before setting up another join. Raising the MCP altitude or adding manual thrust alone does not prove that a descent mode has changed to a climb.

1. `to-ga-left` in `cockpit_engines_and_thrust`. Verify thrust and the FMA (`fma-pitch`, `fma-roll`, `fma-autothrottle`) and read pitch, airspeed, vertical speed and `height-agl` immediately. If descent persists or terrain clearance is shrinking, establish the escape climb rather than waiting for the higher MCP altitude to take effect. A verified altitude setpoint is not a verified climb.
2. If neither CMD is on, control the airplane immediately, even if the FMA still says `G/S` / `VOR/LOC`. Use the predictive input scan in `fly-the-airplane` to stabilize pitch and bank, with a fresh read after each axis change before configuration or another mode action. Zero elevator is not an attitude hold, and a balloon's positive vertical speed alone does not confirm a controlled missed approach.
3. Follow the published missed approach: lateral path, altitude and configuration schedule. Do not turn away to build another intercept until positive climb and safe terrain clearance are confirmed; keep the primary-flight scan through the transition, not just position and DME reads. The chart is a `flightsim-pilot` document (`list_documents` with kind `chart`); do not invent the altitude. A route skill may list it.
4. Retract the gear after a positive rate of climb and the flaps on the scheduled speeds. Set the missed approach altitude on the MCP.
5. The flight phase returns to `climb`. Read the situation again with `assess-flight-state`, then fly another approach with `descent-and-approach` or tell the user you cannot continue safely.

Issue each mode action once, observe its effect, and decide the next action from the resulting airplane state.
