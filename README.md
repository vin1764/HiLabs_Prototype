# Case Management Workbench — clickable prototype

Single-file prototype of a nurse case manager's workbench (HiLabs APM caselet). Synthetic data only — no real PHI.

**Open it:** double-click `index.html` (no server, no build). All content comes from `workbench_state.json`, embedded verbatim in the `<script id="state">` block. Routes: `#queue`, `#member/<id>`, `#outreach/<id>`, `#audit`.

**Walkthroughs for judges**

1. **Harold Benning (SYN-10041)** — rank 1, "9 h left". On Member 360 click *Confirm resolution* on all three conflicts: confidence rises 10 → 23 → 37 → 50 and each confirmation lands in the audit drawer. *Start outreach* → best number 555-0118 (Ruth, around 18:00) → set outcome *Reached* → *Log attempt* → tick talking points → *Confirm all pre-filled* per section (30/40) → type the 10 conversation fields → *Confirm & sign*.
2. **Marcus Reyes (SYN-10590)** — rank 2, "1 day" (FUA closes tomorrow 5 pm). Header shows *1 linked record* (termed SYN-10112); Timeline lists 7 ED registrations across both records and the 3 Nov overdose claim greyed "Not yet received — expected 22 Nov 2026". Outreach: 555-0131 marked *Not in service*, 555-0177 best (ADT PID + member called from it), shelter case worker 555-0190 second; *Send text* first.
3. **Priya Lambert (SYN-10688)** — rank 3, "4 h left". Signals: home BP 162/108 from the vendor platform and PHQ-9 14. Care gaps tab tags PDS-E ("Not triggered — screen exists only in EHR") and the ACOG BP rule "Would be missed by claims-only logic"; Timeline shows the newborn (-K) claims styled "Newborn — billed on mother's ID".

Screenshots of the three screens at 1440×900 are in `screenshots/`. Checked with headless Chromium: no horizontal overflow at 1280.

**Calls made where the brief left room** — the top-bar clock starts at 8:04 am (as specified) and advances in real time; the queue countdown uses `queue.hours_to_nearest_window` (falls back to the nearest care gap's `days_left` when no window is open); confidence lifts on confirmation are the conflict's share of the capped −40 penalty (so Harold lands on exactly 50); *Dismiss* leaves confidence unchanged; the "Show only unreachable" filter is empty with this dataset because every member has a number scoring above 0 (explained on screen); *Send text* stays enabled when there is no opt-in on file but shows the TCPA warning and records the override in the audit log; the safety-signal strip sits between the confidence bar and the conflict cards so all five above-the-fold items fit in 900 px; pharmacy rows have no receipt date, so their freshness stamp uses the fill date ("adjudicated at fill"); talking points, text templates and the per-patient pre-filled narrative fields are hand-written from the data file and cite their source rows. State lives in memory — reload resets the demo.
