# Duplicate Finder — User Guide

## How to Use the Tool

**1. Download the tool**  
- Click the dropdown arrow next to the green **Code** button.  
- Select **Download ZIP**.  
- Extract the ZIP file to access `duplicates_finder.exe`.
<img width="946" height="428" alt="Screenshot 2026-06-25 081210" src="https://github.com/user-attachments/assets/5338334a-b958-4553-9c3e-4d7c1f294047" />

---

**2. Export your data from Salesforce**  
- Run a Salesforce report using the required columns listed below.  
- Export the file as `.csv` or `.xlsx`.

---

**3. Launch the application**  
- Double-click `duplicates_finder.exe`.

If Windows Defender blocks the app:
1. Right-click the file and select **Properties**.  
2. In the **General** tab, check **Unblock**.  
3. Click **OK**, then reopen the app.

---

**4. Select the record type**  
- Choose **Contact** or **Account** from the dropdown.  
- The tool may auto-detect this based on your file.

---

**5. Upload your file**  
- Click **Browse** and select your exported file.

---

**6. Run the tool**  
- Click **Run Script**.  
- A progress indicator will display while processing.

---

**7. Review the output**  
- The Excel report is saved in the same folder as your input file:
  - `contact_duplicate_report.xlsx`
  - `account_duplicate_report.xlsx`

- Each row represents a suspected duplicate pair.  
- Use **Link1** and **Link2** to open records directly in Salesforce for review and merging.

---

## Before You Export — Required Columns

### Accounts

| Column                    |
| :------------------------ |
| Record Link               |
| Account Name              |
| Account Type              |
| Account Record Type       |
| Parent Account            |
| Website                   |
| Phone                     |
| Fax                       |
| Billing Street            |
| Billing City              |
| Billing State/Province    |
| Billing Zip/Postal Code   |
| Billing Country           |
| Secondary Street          |
| Secondary City            |
| Secondary State/Province  |
| Secondary Zip/Postal Code |
| Secondary Country         |
| Account Id 18 Digits      |
| Parent Account ID         |

### Contacts

| Column                  | Notes          |
| :---------------------- | :------------- |
| Record Link             |                |
| First Name              |                |
| Last Name               |                |
| Account Name            |                |
| Title                   |                |
| Role Code               |                |
| Mailing Street          |                |
| Mailing State/Province  |                |
| Mailing City            |                |
| Mailing Zip/Postal Code |                |
| Mailing Country         |                |
| Phone                   |                |
| Alternate Phone         |                |
| Email                   |                |
| Mobile                  |                |
| Alternate Email         |                |
| Confirmed Duplicate     | See note below |

> **Confirmed Duplicate** — On the first cleanup pass, we do **not** filter for records where *Confirmed Duplicate = false*. This ensures all potential duplicates are captured.
>
> After the initial cleanup, some records may legitimately exist more than once (e.g., a contact associated with multiple companies).
>
> For the second pass, filter where *Confirmed Duplicate = false* to avoid reprocessing validated duplicates.
>
> Additionally, consider excluding inactive contacts (e.g., former employees) during the second pass to improve accuracy.

---

## Output Columns

### Contact Report

| Column             | Description                                                                    |
| :----------------- | :----------------------------------------------------------------------------- |
| Record1 / Record2  | Row numbers from your input file                                               |
| Score              | Confidence score — higher means more certain                                   |
| Link1 / Link2      | Clickable links to the Salesforce records                                      |
| Full Name 1 / 2    | Full names as they appear in Salesforce                                        |
| Title 1 / 2        | Job titles                                                                     |
| Account Name 1 / 2 | Company names                                                                  |
| Email 1 / 2        | Primary email addresses                                                        |
| Mobile 1 / 2       | Mobile numbers                                                                 |
| Notes              | Indicates if records share an account or reflect a company change              |

### Account Report

| Column                        | Description                                                                 |
| :---------------------------- | :-------------------------------------------------------------------------- |
| Record1 Index / Record2 Index | Row numbers corresponding to each record from the input file                |
| Score                         | Confidence score indicating likelihood of a duplicate match                 |
| Link1 / Link2                 | Clickable links to Salesforce account records                               |
| Account Type 1 / 2            | Account types for each record                                               |
| Account Record Type 1 / 2     | Salesforce record types                                                     |
| Account Name 1 / 2            | Account names                                                               |
| Parent Account 1 / 2          | Parent account names                                                        |
| Website 1 / 2                 | Website URLs                                                                |
| Root 1 / 2                    | Ensures records are not in the same hierarchy                               |
| Phone 1 / 2                   | Phone numbers                                                               |
| Billing Street 1 / 2          | Billing addresses                                                           |

---

## What This Tool Does

This tool scans a Salesforce export for duplicate records and generates a structured Excel report of suspected duplicate pairs, allowing for efficient review and merging directly in Salesforce.

---

## Why Not Just Use Excel?

Excel only detects **exact matches**, which misses real-world duplicates such as:
- Variations in names (e.g., *Bill* vs *William*)
- Contacts changing companies (different emails, same person)
- Phone number formatting differences
- Website URL inconsistencies

This tool normalizes and applies fuzzy matching to catch these cases.

---

## Why Multiple Matching Keys?

The tool groups records before comparing them to improve speed and accuracy:

- Name-only matching misses typos  
- Phone-only misses updated or missing numbers  
- Email-only misses job changes  

By combining multiple signals (name, phone, email), the tool captures duplicates more effectively while staying performant.
