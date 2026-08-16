# Monitoring Manager Assignment — Simplified Task List

## 1. Prepare and reconcile the data

- [ ] Use `CCP_Monitoring_Data.xlsx` as the district-level quarterly snapshot (68 rows).
- [ ] Use `Average_sessions_details.xlsx` as the facility-level historical dataset (658 rows).
- [ ] Create **Total Sessions This Quarter** for each district by summing ANC, PNC, SCANU, General Medicine, General Surgery, and NCD sessions.
- [ ] Standardize location names before joining the files. Check variants such as `Barishal/Barisal`, `Bogura/Bogra`, `Narsingdi/Narshingdi`, `Jhalokathi/Jhalakathi`, `Lakshmipur/Laxmipur`, `Shariatpur/Sariatpur`, `Thakurgaon/Thakurgaong`, and the apostrophe in `Cox's Bazar`.
- [ ] Decide how to handle city rows in the monitoring file (`Chattogram City`, `Dhaka North City`, `Dhaka South City`, and `Khulna City`) because they do not have direct district-name matches in the historical file.
- [ ] Flag that district facility counts often differ between the two files, although both contain 658 facilities in total. Treat the sources as different reporting layers/time windows, not perfectly reconciled records.

## 2. Complete the quantitative analysis

- [ ] Roll the 68 district rows up to the 8 divisions.
- [ ] Calculate each division's weighted **% Sessions vs Target** using district total-session volume as the weight: `SUM(district % × district total sessions) / SUM(district total sessions)`.
- [ ] Rank all 8 divisions from strongest to weakest.
- [ ] Select any 5 districts that can be matched across both files.
- [ ] For each selected district, calculate expected quarterly sessions as: `SUM(facility average monthly sessions) × 3`.
- [ ] Compare expected quarterly sessions with reported quarterly sessions; show the absolute and percentage gap.
- [ ] Identify apparent inconsistencies, propose plausible hypotheses, and state when the figures still need verification rather than assuming the sources should agree.

## 3. Turn the analysis into management recommendations

- [ ] Recommend 3 divisions for management attention this quarter.
- [ ] Base the choice on more than one dimension—for example target achievement, session volume, participant reach, quality-assessment coverage, trainers, or facility-level consistency.
- [ ] Describe a dashboard for the Implementation Team. For every proposed visual, state the single decision or question it should support.
- [ ] Write a **200–250 word executive summary** for the Country Director, focused on the most important findings, risks, and actions.

## 4. Design monitoring for the 15 remote expansion facilities

- [ ] Propose 4–5 priority indicators and explain why each matters.
- [ ] Design an offline-first collection and synchronization process for unreliable internet.
- [ ] Recommend a realistic field-visit frequency for facilities located 4–6 hours from the divisional office, with justification.
- [ ] Identify the top 3 expansion-specific risks and give one mitigation for each.
- [ ] In **4–6 sentences**, explain how to support one monitoring officer covering 5 facilities while protecting data quality without unrealistic travel.

## 5. Improve the draft monitoring plan

- [ ] Identify at least 3 specific weaknesses in the WhatsApp/self-reporting/master-spreadsheet/quarterly-review approach.
- [ ] For each weakness, explain the concrete operational or data-quality risk to Noora Health.
- [ ] Rewrite the plan in **3–5 sentences** with clearer indicators, standardized collection, validation, ownership/access controls, review cadence, and escalation rules.
- [ ] Keep routine program monitoring data separate from research data unless governance, consent, definitions, and access are explicitly aligned.

## 6. Synthesize the qualitative feedback

- [ ] Group all 8 field notes into 3–4 meaningful themes.
- [ ] For each theme, provide: a short label, one sentence describing the underlying program pattern, and an isolated-versus-systemic judgment with reasoning.
- [ ] Write a **100–150 word Program Learning Note** for the design and implementation teams.
- [ ] Recommend one concrete process or tool change that captures and follows up qualitative feedback systematically.

## 7. Final submission check

- [ ] Combine all sections into one clear, leadership-ready document.
- [ ] Show formulas, assumptions, selected districts, and major data limitations.
- [ ] Check all required word and sentence limits.
- [ ] Cite any external references used; no citations are needed for calculations based only on the supplied files.
- [ ] Export the final document as a **PDF**.
- [ ] Submit within 72 hours of receiving the assignment, by **11:59 PM**.

