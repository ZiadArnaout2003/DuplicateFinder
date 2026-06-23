# Duplicate Finder — User Guide

## What This Tool Does

This tool scans a Salesforce export for duplicate records and produces an Excel report listing every suspected duplicate pair, with a direct link to each record so you can review and merge them in Salesforce.

---

## Why Not Just Use Excel?

Excel's built-in duplicate detection only catches **exact matches**. It would catch a name typed twice, but it silently misses the cases that actually matter in a real CRM:

- **Bill Chen** and **William Chen** — same person, nickname on file
- **Sarah Mitchell** at Initech vs **Sarah Mitchell** at Globex — same person who changed companies, now has two records with different emails and different accounts
- A phone number stored as `(514) 555-1234` in one record and `5145551234` in another — same number, different formatting
- A website stored as `https://www.acme.com` in one record and `acme.com` in another

This tool normalizes all of that before comparing, and uses fuzzy matching to catch near-matches — so it finds the duplicates that have been quietly sitting in your CRM for years.

---

## Why Are There Multiple Matching Keys?

The tool groups records into "blocks" before comparing them, so it only compares records that are plausibly related rather than every record against every other. Multiple blocking keys are used because any single key would miss legitimate duplicates:

- **Name-only blocking** would miss two records for the same person if one has a typo in the last name
- **Phone-only blocking** would miss records where the phone number was updated or left blank
- **Email-only blocking** would miss records where the person changed companies and has a new work email

By blocking on name stem, phone prefix, and email prefix in parallel, the tool catches duplicates that would slip through any single key — and still only compares pairs that have at least one signal in common, keeping the process fast.

---

## Before You Export — Required Columns

Make sure your Salesforce report includes exactly these columns. Extra columns are ignored; missing required ones will cause a validation error.

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

> **Confirmed Duplicate** — This column is used on the **second run onwards**. On your first cleanup pass, leave it blank or omit it. After that initial cleanup, some people genuinely belong in the system twice — for example, a contact who works at two companies simultaneously will have two legitimate records. Set `Confirmed Duplicate = True` on those records so the tool knows to skip them on future runs and doesn't keep flagging them as duplicates.

---

## How to Use It

**1. Download the tool**

Download `duplicates_finder.exe` from the Releases section of this repository and save it anywhere on your computer. No installation required.

**2. Export your data from Salesforce**

Run a Salesforce report with the columns listed above and export it as a `.csv` or `.xlsx` file.

**3. Run the tool**

Double-click `duplicates_finder.exe`. The app window will open.

**4. Select the record type**

Use the dropdown to choose **Contact** or **Account** depending on what you exported.

**5. Upload your file**

Click **Browse**, navigate to your exported file, and select it. The tool will auto-detect the record type from the column headers and set the dropdown for you if it can.

**6. Run**

Click **Run Script**. A progress indicator will appear while the tool processes your file.

**7. Review the report**

When processing completes, an Excel file will be saved in the same folder as your export:

- `contact_duplicate_report.xlsx` for contacts
- `account_duplicate_report.xlsx` for accounts

Each row in the report is one suspected duplicate pair. The **Link1** and **Link2** columns are clickable and open the records directly in Salesforce so you can review and merge them.

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
| Notes              | Flags whether the pair shares the same account or may reflect a company change |

### Account Report

| Column                        | Description                               |
| :---------------------------- | :---------------------------------------- |
| Record1 Index / Record2 Index | Row numbers from your input file          |
| Score                         | Confidence score                          |
| Link1 / Link2                 | Clickable links to the Salesforce records |
| Account Type 1 / 2            | Account types                             |
| Account Record Type 1 / 2     | Account record types                      |
| Account Name 1 / 2            | Account names                             |
| Parent Account 1 / 2          | Parent account names                      |
| Website 1 / 2                 | Website URLs                              |
| Phone 1 / 2                   | Phone numbers                             |
| Billing Street 1 / 2          | Billing street addresses                  |
