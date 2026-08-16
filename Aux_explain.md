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

The original Excel files are never overwritten. Cleaned values and calculated fields exist only inside R while the document is running.

## R Markdown header

The YAML header is the section between the opening and closing `---` lines. It is not an R chunk, but it controls the finished document.

- `title`, `author`, and `date` create the report heading.
- `html_document` and `pdf_document` allow the RMD to produce either HTML or PDF.
- `toc: true` creates a table of contents.
- `number_sections: true` numbers PDF sections.
- `geometry` and `fontsize` control the PDF page margins and text size.
- The color settings create a darker pink for main questions and a softer pink for smaller questions in both PDF and HTML output.

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

- Bar length shows target achievement.
- Bar color shows weighted QA coverage.
- Text labels show the exact target percentage.

The chunk uses `echo=FALSE`, so the final report shows the chart but not the plotting code.

### Why it is used

The chart lets managers compare performance and quality coverage together more quickly than reading a numeric table alone.

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

## Sections B and C

Sections B and C contain written recommendations rather than data-driven calculations, so they do not need separate R chunks. They address:

- monitoring 15 remote facilities;
- indicators, offline collection, travel frequency, and risks;
- support for the monitoring officer;
- weaknesses in the draft monitoring plan;
- the rewritten monitoring plan.

Keeping these as normal Markdown makes them easier to read and edit.

## Chunk 16: `learning-note`

### What it does

- Combines the qualitative findings into one Program Learning Note.
- Counts the words.
- Stops rendering if the result is outside the required 100–150 words.
- Prints the note as normal report text.

### Why it is used

The chunk enforces the assignment's word limit and keeps the final note together as one reusable object. The underlying theme synthesis remains in the Markdown table immediately above it.

## Chunk 17: `export-workbook`

### What it does

- Places the cleaned source data and important derived tables into the named `export_sheets` list.
- Assigns each data frame a clear Excel worksheet name.
- Uses `write_xlsx()` to create `Updated_Data.xlsx`.

The workbook contains these worksheets:

- `Clean_Monitoring`
- `Clean_Facilities`
- `Division_Results`
- `District_Comparison`
- `Five_Districts`
- `Priority_Divisions`
- `Priority_Actions`
- `Issue_Log`
- `Validation_Checks`
- `Location_Fixes`
- `Facility_Name_Fixes`
- `Anomaly_Review`
- `Averaging_Windows`

### Why it is used

The analysis is easier to review and reuse when the cleaned records, calculated results, and data-quality logs are available outside the rendered report. A named list is the simplest way for `writexl` to create several worksheets in one operation.

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
| `filter()` | Keep rows meeting a condition. |
| `select()` | Keep or arrange selected columns. |
| `group_by()` | Temporarily divide rows into groups. |
| `summarise()` | Calculate one or more results for each group. |
| `arrange()` | Sort rows. |
| `full_join()` | Combine two tables while retaining unmatched rows from both. |
| `case_when()` | Assign different results for different conditions. |
| `weighted.mean()` | Calculate an average where larger observations receive more influence. |
| `ntile()` | Divide ordered observations into approximately equal-sized groups. |
| `kable()` | Format an R table for the report. |
| `stopifnot()` | Stop execution when an essential validation rule fails. |
| `n_distinct()` | Count unique values. |
