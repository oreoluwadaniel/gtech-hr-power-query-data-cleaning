## Data Quality Audit

Source file: `GTech_Solutions_HR_Data.csv`, 1,530 rows, 25 columns. Row and column counts were confirmed by direct inspection of the raw file (header row plus 1,530 data rows).

Every issue below was verified by one of two methods: direct line-by-line inspection of the raw CSV, or targeted pattern searches across the full file. Where a specific figure comes from the project's own Power Query Column Quality / Column Distribution audit (documented in the project's STAR write-up) rather than an independent recount, that is stated explicitly.

---

### 1. Consistency: Gender recorded in at least a dozen forms

**Problem**
The Gender column held the same two underlying values (male, female) spelled and cased in many different ways, and mixed in honorifics that are not a gender at all.

**Evidence**
Direct inspection of the raw file found these tokens used interchangeably in the Gender column: `Male`, `MALE`, `male`, `M`, `m`, `Female`, `FEMALE`, `female`, `F`, `f`, `Mr`, `Ms`. A full-file pattern search confirms every one of the 1,530 rows falls into this closed set of variants, meaning the column has no missing values, only inconsistent ones.

**Transformation**
The raw text was normalised with `Text.Upper()` and mapped through conditional logic to two clean values, Male and Female. Where a token was genuinely ambiguous (an honorific alone, without a corresponding first-name cue) it was set to null rather than guessed.

**Power Query Skill**
Custom Column with `Text.Upper()` case normalisation, conditional (if/else) value mapping, Replace Values.

**Technical Impact**
Grouping or filtering by Gender in a report would silently split the same population into multiple buckets (`Male`, `MALE`, `M`, `m` each treated as a separate category), understating true counts in every bucket.

**Business Impact**
A gender balance report would show more categories than genders exist, making the workforce look more fragmented than it is and undermining trust in the headcount numbers.

**Validation**
Post-cleaning, Power Query's Column Distribution view for Gender should show exactly two distinct values with zero errors and zero empties (aside from any records intentionally nulled for ambiguity).

---

### 2. Standardization: Department names fragmented across dozens of variants

**Problem**
The same department was written with different casing, added suffixes ("Dept"), and underscores in place of spaces.

**Evidence**
Direct inspection of a sample spanning the start, middle, and end of the file turned up more than 25 distinct department strings for what is clearly a fixed set of business units, including: `LEGAL`, `Legal`, `legal`, `SUPPLY CHAIN`, `Supply Chain Dept`, `Supply_Chain`, `supply chain`, `CUSTOMER SERVICE`, `customer service`, `Customer Service Dept`, `Operations`, `Operations Dept`, `Finance`, `Finance Dept`, `finance`, `Marketing`, `Marketing Dept`, `marketing`, `EXECUTIVE`, `Executive`, `executive`, `Executive Dept`, `SALES`, `Sales`, `sales`, `Sales Dept`, `Human Resources`, `HUMAN RESOURCES`, `human resources`, `Human_Resources`, `Human Resources Dept`, `Technology`, `TECHNOLOGY`, `technology`. The project's own Power Query audit recorded 43 variants across the full file resolving to 11 real departments.

**Transformation**
Case was normalised, underscores replaced with spaces, and suffixes like "Dept" stripped, then each cleaned string was mapped to one of 11 canonical department names via conditional logic.

**Power Query Skill**
Text.Upper()/Text.Clean(), Text.Replace() for underscores, Custom Column mapping logic, Replace Values.

**Technical Impact**
A GROUP BY or pivot on raw Department would produce dozens of rows instead of 11, and any join to a separate departments lookup table would fail to match on the unnormalised spellings.

**Business Impact**
An HR director asking how many people work in Supply Chain would get a fragmented answer split across several near-identical labels rather than a single trustworthy number.

**Validation**
Post-cleaning Column Distribution for Department should show exactly 11 (or the number of standardised business units the company actually operates) distinct values.

---

### 3. Standardization: Employment_Type variants for a small set of real categories

**Problem**
Employment type had abbreviations, hyphenation, and case differences describing the same handful of real categories.

**Evidence**
Directly observed values include `Full Time`, `FT`, `FULL-TIME`, `Full-Time`, `full-time`, `CONTRACT`, `Contract`, `CONTR`, `Contractor`, `PT`, `Part-Time`, `part-time`, `PART-TIME`, `Part Time`.

**Transformation**
Each variant was normalised and mapped to one of the underlying categories (Full-Time, Part-Time, Contract, Contractor).

**Power Query Skill**
Custom Column conditional mapping, Text.Upper() normalisation, Replace Values.

**Technical Impact**
Headcount-by-employment-type reporting would be split across more than a dozen labels rather than 3 to 4 real categories.

**Business Impact**
Workforce composition analysis (how many contractors versus full-time staff) would be understated or misread because the same category is being counted as if it were several.

**Validation**
Post-cleaning distinct value count for Employment_Type should match the number of employment categories the company actually uses.

---

### 4. Consistency: Performance_Rating_2024 mixed three different rating scales

**Problem**
The same performance concept was recorded as a number (1 to 5), a letter grade (A to D), and a text descriptor (Excellent, Exceeds Expectations, Good, Meets Expectations, Average, Below Average, Needs Improvement) within the same column, depending on which manager or period entered the data. Placeholder text (`N/A`, `NIL`, `TBD`, `#N/A`) was also mixed into the same field.

**Evidence**
All three scales, plus the placeholder tokens, were found directly in the raw file within a single column across the sampled rows.

**Transformation**
Each of the three scales was mapped onto a single five-point descriptor scale (the richest of the three original scales), with placeholder tokens converted to null.

**Power Query Skill**
Custom Column conditional mapping across mixed data types, Replace Values for pseudo-nulls.

**Technical Impact**
Any attempt to average or rank performance scores directly would fail or produce nonsense, since letters, numbers, and text cannot be aggregated together.

**Business Impact**
Performance distribution reporting, one of the direct downstream uses of this data, would have been impossible without resolving this, since the three scales cannot be compared against each other as recorded.

**Validation**
Post-cleaning, Performance_Rating_2024 should contain only the five agreed descriptor values plus nulls, with zero numeric or letter-grade entries remaining.

---

### 5. Validity / Structural Quality: Monthly_Gross_Salary_NGN stored as unparseable text

**Problem**
Salary was entered as free text with currency symbols, prefixes, suffixes, thousands separators, and a "K" shorthand, rather than as a number.

**Evidence**
Directly observed formats in the raw file include `N609,000`, `334000`, `124.0K`, `269000`, `N377,000`, `#219,000`, `#176,000`, `237000`, `NGN 631000`, `240000 NGN`, `N439,000`, `355,000`, `369000 NGN`, and a naira-sign encoding corruption rendering as `??124,000` / `??339,000` / `??2,006,000` (a mojibake artifact of the naira symbol being saved in an encoding Excel/CSV did not preserve). A full-file pattern search found 778 rows containing one of the non-numeric salary notations (N-prefix, #-prefix, NGN prefix/suffix, or K-suffix), out of 1,530 rows, meaning roughly half the file used a non-plain-number format.

**Transformation**
A multi-step M formula stripped currency symbols and letters, detected the "K" suffix and multiplied the numeric portion by 1,000, removed thousands separators, and converted the result to a Whole Number data type.

**Power Query Skill**
Custom Column with `Text.Remove()` / `Text.Replace()` for symbol stripping, conditional logic for the K-suffix multiplier, `Number.From()` type conversion.

**Technical Impact**
As text, the column cannot be summed, averaged, or used in any numeric visual. Any attempt to chart salary would either error out or silently treat every value as a non-numeric category.

**Business Impact**
Salary benchmarking, one of the explicit intended uses of this dataset, is not possible at all while this column remains text. Payroll totals, department averages, and pay-equity comparisons all depend on this fix.

**Validation**
Post-cleaning, the column's data type should read as Whole Number in Power Query, and Column Quality should show 0% errors after conversion, meaning every value survived the parsing logic.

---

### 6. Validity / Structural Quality: Date_of_Birth and Date_of_Hire mixed at least five distinct formats

**Problem**
Dates were entered in day/month/year, month/day/year, and dotted, dashed, and month-name formats within the same column, with no way to distinguish which convention applies to an ambiguous value like `6/3/2021` on sight.

**Evidence**
Directly observed formats in the raw file include `6/6/1991` (M/D/YYYY), `27-Apr-80` (D-Mon-YY), `22/02/1991` (D/M/YYYY), `21.08.1990` (D.M.YYYY), `11/13/1990` (unambiguous M/D/YYYY, since 13 cannot be a month), `06.01.1994` (D.M.YYYY), and `18/02/2019` (D/M/YYYY). A screenshot of the Power Query editor captured mid-transformation shows a new `Hire_Date` column sitting alongside the untouched raw `Date_of_Hire` text column. Comparing the two directly confirms the parsing logic: raw `3/6/2021` becomes `6/3/2021` in the new column (3 June, read as day/month first), and raw `18/02/2019` becomes `2/18/2019` (18 February, again read as day/month first). This matches a day-first (en-GB) parse attempted before a month-first (en-US) fallback.

**Transformation**
A custom M formula attempted `Date.FromText(raw, [Culture='en-GB'])` first, falling back to `[Culture='en-US']` if that failed, and returning null if neither parse succeeded. The result was then set to the Date data type.

**Power Query Skill**
Custom Column with `try ... otherwise try ... otherwise null` error-handling pattern, `Date.FromText()` with explicit culture arguments, data type conversion to Date.

**Technical Impact**
Text dates cannot be used in date-based calculations. Tenure, age, and any time-series or "days since" calculation is impossible until the column is a true Date type, and an unhandled mixed-format column would misparse a meaningful share of rows if converted with a single, non-culture-aware rule.

**Business Impact**
Tenure analytics and age-based demographic analysis, both explicit intended uses of this dataset, require working date arithmetic. Getting the day/month order wrong for ambiguous values (which this two-pass logic is specifically designed to avoid) would silently produce wrong tenure and age figures rather than visible errors.

**Validation**
Post-cleaning, both date columns should show 0% errors in Column Quality and a Date data type, with spot checks against the original raw strings (as demonstrated above) confirming the day/month order was resolved correctly rather than swapped.

---

### 7. Completeness / Validity: Pseudo-null placeholder text used instead of blank fields

**Problem**
Several columns used the words `TBD`, `NIL`, `Unknown`, `Not Available`, `N/A`, `#N/A`, `NULL`, and `n/a` in place of an actual blank when a value was not known, rather than leaving the field empty.

**Evidence**
A full-file pattern search found 715 occurrences of these placeholder tokens. Direct inspection confirms they appear across at least ten columns, including Education_Level, Years_of_Experience, Annual_Leave_Balance_Days, Performance_Rating_2024, Employment_Status, Bank_Name, Manager_Employee_ID, Office_Location, National_ID_Number, and Tax_Identification_Number.

**Transformation**
Each placeholder token was converted to a true null using Replace Values, rather than being treated as a real data point.

**Power Query Skill**
Replace Values, applied consistently across the affected columns.

**Technical Impact**
Left as text, these placeholders are counted in group-bys and shown as a category in charts and filters, meaning "TBD" or "Unknown" can appear as if it were a legitimate answer alongside real values.

**Business Impact**
A completeness report on Education_Level, for example, would understate how much data is genuinely missing, because "Unknown" and a true blank are functionally the same gap but were being tracked differently.

**Validation**
Post-cleaning, Column Quality for the affected columns should report the placeholder-driven percentage as Empty rather than Valid, giving an accurate read on true completeness.

---

### 8. Uniqueness: Employee_ID had duplicate entries

**Problem**
The primary employee identifier was not unique across the file.

**Evidence**
This figure is sourced from the project's own Power Query audit, documented in the project's STAR write-up: 30 duplicate Employee_ID rows were identified, and removing them reduced the row count from 1,530 to 1,500. The starting figure of 1,530 rows was independently confirmed by direct inspection of the raw CSV.

**Transformation**
Power Query's Remove Duplicates was applied to the Employee_ID column, keeping the first occurrence of each ID.

**Power Query Skill**
Remove Duplicates on a designated key column.

**Technical Impact**
A duplicated ID breaks its use as a join key against any other employee-related table (payroll, attendance, and so on), and inflates any headcount figure computed by counting rows.

**Business Impact**
Reported headcount would be overstated by 30 employees (2% of the workforce) until the duplicates were removed, and any join built on Employee_ID before this fix would silently produce fan-out duplication in downstream reports.

**Validation**
Post-cleaning, a count of distinct Employee_ID values should equal the total row count (1,500), confirming true one-row-per-employee uniqueness.

---

### 9. Structural Quality: Phone_Number values corrupted by scientific notation

**Problem**
A share of phone numbers were stored as text but read by spreadsheet software as long numbers, which converted them into scientific notation, rounding away digits in the process.

**Evidence**
Directly observed raw values include `2.34E+12` and `2.34E+13` appearing repeatedly across the Phone_Number column, alongside properly preserved formats like `0070-228-8135`, `+234 085 549 1844`, and plain digit strings like `846129050`.

**Transformation**
This issue is visible in the raw file evidence. The cleaned dataset provided was not independently inspected for the specific handling of this column, since the sandbox environment used to open the .xlsx file was unavailable during this review (see the verification note in the Cleaning Process document).

**Power Query Skill**
Would require Text.From() with an explicit precision-preserving conversion, or re-importing the source column as Text rather than allowing automatic type detection.

**Technical Impact**
Once a phone number has been rounded into scientific notation, the original digits cannot be recovered. This is data loss, not a formatting inconvenience.

**Business Impact**
Any contact list, SMS campaign, or verification process built on this column would fail silently for every affected employee, since the stored value no longer matches their real phone number.

**Validation**
A working validation check is a regular-expression or length check confirming every Phone_Number value is a plausible Nigerian phone number pattern, with zero values matching an `E+` scientific notation pattern.

---

### 10. Standardization: Office_Location casing and naming inconsistency

**Problem**
The same office was written in different cases and, in at least one case, an entirely different phrasing.

**Evidence**
Directly observed values for what is evidently the same Lagos office include `lag hq`, `LAGOS HQ`, `Lagos HQ`, `lagos hq`, and `Lagos - Head Office`.

**Transformation**
Not independently confirmed against the cleaned file (see verification note in the Cleaning Process document); flagged here as a raw-data finding for completeness of the audit.

**Power Query Skill**
Would require the same case-normalisation and conditional-mapping pattern used for Department and Employment_Type.

**Technical Impact**
Location-based headcount and office-capacity reporting would fragment the same office into multiple rows.

**Business Impact**
Facilities or regional headcount planning would misread how many staff are actually based at each office.

---

### 11. Structural Quality: Leading and trailing whitespace in text fields

**Problem**
A number of First_Name and Last_Name values carry a leading or trailing space, likely from manual data entry.

**Evidence**
Directly observed in the raw file: ` Titi`, ` Femi`, `Ifeanyi ` (trailing), ` Obiageli`, `Adewale ` (trailing), `Biodun ` (trailing).

**Power Query Skill**
Trim (`Text.Trim()`), applied across text columns.

**Technical Impact**
Untrimmed whitespace breaks exact-match filtering, grouping, and joins on name fields, since `"Femi"` and `" Femi"` are not equal as strings.

**Business Impact**
Searching or filtering an employee list by name would silently miss records where the name carries invisible whitespace.

---

### 12. Analytical note: Date_of_Exit sparsity is expected, not a defect

**Observation**
Power Query's Column Quality view (captured in project screenshots) shows Date_of_Exit at roughly 13% valid and 87% empty. This is not a data quality problem to fix; the majority of employees are still active and correctly have no exit date. It is flagged here to distinguish a legitimately sparse column from the true completeness issues documented above (item 7).

---

## Summary by Dimension

| Dimension | Issues Found |
|---|---|
| Completeness | Pseudo-null placeholders masking true missing values across 8 to 10+ columns |
| Accuracy | Phone numbers corrupted by scientific-notation rounding |
| Consistency | Gender, Performance_Rating_2024 scale mixing |
| Validity | Salary stored as text, dates stored as unparsed mixed-format text |
| Uniqueness | 30 duplicate Employee_ID rows |
| Standardization | Department, Employment_Type, Office_Location variant spellings and casing |
| Structural Quality | Leading/trailing whitespace, naira-symbol encoding corruption, comma-formatted numeric ID fields |
| Analytical Readiness | None of the above could be reliably grouped, joined, summed, or date-calculated in the raw state |
