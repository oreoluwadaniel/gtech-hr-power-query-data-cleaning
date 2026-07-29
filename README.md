# GTech Solutions HR Data Cleaning & Quality Transformation

A Power Query data cleaning project built on a real-world-style HR file from a fictional Nigerian technology company, GTech Solutions. The point of this project is not HR analytics. It is showing, with evidence, what it takes to turn a genuinely messy operational dataset into something an analyst can trust.

## Overview

GTech Solutions is a fictional technology company operating across Lagos, Abuja, Port Harcourt, and Kano. Its HR team kept a single employee master file: 1,530 rows and 25 columns covering personal details, employment status, salary, performance ratings, and office location. The file had accumulated years of manual entry from different people with no shared data-entry standard, and it showed. This project audits that file, documents every quality problem found, cleans it in Power Query, and explains why each fix matters.

## Business Problem

Before any HR dashboard or workforce report can be trusted, the data behind it has to be reliable. In this file it was not. The same department appeared under more than two dozen different spellings. Gender was recorded a dozen different ways. Salary was stored as free text mixing three different currency notations and a shorthand for thousands. Dates were entered in at least five different formats within the same column. Thirty employees had duplicate IDs. And several columns used words like "TBD," "NIL," and "Unknown" in place of a blank field, which is not the same thing to a reporting tool.

None of this is unusual. It is what happens in almost every organisation that collects data across different teams, systems, and years without enforcing a standard. The cost is that a question as simple as "how many people work in Supply Chain" does not have a single reliable answer until the data is cleaned.

## Project Goal

Take the raw HR file, identify every quality issue systematically, and produce a clean, structured, analysis-ready dataset using Power Query, with the transformation logic documented well enough that someone else could review, maintain, or re-run it.

## Dataset

Source file: `GTech_Solutions_HR_Data.csv`. 1,530 employee records across 25 columns, spanning personal details, employment terms, compensation, performance, and location fields. Row and column counts were confirmed by direct inspection of the raw file.

## Data Quality Assessment

The raw file was profiled column by column using Power Query's Column Quality and Column Distribution views before any cleaning began. A key finding from that audit is worth stating plainly: several columns showed close to 100% "Valid" under Column Quality while still being unusable for analysis, because Column Quality measures whether a cell is null or an error, not whether the value is written consistently. Gender, for example, was 100% populated but recorded in a dozen different spellings and cases. That distinction, between completeness and consistency, shaped how this dataset needed to be approached.

The full breakdown of every issue found, with raw evidence, the specific transformation applied, and why it mattered, is in [`documentation/data-quality-audit.md`](documentation/data-quality-audit.md).

## Cleaning Approach

The work followed an audit-first sequence: profile everything before changing anything, fix the highest-risk structural issues first (duplicate IDs, since they break every downstream count and join), then work through type conversions (dates, salary), then categorical standardisation, then null handling, then add calculated fields once the base data could support them. The full step-by-step reconstruction, including specific before/after examples pulled directly from the raw file and project screenshots, is in [`documentation/data-cleaning-process.md`](documentation/data-cleaning-process.md).

## Power Query Transformation Process

Every transformation was applied inside Power Query, the transformation layer built into Power BI Desktop. Simple fixes used built-in UI actions (Remove Duplicates, Replace Values, Trim, data type changes). The date and salary columns needed more than the UI could offer, so custom M formulas were written for those: a two-pass date parser that tries a day-first interpretation before falling back to month-first, and a multi-step salary parser that strips currency symbols, detects a "K" shorthand for thousands, and converts the result to a number. Every step is recorded automatically in the Applied Steps panel, which makes the entire pipeline auditable and means it re-runs automatically the next time the source file refreshes.

## Key Data Quality Issues

| Issue | Column(s) Affected | Verified How |
|---|---|---|
| Inconsistent casing and spelling | Gender, Department, Employment_Type, Office_Location | Directly observed in raw file |
| Mixed date formats within one column | Date_of_Birth, Date_of_Hire | Directly observed in raw file; parsing logic confirmed via screenshot comparison |
| Salary stored as unparseable text with 8 formats | Monthly_Gross_Salary_NGN | Directly observed in raw file; format-pattern search across full file |
| Three mixed rating scales in one column | Performance_Rating_2024 | Directly observed in raw file |
| Placeholder text used instead of true nulls | At least 10 columns | Full-file pattern search, 715 occurrences confirmed |
| Duplicate primary key | Employee_ID | Documented in the project's own cleaning record; starting row count of 1,530 independently confirmed |
| Scientific-notation corruption | Phone_Number | Directly observed in raw file |

## Before vs After

| Data Quality Issue | Before (Raw) | Cleaning Action | After | Why It Matters |
|---|---|---|---|---|
| Salary format | `N609,000`, `124.0K`, `NGN 631000`, `240000 NGN` (text, 8 formats observed) | Stripped symbols, resolved K-shorthand, converted to number | Whole Number | Enables salary benchmarking, payroll totals, and averages, none of which are possible on a text column |
| Date format | Raw `3/6/2021` and `18/02/2019` in the same column as `6/6/1991` and `21.08.1990` | Two-pass culture-aware date parsing (day-first, then month-first, then null) | `6/3/2021` and `2/18/2019` (correctly parsed as 3 June and 18 February) | Enables tenure and age calculations without misreading day/month order |
| Gender | `Male`, `MALE`, `male`, `M`, `m`, `Mr`, `Female`, `FEMALE`, `female`, `F`, `f`, `Ms` | Case normalisation and conditional mapping | Male / Female | Accurate headcount and demographic reporting without fragmented categories |
| Department | `LEGAL`, `Legal`, `legal`, `Supply Chain Dept`, `Supply_Chain`, `supply chain`, and 25+ more variants observed directly | Case normalisation, underscore/suffix cleanup, mapped to canonical list | 11 standardised departments | Reliable departmental headcount and reporting |
| Pseudo-nulls | `TBD`, `NIL`, `Unknown`, `Not Available`, `N/A`, `#N/A`, `NULL`, `n/a` used across 10+ columns | Replace Values to true null | Proper null | Accurate completeness reporting; placeholders no longer counted as real data points |
| Duplicate records | 1,530 rows, 30 duplicate Employee_ID values (per project documentation) | Remove Duplicates on Employee_ID | 1,500 rows | Employee_ID can be trusted as a join key; headcount is no longer inflated |

## Validation

Two checks confirm the cleaning held: Column Quality on the cleaned dataset should show 0% errors on the converted date and salary columns (meaning every value survived the parsing logic without falling through to null unintentionally), and Column Distribution on each standardised categorical column should show only the agreed canonical values, no leftover variants. A distinct-count check on Employee_ID should equal the total row count, confirming true uniqueness.

## Data Quality Findings

These are the findings that came out of the audit itself, separate from anything about HR analytics: placeholder text was being used interchangeably with true nulls across a wide share of the file, which would have understated real completeness gaps if left as-is. Categorical fields with a small number of real values were fragmented into dozens of raw variants, which would have split single populations into many reporting buckets. A duplicated primary key existed in roughly 2% of records, which would have inflated any row-based headcount. Two of the most analytically important columns, salary and hire date, were stored in types that made them unusable for calculation until converted.

## Business Value

None of these are cosmetic fixes. A salary column stored as text cannot be summed. A department field with 43 spellings cannot be grouped into 11 real answers. A duplicated employee ID corrupts any join or headcount built on top of it. Each fix in this project removes a specific, identifiable way the raw file would have produced a wrong number, a broken calculation, or a misleading report if it had been used as-is.

## Recommendations

Standardise data entry for categorical fields (Gender, Department, Employment_Type, Office_Location) with a controlled list or dropdown at the point of entry, rather than free text, to prevent the variant sprawl seen in this file from recurring. Enforce a single date format at entry or import. Require Employee_ID uniqueness at the point of creation, ideally system-generated rather than manually entered. Replace the practice of typing "TBD" or "Unknown" into a field with actually leaving it blank, so completeness can be measured accurately. Where the source system cannot enforce these rules directly, keep this Power Query pipeline in place as a standing data-quality gate that runs on every refresh rather than a one-time cleanup.

## Skills Demonstrated

**Data Cleaning:** duplicate removal, null handling and pseudo-null detection, text trimming, category standardisation, multi-format parsing.

**Power Query:** Column Quality and Column Distribution profiling, Remove Duplicates, Replace Values, Custom Columns, custom M formulas for date and currency parsing, data type conversion, Applied Steps documentation.

**Data Quality:** completeness vs consistency distinction, validity checks, uniqueness enforcement, structural quality review (encoding corruption, scientific-notation data loss).

**Business Analysis:** connecting each raw-data defect to a specific downstream reporting or calculation failure it would cause if left unresolved.

**Documentation:** reconstructing and recording a repeatable transformation process with verifiable evidence, including being explicit about which claims were independently confirmed versus sourced from prior project notes.

## Tools Used

Power BI Desktop, Power Query Editor, M formula language, Column Quality and Column Distribution profiling views.

## Repository Structure

```
gtech-hr-power-query-data-cleaning/
├── README.md
├── data/
│   ├── raw/                 GTech_Solutions_HR_Data.csv (source file, unmodified)
│   └── cleaned/              cleaned dataset
├── screenshots/               Power Query profiling views referenced in the documentation
├── documentation/
│   ├── data-cleaning-process.md
│   ├── data-quality-audit.md
│   └── data-dictionary.md
└── LICENSE
```

## How to Use the Project

Open `data/raw/GTech_Solutions_HR_Data.csv` to see the source file in its original state. Read `documentation/data-quality-audit.md` for the full issue-by-issue breakdown with evidence. Read `documentation/data-cleaning-process.md` for the reconstructed step-by-step pipeline. The cleaned output is in `data/cleaned/`.

## Conclusion

Reliable analysis starts with reliable data. This project shows the full path from a messy operational file to a documented, defensible cleaning process: auditing before touching anything, fixing the highest-risk issues first, using Power Query's M language where the built-in UI transformations were not enough, and being explicit about what was verified directly versus what is sourced from prior documentation. That is the standard this project was held to throughout, and it is the standard any HR, finance, or operational dataset needs before it is trusted for reporting.
