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
