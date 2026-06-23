# 🏢 Salesforce Account Duplicate Finder

A systematic score-based duplicate detection pipeline designed for **high accuracy and clean Excel reporting.** The tool processes large datasets and outputs a ready-to-review spreadsheet with direct Salesforce links.

---

## 🚀 How It Works

The system identifies duplicate Account records in a 4-step process built to handle the messiness of real-world CRM data.

### 🧼 Step 1 — Data Loading & Normalization

Before running any matches, every record is cleansed so formatting differences don't hide real duplicates. Original values are always preserved for the output report.

| Field                                 | What happens                                           | Example                                        |
| :------------------------------------ | :----------------------------------------------------- | :--------------------------------------------- |
| **Account Name**                      | Legal suffixes abbreviated, punctuation stripped       | `"Acme Solutions Inc."` → `"acme sol inc"`     |
| **Phone / Fax**                       | All non-digits removed                                 | `"(514) 555-1234"` → `"5145551234"`            |
| **Website**                           | Scheme, port, path, and common subdomains stripped     | `"https://mail.acme.com/login"` → `"acme.com"` |
| **Billing Street / Secondary Street** | Abbreviations expanded                                 | `"123 Main St."` → `"123 main street"`         |
| **Billing Zip / Secondary Zip**       | Hyphen suffix removed                                  | `"H3A 1B2-4500"` → `"H3A 1B2"`                 |
| **All text fields**                   | Lowercased, whitespace collapsed, `NaN`/`None` cleared |                                                |

The account **parent hierarchy** is also resolved at this stage. Every record is stamped with a `Root ID 18 Digits` — its ultimate ancestor in the Salesforce hierarchy — using path-compression memoization. Any two records that share a root are intentionally related and are excluded from duplicate consideration entirely.

---

### 🔀 Step 2 — Smart Grouping (Multi-Pass Blocking)

Records are assigned block keys and only records sharing at least one key are compared — keeping the process fast even at scale.

| Pass        | Key format                           | Fields used                  |
| :---------- | :----------------------------------- | :--------------------------- |
| 1 — Name    | `name_{first 8 chars of first word}` | Account Name (normalized)    |
| 2 — Phone   | `ph_{first 7 digits}`                | Phone, Secondary Phone       |
| 3 — Website | `web_{domain prefix}`                | Website (normalized)         |
| 4 — Address | `addr_{zip5}_{street prefix}`        | Billing Zip + Billing Street |

A record can belong to multiple blocks. A `detected_pairs` set ensures each pair is only evaluated once regardless of how many blocks it shares.

---

### 🧮 Step 3 — Confidence Scoring

#### Hard Filters (applied before any scoring)

| Filter                                    | Effect                                                |
| :---------------------------------------- | :---------------------------------------------------- |
| Same corporate hierarchy (shared Root ID) | Score forced to 0 — intentionally related records     |
| Shell / record type mismatch              | Score forced to 0 — different account classifications |
| Billing Country mismatch                  | Score forced to 0 — different regions excluded        |

#### Signal Points

| Signal                       | Points  | Condition                                                               |
| :--------------------------- | :-----: | :---------------------------------------------------------------------- |
| Website — exact              | **+40** | Normalized domains are identical                                        |
| Website — fuzzy              |   +25   | Similarity > 90%                                                        |
| Phone match                  | **+40** | Any phone field intersects (Phone, Secondary Phone, Fax, Secondary Fax) |
| Street + zip + country match | **+35** | Billing or Secondary address matches exactly across all three fields    |
| Street match only            |   +20   | Street matches, zip differs                                             |
| Zip match only               |   +15   | Zip matches, street differs                                             |
| Account name — near          | **+30** | Similarity ≥ 95%                                                        |
| Account name — fuzzy         |   +15   | Similarity ≥ 85%                                                        |
| Account name — loose         |   +5    | Similarity ≥ 75%                                                        |

#### 🎯 Duplicate Threshold: 70 Points

A pair is flagged as a duplicate when its total score reaches **70 or more**. No single signal can reach 70 alone — at least two strong independent signals must agree.

**Examples that pass:**

| Combination                            | Score | Result       |
| :------------------------------------- | :---: | :----------- |
| Phone (+40) + Name near (+30)          |  70   | ✅ Duplicate |
| Website exact (+40) + Street+Zip (+35) |  75   | ✅ Duplicate |
| Phone (+40) + Website exact (+40)      |  80   | ✅ Duplicate |
| Website exact (+40) + Name near (+30)  |  70   | ✅ Duplicate |

**Examples that are safely rejected:**

| Combination                           | Score | Result                                      |
| :------------------------------------ | :---: | :------------------------------------------ |
| Name near (+30) + Zip only (+15)      |  45   | ❌ Rejected — subsidiaries can share a zip  |
| Website fuzzy (+25) + Name near (+30) |  55   | ❌ Rejected — not enough independent signal |

---

### 📤 Step 4 — Excel Report

Confirmed pairs are written to `account_duplicate_report.xlsx`. Every row contains the **original, un-normalized** values for easy review.

| Column                            | Description                                                                |
| :-------------------------------- | :------------------------------------------------------------------------- |
| `Record1 Index` / `Record2 Index` | Row positions in the input file (1-based, matches spreadsheet row numbers) |
| `Score`                           | Total confidence score                                                     |
| `Link1` / `Link2`                 | Salesforce record URLs (clickable hyperlinks)                              |
| `Account Type 1` / `2`            | Original account types                                                     |
| `Account Record Type 1` / `2`     | Original account record types                                              |
| `Account Name 1` / `2`            | Original account names                                                     |
| `Parent Account 1` / `2`          | Original parent account names                                              |
| `Website 1` / `2`                 | Original website URLs                                                      |
| `Phone 1` / `2`                   | Original phone numbers                                                     |
| `Billing Street 1` / `2`          | Original billing street addresses                                          |

---

## 🛠️ Usage

File upload and processing is all done through the graphical user interface (GUI).The report is saved to `account_duplicate_report.xlsx` in the working directory. Both `.csv` and `.xlsx`/`.xlsm` input files are supported.

---
