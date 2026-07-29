## Data Cleaning Process

This document reconstructs the Power Query cleaning pipeline applied to `GTech_Solutions_HR_Data.csv`, based on direct inspection of the raw file, the project's own STAR-format documentation of the work, and Power Query interface screenshots captured during the project.

### Verification scope and method

Two sources of evidence were used, and they are not treated as equally strong:

1. **Directly verified from the raw file.** Row count (1,530), column count (25), the specific messy values quoted throughout this document, and the pseudo-null token counts were confirmed by reading the raw CSV and running targeted pattern searches against it.
2. **Sourced from the project's own documentation.** Specific counts describing the *cleaned* output (duplicate rows removed, exact variant counts per column, final standardised category lists) come from the project's STAR write-up, written by the person who performed the cleaning. During this review, the cleaned `.xlsx` file could not be opened for independent row-by-row verification because the sandbox environment required to read a binary spreadsheet file was unavailable at the time of writing. Where a claim rests only on source #2, it is marked as such rather than presented as independently re-derived.

This distinction matters because the instructions for this project are explicit that nothing should be presented as verified unless it actually was.

### Pipeline sequence

The cleaning work followed this sequence, reconstructed from the Applied Steps described in the STAR document and cross-checked against the raw data wherever the raw file itself provides evidence:

**1. Audit and load**
Power Query's Column Quality and Column Distribution views were used to profile every column before any changes were made. This is visible directly in the project's screenshots: one screenshot shows Employee_ID, First_Name, Last_Name, Gender, Date_of_Birth, and Email_Address all reporting close to 100% "Valid" under Column Quality, despite Gender clearly containing a dozen inconsistent spellings in the values themselves (`Male`, `FEMALE`, `Ms`, `male`, `Mr`, `female`...). This is an important and correct observation embedded in the audit approach: Power Query's Column Quality metric measures null/error rate, not semantic consistency, so a column can be 100% "valid" and still need standardisation work. A second screenshot shows Employment_Status, Date_of_Hire, and Date_of_Exit profiled the same way, with Date_of_Exit reporting roughly 13% valid and 87% empty, which is expected given most employees are still active.

**2. Removed duplicate Employee IDs**
Power Query's Remove Duplicates function was applied to the Employee_ID column. Per the project's documentation, this removed 30 duplicate rows, taking the dataset from 1,530 to 1,500 rows. This was prioritised early because a non-unique primary key blocks reliable joins and headcount counts for every later step.

**3. Parsed inconsistent date formats**
Both Date_of_Birth and Date_of_Hire mixed several date formats in the same column. A custom M formula was used: `try Date.FromText(raw, [Culture='en-GB']) otherwise try Date.FromText(raw, [Culture='en-US']) otherwise null`, attempting a day-first parse before falling back to a month-first parse, and returning null only if neither succeeded. Direct comparison of a Power Query screenshot (showing a parsed `Hire_Date` column next to the still-raw `Date_of_Hire` column) confirms this logic was actually applied: raw `3/6/2021` became `6/3/2021` (parsed as 3 June, then displayed month-first), and raw `18/02/2019` became `2/18/2019` (parsed as 18 February, then displayed month-first). Both results are consistent with a day-first parse taking priority, matching the documented formula. The resulting column was set to the Date data type.

**4. Cleaned and converted the salary column**
Monthly_Gross_Salary_NGN mixed currency symbols (`N`, `#`, `NGN`, and a corrupted naira-sign encoding rendering as `??`), thousands separators, a `K` shorthand for thousands, and trailing currency text, all within the same column, confirmed directly in the raw file. A multi-step M formula stripped symbols and letters, detected the `K` suffix and multiplied by 1,000 where present, and converted the cleaned string to a Whole Number.

**5. Standardised categorical columns**
Gender, Department, Employment_Type, and Performance_Rating_2024 were each normalised with `Text.Upper()` for case, then mapped through conditional (if/else) logic to a single clean value per underlying category. Ambiguous values were set to null instead of guessed. This is directly supported by raw-file evidence for all four columns (see the Data Quality Audit for the specific variant lists observed).

**6. Replaced pseudo-nulls with true nulls**
The placeholder tokens `TBD`, `NIL`, `Unknown`, `Not Available`, `N/A`, `#N/A`, `NULL`, and `n/a` were converted to proper null values using Replace Values. A full-file pattern search independently confirmed 715 occurrences of these tokens spread across at least ten columns in the raw data, supporting that this was a real, widespread issue rather than an isolated one.

**7. Added calculated helper columns**
Four columns were added once the base data was clean: `Age_Years` (from Date_of_Birth), `Tenure_Years` (from Date_of_Hire to either Date_of_Exit or the current date), `Full_Name` (First_Name and Last_Name merged with a space), and `Is_Active` (a flag derived from Employment_Status). These are documented in the project's STAR write-up. They could not be independently confirmed in the cleaned output during this review for the reason noted above (sandbox unavailability), so they are presented here as the author's documented additions rather than independently re-verified.

### Additional raw-data findings not covered in the original STAR summary

Direct inspection of the raw file surfaced a few structural issues beyond the ones summarised in the project's own write-up. These are documented in full in the Data Quality Audit and are worth naming here because they represent additional, independently-found evidence of data quality problems in the source file: phone numbers corrupted into scientific notation by automatic type detection, inconsistent casing in Office_Location, leading/trailing whitespace in name fields, and comma-formatted (rather than plain-digit) Bank_Account_Number values in some rows.

### Tools used

Power BI Desktop housed the Power Query Editor used for the entire cleaning process. Every transformation was written either through the Power Query UI or directly in the M formula language (used specifically for the date-parsing and salary-parsing logic, which required conditional handling beyond what the built-in UI transformations support on their own). Each step is recorded automatically in the Applied Steps panel, which is what makes the pipeline auditable and re-runnable against a refreshed source file.
