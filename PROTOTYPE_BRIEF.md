# Build brief v4 — Case Management Workbench clickable prototype

> v4 changes vs v3 (after the third review): `pending_link` on verification items; `brief_composed[]` (engine-cited brief) beside the authored `eight_am_line`; `reachability.window_short` and per-number `label`; channel can be "Call first — clinical window closes in N h"; newborn-claims item; simulation is 48 members with a 20-seed sweep, `attempts_today` per row, `escalated_n`, `seed`; assessment classes are now four (13 pre-fill / 7 pre-fill-verify / 6 derivable / 14 must-ask). Build from the v3 `workbench_state.json`.

You are building a clickable, single-file HTML prototype of a **Case Management Workbench** for a US health-plan nurse case manager, for a product case competition (HiLabs APM caselet). It must look like a finished product, not wireframe boxes, and it must be driven entirely by the attached `workbench_state.json` (synthetic, fictional patients — no real PHI).

If a v1/v2 build already exists in this repo, upgrade it to this brief rather than starting over; the data file's field names changed (see Data model) — do not keep any old field name.

## Hard constraints

- **One self-contained `index.html`**: inline CSS and JS, no build step, no server, no external calls except (optionally) a Google Fonts link and one icon library from cdnjs. Opens from disk by double-click.
- **Data source = `workbench_state.json`, embedded** in the HTML as `<script type="application/json" id="state">` (paste the file's contents verbatim). All screen content comes from it. Do not invent patients, numbers, dates or drugs.
- **Three screens, routed by hash** (`#queue`, `#member/<id>`, `#outreach/<id>`), persistent left nav, top bar: "Mon 9 Nov 2026 · 8:04 am · Maria Alvarez, RN · shift 08:00–17:00 · evening team 17:00–20:00 · 48 active cases · 6 new alerts".
- **Human in the loop is visible everywhere**: anything AI-produced carries an `AI draft` chip; it becomes `Clinician confirmed · Maria · 08:xx` only through a Confirm button on the **outreach screen after the attempt is logged as Reached** (never on the member 360), and every confirmation appends a line to a visible audit log.
- Desktop first (1440 px), no breakage at 1280. Keyboard-accessible buttons. No emoji. Colour never alone (always a label).

## Data model (from `workbench_state.json`)

- `today`, `case_manager {name, shift, active_cases, new_alerts_since_friday, capacity_attempt_slots (18), capacity_today_members (9)}`
- `queue[]` — already in display order (deadline members first, soonest window first; then by priority). Per patient: `patient_id, name, age, plan, program, priority (0–100), priority_reasons[3] (Urgency / Acuity / Signals, each with its points and reason), hours_to_nearest_window (number or null), deadline_today (bool), bucket ("Today (deadline)" | "Today" | "Tomorrow (bumped by capacity)" | "This week"), confidence (0–100), confidence_reasons[], confidence_label ("complete" | "adequate" | "sparse picture — verify by conversation"), verify_on_call[] {type, severity, sources[], text, action, confidence, pending_link (bool — item exists only because an unconfirmed linked record was read; render a grey chip "pending link confirmation")}, signals[] {text, severity, event_date, citation}, reachability {best_number, best_score, evidence[], channel (may start "Call first — clinical window closes in N h" — then the text composer is disabled with that reason), window, window_short, daytime_number, evening_handoff (bool), sms_consent, all_numbers[] {number, score, evidence[], text_allowed, harvested, label (named other person, e.g. "wife, Ruth")}, representative, interpreter (bool)}, next_best_action[] (ordered steps), eight_am_line (authored prose), eight_am_line_source, brief_composed[] {text, cite} (the engine's cited sentences — render as the "Evidence" popover under the 8 am line, one row per sentence with its citation), linked_records[] {member_id, product, evidence[], status}, linked_ids[], newborn_claims_excluded (int), own_utilisation_claims (int)`
- `caseload_simulation {seed, members (48), deadline_members, capacity_overflow, escalated_members[], escalated_n, attempts_planned, overflow_rule, shown_seed_note, sweep_20_seeds {deadline_members, today, tomorrow_bumped, this_week, capacity_overflow, due_today_share, escalated, seeds_with_bump, seeds_with_overflow} (strings like "5–11"), today_member_slots, buckets{}, due_today_share, histogram{}, rows[] {member, tier, hours, priority, deadline_today, bucket, attempts_today}}`
- `tables` — raw records per dataset, each with `member_id` (use `linked_ids` to gather a person's records; records whose member_id is in `linked_records` are MASKED until the link is confirmed; a claim whose member_id is `<id>-K` is a dependent (newborn) billed under the mother's ID — list those under "Newborn encounters (excluded from her utilisation)", never in her utilisation counts): `member, consent, adt (has pid_phone, dg1_text, discharge_disposition, facility_mrn), pharmacy (has dur_override, claim_status, visible_in_extract, visible_today, gap_days), med_history, claims (has visible_today, received_date, status), um_auth, contact_history, other_contacts (has phi_ok), clinical (has stopped_meds, narrative_summary_received, repeat_labs_ordered, language_noted), labs, hra_sdoh (has status_flag, age_months), risk_scores, care_gaps (has kind, due_datetime, days_left, status), case_notes, call_center (caller_ani), rpm_home (vendor_alert, received_time), benefits, appointments`
- `assessment_map[]` {section, field, source_rule, dataset, cls ("pre-fill" | "pre-fill (verify on call)" | "derivable" | "must-ask")}, `assessment_counts` (13 / 7 / 6 / 14)
- `rules{}` — plain-English scoring rules for the "How is this scored?" popover (priority, bucket, confidence, reachability, clinical_rule_content, human_in_loop).
- `imperfections[]` — reference only; never display as a list.

## Screen 1 — Prioritised work queue (`#queue`)

- Header tiles: *Deadline today* (`caseload_simulation.deadline_members`, i.e. the whole caseload, not just the five) · *Over capacity* (`capacity_overflow`; sub-text `escalated_n` + " deadline members get one attempt today — evening team takes attempt 2") · *New alerts since Friday* (6) · *Attempt slots* (`attempts_planned` + " / 18 planned") · *Sparse pictures* (count with `confidence < 50`).
- One list of cards in `queue` order. Each card: bucket label; name, age, plan, program; **countdown** from `hours_to_nearest_window` ("2 h", "9 h", "4 days", or "no window" when null); **priority bar in three segments** labelled Urgency / Acuity / Signals (parse the points from `priority_reasons`); **confidence bar** with its `confidence_label` (red < 50, amber 50–74, green ≥ 75); top two `signals` as chips (severity-coloured, with event date); `next_best_action[0]` with an `AI draft` chip and a `Start` button → `#member/<id>`.
- Below the five cards, a collapsed section "Rest of caseload (simulated, 43 members — seed N shown)" that renders `caseload_simulation.rows` (including `attempts_today`) as a compact table and the histogram as a small bar chart, with `overflow_rule` and a one-line "Across 20 seeds: deadline 5–11, bumped 0–3, overflow in 3/20" from `sweep_20_seeds`. This is what shows judges the capacity cut on a real caseload.
- Sort controls: by display order (default), by nearest window, by confidence ascending. Filter: "unreachable" (`reachability.best_score < 0`).
- "How is this scored?" → popover with `rules.priority`, `rules.bucket`, `rules.confidence`.

## Screen 2 — Member 360 (`#member/<id>`)

Left 2/3 content, right 1/3 action rail.

**Above the fold:**
1. Identity header: name, age, sex, plan, program, PCP, language (from `member`); `linked_records` as a chip "Possible linked record — masked until confirmed" with its `evidence` on hover; `reachability.representative` text beside the phone numbers (red "No representative on file" when it starts with "None"); interpreter flag when `reachability.interpreter`.
2. **The 8 am line** (`eight_am_line`), labelled "What matters today · AI summary", `AI draft` chip (no Confirm here — it is confirmed on the outreach screen); an "Evidence" link opens `brief_composed[]` as a two-column list (sentence · record cited).
3. **Confidence bar** with `confidence_label` and a "why" popover listing `confidence_reasons`.
4. **Verify on the call** — `verify_on_call[]` as cards: type, severity, the sources side by side, the question (`text`), the coordination action (`action`), a confidence %; a grey chip "pending link confirmation" when `pending_link`. No Confirm/Dismiss buttons here; a note "Verified on the outreach screen after the member is reached".
5. **Safety signals** (`signals[]`) with severity and event date.

**Tabs below the fold:** Timeline · Medications · Clinical & labs · Care gaps · Contact & consent · Notes.
- *Timeline*: merged chronological list from `adt, claims, pharmacy, contact_history, clinical, labs, rpm_home, um_auth, appointments` across `linked_ids`, newest first. Records belonging to a linked (unconfirmed) record are shown as masked rows ("Linked record — content masked until confirmed"). Every item carries a **freshness stamp**: event date, received date, band — green ≤ 7 days, amber 8–45, grey > 45 (claims: green ≤ 45, amber 46–90, grey > 90). Claims with `visible_today == false` render greyed with "Not yet received — expected <received_date>". Newborn `-K` claims render in a distinct style labelled "Newborn — billed on mother's ID (excluded from her utilisation)".
- *Medications*: `pharmacy` rows with `days_since_fill`, `gap_days`, `claim_status`, `dur_override`; a reversed status renders "Not picked up (returned to stock)"; a rejected status renders "Rejected — never dispensed"; a `dur_override` renders a red chip "DUR override at the counter" with the code text. Rows named in a `verify_on_call` item are highlighted. Show `med_history` beneath as "Medication history (Surescripts)".
- *Clinical & labs*: `clinical` documents with banners when `narrative_summary_received == "No"` ("Summary not received — day N") and when `repeat_labs_ordered` is "Not stated"; `stopped_meds` shown as a STOPPED list; `labs` table; a lab with blank `loinc` renders "Present, unreadable (scanned PDF)" with a button `Extract with AI` that reveals the value text with an `AI draft` chip; `rpm_home` rows with `vendor_alert` and `received_time`.
- *Care gaps*: `care_gaps` as tasks with `due_datetime`, `days_left`, `kind`; rows whose status starts with "Not triggered", "No result", "Shows", "Flagged", "Open in plan system" get the tag "Would be missed by claims-only logic".
- *Contact & consent*: `contact_history` + `call_center` + `consent` row + HRA `best_time` + `other_contacts` with their `phi_ok` text.
- *Notes*: `case_notes`.

**Right rail:** `next_best_action` as an ordered list (each step `AI draft`); reachability summary — `daytime_number` labelled "Now (your shift)" and `best_number` labelled "Evening team" when `evening_handoff` is true, else a single best number; `channel`; `window`; `sms_consent`; buttons `Start outreach` → `#outreach/<id>`, `Create task`, `Mark reviewed`.

## Screen 3 — Outreach & documentation (`#outreach/<id>`)

Three columns.
- **Left — Reach:** `reachability.all_numbers` ranked with scores (floor −50) and evidence lines; chips: `Text allowed` (green) when `text_allowed`, `Call only — harvested, no SMS consent` (amber) when `harvested`, `No SMS consent` (grey) otherwise; `representative` text; interpreter flag; the daytime/evening split with a `Hand to evening team` button that logs a task. `Send text` is enabled only for numbers with `text_allowed`; the pre-filled text contains no clinical content ("Hi, this is your care manager at <plan>. Please call us at 555-0100 when you can."). `Log attempt` form (outcome: Reached / Voicemail / No answer / Not in service / Wrong person / Relay asked to call back) that appends to the Contact tab and, on Reached, unlocks the middle and right columns.
- **Middle — The call (AI draft):** the `verify_on_call` items become the talking points, each with a `Confirm` (what the member said, free text) and `Could not verify`; plus 2–3 scripted questions derived from `signals` (e.g., severe-range BP → "headache, visual change, RUQ pain, breathlessness?"; MOUD gap → "when did you last take buprenorphine?"); confirming an item writes an audit line and, if the item is of type identity, unmasks the linked record's rows across the app.
- **Right — Documentation:** render `assessment_map` grouped by `section`, 40 fields; `pre-fill` fields pre-populated from data (diagnoses, utilisation, appointments, auths, gaps, care team, coverage, employment) each with an `AI draft` chip; `pre-fill (verify on call)` fields pre-populated the same way but with an amber `verify` chip and a one-line prompt (meds, allergies, housing, food, transport, phone minutes, contact details); `derivable` fields filled after the call from confirmed items; `must-ask` fields empty inputs. A `Confirm section` button per section (never a global confirm-all). Counter: "Confirmed 0 / 40 · 13 pre-filled · 7 to verify · 6 derived · 14 to ask". Care-plan goals box. Call note auto-composed from confirmed items and sections. One `Confirm & sign` → toast "Written once to assessment, care plan, note and audit form".

Audit log drawer (bottom, collapsible) listing every confirmation with time and the evidence shown.

## Patient walkthroughs the judges will click (make these work end to end)

1. **Priya Lambert (SYN-10688)** — queue order 1, "2 h", deadline bucket, confidence 100. Signals: severe-range home BP 164/102 received Mon 06:00; PHQ-9 14 with no follow-up booked. Care gaps show the plan's severe-BP rule and "PND-E not triggered — office note not in the ECDS feed" tagged *missed by claims-only logic*. Timeline shows three newborn `-K` claims. Outreach: cell 555-0156, Text allowed; `next_best_action[0]` says call now with the symptom screen; log Reached → scripted BP questions → confirm → book BH follow-up (task) → sign.
2. **Marcus Reyes (SYN-10590)** — order 2, "4 h", confidence 40 sparse. 360 header: "Possible linked record SYN-10112 — masked"; timeline rows from SYN-10112 masked; overdose claim greyed "expected 2026-11-25". Outreach: 555-0131 "not in service"; 555-0177 ranked first, chip *Call only — harvested, no SMS consent*; Dana 555-0190 listed with "may ask member to call back only"; log "Relay asked to call back", then "Reached"; confirm the identity item → the SYN-10112 rows unmask and the timeline shows 7 ED visits; confirm the HRA/Z59.01 item; book the MOUD visit.
3. **Harold Benning (SYN-10041)** — order 3, "9 h". 360 shows the "Summary not received — day 4" banner, `repeat_labs_ordered` "Not stated", two verify items (lisinopril and KCl dispensed Friday with DUR override chips, STOPPED on the CCD). Right rail shows daytime number 555-0102 ("Now") and 555-0118 ("Evening team"). Outreach: log No answer on 555-0102 → `Hand to evening team` logs the task → (demo) log Reached on 555-0118 at 18:05 → confirm both items → book cardiology → sign.

## Visual direction

- Clinical-calm: off-white background, one accent (deep teal or navy), severity colours only on chips and bars (red/amber/green, always with a text label), generous whitespace, 14–15 px body, Inter or system sans.
- Every number traceable: hovering a score, a chip or a countdown shows where it came from.

## Deliver

- `index.html` (single file), a 10-line README with how to open it and the three walkthroughs, and `workbench_state.json` beside it.
- Open the file in a headless browser and save screenshots of the three screens at 1440×900 into `screenshots/` (queue.png, member360.png, outreach.png); check nothing overflows at 1280 width.
- Commit and push to the repo after each screen works.
