## Data Dictionary: Cleaned HR Dataset

This dictionary covers the 25 source columns confirmed in the raw file, plus the 4 calculated helper columns documented in the project's cleaning process. Data types listed for the source columns reflect the type each column should hold after cleaning, based on the transformations documented in `data-cleaning-process.md`. Business meaning is only stated where it can be reasonably inferred from the column name and its role in the raw data; anything uncertain is marked as such rather than guessed.

| Column | Data Type (Cleaned) | Description | Business Meaning |
|---|---|---|---|
| Employee_ID | Text (unique key) | Unique identifier for each employee, format `EMP-#####`. | Primary key for joining to other HR or payroll tables. Must be unique after duplicate removal. |
| First_Name | Text | Employee's first name. | Identity field. |
| Last_Name | Text | Employee's surname. | Identity field. |
| Gender | Text (Male / Female) | Standardised to two values. | Used for demographic and diversity reporting. |
| Date_of_Birth | Date | Employee's date of birth. | Basis for calculated Age_Years. |
| Email_Address | Text | Work email address. | Contact and identity field. Domain conventions vary across the file (`gtechsolutions.com.ng`, `g-tech.com.ng`, `gtechsol.ng`); not confirmed whether this reflects legitimate sub-brands or an unresolved inconsistency. |
| Phone_Number | Text | Employee's phone number. | Contact field. Flagged in the audit for scientific-notation corruption in a share of raw rows; not confirmed whether the cleaned file recovered or lost these values. |
| State_of_Origin | Text | Nigerian state associated with the employee. | Demographic/reporting field. |
| Department | Text (standardised category) | The business unit the employee belongs to. | Standardised to the company's real department list; used for headcount and departmental reporting. |
| Job_Title | Text | The employee's job title. | Used for role-level analysis and org structure. |
| Employment_Type | Text (standardised category) | Full-Time, Part-Time, Contract, or Contractor. | Used for workforce composition reporting. |
| Employment_Status | Text (standardised category) | Active, Inactive, Terminated, Suspended, or On Leave. | Basis for the calculated Is_Active flag and attrition tracking. |
| Date_of_Hire | Date | Date the employee joined the company. | Basis for calculated Tenure_Years. |
| Date_of_Exit | Date (nullable) | Date the employee left the company, where applicable. | Legitimately empty for active employees; used for attrition timing analysis. |
| Monthly_Gross_Salary_NGN | Whole Number | Monthly gross salary in Nigerian Naira. | Basis for salary benchmarking and payroll totals. |
| Bank_Name | Text | Employee's bank for payroll purposes. | Payroll/finance reference field. |
| Bank_Account_Number | Text | Employee's bank account number. | Should remain text (not numeric) to avoid losing leading digits or gaining unintended thousands-separator formatting, an issue observed in the raw file. |
| Education_Level | Text | Highest qualification (for example B.Sc, M.Sc, HND, Ph.D). | Demographic/reporting field. |
| Years_of_Experience | Whole Number | Years of professional experience. | Raw file mixed numeric and text formats (`10+ years`, `5 yrs`, `1 year`); cleaned type assumes this was resolved to a plain number, not independently confirmed against the cleaned file. |
| Annual_Leave_Balance_Days | Whole Number | Remaining annual leave balance, in days. | Operational HR field. |
| Performance_Rating_2024 | Text (standardised 5-point descriptor scale) | 2024 performance rating. | Standardised to one descriptor scale; used for performance distribution reporting. |
| Manager_Employee_ID | Text (nullable) | Employee_ID of the employee's direct manager. | Legitimately null for top-level executives; should match the Employee_ID key format. |
| Office_Location | Text | The employee's assigned office. | Flagged in the audit for casing/naming inconsistency in the raw file; not independently confirmed as standardised in the cleaned output. |
| National_ID_Number | Text | National Identification Number. | Sensitive personal identifier; should remain text to preserve leading digits. |
| Tax_Identification_Number | Text | Tax Identification Number. | Sensitive personal identifier; should remain text to preserve exact formatting. |

### Calculated helper columns (documented, added after base cleaning)

| Column | Data Type | Description | Business Meaning |
|---|---|---|---|
| Age_Years | Whole Number | Calculated from Date_of_Birth to the current date. | Enables age-based demographic analysis without a separate DAX measure. |
| Tenure_Years | Whole Number | Calculated from Date_of_Hire to Date_of_Exit (if present) or the current date. | Enables tenure analytics without a separate DAX measure. |
| Full_Name | Text | First_Name and Last_Name merged with a single space. | Convenience field for reporting and search. |
| Is_Active | Boolean / Flag | Derived from Employment_Status. | Simplifies attrition and headcount filtering to a single true/false field. |

**Note on verification:** The four calculated columns and the exact post-cleaning values for Years_of_Experience, Office_Location, and Phone_Number are documented per the project's own STAR write-up but were not independently re-opened and checked cell-by-cell in the cleaned `.xlsx` file during this review, because the sandbox environment needed to read a binary spreadsheet file was unavailable at review time. This is stated here rather than presented as independently confirmed.
