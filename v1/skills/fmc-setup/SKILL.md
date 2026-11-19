---
name: fmc-setup
description: "Check or enter the route and performance data in the Zibo 737-800X FMC through the CDU: reference airport, IRS position, route, SID, STAR, approach, activation, discontinuities, PERF INIT and TAKEOFF REF. Use when the FMC has no route, the route or performance data look wrong, or before takeoff."
---

# FMC setup

Use `fly-the-airplane` for how the tools work. Read the CDU first (`read_group` on `fmc-cdu`), then work with `cockpit_fmc_cdu`: page keys, line select keys, `exec`, and `cdu-type` for text (letters, digits, space, `.`, `-`, `/`). Take every value from the flight plan document (`list_documents` with category `flight` and kind `plan`, then `read_document`) or from the route skill for the route you are flying; do not invent values. Clearance (ATC) is not available from any tool: it comes from the user. If the CDU already holds the route, verify it and change only what is wrong.

## How the CDU behaves

- `CLR` deletes **one character** and an error message (`INVALID ENTRY`, `NOT IN DATA BASE`) hides the text you typed but does not remove it. Press `clr` until `scratchpad` is empty before you type again.
- Check the CDU after every entry. `exec` only when the light is on.
- Line select keys: the key next to a field is `lsk-<row><l|r>` (row 1 at the top, 6 at the bottom, `l` left, `r` right). Match both row and side to the displayed label; a right-aligned field is not entered with the left key on that row. On list pages (SIDs, STARs, approaches, transitions) press the key beside the name you want. Use `next-page` and `prev-page` when the name is not on the page. Selected entries are marked `<SEL>`.
- `<INDEX` (`lsk-6l`) on a DEP/ARR page leads to the INIT/REF INDEX, not to the DEP/ARR index; press `dep-arr` to get back to the DEP/ARR index.
- If the CDU does not accept an entry, read the message, scratchpad and target field; check format, row and side before correcting and retrying. On the ground, do not depart with required setup incomplete. Airborne, an unresolved CDU field is not a reason to stop flying: refresh the flight-deck indications and follow `descent-and-approach` or the current phase procedure. Report what is missing and continue only with verified guidance and performance data; if they cannot be established safely, fly the applicable missed approach or recovery rather than end while inbound.

## Verify an existing route

Open `legs` and read the lines. They must follow the SID, the route and the STAR from the plan without a `ROUTE DISCONTINUITY`. Check PERF INIT and TAKEOFF REF as in steps 9 and 10. The FMC works in thousands of pounds, so compare the ZFW with the airplane: ZFW in thousands of pounds = (`takeoff-data.gross-weight` − `takeoff-data.fuel-total`) × 2.2046 / 1000.

## Enter a route from an empty CDU

1. **Open the FMC.** On MENU press `lsk-1l` (<FMC), then `lsk-6r` (POS INIT).
2. **Reference airport.** Type the departure ICAO, press `lsk-2l`.
3. **IRS.** In `cockpit_irs` set `irs-left-mode` and `irs-right-mode` to nav. Back on POS INIT type your present position as hemisphere, degrees, minutes with one decimal, for example `S3355.8E15110.3`, and press `lsk-4r` (SET IRS POS). Take the position from the airplane readings. Alignment takes minutes.
4. **Route.** Press `lsk-6r` (ROUTE). Type the origin and press `lsk-1l` (ORIGIN), the destination and `lsk-1r` (DEST), then the runway without the `RW` prefix (for example `16R`) and `lsk-3l` (RUNWAY).
5. **Departure.** `dep-arr`, `lsk-1l` (DEP). The page lists SIDs on the left and runways on the right. Press the key on the SID row from the plan, then on its transition row. The database may spell a SID differently from the chart (the number differs): read the list.
6. **Arrival.** `dep-arr`, `lsk-2r` (ARR). STARs are on the left and approaches on the right, over several pages. Press the right key on the approach row (and a transition if the plan has one) and the left key on the STAR row. Both must show `<SEL>`. No `exec` is needed until the route is activated.
7. **Activate.** `rte`, `lsk-6r` (ACTIVATE), `exec`.
8. **Close discontinuities.** On `legs` a `ROUTE DISCONTINUITY` between the SID and the STAR is closed by pressing the line select key next to the fix after it (it is copied to the scratchpad), then the key next to the `*****` line above it, and `exec`. Read the legs again against the plan.
9. **Performance.** `init-ref`, then `lsk-6l` (INDEX), then `lsk-3l` (PERF). **The FMC works in thousands of pounds, not kilograms** (the fuel on the page, `11.4`, is 5171 kg). Convert from the airplane readings as above, for example 57 159 kg → `126.0`. Enter ZFW (`lsk-3l`); the gross weight on the page must then match the reading × 2.2046 / 1000. Also reserves (`lsk-4l`), cost index (`lsk-5l`) and cruise altitude (`lsk-1r`), then `exec` if the light is on. Use the values from the plan, from the route skill or from the user: there is usually no OFP and no ATC. The cost index only changes the VNAV climb, cruise and descent speeds, and the cruise altitude must be above the altitude you set in the MCP. Entering kilograms gives INVALID ENTRY.
10. **Takeoff data.** `init-ref`, `lsk-6l` (INDEX), `lsk-4l` (TAKEOFF). Type the takeoff flaps (`lsk-1l`) and the CG in % MAC (`lsk-3l`; from the load sheet, or from `takeoff-data.actual-cg` if you have none). The page then shows suggested V1, VR and V2 and the calculated trim: press `lsk-1r`, `lsk-2r` and `lsk-3r` once each to accept them (a second press on the same key copies the value back into the scratchpad). Enter only the CG in `lsk-3l`, in % MAC such as 19.4; the trim shows next to it once the FMC has calculated it and is not typed. Check `takeoff-data`: `fmc-cg` must agree with `actual-cg`, `v1-set`, `vr-set` and `v2-set` must be non-zero and match the page, and `trim-calculated` is the stabilizer trim for takeoff. Set the flap lever to the takeoff flaps (`flaps`) and the trim with `set-stabilizer-trim` (the value of `trim-calculated`); `flaps-match-fmc` and `trim-matches-fmc` then become true. They are checks, not entries: before you set them they read false.

Continue with `before-takeoff`.
