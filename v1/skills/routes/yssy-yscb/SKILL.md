---
name: yssy-yscb
description: Route details for Sydney YSSY to Canberra YSCB in the Zibo 737-800X: runway 16R, SID GROOK2 WOL, STAR LECE1W, the FMC legs, default performance values, the Canberra ILS Z runway 35 chart numbers and the missed approach. Use when the flight departs YSSY, arrives at YSCB or passes the fixes between them.
---

# Sydney to Canberra

Facts about the route only. How to fly each part is in the phase skills (`fmc-setup`, `before-takeoff`, `takeoff-straight-ahead`, `climb-and-cruise`, `descent-and-approach`, `landing`, `go-around`). Confirm every number against the current chart (`list_documents` with kind `chart` and the ICAO, then `read_document`); it can be a PDF that some models cannot read, in which case use the numbers here and tell the user.

## Departure

- Runway 16R at YSSY, heading about 155. SID **GROOK2** with the **WOL** transition (the database has GROOK2, not GROOK1; transitions are KADOM, STUIE, WOL), then H65 to LEECE.
- Sydney elevation and the clearance are not known to any tool; without ATC use an initial altitude consistent with the published departure restrictions and terrain.

## Arrival

- Select the approach named by the task or arrival procedure before reading its profile. **ILS-Y uses CB DME** (CB 116.7 MHz); **ILS-Z uses ICB DME** paired with the ILS. The profile below is **Z only**. For ILS-Y, read the Y chart and use its distances, altitude restrictions, minima and missed approach; do not substitute Z profile distances even though both use the same ILS frequency and inbound course.
- STAR **LECE1W** and **ILS Y 35** joined at MOMBI. The FMC legs run: WOL, LECE, AKLIM, EKASA, SLICK, FOXLO, MOMBI, DAMKO, RW35, without a discontinuity. After activating the route there is a discontinuity between WOL and LECE: close it as described in `fmc-setup`.
- Default performance values when nobody gives others (there is no OFP): ZFW 126.0 (thousands of pounds, from the weight readings), reserves 2.0, cost index 30, cruise FL220, takeoff flaps 5.
- The leg is short. From FL220 you need roughly a 3.5–4° path to be at about 5000 ft by about 10 DME from ICB; LVL CHG alone is too shallow. Start the descent early, use `speedbrake` value 2, and be at about 160 kt before the FAF. Do not let the speed ride to 250 or more.

## Canberra ILS Z RWY 35

Chart `SCBII01`, 19 Mar 2026; confirm against the current chart.

- ILS/DME **ICB, 109.5 MHz**, final course **348° magnetic**. The DME is paired with the ILS, so `nav-1-dme` is the distance to ICB. The CB VOR/DME is 116.7 and is not the ILS.
- Required: DME or GNSS for the ILS, DME for LOC only.
- Fixes by ICB DME: DAMKO 4, MOMBI 9.1 (holding, max 170 kt at 5100 ft, 210 kt at 5600 ft), KATIA 11.1, MENZI 13.1. The 3° path is 3140 ft at 4 DME, 3460 at 5, 3810 at 6.1, 4760 at 9.1, 5050 at 10 and 5400 at 11.1. From 5000 ft level the glideslope is captured from below at about 10 DME.
- Minima: ILS CAT I 2170 ft (301 ft above the threshold) with RVR 750 m; LOC only 2700 ft. Aerodrome elevation 1887 ft, threshold 35 elevation 1869 ft.
- Missed approach: track 348°, climb to 5100 ft or as directed.
- Radios: ATIS 116.7 / 127.45, approach 124.5, tower 118.7 (no ATC is available here).
- Tune 109.5 into Nav 1 and Nav 2 with course 348. On the simulator `set-nav-1-course` did not always take and a zero course biased APP and contributed to a lateral excursion after touchdown: read it back and confirm 348.
- A flown attempt on ILS Y 35 lost the glideslope because of unnecessary `cmd-a` / `cmd-b` presses on final: see `ils-final-hands-off`.
