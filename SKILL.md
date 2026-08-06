---
name: "ibd-companion"
description: "AI-assisted, local-first IBD recordkeeping for Crohn's disease and ulcerative colitis."
---

# IBD Companion

Use this skill only for the user's private IBD tracking, including Crohn's disease and ulcerative colitis. The SQLite database is the sole source of truth. Keep health data out of Git, MEMORY.md, ordinary notes, Notion, and other channels.

## Scope and safety

- This is a personal record and planning aid, not a diagnostic or prescribing tool.
- Never diagnose disease activity, infer causality, change medication, invent medical schedules, units, reference ranges, abnormal flags, or medical conclusions.
- Keep unknown information NULL. Preserve original wording for symptoms, factors, and clinician conclusions.
- Escalate urgent red-flag symptoms to professional care rather than offering routine trend analysis.
- Do not modify an external health log unless the user separately authorizes it.

## Storage and commands

Base tracker: `scripts/ibd_db.py`.

Care-context extension: `scripts/ibd_care.py`.

Both use `${IBD_DB_PATH:-${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/private/ibd/ibd.sqlite3}`. Opening through `ibd_care.py` applies migrations through V6, preserving legacy profile snapshots, review parents, and treatment-plan occurrences without copying medical facts.

On first use, either script creates an empty private database at that path if it does not already exist. Creating an empty database does not authorize recording medical facts or creating external reminders.

## Data model: separate meanings, separate sources

1. **Thin disease profile — durable global context only.** `disease_profile` stores confirmed disease type, earliest symptom date, an optional link to the actual diagnosis `checkups` record, and sparse timeless notes. It does not store current status, affected locations, a standalone diagnosis date, treatment state, symptom-baseline content, or dated medical events. Procedures, admissions, complications, and other dated facts belong in `checkups`/review history.
2. **Treatment and monitoring — what happened around an injection.** `injections` stores infusion facts. A blood draw is one `checkups(kind='lab')` row, optionally linked to the injection it monitors. Its values live once in `checkup_results`.
3. **Large reviews — one parent, many tests.** `review_episodes` is one diagnosis/large-review parent. `review_components` links its lab, endoscopy, CT, MRI/MRE, pathology, visit, or assessment checkups. A lab can simultaneously belong to an injection and a review without copying the lab or its results.
4. **Reports and observations — separate shapes.** Atomic comparable values belong in `checkup_results`; versioned narrative reports and private attachment references belong in `checkup_reports`.
5. **Plans — what is expected.** `treatment_plans` generates injections and optional injection-monitoring labs; `review_plans` generates calendar-month large-review parents. There is no parallel `followup` plan layer. A one-off visit or test is a direct `checkups` fact, while a multi-test review uses the `review` hierarchy.
6. **Confirmed medical conclusions — what a clinician explicitly assessed.** `checkup_assessments` is tied to a completed assessment-bearing checkup, which can be a component of a review. It is never generated from metrics or symptoms.
7. **Personal symptom baseline — comparison, not clinical status.** `symptom_baselines` stores draft, candidate, confirmed, and historical versions. Only one explicitly confirmed baseline is enabled; it must never be silently re-learned from recent logs.

## Profile and diagnosis workflow

- Use `ibd_care.py set-profile` only for disease type, earliest symptom date, diagnosis-checkup link, and small timeless notes. A dated procedure, admission, complication, symptom episode, or clinician conclusion must be recorded in its dated medical record and must not be duplicated in profile notes.
- Do not add a current-baseline field or copy baseline values into the profile. `current-summary` resolves the unique explicitly confirmed row from `symptom_baselines` dynamically.
- Create or retain the diagnosis as a completed `checkups` record first, then link it with `--diagnosis-checkup-id`.
- Do not create a diagnosis event from an old standalone date unless the user explicitly confirms the factual details. The V2→V3 migration preserves legacy profile fields in `disease_profile_legacy_snapshots` instead.

## Injection monitoring, review components, reports, and conclusions

- Each actual blood draw, visit, endoscopy, imaging test, or pathology test is one `checkups` record. A large review itself is a `review_episodes` parent, not a `checkups` row.
- Use one lab checkup for one actual draw. If it serves both an injection and a large review, keep its injection link and add one `review_components(role='lab')` link. Never copy its values into a second lab row.
- Create a review parent with `schedule-review`, then link its actual tests using `link-review-component`. Use `review-detail` to read the parent and every linked child.
- Report values belong in `checkup_results`. Use report-provided values, units, reference bounds, and abnormal flags only. Never fill a missing range or flag from a generic threshold.
- New databases seed only the medicine-agnostic starter metrics: CRP, ESR, HGB, WBC, PLT, and albumin. Drug-specific monitoring and any other metric must be explicitly adopted before a result can be recorded.
- Use `add-checkup-report` for endoscopy, CT, MRI/MRE, pathology, visit, or other report wording. Preserve original wording; optional findings/impression are transcriptions, not model-generated interpretations. An attachment reference must remain in private storage.
- After a completed review, record `record-checkup-assessment` on the completed assessment-bearing visit/checkup only when the clinician's conclusion or the user's explicit confirmed transcription supports it. Link that checkup to the review using role `assessment` or `visit`.
- A corrected conclusion creates a new assessment version and preserves the previous one. Never treat a lab result, symptom trend, or an unconfirmed report interpretation as a formal assessment.

## Custom metric workflow

1. When the user asks to record or track a metric, first run `ibd_db.py metric-list` and use an existing definition when it matches the same measurement.
2. If no matching definition exists, explain the needed fields and obtain the user's explicit confirmation of its display name, numeric/qualitative type, and optional default unit. Then create it with `ibd_db.py metric-add` using a stable lowercase `snake_case` code.
3. Create or locate the one actual `lab` checkup for the source draw, then record the value with `add-result`. Preserve the report's actual unit, reference range, and abnormal flag; a default unit is only a prompt.
4. Never change a metric's code or value type after it has results. Use `metric-deactivate` to stop new entries while retaining historical results.
5. Do not invent a metric, unit, reference range, abnormal flag, drug-monitoring need, or medical interpretation from context alone.

## Symptom baseline workflow

- A user can enter a usual-state baseline with `create-manual-baseline`; it starts as **draft**.
- `generate-baseline-candidate` can be run only on an explicitly requested date range. It requires at least seven days logged as `usual` or explicit return-to-usual and stores the dates/method as evidence.
- Show a draft/candidate and its evidence before `confirm-baseline`. The user must explicitly confirm it before it becomes the enabled baseline.
- Confirming a new baseline marks the old enabled one **historical**. Never update an enabled baseline from new logs automatically, especially during sustained worsening.
- A symptom baseline measures personal usual state only. It is separate from clinician-confirmed active/remission/flare status.

## Recurring-plan rules

- Run `generate-injections` or `generate-reviews` through a clearly stated planning horizon after the user has created or changed a plan. Preview the generated events to the user; creating database rows is permitted only as part of an explicitly requested plan setup or generation.
- Generation is idempotent: the same plan and planned timestamp never creates a duplicate injection, checkup, or lab.
- Generation creates **planned** injection records and, if configured, a linked pre-injection `lab` checkup. It may create pending database reminder rows; it never creates an external Apple Reminder without explicit approval.
- A plan uses its explicit first planned date as its cadence anchor. A one-off delay changes that event and its own reminder only. It does **not** silently shift later events, because whether the next cycle follows the original plan or the completed date is a medical-plan decision. Ask before creating a revised plan for future cycles.
- A plan can be deactivated without deleting its historical generated events. Do not delete medical history.
- Use `review_plans` for multi-test large reviews. The interval is in calendar months, not approximate day counts. Separate bounded plans may represent phases such as an explicitly confirmed first-year cadence and a later annual cadence.
- Never infer or auto-create a later review phase from diagnosis date, elapsed time, completed reviews, or an earlier plan. A six-month-to-annual transition requires an explicitly confirmed clinician arrangement or explicit effective/due boundary supplied by the user. Generating one bounded plan never activates another plan.
- `generate-reviews` creates review parents and internal review reminders only. It does not invent which tests will be required; add/link component tests only from an explicit plan or actual record.
- A large review that occurs on an injection date does not duplicate the monitoring lab. Link the existing lab to both contexts.

## Daily symptoms and factors

Continue using `ibd_db.py record-symptom`, `daily-summary`, `factor-term-add`, and `record-factor`.

- Classify stool count as `day_total` only when cumulative as of the observation, or `increment` only when newly added after prior entries. Ask when ambiguous.
- Daily aggregation uses the latest active day total then only later increments. Superseded entries are excluded.
- Set `back_to_usual=1` only when a return to usual/baseline is explicit. Improvement alone remains NULL.
- Resolve factor labels through canonical terms; preserve the user label in `raw_label`; do not merge distinct factors without clear meaning.
- `period-summary --analysis` may describe days explicitly recorded as usual, but those descriptive values are not a confirmed baseline and must not update one.

## Optional external reminders

- Internal pending reminder rows are planning records, not notifications. External reminder delivery is **off by default**.
- If the user explicitly asks to enable delivery, first confirm the notification channel and timing. A single cron dispatcher is sufficient when cron delivery is the chosen channel; Apple Reminders is an alternative only with separate explicit approval.
- After an external reminder is created or delivered, retain its external system/id through `reminder-link`, and update the internal reminder status through `reminder-status`. Do not create a cron job or external reminder merely because a planned injection/checkup exists.
- When the user asks whether this feature can be enabled, explain that it is available and offer setup; do not treat that question as authorization to schedule it.

## Summaries

- For dynamic current context use `ibd_care.py current-summary`: thin profile/diagnosis link, latest confirmed checkup assessment, latest completed review, active treatment plan, and confirmed symptom baseline.
- Use `ibd_care.py overview` when review episodes/plans, treatment occurrence provenance, and pending baseline drafts/candidates are also needed.
- Use `ibd_care.py review-detail --id <review-id>` for one review's linked checks, atomic results, current reports, and current confirmed assessments.
- For symptom/result trends use `ibd_db.py period-summary --analysis`.

In a natural-language summary:

- identify the source separately: clinician-confirmed assessment, current plan, confirmed personal baseline, daily records, and lab results;
- never call a metric trend or symptom pattern proof of inflammation, disease status, or causality;
- label factor comparisons exploratory and insufficient below three exposed or comparison days;
- do not treat affected locations, symptoms, or trend patterns alone as proof of disease activity.
