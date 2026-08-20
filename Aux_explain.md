# Explanation of the R Markdown Code

This document explains every R code chunk in `Result.rmd` in plain language. A **chunk** is a block beginning with ```` ```{r chunk-name} ````. R Markdown runs the chunks from top to bottom, so later chunks can use objects created earlier.

## Overall workflow

The chunks follow this sequence:

1. Set up R and load libraries.
2. Define a few reusable cleaning and review functions.
3. Import and clean both Excel workbooks.
4. Check data quality and compare the sources.
5. Produce correction and review tables.
6. Calculate division and district results.
7. Generate management priorities and written summaries.
8. Export cleaned and derived data to one multi-sheet Excel workbook.
9. Reshape the current snapshot into a flat, dashboard-safe district table.
10. Export a Looker Studio package with documentation, chart specifications, action tracking, and a future monthly template.

The two original source workbooks are never overwritten. Cleaned values and calculated fields are created inside R while the document runs, then copied into clearly named output workbooks and CSV files.

## Authoritative assignment structure

The report uses `Monitoring Manager Assignment.md` as the authoritative transcription. It contains nine required tasks:

| Required task | Report location | Main output |
|---|---|---|
| A1 | Division roll-up and ranking | Eight-division weighted ranking |
| A2 | Five-district comparison | Reported/expected sessions, gaps, assessment, hypotheses |
| A3 | Management priorities and dashboard | Three-division shortlist, actions, decision-led visuals |
| A4 | Executive summary | 200–250 word Country Director brief |
| B1 | Remote expansion monitoring | Five indicators, offline collection, visit plan, three risks |
| B2 | Officer support | Five-sentence managerial response |
| C1 | Draft monitoring plan | Five weaknesses/risks and one four-sentence rewrite |
| D1 | Qualitative themes | Four themes covering all eight notes |
| D2 | Program Learning Note | 100–150 word note and one systematic feedback change |

Data preparation, validation, exports, limitations, references, and Additional Content support these tasks but are not presented as extra assignment questions.

## Formula guide used in the task answers

### Task A1

District sessions:

```text
S_d = ANC_d + PNC_d + SCANU_d + GM_d + GS_d + NCD_d
```

Division weighted target result:

```text
Weighted Target %_v =
SUM(Target %_d × S_d) ÷ SUM(S_d)
```

The percentage remains on the source's 0–100 scale. `arrange(desc(weighted_target_pct))` sorts divisions, and `row_number()` creates the rank.

### Task A2

```text
reported_d = SUM(the six quarterly topic fields)
expected_d = 3 × SUM(facility average monthly sessions in district d)
gap_d = reported_d - expected_d
gap_percent_d = gap_d ÷ expected_d × 100
```

The code uses the 20% absolute gap only as a review flag. The files do not provide enough information to make it an official tolerance threshold.

### Task A3

Each risk component is converted to the same 0–1 range:

```text
normalized_x = (x - minimum_x) ÷ (maximum_x - minimum_x)
```

The composite score is:

```text
100 × (
  0.30 × target_risk
  + 0.30 × qa_risk
  + 0.15 × productivity_risk
  + 0.15 × workload_risk
  + 0.10 × scale_risk
)
```

This is a transparent management decision rule, not a statistically estimated model or organizational policy.

### Task B1

The five indicator functions are:

```text
session achievement = delivered sessions ÷ planned sessions × 100
participants = patients + caregivers
participants/session = participants ÷ delivered sessions
nurse continuity = active trained nurses ÷ planned trained posts × 100
quality score = checklist items passed ÷ checklist items assessed × 100
timely completeness = complete reports on time ÷ expected reports × 100
```

Quarterly visit load for one officer covering five facilities is:

```text
5 facilities × 1 visit = 5 visits per quarter
5 visits ÷ 3 months = 1.67 planned visits per month
```

### Tasks A4, B2, C1, and D2

Length constraints are treated as validation functions:

```text
200 <= executive_summary_word_count <= 250
4 <= B2_sentence_count <= 6
3 <= rewritten_plan_sentence_count <= 5
100 <= learning_note_word_count <= 150
```

## R Markdown header

The YAML header is the section between the opening and closing `---` lines. It is not an R chunk, but it controls the finished document.

- `title`, `author`, and `date` create the report heading.
- `html_document` and `pdf_document` allow the RMD to produce either HTML or PDF.
- `toc: true` creates a table of contents.
- `number_sections: true` numbers PDF sections.
- `geometry` and `fontsize` control the PDF page margins and text size.
- The HTML style creates a reddish-pink background, darker border, and matching text for task boxes. The separate `subquestion` class uses dark-green text, a green border, and a pale-green background for the Data Processing note.
- The LaTeX header uses `xcolor`, `fcolorbox`, and a saved `minipage` to reproduce both color treatments in PDF without requiring an extra LaTeX package.
- `mainquestion` is used only for the nine authoritative task statements. `subquestion` gives the supporting Data Processing note its separate green visual identity; it is not an additional assignment task.

## Chunk 1: `setup`

### What it does

- Sets common options for every later chunk using `knitr::opts_chunk$set()`.
- Loads the required R libraries:
  - `dplyr` for filtering, grouping, joining, and calculating columns.
  - `ggplot2` for charts.
  - `knitr` for report tables.
  - `readxl` for reading Excel workbooks.
  - `scales` for percentage labels and rescaling indicators.
  - `stringr` for cleaning and comparing text.
  - `writexl` for writing multiple data frames to one Excel workbook.
- Turns off unnecessary console messages and scientific notation.

### Why it is used

This gives every later chunk the same settings and makes the required commands available. The option `include=FALSE` means the setup code runs but is not printed in the final report, because readers do not need to see package-loading messages.

## Chunk 2: `import-functions`

### What it does

This chunk defines three small reusable functions and two reviewed correction lists.

#### `clean_text(x)`

- Converts values to text.
- changes smart apostrophes such as `’` to the standard `'` character;
- removes leading, trailing, and repeated spaces.

It returns cleaned display text. It is used for division, district, facility-type, and facility-name fields.

#### `string_key(x)`

- Applies `clean_text()`;
- converts special characters to plain ASCII characters;
- changes text to lowercase;
- removes punctuation and spaces.

It returns a comparison-only key. For example, formatting variations of a name become the same key, allowing hidden duplicates to be detected. This key never replaces the readable name in the report.

#### `district_name_crosswalk`

This is a reviewed list of known district spelling variants and their selected standard forms, such as `Bogra` to `Bogura`. A fixed crosswalk is safer than letting R guess the correct spelling.

#### `facility_location_patterns`

This applies the same idea when a misspelled district appears inside a longer facility name. The `\b` notation means “whole word,” preventing R from replacing letters that merely occur inside another word.

#### `iqr_outlier(x, multiplier = 3)`

- Calculates the middle 50% of a numeric distribution.
- Builds a conservative boundary three interquartile ranges below and above that middle range.
- Returns `TRUE` for values outside the boundary and `FALSE` for the rest.

It only creates review flags. It does not delete, cap, or replace unusual values because an unusual observation may represent real performance.

### Why this chunk is used

The same cleaning and review logic is required in several places. Defining it once keeps the rules consistent and avoids repeating long expressions throughout the analysis.

## Chunk 3: `import-and-clean`

### What it does

#### Imports the workbooks

```r
monitor_raw <- read_excel("CCP_Monitoring_Data.xlsx")
history_raw <- read_excel("Average_sessions_details.xlsx")
```

These two lines load the district monitoring data and facility historical data directly with `readxl`.

#### Renames columns

The source headers are renamed to short, consistent R names. For example:

- `General Surgury Sessions` becomes `general_surgery_sessions`;
- `% Sessions vs Target` becomes `sessions_vs_target_pct`.

Only column names change at this stage; source values do not.

#### Confirms required structure and types

- `required_monitor` and `required_history` list the columns that must exist.
- `stopifnot()` stops the document if a required column is absent.
- The numeric-type check stops the document if a required numeric field was imported as mixed text.

Stopping is safer than silently producing calculations from incomplete or incorrectly typed data.

#### Preserves original text and creates cleaned text

The chunk keeps `division_original`, `district_original`, and `facility_name_original` before applying corrections. This makes every correction auditable.

It then:

- standardizes apostrophes and whitespace;
- changes `Barisal` to `Barishal`;
- applies the reviewed district crosswalk;
- corrects repeated or poorly spaced commas in facility names;
- removes trailing commas;
- corrects reviewed location names inside facility names.

`facility_name_fixes` stores the original and corrected versions of every changed facility name.

#### Calculates monitoring fields

- `total_sessions_quarter` sums the six condition-area session columns.
- `total_participants` adds patients and caregivers.
- `calculated_sessions_week` divides quarterly sessions by 13 weeks.
- `calculated_participants_session` divides total participants by total sessions.

These calculated fields are used to check the source averages and support later analysis.

#### Creates a district comparison key

The monitoring file contains four city reporting units that do not exist separately in the historical file. The code keeps the original city names but maps them to their parent districts only in `comparison_district`:

- Dhaka North City and Dhaka South City → Dhaka;
- Chattogram City → Chattogram;
- Khulna City → Khulna.

#### Calculates historical checks

- Infers the number of months used to calculate session and participant averages.
- Calculates participants per session from both totals and monthly averages.
- Applies conservative outlier flags within facility type.

The monitoring data receive similar outlier flags within division.

### Why it is used

This chunk creates the clean, analysis-ready versions of both sources while preserving the raw values for audit. All later calculations rely on the `monitor` and `history` objects created here.

## Chunk 4: `validation-audit`

### What it does

#### Aggregates each source to district level

- `monitor_district` totals monitoring facilities, quarterly sessions, and participants by division and comparison district.
- `history_district` counts historical facility rows and calculates expected quarterly sessions and participants using monthly averages multiplied by three.

#### Joins the sources

`full_join()` creates `district_comparison_all`. A full join keeps districts from either file, even when a district appears in only one of them.

For every district, the chunk calculates:

- whether the district matched;
- the difference in facility counts;
- the session difference;
- the percentage session difference.

#### Runs validation tests

The code checks for:

- non-finite, negative, zero, or fractional count values;
- exact and normalized-key duplicates;
- missing required values;
- invalid percentages;
- conflicting division assignments;
- inconsistent weekly-session and participant-per-session formulas;
- disagreement between historical session and participant averaging periods;
- start years that are in the future;
- unmatched district geographies.

#### Builds two compact outputs

- `validation_checks` records the number of failures for each test.
- `issue_log` records actual issues, how many observations they affect, and how they were treated.

`kable(issue_log)` displays the issue summary as a readable table.

### Why it is used

This chunk checks more than the problems mentioned in the assignment. It looks for problems in names, numbers, formulas, repeated rows, and links between the two files. It also separates mistakes that R can safely fix from differences that a person needs to check.

## Chunk 5: `location-fixes-table`

### What it does

- Finds division and district values where the original text differs from the cleaned value.
- Combines corrections from both workbooks.
- Removes repeated correction combinations.
- Displays original and corrected values with `kable()`.

### Why it is used

The table provides an audit trail. A reviewer can see exactly which location labels changed instead of being asked to trust an invisible cleaning step.

## Chunk 6: `facility-name-fixes`

### What it does

Displays the first 10 records from `facility_name_fixes`, including the Excel row, original name, and corrected name.

### Why it is used

There are 36 facility-name changes, so printing all of them in the main report would be distracting. Showing representative examples keeps the report readable while the complete correction object remains available in R. The full list is also documented in `issues.md`.

## Chunk 7: `anomaly-review`

### What it does

- Collects every unusual-value flag into one `anomaly_review` table.
- Records the source, Excel row, location, indicator, and value.
- Combines flags for:
  - monitoring target achievement;
  - monitoring QA coverage;
  - monitoring sessions per facility;
  - monitoring participants per session;
  - historical average monthly sessions;
  - historical participants per session.
- Creates `anomaly_display`, which shows all monitoring flags and the first eight historical flags.

### Why it is used

This table points to values worth checking without calling them errors. Showing only some rows keeps the report short and readable.

## Chunk 8: `averaging-window-review`

### What it does

Groups historical facilities by `Active since` year and reports:

- the number of facilities;
- minimum implied observation months;
- median implied observation months;
- maximum implied observation months.

### Why it is used

The older file gives a start year, but not a start month or a clear end date. This table shows why the exact period behind each average cannot be worked out, even though the monthly averages agree with the totals.

## Chunk 9: `division-results`

### What it does

Groups monitoring rows by division and calculates:

- number of reporting units and facilities;
- total sessions and participants;
- total trainers;
- session-volume-weighted target achievement;
- facility-count-weighted QA coverage;
- sessions per facility;
- facilities per trainer.

It then ranks the eight divisions from highest to lowest weighted target achievement and creates `division_rank_table` for display.

### Why it is used

This directly answers the first quantitative assignment question. Weighting prevents a small district from influencing the division result as much as a district delivering many sessions.

## Chunk 10: `division-chart`

### What it does

Creates a horizontal bar chart of weighted target achievement by division.

- The horizontal axis shows session-volume-weighted `% Sessions vs Target`.
- The vertical axis lists the eight divisions.
- Bar length shows target achievement.
- The orange-to-green bar color shows facility-weighted QA coverage, with orange lower and green higher.
- Text labels show the exact target percentage.

The chunk uses `echo=FALSE`, so the final report shows the chart but not the plotting code.

Immediately below the chart, the `figurecaption` fenced div prints a centered **Fig. 1** caption in both output formats. CSS centers it in HTML, while the custom LaTeX `figurecaption` environment centers it in PDF. The caption explains the axes, labels, color scale, and the fact that delivery and QA are separate indicators.

### Why it is used

The chart lets managers compare performance and quality coverage together more quickly than reading a numeric table alone.

## Table-numbering approach

The report contains both R-generated `kable()` tables and ordinary Markdown tables. To keep one uninterrupted sequence across both types, visible captions are numbered manually in document order from **Table 1** through **Table 15**. This avoids separate automatic counters producing duplicate numbers for different table-rendering methods. When adding or deleting a displayed table, update every later caption and verify that the sequence remains complete with no repeated or skipped number.

## Chunk 11: `district-comparison`

### What it does

#### Selects five districts using a clear rule

- Keeps districts present in both sources.
- Sorts them by reported quarterly sessions.
- Divides them into five volume groups using `ntile()`.
- Selects the middle district from each group.

This provides five districts across the delivery-volume range without manually choosing favorable examples.

#### Compares reported and expected sessions

For each selected district, it uses:

```text
Expected quarterly sessions = sum of facility average monthly sessions × 3
```

It calculates the absolute and percentage difference from the monitoring snapshot.

#### Classifies the difference

- More than 20% difference: `Does not align`.
- Up to 20% difference with different facility counts: `Totals close; lists differ`.
- Up to 20% difference with matching facility counts: `Similar`.

The code also assigns a plausible verification hypothesis based on the session direction and facility-count difference.

### Why it is used

This answers the assignment's five-district comparison requirement using a transparent selection method and explicit review thresholds. The hypotheses guide verification but do not claim to prove the cause.

## Chunk 12: `district-hypotheses`

### What it does

Loops through the five selected districts and prints one formatted bullet for each district's assessment and hypothesis.

- `echo=FALSE` hides the R loop.
- `results='asis'` tells R Markdown to treat the generated bullets as normal Markdown formatting.

### Why it is used

The preceding chunk produces a compact numeric table. This chunk turns the stored hypotheses into readable explanations without manually rewriting district names or results.

## Chunk 13: `priority-analysis`

### What it does

Creates five scaled warning signals for each division:

- target underachievement;
- low QA coverage;
- low sessions per facility;
- high facilities per trainer;
- the number of facilities affected.

It combines them into a 0–100 priority score using these weights:

- 30% target risk;
- 30% QA risk;
- 15% productivity risk;
- 15% trainer-workload risk;
- 10% program size.

The three highest-scoring divisions become `priority_three` and are displayed in `priority_table` with their component indicators.

### Why it is used

The assignment asks managers to consider more than one measure. The score gives them a clear starting list while keeping program size from dominating the decision. The report also warns that the weights are a management choice, so managers should review the separate measures before acting.

## Chunk 14: `priority-actions`

### What it does

- Gives a plain-language reason for each of the three selected divisions.
- Assigns one practical first action for the next 30 days.
- Prints the result as three short bullets and also keeps it as a data frame for Excel export.

### Why it is used

A score alone does not tell a manager what to do. This chunk turns the shortlist into specific follow-up work while keeping the figures and actions together.

## Chunk 15: `executive-summary`

### What it does

- Pulls the strongest and weakest divisions from `division_results`.
- Pulls the three priority divisions from `priority_three`.
- Pulls sampled districts that do not align from `five_districts`.
- Inserts these live results into a Country Director summary.
- Counts the words.
- Uses `stopifnot()` to stop rendering if the summary is outside the required 200–250 words.
- Prints the summary as normal report text.

The code is hidden using `echo=FALSE`, and `results='asis'` renders the text without code-style formatting.

### Why it is used

The text uses the latest calculated results. If the source data changes, the ranking names change too. The word-count check makes sure the summary stays within the assignment limit.

## Tasks B1, B2, and C1

These answers are written in Markdown because they are operational recommendations rather than calculations from the two supplied workbooks. Task B1 still shows mathematical definitions for all calculated indicators and the visit workload. The formulas are displayed directly in the report so a reader can distinguish a count, rate, ratio, and workload assumption.

Task B1 is organized exactly around the requested components:

1. five indicators, their functions, and reasons;
2. offline-first data collection;
3. realistic visit frequency and workload calculation;
4. three risks and one mitigation for each.

Task B2 remains one five-sentence answer. The visible check `4 <= 5 <= 6` shows that it meets the requested range.

Task C1 is intentionally kept as **one combined task**. The earlier draft incorrectly displayed separate invented questions for weaknesses and rewriting. The corrected report contains one pink C1 task statement followed by two answer subsections: the weakness/risk table and the four-sentence rewritten plan. The visible check `3 <= 4 <= 5` confirms its length.

Keeping these answers as ordinary Markdown makes the operational reasoning readable while preserving the exact assignment structure.

## Chunk 16: `learning-note`

### What it does

- Combines the qualitative findings into one Program Learning Note.
- Counts the words.
- Stops rendering if the result is outside the required 100–150 words.
- Prints the note as normal report text.

### Why it is used

The chunk enforces the assignment's word limit and keeps the final note together as one reusable object. The underlying theme synthesis remains in the Markdown table immediately above it.

Task D1 now numbers and reproduces all eight supplied notes before the synthesis. The table maps notes `1,8`, `3,5,6`, `2`, and `4,7` to four themes, so every note is accounted for. Each row separates the one-sentence underlying pattern from the isolated/systemic judgment and reasoning.

Task D2 displays its exact task statement, the word-count function, the generated learning note, the checked count, and one explicit feedback-and-action-log recommendation.

## Chunk 17: `export-workbook`

### What it does

- Defines `format_export_label()` to turn internal snake-case names into readable Excel labels while restoring program abbreviations such as ANC, PNC, SCANU, NCD, CCP, QA, ID, and IQR.
- Defines `format_export_text()` to format only text that actually looks like a snake-case code. Ordinary sentences and proper names retain their wording.
- Defines `prepare_export_sheet()` to apply those presentation rules to an exported copy of each table, not to the analysis objects used in calculations.
- Places the cleaned source data and important derived tables into the named `export_sheets` list.
- Replaces underscores in worksheet names with spaces.
- Uses `write_xlsx()` to create `Updated_Data.xlsx`.

The workbook contains these worksheets:

- `Clean Monitoring`
- `Clean Facilities`
- `Division Results`
- `District Comparison`
- `Five Districts`
- `Priority Divisions`
- `Priority Actions`
- `Issue Log`
- `Validation Checks`
- `Location Fixes`
- `Facility Name Fixes`
- `Anomaly Review`
- `Averaging Windows`

### Why it is used

The analysis is easier to review and reuse when the cleaned records, calculated results, and data-quality logs are available outside the rendered report. A named list is the simplest way for `writexl` to create several worksheets in one operation. Formatting only exported copies is important: readable labels improve the workbook, while stable snake-case names keep the R calculations reproducible.

## Chunk 18: `dashboard-export`

### Purpose and design principle

This chunk creates the files used by the optional Looker Studio portfolio extension. Its central rule is **one table, one declared grain**. The main table contains one row per consolidated district for one current snapshot. It does not mix district rows, division totals, facility rows, and action rows in the same table, because mixing grains is a common cause of doubled or meaningless dashboard totals.

The chunk creates six R objects:

1. `dashboard_data` — the main 64-row district fact table;
2. `dashboard_division_summary` — an eight-row division table;
3. `dashboard_action_tracker` — a three-row proposed follow-up table;
4. `dashboard_dictionary` — field definitions and aggregation rules;
5. `dashboard_chart_plan` — the visual-to-variable specification;
6. `dashboard_monthly_template` — an empty, header-only future reporting structure.

It then writes `Looker_Dashboard_Package.xlsx` and `Looker_Dashboard_Data.csv`.

### Part 1: consolidate monitoring rows to district grain

```r
dashboard_data <- monitor |>
  group_by(division, comparison_district) |>
  summarise(...)
```

`comparison_district` is used instead of the source `district` because the source includes four city reporting units that do not exist as separate rows in the historical workbook. The documented comparison rule groups:

- `Chattogram City` into `Chattogram`;
- `Dhaka North City` and `Dhaka South City` into `Dhaka`;
- `Khulna City` into `Khulna`.

This reduces 68 monitoring reporting-unit rows to 64 comparable district rows. The source labels remain available in `monitor` and `Updated_Data.xlsx`; only the dashboard model uses the consolidated grain.

Inside `summarise()`:

- `reporting_units = n()` counts how many original monitoring rows contributed to the district.
- `includes_city_reporting_unit = any(...)` records whether consolidation occurred.
- `sum()` produces additive totals for facilities, trainers, six session topics, sessions, patients, caregivers, and participants.
- `any_monitoring_anomaly` is `TRUE` if at least one contributing source row has an unusual-value flag.

### Part 2: create weighted helper numerators

Before grouping, the first `mutate()` calculates a numerator on every original monitoring row:

```r
row_target_weighted_numerator =
  (sessions_vs_target_pct / 100) * total_sessions_quarter

row_qa_weighted_numerator =
  (ccp_quality_assessment_coverage_pct / 100) * facilities
```

`summarise()` then sums the row-level numerators within each consolidated district. Calculating them before grouping is important for Dhaka, Chattogram, and Khulna, where several source reporting-unit rows contribute to one district. It also prevents a newly summarized column such as `total_sessions_quarter` from being accidentally reused in a later expression inside the same `summarise()` call.

These helpers solve an important dashboard problem. If Looker Studio simply averages district percentages, a district with 100 sessions receives the same weight as a district with 1,000 sessions. Instead, a filter-safe result is calculated as:

```text
SUM(target_weighted_numerator) / SUM(total_sessions_quarter)
```

QA follows the same pattern, using facilities as the weight:

```text
SUM(qa_weighted_numerator) / SUM(facilities)
```

The rate is stored as a decimal from 0 to 1. For example, 85% is stored as `0.85`, because Looker Studio's Percent type expects a decimal fraction. The original analysis retains percentages on the 0–100 scale; only the dashboard export uses 0–1 rates.

### Part 3: join the historical benchmark

`left_join()` adds the following district-comparison fields:

- historical facility count;
- expected quarterly sessions;
- expected quarterly participants;
- facility-count difference;
- session difference and percentage difference.

A left join is used because the monitoring snapshot defines the dashboard population. Earlier validation confirmed that all 64 consolidated monitoring districts match the historical source after normalization.

The expected value remains a **diagnostic benchmark**, calculated from historical average monthly sessions multiplied by three. It is not a formal target and should not be interpreted as one.

### Part 4: create IDs, labels, ratios, and flags

#### Record and reporting-period fields

- `record_id` combines `CURRENT`, division, and district after removing punctuation. This gives every current row a stable unique key.
- `reporting_period_id` is `CURRENT_SNAPSHOT` because the source does not identify the quarter.
- `reporting_period_label` explicitly says the source date was not supplied.
- `reporting_period_end` is deliberately blank. A fake date would make a trend chart look valid when it is not.
- `reporting_grain` states that the table uses consolidated districts.

When real monthly data become available, the unique key should include the actual period, for example `2026_08_DHAKA_DHAKA`.

#### Operational ratios

The chunk calculates:

- `sessions_per_facility = total_sessions / facilities`;
- `participants_per_session = total_participants / total_sessions`;
- `facilities_per_trainer = facilities / trainers`;
- patient and caregiver shares;
- the absolute and percentage difference from the historical estimate.

These district ratios are safe when a chart displays one district per row. For a total or scorecard across many districts, the ratio should be recalculated from summed base fields—for example, `SUM(total_participants) / SUM(total_sessions_quarter)`—rather than averaging district ratios.

#### Demonstration status rules

`case_when()` assigns readable categories:

- delivery is **On track** at 85% or above, **Watch** at 70%–84.99%, and **Critical** below 70%;
- QA is **Strong** at 80% or above, **Watch** at 60%–79.99%, and **Critical** below 60%;
- overall attention is **Urgent** if either delivery or QA is below 60%, **Watch** if either misses the higher threshold, and **On track** otherwise.

These are portfolio demonstration thresholds, not Noora Health policy. `threshold_note` travels with every row so the assumption is not separated from the data. Managers should confirm or replace these cutoffs before operational use.

`attention_sort` converts the status into `1`, `2`, or `3`, allowing tables to sort Urgent before Watch before On track. `attention_flag` converts Urgent/Watch to `1`, allowing a scorecard to sum the number of affected districts.

`cross_file_review_flag` becomes `1` when either:

- current and historical facility counts differ; or
- the session result differs from the historical three-month estimate by more than 20%.

This is a data-review flag, not proof of low performance.

### Part 5: arrange the dashboard columns

`select()` puts identifiers and filters first, status fields second, additive metrics third, ratios fourth, and helper/audit fields last. A deliberate column order makes the field list easier to navigate inside Looker Studio.

`arrange()` puts Urgent records first, then weaker target and QA results. The original source is not altered; this order affects only the dashboard export.

### Part 6: create the division summary

`dashboard_division_summary` uses `division_priority`, which already contains:

- division rank;
- facilities, sessions, participants, and trainers;
- weighted target and QA results;
- sessions per facility and facilities per trainer;
- the overall priority score and its five normalized risk components.

This table is useful when a visual must show the already calculated priority score. It is kept separate from district records so division totals are not repeated on every district row.

### Part 7: create the action tracker

`dashboard_action_tracker` converts the three recommended division actions into structured management records. Each row has:

- a unique `action_id`;
- division;
- finding;
- recommended action;
- status;
- owner;
- target date;
- latest update;
- last-updated date;
- evidence link.

The last five operational fields are deliberately blank or say that no update was supplied. The analysis can recommend an action, but it cannot truthfully invent an owner, deadline, or progress update. A manager should fill these fields after agreeing the action.

### Part 8: create the data dictionary

`tibble::tribble()` is used to enter documentation row by row. Every dictionary record gives:

- exact field name;
- Looker Studio type;
- dashboard role;
- safe default aggregation;
- plain-language meaning.

The dictionary distinguishes additive metrics from non-additive rates and ratios. This distinction is essential: sessions can be summed, but percentages and per-unit ratios should not be summed and often should not be averaged.

### Part 9: create the chart plan

`dashboard_chart_plan` records nine proposed visuals. Each row specifies the page, order, chart type, dimension, metrics/formulas, and one management question. This demonstrates that chart choice follows a decision need.

Examples:

- scorecards summarize a single key metric;
- a horizontal bar compares and ranks divisions;
- a bubble scatter shows the relationship between delivery and QA while bubble size represents volume;
- a 100% stacked bar compares session composition;
- an exception table supports follow-up on named districts;
- an action table supports accountability;
- a time series is reserved for future dated data.

### Part 10: create the future monthly template

`dashboard_monthly_template` is a zero-row tibble. The column types are defined, but no rows are invented. Its fields support future tracking of:

- reporting period and facility identity;
- report due and received dates;
- sessions delivered and planned;
- patients and caregivers;
- QA checks completed and planned;
- active trained nurses;
- completeness;
- linked action ownership and status.

Once real monthly rows exist, this structure can support time series, reporting timeliness, staffing continuity, and overdue-action views. The current snapshot cannot.

### Part 11: validate the export

The `stopifnot()` block prevents export when a key dashboard assumption fails. It confirms:

- exactly 64 district rows;
- every `record_id` is unique;
- the data dictionary has exactly one entry for every dashboard field and no undocumented field;
- dashboard session totals equal monitoring session totals;
- dashboard participant totals equal monitoring participant totals;
- every CSV header begins correctly and contains only letters, numbers, or underscores;
- text fields contain no embedded line breaks that could break CSV upload.

These tests are deliberately placed immediately before writing files. If an upstream change breaks the dashboard model, rendering stops instead of silently publishing a bad export.

### Part 12: write the files

```r
write_xlsx(dashboard_export_sheets, "Looker_Dashboard_Package.xlsx")
```

The named list becomes six Excel worksheets.

```r
write.csv(
  dashboard_data,
  "Looker_Dashboard_Data.csv",
  row.names = FALSE,
  na = "",
  fileEncoding = "UTF-8"
)
```

The CSV is the simplest direct upload. `row.names = FALSE` prevents R from adding a meaningless first column. `na = ""` writes blank fields as empty cells. UTF-8 protects punctuation and location names.

### Why two dashboard files are exported

The CSV provides one clean fact table for quick connection. The Excel package provides supporting governance material and additional data sources. Looker Studio connects to one Google Sheets worksheet at a time, so the action tracker should be connected as a second data source if it is displayed on a separate page.

`Updated_Data.xlsx` remains the audit workbook. `Looker_Dashboard_Package.xlsx` is the presentation/refresh package. Keeping these purposes separate makes both files easier to understand.

## Additional Content narrative after Chunk 18

The remainder of the Additional Content section is Markdown rather than executable R. It explains:

- how to connect the CSV or a Google Sheets worksheet;
- which Looker Studio data type and aggregation to assign to each field group;
- the three required filter-safe calculated fields;
- the exact dimensions and metrics for each visual;
- dashboard page layout and controls;
- why maps and time series are currently inappropriate;
- a safe refresh and governance routine.

This narrative is part of the technical work. A dashboard is not complete merely because a chart renders; users also need metric definitions, known limitations, refresh instructions, and ownership rules.

## Final limitations section

The final section is ordinary Markdown, not code. It records assumptions that the calculations cannot resolve, including:

- unknown reporting and averaging windows;
- district facility-count disagreement;
- lack of stable cross-file facility IDs;
- the city-to-parent-district comparison assumption;
- rounded source averages.

This section matters because code cannot fill in facts that are missing from the source files.

## Short glossary of common R commands

| Command | Plain-language meaning |
|---|---|
| `<-` | Save the result on the right under the name on the left. |
| `|>` | Pass the current result to the next operation. |
| `mutate()` | Add or change columns. |
| `transmute()` | Create columns and keep only the columns named in that operation. |
| `filter()` | Keep rows meeting a condition. |
| `select()` | Keep or arrange selected columns. |
| `group_by()` | Temporarily divide rows into groups. |
| `summarise()` | Calculate one or more results for each group. |
| `arrange()` | Sort rows. |
| `left_join()` | Add matching columns while keeping every row from the table on the left. |
| `full_join()` | Combine two tables while retaining unmatched rows from both. |
| `case_when()` | Assign different results for different conditions. |
| `if_else()` | Choose one of two values using a true/false condition. |
| `any()` | Return `TRUE` when at least one value in a group is true. |
| `weighted.mean()` | Calculate an average where larger observations receive more influence. |
| `ntile()` | Divide ordered observations into approximately equal-sized groups. |
| `kable()` | Format an R table for the report. |
| `stopifnot()` | Stop execution when an essential validation rule fails. |
| `n_distinct()` | Count unique values. |
| `tribble()` | Create a small table by entering values row by row. |
| `write_xlsx()` | Write one or more data frames to an Excel workbook. |
| `write.csv()` | Write a flat comma-separated text file. |
