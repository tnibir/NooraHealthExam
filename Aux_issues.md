# Data Issues Register

This register documents the issues detected in `CCP_Monitoring_Data.xlsx` and `Average_sessions_details.xlsx`, organized by file and column. Excel row numbers include the header row. The treatment follows three rules:

- **Corrected:** the intended value is clear from a spelling, punctuation, whitespace, or header error.
- **Flagged:** the value may be valid, but the source data are insufficient to change it safely.
- **Passed:** automated type, completeness, range, uniqueness, or internal-consistency checks found no problem.

## 1. `CCP_Monitoring_Data.xlsx`

### Workbook structure

The worksheet's stored range extends to a completely blank final column (`Q`). It has no header or data. `readxl::read_excel()` ignores this trailing blank column automatically, so no manual deletion or value change is required. The 68 substantive district/reporting-unit rows were retained.

### `Division`

- Eight valid division values are present.
- No missing values, inconsistent capitalization, leading/trailing spaces, or district-to-division conflicts were found.
- No correction was required.

### `District`

Two confirmed text issues were corrected:

| Excel row | Original | Corrected | Issue |
|---:|---|---|---|
| 7 | `Cox’s Bazar` | `Cox's Bazar` | Smart apostrophe prevented an exact text match. |
| 23 | `Narshingdi` | `Narsingdi` | Spelling differed from the corresponding historical-data district. |

Four rows are city reporting units rather than districts:

| Excel row | Division | Source value | Comparison key used |
|---:|---|---|---|
| 66 | Dhaka | Dhaka North City | Dhaka |
| 67 | Dhaka | Dhaka South City | Dhaka |
| 68 | Chattogram | Chattogram City | Chattogram |
| 69 | Khulna | Khulna City | Khulna |

The original city labels remain unchanged for monitoring analysis. Parent-district keys are used only when comparing with the historical workbook, which has no equivalent city rows. This is an analytical assumption and should be confirmed against a facility crosswalk.

### `Facilities`

- Values are complete, positive integers.
- The column totals 658 facilities, equal to the 658 facility rows in the historical workbook.
- Despite equal totals, facility allocation differs in 59 of the 64 consolidated districts. This cannot be corrected safely without dated facility rosters and stable facility IDs. The complete district discrepancy table appears in Section 3.

### Session columns

Columns checked:

- `ANC Sessions`
- `PNC Sessions`
- `SCANU Sessions`
- `General Medicine Sessions`
- `General Surgury Sessions`
- `NCD Sessions`

Issues and treatment:

- The header `General Surgury Sessions` contains a spelling error. It is renamed `general_surgery_sessions` in R; its values are unchanged.
- All six columns contain complete, positive integer values.
- No negative, zero, fractional, infinite, or non-numeric values were found.
- District quarterly totals are recalculated as the row-wise sum of these six columns.

### `Total Patients` and `Total Caregivers`

- Both columns contain complete, positive integer values.
- No duplicate-field, type, range, or internal-consistency problem was found.
- Their sum is used as total participants for checking `Avg Participants/Session`.

### `Trainers`

- Values are complete, positive integers.
- No value was altered.
- Differences in trainer coverage are operational indicators, not demonstrated data errors.

### `Avg Sessions/Week`

- Every row agrees with the recalculated quarterly session total divided by 13 weeks, allowing for the source column's one-decimal rounding.
- There were zero formula inconsistencies, so the supplied values were retained as an audit field.

### `Avg Participants/Session`

- Every row equals the rounded value of `(Total Patients + Total Caregivers) / Total Sessions`.
- There were zero formula inconsistencies.
- The source column is rounded to whole participants; the analysis also retains the unrounded calculated ratio.

### `% Sessions vs Target`

- All values are numeric and within 0–100%.
- Four values are unusually low under a conservative three-IQR check within their divisions:

| Excel row | Division | District | Value |
|---:|---|---|---:|
| 17 | Chattogram | Noakhali | 44.2% |
| 19 | Dhaka | Dhaka | 35.7% |
| 27 | Dhaka | Narayanganj | 39.6% |
| 54 | Rangpur | Dinajpur | 38.0% |

These are management and source-verification flags, not confirmed data-entry errors. They remain unchanged.

### `CCP Quality Assessment Coverage %`

- All values are numeric and within 0–100%.
- No missing values or conservative three-IQR anomalies were found.
- Values remain unchanged.

## 2. `Average_sessions_details.xlsx`

### `Active since`

- Values are complete integer years from 2022 through 2026; no future year was found.
- The column supplies only a year, not an activation month or full date.
- The implied averaging periods vary considerably among facilities with the same activation year:

| Active since | Facilities | Minimum implied months | Median implied months | Maximum implied months |
|---:|---:|---:|---:|---:|
| 2022 | 20 | 22 | 31.5 | 41 |
| 2023 | 37 | 8 | 22.0 | 30 |
| 2024 | 338 | 2 | 12.0 | 19 |
| 2025 | 170 | 1 | 5.0 | 9 |
| 2026 | 93 | 1 | 4.0 | 7 |

The session and participant averages imply the same whole-number denominator for every facility, so the averages are internally consistent. However, the exact time window cannot be reconstructed from a year-only field. The analysis uses the supplied averages and treats `average monthly sessions × 3` as a diagnostic estimate, not an exact forecast.

### `Division`

The workbook contains both `Barisal` and `Barishal`. `Barisal` is standardized to `Barishal` for 10 facility rows:

| Original division | District | Rows affected | Corrected division |
|---|---|---:|---|
| Barisal | Barguna | 2 | Barishal |
| Barisal | Bhola | 1 | Barishal |
| Barisal | Jhalokathi | 2 | Barishal |
| Barisal | Patuakhali | 3 | Barishal |
| Barisal | Pirojpur | 2 | Barishal |

After correction, each district maps to only one division across both workbooks.

### `District`

The following spelling variants were standardized:

| Original | Corrected | Facility rows affected |
|---|---|---:|
| Jhalakathi | Jhalokathi | 5 |
| Laxmipur | Lakshmipur | 2 |
| Sariatpur | Shariatpur | 2 |
| Bogra | Bogura | 4 |
| Thakurgaong | Thakurgaon | 3 |

Some `Jhalokathi` rows also contained the `Barisal` division variant described above. After normalization and city-to-parent comparison mapping, all 64 district geographies match across the two sources.

### `Facility Type`

The assignment brief describes eight facility categories, but the workbook contains nine:

| Facility type | Rows |
|---|---:|
| Upazila Health Complex | 422 |
| Union Health and Family Welfare Center | 79 |
| District Hospital | 63 |
| MCWC | 60 |
| Medical College Hospital | 25 |
| Specialized Hospital | 5 |
| MCHTI | 2 |
| Medical University | 1 |
| MFSTC | 1 |

No category was deleted or merged because the workbook provides no evidence that one category is erroneous. This remains a documentation/data-definition discrepancy that should be clarified with the data owner.

### `Facility name`

The column contains 658 unique facility names. No exact or normalized-key duplicates were found. Thirty-six names required safe formatting or embedded-location corrections:

| Excel row | Original | Corrected |
|---:|---|---|
| 56 | Adhunik District Sadar Hospital, Khagrachari | Adhunik District Sadar Hospital, Khagrachhari |
| 130 | Belabo Upazila Health Complex, Narshingdi | Belabo Upazila Health Complex, Narsingdi |
| 131 | Monohardi Upazila Health Complex, Narshingdi | Monohardi Upazila Health Complex, Narsingdi |
| 132 | Palash Upazila Health Complex, Narshingdi | Palash Upazila Health Complex, Narsingdi |
| 133 | Raipura Upazila Health Complex, Narshingdi | Raipura Upazila Health Complex, Narsingdi |
| 134 | Shibpur Upazila Health Complex, Narshingdi | Shibpur Upazila Health Complex, Narsingdi |
| 153 | Ghatail Upazila Health Complex,Tangail | Ghatail Upazila Health Complex, Tangail |
| 154 | Kalihati Upazila Health Complex,Tangail | Kalihati Upazila Health Complex, Tangail |
| 155 | Nagarpur Upazila Health Complex,,Tangail | Nagarpur Upazila Health Complex, Tangail |
| 156 | Mirzapur Upazila Health Complex,Tangail | Mirzapur Upazila Health Complex, Tangail |
| 157 | Gopalpur Upazila Health Complex,,Tangail | Gopalpur Upazila Health Complex, Tangail |
| 158 | Delduar Upazila Health Complex,Tangail | Delduar Upazila Health Complex, Tangail |
| 159 | Bhuapur Upazila Health Complex,Tangail | Bhuapur Upazila Health Complex, Tangail |
| 160 | Dhanbari Upazila Health Complex,Tangail | Dhanbari Upazila Health Complex, Tangail |
| 204 | Manpura Upazila Health Complex, Bola | Manpura Upazila Health Complex, Bhola |
| 219 | Kathalia Upazila Health Complex, Jhalakathi | Kathalia Upazila Health Complex, Jhalokathi |
| 220 | Nalchiti Upazila Health Complex, Jhalakathi | Nalchiti Upazila Health Complex, Jhalokathi |
| 221 | Rajapur Upazila Health Complex, Jhalakathi | Rajapur Upazila Health Complex, Jhalokathi |
| 281 | Barlekha Upazila Health Complex, Moulavibazar | Barlekha Upazila Health Complex, Moulvibazar |
| 387 | Jhalakathi District Hospital | Jhalokathi District Hospital |
| 404 | Kustia Medical College & Hospital, Kustia | Kushtia Medical College & Hospital, Kushtia |
| 432 | Hathazari Upazila Health Complex, Chaattogram | Hathazari Upazila Health Complex, Chattogram |
| 500 | Mother and Child Welfare Centre (MCWC), Kishoreganj `[line break]` Sadar, Kishoreganj | Mother and Child Welfare Centre (MCWC), Kishoreganj Sadar, Kishoreganj |
| 505 | Mother and Child Welfare Centre (MCWC), Cox’s Bazar Sadar, Cox’s Bazar | Mother and Child Welfare Centre (MCWC), Cox's Bazar Sadar, Cox's Bazar |
| 507 | Mother and Child Welfare Centre (MCWC), Khagrachari Sadar, Khagrachari | Mother and Child Welfare Centre (MCWC), Khagrachhari Sadar, Khagrachhari |
| 510 | Mother and Child Welfare Centre (MCWC), Laksmipur Sadar, Laksmipur | Mother and Child Welfare Centre (MCWC), Lakshmipur Sadar, Lakshmipur |
| 516 | Mother and Child Welfare Centre (MCWC), Jhalakathi Sadar, Jhalakathi | Mother and Child Welfare Centre (MCWC), Jhalokathi Sadar, Jhalokathi |
| 567 | Kamarpukur Union Health & Family welfare Centre, Syedpur, Nilphamari, | Kamarpukur Union Health & Family welfare Centre, Syedpur, Nilphamari |
| 588 | Sonapur Union Health & Family welfare Centre, Raipur, Laxmipur | Sonapur Union Health & Family welfare Centre, Raipur, Lakshmipur |
| 589 | Charpata Union Health & Family welfare Centre,Raipur, Laxmipur | Charpata Union Health & Family welfare Centre, Raipur, Lakshmipur |
| 592 | Chengmari Union Health & Family welfare Centre,Mithapukur,Rangpur | Chengmari Union Health & Family welfare Centre, Mithapukur, Rangpur |
| 593 | Gopalpur Union Health & Family welfare Centre,Mithapukur,Rangpur | Gopalpur Union Health & Family welfare Centre, Mithapukur, Rangpur |
| 597 | shapleja ,Union Health & Family welfare Centre, Mathbaria, Pirojpur | shapleja, Union Health & Family welfare Centre, Mathbaria, Pirojpur |
| 602 | Pangashia, Union Health & Family welfare Centre, Dumki, Patukhali | Pangashia, Union Health & Family welfare Centre, Dumki, Patuakhali |
| 603 | Bohorampur, Union Health & Family welfare Centre, Dashmina, Patukhali | Bohorampur, Union Health & Family welfare Centre, Dashmina, Patuakhali |
| 638 | Telikhal Union Health & Family welfare Centre, Companiganj, Sulhet | Telikhal Union Health & Family welfare Centre, Companiganj, Sylhet |

Only demonstrable punctuation, whitespace, line-break, trailing-comma, and location-name problems were corrected. Other words and capitalization were left unchanged because no authoritative facility master list was supplied.

### `Total number of sessions`

- Values are complete, positive integers.
- No invalid type, negative value, duplicate-field problem, or internal inconsistency was found.
- Cumulative totals naturally vary with activation and observation length; they are not compared directly with the quarterly monitoring snapshot.

### `Average monthly sessions`

- Values are complete, numeric, finite, and positive.
- Multiplying each value by its implied whole-number month count reproduces `Total number of sessions` within stored precision.
- Twenty values are unusually high relative to their facility type under a conservative three-IQR rule:

| Excel row | Division | District | Average monthly sessions |
|---:|---|---|---:|
| 222 | Barishal | Patuakhali | 47.78 |
| 404 | Khulna | Kushtia | 39.29 |
| 555 | Rajshahi | Bogura | 34.00 |
| 142 | Dhaka | Munshiganj | 33.11 |
| 107 | Mymensingh | Sherpur | 31.25 |
| 379 | Chattogram | Brahmanbaria | 28.00 |
| 41 | Mymensingh | Mymensingh | 27.55 |
| 235 | Rangpur | Dinajpur | 27.42 |
| 83 | Khulna | Jhenaidah | 26.13 |
| 175 | Khulna | Khulna | 25.56 |
| 27 | Sylhet | Habiganj | 25.42 |
| 36 | Khulna | Satkhira | 25.23 |
| 540 | Rangpur | Nilphamari | 24.83 |
| 262 | Rangpur | Nilphamari | 23.50 |
| 98 | Mymensingh | Jamalpur | 23.24 |
| 302 | Rajshahi | Naogaon | 23.11 |
| 416 | Chattogram | Khagrachhari | 23.00 |
| 344 | Khulna | Khulna | 22.78 |
| 217 | Barishal | Barguna | 22.60 |
| 4 | Dhaka | Dhaka | 11.61 |

Row 4 is flagged relative to the very small `Specialized Hospital` group, not because 11.61 is globally high. All 20 values remain unchanged and require source-register or contextual review before being called errors.

### `Total number of participants`

- Values are complete, positive integers.
- No invalid type, range, or internal-consistency issue was found.

### `Average monthly participants`

- Values are complete, numeric, finite, and positive.
- The implied month denominator agrees with that calculated from sessions for every facility.
- `Average monthly participants / Average monthly sessions` agrees with `Total participants / Total sessions` for every facility within stored precision.
- No value was altered.

## 3. Cross-file facility-count disagreement

After spelling correction and consolidation of city reporting units to parent districts, all 64 district geographies match. However, only five districts have the same facility count in both sources. The following 59 do not. `Gap` equals monitoring facilities minus historical facility rows.

| Division | District | Monitoring | Historical | Gap |
|---|---|---:|---:|---:|
| Barishal | Barguna | 21 | 9 | 12 |
| Barishal | Barishal | 10 | 11 | -1 |
| Barishal | Bhola | 11 | 8 | 3 |
| Barishal | Jhalokathi | 10 | 7 | 3 |
| Barishal | Patuakhali | 5 | 10 | -5 |
| Barishal | Pirojpur | 11 | 10 | 1 |
| Chattogram | Bandarban | 11 | 8 | 3 |
| Chattogram | Brahmanbaria | 8 | 10 | -2 |
| Chattogram | Chandpur | 7 | 8 | -1 |
| Chattogram | Chattogram | 21 | 20 | 1 |
| Chattogram | Cox's Bazar | 6 | 9 | -3 |
| Chattogram | Cumilla | 13 | 24 | -11 |
| Chattogram | Feni | 22 | 7 | 15 |
| Chattogram | Khagrachhari | 12 | 9 | 3 |
| Chattogram | Noakhali | 9 | 12 | -3 |
| Chattogram | Rangamati | 11 | 13 | -2 |
| Dhaka | Dhaka | 35 | 21 | 14 |
| Dhaka | Faridpur | 6 | 11 | -5 |
| Dhaka | Gazipur | 5 | 8 | -3 |
| Dhaka | Gopalganj | 8 | 6 | 2 |
| Dhaka | Kishoreganj | 6 | 19 | -13 |
| Dhaka | Madaripur | 12 | 5 | 7 |
| Dhaka | Manikganj | 8 | 9 | -1 |
| Dhaka | Munshiganj | 10 | 7 | 3 |
| Dhaka | Narayanganj | 15 | 7 | 8 |
| Dhaka | Narsingdi | 9 | 8 | 1 |
| Dhaka | Rajbari | 16 | 6 | 10 |
| Dhaka | Shariatpur | 7 | 8 | -1 |
| Dhaka | Tangail | 8 | 16 | -8 |
| Khulna | Bagerhat | 7 | 13 | -6 |
| Khulna | Jashore | 11 | 9 | 2 |
| Khulna | Khulna | 18 | 17 | 1 |
| Khulna | Kushtia | 11 | 8 | 3 |
| Khulna | Magura | 11 | 5 | 6 |
| Khulna | Meherpur | 7 | 4 | 3 |
| Khulna | Narail | 7 | 4 | 3 |
| Khulna | Satkhira | 6 | 10 | -4 |
| Mymensingh | Jamalpur | 7 | 9 | -2 |
| Mymensingh | Mymensingh | 12 | 17 | -5 |
| Mymensingh | Netrokona | 6 | 10 | -4 |
| Mymensingh | Sherpur | 5 | 8 | -3 |
| Rajshahi | Bogura | 12 | 17 | -5 |
| Rajshahi | Chapainawabganj | 9 | 5 | 4 |
| Rajshahi | Joypurhat | 16 | 6 | 10 |
| Rajshahi | Naogaon | 7 | 12 | -5 |
| Rajshahi | Natore | 14 | 9 | 5 |
| Rajshahi | Pabna | 11 | 10 | 1 |
| Rajshahi | Rajshahi | 8 | 14 | -6 |
| Rajshahi | Sirajganj | 11 | 13 | -2 |
| Rangpur | Dinajpur | 7 | 15 | -8 |
| Rangpur | Gaibandha | 6 | 8 | -2 |
| Rangpur | Kurigram | 8 | 11 | -3 |
| Rangpur | Lalmonirhat | 16 | 6 | 10 |
| Rangpur | Nilphamari | 6 | 9 | -3 |
| Rangpur | Panchagarh | 9 | 6 | 3 |
| Rangpur | Rangpur | 10 | 11 | -1 |
| Rangpur | Thakurgaon | 5 | 9 | -4 |
| Sylhet | Sunamganj | 8 | 15 | -7 |
| Sylhet | Sylhet | 10 | 18 | -8 |

The five exact facility-count matches are Lakshmipur, Chuadanga, Jhenaidah, Habiganj, and Moulvibazar. Because both workbooks total 658 facilities, the problem is allocation or scope by district rather than a difference in the overall program total. Plausible explanations include different active-facility rosters, city/district reporting definitions, activation dates, transfers between reporting groups, or data-generation errors. None can be selected as the definitive explanation from the supplied files alone.

## 4. Checks that passed across both files

The following automated checks returned zero failures:

- Required-column and numeric-type validation
- Missing required values
- Exact duplicate rows
- Duplicate district or facility names after ignoring case, punctuation, and spacing
- Infinite or `NaN` values
- Negative values
- Zero/non-positive count values
- Fractional values in count fields
- Percentages outside 0–100%
- Monitoring weekly-session and participant-per-session formula checks
- Historical session/participant averaging-denominator checks
- Future activation years
- District-to-division conflicts
- Unmatched district geographies after documented normalization

Passing these checks does not prove that every observation is correct; it means no additional issue was detectable from the supplied fields without external source records.

## 5. Dashboard-readiness and Looker Studio issues

This section records issues that matter specifically when the analytical data are converted into a dashboard. They are not necessarily errors in the source workbook; many are modelling, aggregation, or governance risks that could produce a misleading dashboard if left undocumented.

### 5.1 No dated reporting period

The monitoring workbook describes a quarter but does not identify its start date, end date, quarter number, or year. Therefore:

- `reporting_period_id` is set to `CURRENT_SNAPSHOT`;
- `reporting_period_label` says that the source date was not supplied;
- `reporting_period_end` is blank;
- no current time-series chart or date-range control is valid.

This is **flagged, not corrected**. Assigning an assumed date would create false information. The `monthly_upload_template` worksheet shows the date field that should be populated during future reporting.

### 5.2 Reporting grain changes from 68 units to 64 consolidated districts

The original monitoring workbook has 68 reporting-unit rows. The dashboard source has 64 consolidated district rows because four city units are grouped with their parent districts for cross-source consistency:

| Dashboard district | Original rows included | Dashboard reporting-unit count |
|---|---|---:|
| Chattogram | Chattogram and Chattogram City | 2 |
| Dhaka | Dhaka, Dhaka North City, and Dhaka South City | 3 |
| Khulna | Khulna and Khulna City | 2 |

All other districts have one contributing monitoring row. `includes_city_reporting_unit` and `reporting_units` retain this information in the dashboard data.

This consolidation is appropriate for the stated district comparison and prevents duplicated historical benchmarks. However, it means the 64-row dashboard cannot separately compare Dhaka North City with Dhaka South City. If managers need city-level accountability, the historical data require a matching city/facility crosswalk and the dashboard should use a separate 68-row reporting-unit source.

### 5.3 Percentages use different storage scales in analysis and dashboard files

The source and main R analysis store target and QA percentages on a 0–100 scale; for example, 85% is stored as `85`. The dashboard export stores rates on a 0–1 scale; the same value is `0.85`.

This conversion is deliberate because Looker Studio's Percent field type treats `0.85` as 85%. If a user sets a 0–100 source field to Percent, Looker Studio may display 8,500%. The `data_dictionary` worksheet records the intended types.

Current dashboard checks confirm:

- `sessions_vs_target_rate` ranges from 0.38 to 1.00;
- `qa_coverage_rate` ranges from 0.51 to 0.916;
- neither field is missing;
- neither field is outside 0–1 after conversion.

### 5.4 Rates and ratios are not additive

The following fields must never be summed:

- `sessions_vs_target_rate`;
- `qa_coverage_rate`;
- `benchmark_session_gap_rate`;
- `sessions_per_facility`;
- `participants_per_session`;
- `facilities_per_trainer`;
- patient and caregiver shares.

Simple averages can also be wrong. For example, averaging district target rates gives every district equal influence even when their session volumes differ. The dashboard therefore includes two helper fields:

```text
target_weighted_numerator = district target rate × district sessions
qa_weighted_numerator = district QA rate × district facilities
```

The correct filter-aware Looker Studio calculations are:

```text
SUM(target_weighted_numerator) / SUM(total_sessions_quarter)
SUM(qa_weighted_numerator) / SUM(facilities)
SUM(total_participants) / SUM(total_sessions_quarter)
```

The last formula recalculates participants per session across the selected records. The same numerator/denominator principle should be used for sessions per facility and facilities per trainer.

### 5.5 Summary calculations must be made before source names are overwritten

During dashboard development, a test identified a potential R aggregation trap in consolidated districts. If `summarise()` first creates a new column called `total_sessions_quarter`, a later expression with the same name can see the newly summarized value rather than the original row values. That would inflate weighted results for groups containing more than one source row.

The final code avoids this by creating `row_target_weighted_numerator` and `row_qa_weighted_numerator` in `mutate()` **before** `group_by()` and `summarise()`. The grouped step then sums those row-level numerators. After the correction, all district target rates fall between 38% and 100%, and all QA rates fall between 51% and 91.6%, consistent with the source ranges.

This is a useful general lesson: when a grouped calculation depends on a source column that will also be summarized under the same name, calculate the dependent row-level component first or give the summary a different name.

### 5.6 Demonstration status thresholds are assumptions

The dashboard uses portfolio demonstration thresholds:

| Measure | Green/strong | Amber/watch | Red/critical |
|---|---:|---:|---:|
| Sessions vs target | 85% or higher | 70% to below 85% | Below 70% |
| QA coverage | 80% or higher | 60% to below 80% | Below 60% |

Overall attention is Urgent if either measure is below 60%, Watch if either misses its higher threshold, and On track otherwise. In the current 64-row export this produces:

- 20 Urgent districts;
- 31 Watch districts;
- 13 On-track districts.

These categories are **derived assumptions**, not a Noora Health policy supplied in the assignment. They are useful for demonstrating conditional formatting and exception management, but managers should confirm the thresholds and document any replacement rules before operational use.

### 5.7 Cross-file review flags are intentionally broad

`cross_file_review_flag` equals 1 when the district facility count differs between sources or the session total differs from the historical three-month estimate by more than 20%. It flags 62 of the 64 dashboard districts.

The high count does not mean 62 districts performed poorly. It primarily reflects the already documented facility-roster and period differences:

- 59 districts have unequal facility counts;
- 50 districts differ from the historical session estimate by more than 20%;
- the conditions overlap.

The flag should therefore feed a **data-review queue**, not a red performance scorecard. Managers should filter or sort by the size and type of discrepancy rather than treat every flag as equally serious.

### 5.8 The historical estimate is not a target

`expected_quarter_sessions` is calculated as the sum of facility average monthly sessions multiplied by three. Its limitations remain:

- the exact historical averaging window is unknown;
- facility rosters differ by district;
- source city/district definitions differ;
- activation timing may differ;
- an older average may not reflect current operating conditions.

For this reason, `benchmark_session_gap` and `benchmark_session_gap_rate` are labelled diagnostic fields. A bar chart using them must be titled as a historical-alignment or source-review chart—not as a target-performance chart.

### 5.9 Division summary and district data are different grains

The `division_summary` worksheet contains one row per division and includes the pre-calculated priority score. The `dashboard_data` worksheet contains one row per district. They must not be appended into one table or blended on division without a carefully defined one-to-many relationship, because repeated division totals would inflate results.

Recommended use:

- use `dashboard_data` for most scorecards, division/district charts, filters, and exception tables;
- use `division_summary` only for visuals requiring the priority score or its normalized risk components;
- use `action_tracker` as a separate source for management follow-up.

### 5.10 Proposed action fields are not observed progress

The action tracker contains three evidence-based recommended actions, but the assignment does not supply real owners, due dates, status updates, or evidence links. Therefore:

- `action_status` says `Proposed - not yet assigned`;
- `action_owner`, `target_date`, `last_updated_date`, and `evidence_link` are blank;
- `latest_update` says no operational update was supplied.

These placeholders make missing accountability visible without inventing progress. They should be replaced only after a manager agrees an owner and update.

### 5.11 CSV upload can duplicate the snapshot

Looker Studio's CSV file-upload workflow appends additional files to a dataset. Re-uploading the same 64-row snapshot can therefore double sessions, participants, facilities, and flags without producing obvious row errors in the dashboard.

Safe refresh rules:

- when correcting the same snapshot, replace the source or create a new dataset rather than append;
- append only a genuinely new reporting period;
- include the period in the unique key for future data;
- check distinct record count and total sessions after every upload.

A Google Sheets source can be easier for a demonstration because the current worksheet can be replaced in place, but field additions or type changes still require a data-source field refresh.

### 5.12 Mapping limitations

The current files contain text division and district names but no official geographic code, latitude, longitude, or boundary file. A map connector may misinterpret a district name, especially when a name also exists elsewhere.

No map is recommended for the current portfolio version. A ranked bar or bubble plot communicates the management comparisons without geocoding risk. Add a map only after obtaining a validated Bangladesh district code and coordinates or boundaries.

### 5.13 Dashboard export checks that passed

The generated `Looker_Dashboard_Data.csv` currently passes these checks:

- 64 rows and 47 fields;
- 64 unique `record_id` values;
- total sessions reconcile to 12,608 in the cleaned monitoring data;
- total participants reconcile to 44,355;
- all field names begin with a letter or underscore, use only letters/numbers/underscores, and stay within 128 characters;
- no embedded line breaks in text fields;
- no missing target or QA rate;
- dashboard rate ranges match the source after converting from 0–100 to 0–1;
- three dashboard districts correctly record that city reporting units were consolidated;
- the monthly template contains headers but zero invented observations.

These tests show structural readiness for import. They do not resolve the unknown source period, facility-roster disagreement, benchmark comparability, threshold approval, or future action ownership.

## 6. Assignment-structure and task-fidelity corrections

`Monitoring Manager Assignment.md` is now the authoritative task transcription. A task-fidelity review found that the earlier report used several paraphrased “Main question” and “Question” labels. Most answers addressed the intended content, but the labels were not the assignment's actual task statements, and Section C was incorrectly displayed as two questions even though the source contains one combined task.

### Correct required structure

| Section | Required tasks | Correct interpretation |
|---|---:|---|
| A | A1–A4 | Division ranking; five-district comparison; three management priorities plus dashboard; executive summary |
| B | B1–B2 | Remote-facility monitoring approach; support for one monitoring officer |
| C | C1 | One combined weakness/risk analysis and 3–5 sentence rewrite |
| D | D1–D2 | Eight-note thematic synthesis; Program Learning Note and one feedback-process change |

There are exactly nine required task statements. The corrected report contains nine dark pink/reddish task boxes and no invented “Main question” or numbered “Question” labels.

### Supporting work is no longer presented as an assignment question

The data-cleaning section remains because it is necessary for reproducible analysis. Its pale-green box is labelled **“Data Processing”**, visually separating supporting preparation from the reddish-pink assignment task boxes. The export, limitations, references, abbreviations, and Looker Studio extension are also supporting sections rather than new requirements from the assignment transcription.

### Formula-display audit

The corrected report visibly states the mathematical function wherever a task depends on a calculation:

- district total sessions as the sum of six topic counts;
- division target achievement as a session-volume-weighted percentage;
- expected district quarterly sessions as three times the sum of facility monthly averages;
- absolute and percentage gaps between current and expected sessions;
- min-max normalization and the five-component management priority score;
- session achievement, participant reach, participants/session, nurse continuity, QA score, and timely reporting for remote facilities;
- routine visit workload per quarter and per month;
- word-count limits for A4 and D2;
- sentence-count limits for B2 and C1;
- filter-safe weighted formulas in the Looker Studio extension.

This audit distinguishes three kinds of numbers:

1. **Source calculations**, such as district sessions and participants;
2. **Analytical assumptions**, such as the 20% comparison flag and priority-score weights;
3. **Future operational definitions**, such as remote-facility timeliness and QA rates.

Keeping these categories explicit prevents an assumed management rule from being mistaken for a value supplied by Noora Health.

### Qualitative-note traceability

The corrected report reproduces the eight supplied notes and numbers them in their original order. The four-theme table assigns:

- notes 1 and 8 to content relevance and duplication;
- notes 3, 5, and 6 to adaptive timing and inclusive access;
- note 2 to workforce continuity;
- notes 4 and 7 to data-tool reliability and feedback closure.

Every note is included once. The workforce theme is described as a systemic **risk requiring verification**, rather than claiming that one Jashore example proves organization-wide prevalence.

### Remaining interpretation safeguards

- The Markdown assignment says the monitoring file represents 68 districts, while the workbook contains four city-labelled reporting units. The report uses “68 reporting areas” for the source and documents the 64 consolidated comparison districts.
- The Markdown assignment describes eight facility categories, while the historical workbook contains nine distinct values. The report retains all nine and flags the discrepancy.
- The assignment requests “any five districts.” The report uses a volume-stratified, reproducible selection rule; this is an analyst choice, not an extra task requirement.
- The 20% gap threshold and composite-priority weights are disclosed analytical assumptions.
- The Additional Content dashboard remains optional and follows the completed A–D tasks.

### Figure and table presentation audit

- The report contains one displayed chart, labelled **Fig. 1** directly below the image.
- Its centered caption explains the horizontal weighted-target axis, vertical division axis, percentage labels, and orange-to-green QA fill.
- Fifteen displayed tables are captioned sequentially as **Table 1** through **Table 15**.
- The sequence includes both R-generated and Markdown tables, so numbering is maintained explicitly rather than by separate automatic counters.
