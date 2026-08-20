# Monitoring Manager Assignment — Authoritative Task-by-Task Learning Checklist

## How to use this file

`Monitoring Manager Assignment.md` is the authoritative task transcription. The required structure is:

- Section A: four tasks, A1–A4;
- Section B: two tasks, B1–B2;
- Section C: one task, C1;
- Section D: two tasks, D1–D2.

Do not invent extra assignment questions. Data cleaning, validation, exports, and the Looker Studio portfolio extension are supporting work and should be labelled as such.

## Supporting preparation — not a separate assignment task

- [ ] Read both workbooks without altering them:
  - `CCP_Monitoring_Data.xlsx` — district/reporting-unit quarterly snapshot;
  - `Average_sessions_details.xlsx` — facility-level historical averages and totals.
- [ ] Retain original text before applying any name correction.
- [ ] Standardize only demonstrable spelling, punctuation, line-break, and whitespace differences.
- [ ] Create a documented comparison geography for city reporting units; retain original city labels in the cleaned source.
- [ ] Confirm required columns, numeric types, missingness, duplicates, impossible values, and formula consistency.
- [ ] Flag unusual observations for review; do not replace them merely because they are unusual.
- [ ] Document that equal national facility totals do not imply matching district rosters.

# Section A — Quantitative analysis and management recommendations

## Task A1 — Division roll-up and ranking

> Using ‘CCP_Monitoring_Data’, roll up the district-level figures to division level and calculate a division-level "% Sessions vs Target" (weighted by session volume, not a simple average of the district percentages). Rank the 8 divisions from strongest to weakest.

### Required calculations

For district (d):

```text
district_sessions = ANC + PNC + SCANU + General Medicine + General Surgery + NCD
```

For division (v):

```text
weighted_target_percent =
SUM(district_target_percent × district_sessions)
÷ SUM(district_sessions)
```

### Completion checklist

- [ ] Sum the six topic fields for every district row.
- [ ] Group district rows by division.
- [ ] Sum facilities, sessions, participants, and trainers for descriptive context.
- [ ] Calculate target achievement with session volume as the weight.
- [ ] Do not use a simple mean of district percentages.
- [ ] Rank all eight divisions from highest to lowest weighted target result.
- [ ] Explain why weighting is appropriate and state that district target counts were not supplied.
- [ ] Show both the formula and the resulting table.

## Task A2 — Five-district source comparison

> Pick any 5 districts. For each, compare the district's total sessions this quarter (CCP_Monitoring_Data file) against what you'd expect for a quarter based on that district's facilities in Average_sessions_details file (their average monthly sessions × 3). Which districts, if any, look inconsistent between the two sources and for those, what's your hypothesis for the gap? For the rest, do you expect the figures from the two sources to be similar, or would you want to verify the figures?

### Required calculations

```text
reported_sessions_district = SUM(six quarterly session-topic fields)

expected_quarter_sessions_district =
3 × SUM(facility average monthly sessions)

absolute_gap = reported_sessions - expected_sessions

gap_percent =
(reported_sessions - expected_sessions)
÷ expected_sessions × 100
```

### Completion checklist

- [ ] Select exactly five matched districts.
- [ ] State the selection rule so the sample cannot appear cherry-picked.
- [ ] Show monitoring and historical facility counts beside the session comparison.
- [ ] Show reported sessions, expected sessions, absolute or percentage difference, and assessment.
- [ ] State any review threshold as an analytical assumption, not an official pass mark.
- [ ] For each inconsistent district, give a source-grounded hypothesis such as facility-roster difference, missing reports, activation timing, changed activity, or unequal periods.
- [ ] For apparently similar districts, explicitly say whether the figure can be accepted or still needs verification.
- [ ] Avoid claiming that the historical estimate is the formal target.

## Task A3 — Three management priorities and dashboard description

> Recommend the 3 divisions you would prioritize for management attention this quarter. State your indicators, it should combine more than one data dimension, and describe, in words, the dashboard you'd design for the Implementation Team using this kind of data. For each visual you propose, name the one decision or question it should help answer.

### Required multi-dimensional calculation

The report combines five indicators:

1. weighted target achievement;
2. facility-weighted QA coverage;
3. sessions per facility;
4. facilities per trainer;
5. number of facilities affected.

Min-max normalization:

```text
normalized_x = (x - minimum_x) ÷ (maximum_x - minimum_x)
```

Composite score:

```text
priority_score = 100 × (
  0.30 × target_risk
  + 0.30 × qa_risk
  + 0.15 × productivity_risk
  + 0.15 × trainer_workload_risk
  + 0.10 × program_scale_risk
)
```

### Completion checklist

- [ ] State every indicator, direction of risk, weight, and formula.
- [ ] Explain why the score is a management shortlist rather than an official performance standard.
- [ ] Name exactly three divisions.
- [ ] Give the specific values driving each recommendation.
- [ ] Give a practical first action for each division.
- [ ] Describe the Implementation Team dashboard in words.
- [ ] For every visual, state exactly one management decision or question.
- [ ] Separate current-data visuals from future visuals requiring dated monthly facility data.

## Task A4 — Country Director executive summary

> Write a 200-250 word executive summary of your findings addressed to the Country Director, who has 5 minutes to read it before a leadership meeting.

### Completion checklist

- [ ] Address the Country Director directly.
- [ ] Keep the summary between 200 and 250 whitespace-separated words.
- [ ] Lead with the strongest and weakest performance findings.
- [ ] State the three priority divisions and why they matter.
- [ ] Include the major cross-source reliability warning.
- [ ] End with immediate management actions.
- [ ] Keep technical methodology out unless it changes interpretation.
- [ ] Validate automatically: `200 <= word_count <= 250`.

# Section B — Monitoring the remote expansion

## Task B1 — Monitoring approach, data collection, visits, risks, and mitigation

> Propose a monitoring approach for these 15 facilities given the connectivity and travel constraints described. Specify: (a) which 4-5 indicators you would prioritize tracking and why, (b) how data would be collected given intermittent connectivity, and (c) a realistic field-visit frequency, with your reasoning for why that frequency is achievable and sufficient. Identify the top 3 risks specific to this expansion and your mitigation approach for each.

### Indicator formulas

```text
session_achievement_percent = sessions_delivered ÷ sessions_planned × 100

total_participants = patients + caregivers

participants_per_session = total_participants ÷ sessions_delivered

trained_nurse_continuity_percent =
active_trained_nurses ÷ planned_trained_posts × 100

quality_score_percent =
checklist_items_passed ÷ checklist_items_assessed × 100

complete_timely_reporting_percent =
complete_reports_received_by_due_date ÷ expected_reports × 100
```

### Visit-feasibility calculation

For the officer covering five facilities quarterly:

```text
5 facilities × 1 visit per quarter = 5 visits per quarter
5 visits ÷ 3 months = 1.67 planned visits per month
```

For all facilities:

```text
15 facilities × 1 routine visit per quarter = 15 routine visits per quarter
```

### Completion checklist

- [ ] Propose four or five indicators—not more—and explain why each matters.
- [ ] Give the mathematical function for every calculated indicator.
- [ ] Use a paper source register plus an approved offline-capable form.
- [ ] Explain local saving, validation, unique facility-month IDs, delayed synchronization, and upload receipts.
- [ ] Recommend an initial visit and a sustainable routine frequency.
- [ ] Show why the routine travel volume is achievable.
- [ ] Add monthly remote checks and exception-triggered visits.
- [ ] Identify exactly three expansion-specific risks.
- [ ] Pair each risk with a concrete mitigation.

## Task B2 — Support one officer covering five facilities

> One monitoring officer on your team will be responsible for covering 5 of these facilities largely on their own. What would you personally put in place, as their manager, to support them and protect data quality, without requiring them to travel more than is realistic? Answer in 4-6 sentences.

### Completion checklist

- [ ] Use four to six sentences; the report uses five.
- [ ] Cover route planning and advance travel resources.
- [ ] Establish main and backup facility contacts.
- [ ] Use monthly remote checks and clear warning triggers.
- [ ] Include staff safety and leave coverage.
- [ ] Include direct managerial coaching and sampled source verification.
- [ ] Validate: `4 <= sentence_count <= 6`.

# Section C — Draft monitoring plan

## Task C1 — Weaknesses, risks, and rewrite

> Identify at least 3 specific weaknesses in this draft plan, and for each, explain the concrete risk it creates for Noora Health. Then rewrite the draft to fix it, in 3-5 sentences.

This is one combined task. Do not split it into invented “Question 1” and “Question 2.”

### Completion checklist

- [ ] Reproduce or clearly identify the supplied draft before critiquing it.
- [ ] Identify at least three specific weaknesses; the report uses five.
- [ ] Link every weakness to a concrete operational, privacy, governance, or data-quality risk.
- [ ] Address narrow indicators, free-text WhatsApp reporting, unclear validation ownership, research access to the live program file, and quarterly-only review.
- [ ] Rewrite the plan in three to five sentences; the report uses four.
- [ ] Include standard definitions, unique IDs, validation, offline collection, access controls, review cadence, action ownership, and escalation.
- [ ] Validate: `3 <= sentence_count <= 5`.

# Section D — Qualitative synthesis and program learning

## Task D1 — Group eight notes into themes

> Group the 8 notes above into 3-4 themes of your own choosing. For each theme: give it a short label, write one sentence on the underlying pattern it points to, not a summary of the quotes, but what it tells us about the program, and say whether you'd treat it as an isolated incident or something to escalate as systemic, with your reasoning.

### Completion checklist

- [ ] Reproduce or number the eight supplied notes so the grouping is auditable.
- [ ] Use three or four themes; the report uses four.
- [ ] Assign every note exactly once unless an intentional overlap is explained.
- [ ] Give each theme a short label.
- [ ] Write exactly one underlying-pattern sentence per theme.
- [ ] Do not merely restate the quotations.
- [ ] State isolated versus systemic treatment and give reasoning.
- [ ] Treat one observation cautiously: a plausible recurring mechanism can be escalated for verification without claiming proven prevalence.

## Task D2 — Program Learning Note and feedback process

> Draft a 100-150 word Program Learning Note, to share qualitative insight with the design and implementation teams based on your synthesis above. Propose one concrete process or tool change you'd recommend so this kind of feedback gets captured more systematically going forward.

### Completion checklist

- [ ] Write for the design and implementation teams.
- [ ] Base the note on the four themes from Task D1.
- [ ] Keep the note between 100 and 150 whitespace-separated words.
- [ ] Recommend exactly one concrete process/tool change.
- [ ] Specify the minimum feedback-log fields: facility, theme, urgency, owner, due date, status, and closure note.
- [ ] Explain the monthly review and feedback-closure loop.
- [ ] Validate automatically: `100 <= word_count <= 150`.

# Additional Content — Looker Studio portfolio extension

This section is optional supporting content, not a new assignment task.

- [ ] Keep it after the required A–D answers and analytical export.
- [ ] Use one consolidated-district row grain in the main dashboard table.
- [ ] Export the dashboard CSV and multi-sheet package.
- [ ] Document every field, type, safe aggregation, formula, filter, chart, and decision.
- [ ] Use weighted numerators for filter-safe target and QA results.
- [ ] Keep the proposed action tracker separate from observed progress.
- [ ] Do not invent dates, trends, owners, deadlines, or status updates.
- [ ] State that the current source supports a snapshot dashboard, not a genuine time series.

# Final verification

- [ ] Confirm there are exactly nine pink task boxes: A1, A2, A3, A4, B1, B2, C1, D1, and D2.
- [ ] Confirm no invented “Main question” or “Question” labels remain.
- [ ] Confirm every calculation required by a task is shown mathematically.
- [ ] Confirm every table and narrative sits under the task it answers.
- [ ] Confirm the chart has a centered `Fig. 1` caption below it explaining both axes, bar length, labels, and QA color.
- [ ] Confirm all 15 displayed tables are numbered sequentially from `Table 1` through `Table 15`, with no gap or duplicate.
- [ ] Confirm the executive summary and learning note pass word limits.
- [ ] Confirm B2 and the rewritten C plan pass sentence limits.
- [ ] Render and visually inspect both HTML and PDF.
- [ ] Reconcile all exported dashboard totals and identifiers.
