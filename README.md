# William & Mary Admissions Analysis Dashboard

A three-page Power BI report comparing first-year admissions for Fall 2024 and Fall 2025 using William & Mary's public Common Data Sets. Built to practice data preparation, performance reporting, consistency checks, and communicating findings to technical and nontechnical readers.

## Tools

- **Power BI Desktop:** interactive report pages, tables, column charts, and KPI cards.
- **Power Query:** PDF import, column standardization, appending, conditional classification, data types, and pivoting.
- **DAX:** totals, admission and yield rates, quality checks, and year-to-year percentage changes.

## Data sources

- [Common Data Set 2024–2025, Section C](https://www.wm.edu/offices/ir/university_data/cds/cds-2024-2025_c.pdf): Fall 2024 admissions.
- [Common Data Set 2025–2026, Section C](https://www.wm.edu/offices/ir/university_data/cds/cds_2025-26_c.pdf): Fall 2025 admissions.

The Common Data Set publication year differs from the fall reporting year. This project uses aggregate counts, not individual student records. This is an independent portfolio project, not work commissioned by William & Mary.

## Results

| Metric | Fall 2024 | Fall 2025 |
|---|---:|---:|
| Applicants | 17,798 | 16,895 |
| Admitted | 6,064 | 6,245 |
| Enrolled | 1,614 | 1,639 |
| Admission rate | 34.1% | 37.0% |
| Yield rate | 26.6% | 26.2% |

- Applications decreased by **903 (5.1%)**.
- Admitted students increased by **181 (3.0%)**.
- Enrollment increased by **25 (1.5%)**.
- Admission rate increased while yield decreased slightly.

These figures describe a two-year comparison. They do not establish a long-term trend or explain the causes of the changes. Percentage changes use unrounded counts; displayed rates are rounded to one decimal place.

## Report pages

1. **Fall 2025 Admissions:** applicant, admitted, and enrolled totals; admission and yield rates; missing-count and ordering checks; counts by sex category.
2. **Findings & Checks:** plain-language Fall 2025 findings, quality-check definitions and scope, metric definitions, and source information.
3. **2024–2025 Comparison:** annual summary table, clustered column chart, application and enrollment change cards, and findings with source information.

Clicking chart bars filters other visuals. Clear selections before exporting a report intended to show both years. The comparison table's total rates are calculated from combined counts, rather than by averaging the two displayed annual percentages.

## Files

| File | Purpose |
|---|---|
| `WM_Admissions_Analysis.pbix` | Editable Power BI project |
| `WM_Admissions_2024_2025_Report.pdf` | Three-page report export |
| `WM_Admissions_2024_Flat.pdf` | Flattened 2024 source for import |
| `WM_Admissions_2025_Flat.pdf` | Flattened 2025 source for import |
| `README.md` | Project documentation |

The latest uploaded project copy was named `WM_Admissions_Analysis(1).pbix`; it may be renamed to the standard project filename above. This README does not bundle the listed files.

## Preparation workflow

1. Open each official source PDF and print it using **Microsoft Print to PDF**. This makes form-field values readable by the Power BI PDF connector.
2. Import the flattened PDFs with **Get data → PDF → Transform Data**.
3. For 2025, select the admissions table, retain the applicant/admitted/enrolled aggregate sections, and exclude section headers and the separate full-time/part-time breakdown to avoid double counting.
4. For 2024, import **Table003, Table004, Table005, and Table006 (Page 1)**. The Unknown admitted count appears separately in Table005.
5. Standardize the two columns in each 2024 table to `Description` and `Count`, then append the four tables to obtain 12 rows.
6. Add `Stage`, `Sex`, and `Year`; remove `Description`; pivot `Stage` using `Count` with Sum.
7. Set `Sex` to Text and `Year`, `Applicants`, `Admitted`, and `Enrolled` to Whole Number.
8. Append the prepared 2025 rows to the prepared 2024 query and rename the result `Admissions_AllYears`.
9. Close & Apply, create measures and visuals, then save the PBIX and export the PDF.

Table identifiers describe the imported copies used in this project and should be checked against the preview if re-importing. The existing `Admissions_2025` query remains the source for the original Fall 2025 pages. The combined query was created by appending directly to the prepared 2024 query and renaming it.

### Power Query custom columns

Stage:

```powerquery
if Text.Contains([Description], "applied") then "Applicants"
else if Text.Contains([Description], "admitted") then "Admitted"
else "Enrolled"
```

Sex classification for 2024:

```powerquery
if Text.Contains([Description], "unknown") then "Unknown"
else if Text.Contains([Description], "another gender") then "Another Gender"
else if Text.Contains([Description], "women") then "Women"
else "Men"
```

Check Women before the fallback Men category because `women` contains `men`. The stage fallback is appropriate only after retaining the three expected admissions sections.

Year custom column: `2024` or `2025`, according to the source.

## Data structure

One row represents a fall reporting year and a source sex category.

| Column | Type | Definition |
|---|---|---|
| Sex | Text | Category as classified from source descriptions |
| Year | Whole Number | Fall admissions reporting year |
| Applicants | Whole Number | First-time, first-year applicants |
| Admitted | Whole Number | First-time, first-year admitted students |
| Enrolled | Whole Number | First-time, first-year enrolled students |

The combined table contains seven rows:

| Year | Sex | Applicants | Admitted | Enrolled |
|---|---|---:|---:|---:|
| 2024 | Another Gender | 0 | 0 | 0 |
| 2024 | Men | 6,742 | 2,519 | 643 |
| 2024 | Unknown | 9 | 1 | 0 |
| 2024 | Women | 11,047 | 3,544 | 971 |
| 2025 | Men | 6,308 | 2,571 | 647 |
| 2025 | Unknown | 16 | 4 | 1 |
| 2025 | Women | 10,571 | 3,670 | 991 |

The 2024 Another Gender row contains explicit source zeros. No corresponding 2025 row was imported; its absence should not be represented as a verified zero. Unknown and Another Gender remain separate categories.

## DAX measures

Create each definition below as a separate measure.

### Fall 2025 measures

```DAX
Total Applicants = SUM(Admissions_2025[Applicants])
```

```DAX
Total Admitted = SUM(Admissions_2025[Admitted])
```

```DAX
Total Enrolled = SUM(Admissions_2025[Enrolled])
```

```DAX
Admission Rate = DIVIDE([Total Admitted], [Total Applicants])
```

```DAX
Yield Rate = DIVIDE([Total Enrolled], [Total Admitted])
```

```DAX
Invalid Rows =
COUNTROWS(
    FILTER(
        Admissions_2025,
        Admissions_2025[Applicants] < Admissions_2025[Admitted]
            || Admissions_2025[Admitted] < Admissions_2025[Enrolled]
    )
) + 0
```

```DAX
Missing Count Rows =
COUNTROWS(
    FILTER(
        Admissions_2025,
        ISBLANK(Admissions_2025[Applicants])
            || ISBLANK(Admissions_2025[Admitted])
            || ISBLANK(Admissions_2025[Enrolled])
    )
) + 0
```

### Combined-year measures

```DAX
Applicants All Years = SUM(Admissions_AllYears[Applicants])
```

```DAX
Admitted All Years = SUM(Admissions_AllYears[Admitted])
```

```DAX
Enrolled All Years = SUM(Admissions_AllYears[Enrolled])
```

```DAX
Admission Rate All Years =
DIVIDE([Admitted All Years], [Applicants All Years])
```

```DAX
Yield Rate All Years =
DIVIDE([Enrolled All Years], [Admitted All Years])
```

```DAX
Applicants Change % =
VAR Applicants2024 =
    CALCULATE([Applicants All Years], Admissions_AllYears[Year] = 2024)
VAR Applicants2025 =
    CALCULATE([Applicants All Years], Admissions_AllYears[Year] = 2025)
RETURN
    DIVIDE(Applicants2025 - Applicants2024, Applicants2024)
```

```DAX
Enrollment Change % =
VAR Enrolled2024 =
    CALCULATE([Enrolled All Years], Admissions_AllYears[Year] = 2024)
VAR Enrolled2025 =
    CALCULATE([Enrolled All Years], Admissions_AllYears[Year] = 2025)
RETURN
    DIVIDE(Enrolled2025 - Enrolled2024, Enrolled2024)
```

Format rates and change measures as Percentage with one decimal place. The two change measures use fixed 2024 and 2025 comparisons. Their home table may be `Admissions_2025`, but their calculations reference the combined-year measures and table.

## Validation and limitations

- Annual totals and displayed rates match the prepared source counts.
- Fall 2025 missing-count check: **0 rows**.
- Fall 2025 ordering check: **0 rows** violating Applicants ≥ Admitted ≥ Enrolled.
- These two automated checks cover only the three rows in `Admissions_2025`; they do not validate the full combined table, duplicates, negative values, or individual student records.
- The three-page PDF was reviewed for figures, clipping, and readability. The uploaded PBIX archive passed an integrity check and contained all three expected page names.
- Native PBIX refresh and DAX execution were not independently tested outside the user's Power BI session.
- When reopening on another computer, update PDF paths using **Transform data → Data source settings → Change Source**, then refresh and reconcile annual totals.
- SQL, Microsoft Fabric, Tableau, Workday, Banner, Slate, Salesforce, and Qlik are not implemented in this project.

## Skills demonstrated

- PDF data preparation and organization with Power Query.
- Appending tables, conditional columns, data typing, and pivoting.
- DAX aggregation, safe division, variables, and filtered calculations.
- KPI reporting and interactive visual filtering.
- Year-to-year analysis and scoped consistency checks.
- Metric definitions, source documentation, and plain-language findings.

## Resume description

**William & Mary Admissions Analysis Dashboard | Power BI, Power Query, DAX**

Built a three-page admissions dashboard using Power Query and DAX to compare Fall 2024–2025 performance, identify a 5.1% decline in applications and 1.5% enrollment growth, and communicate findings through visualizations, data quality checks, and plain-language summaries.
