# 🛡️ Salesforce Contact Duplicate Finder

A systematic score-based duplicate detection pipeline designed for **high accuracy and clean Excel reporting.** The tool processes large datasets and outputs a formatted, clickable spreadsheet ready for review.

---

## 🚀 How It Works

The system identifies duplicate Contact records in a 4-step process built to handle the messiness of real-world CRM data.

### 🧼 Step 1 — Data Loading & Normalization

Before running any matches, every record is cleansed so formatting differences don't hide real duplicates. Original values are always preserved for the output report.

| Field                                | What happens                                           | Example                                             |
| :----------------------------------- | :----------------------------------------------------- | :-------------------------------------------------- |
| **First Name**                       | Nicknames and common typos standardized                | `"Bill"` → `"William"`, `"Micheal"` → `"Michael"`   |
| **Phone / Mobile / Alternate Phone** | All non-digits removed                                 | `"(555) 666-7777"` → `"5556667777"`                 |
| **Company Names**                    | Legal suffixes abbreviated                             | `"Solutions"` → `"Sol"`, `"Corporation"` → `"Corp"` |
| **All text fields**                  | Lowercased, whitespace collapsed, `NaN`/`None` cleared |                                                     |

---

### 🔀 Step 2 — Smart Grouping (Multi-Pass Blocking)

Instead of comparing every record against every other record, records are grouped into blocks and only records sharing at least one block key are compared — cutting comparisons drastically.

| Pass             | Key format                          | Fields used            |
| :--------------- | :---------------------------------- | :--------------------- |
| 1 — Name Stem    | `lf_{last 3 chars}_{first initial}` | Last Name + First Name |
| 2 — Phone        | `ph_{first 7 digits}`               | Phone                  |
| 3 — Email prefix | `em_{username}`                     | Email (before `@`)     |

A record can belong to multiple blocks. A `detected_pairs` set ensures each pair is only evaluated once regardless of how many blocks it shares.

---

### 🧮 Step 3 — Confidence Scoring

#### Hard Rules (applied before any scoring)

| Rule                       | Effect                                                   |
| :------------------------- | :------------------------------------------------------- |
| Country mismatch           | Score forced to 0 — different regions excluded           |
| Last name similarity < 85% | Score forced to 0 — too dissimilar to be the same person |

#### Signal Points

| Signal                         | Points  | Condition                                                           |
| :----------------------------- | :-----: | :------------------------------------------------------------------ |
| Primary email — exact          | **+50** | Identical primary email addresses                                   |
| Primary email — close          | **+30** | Same domain, usernames differ only by `.` or `_`                    |
| Alternate email — exact        | **+30** | Identical alternate email addresses                                 |
| Alternate email — close        | **+15** | Same domain, usernames differ only by `.` or `_`                    |
| Mobile phone match             | **+30** | Shared mobile number (personal → higher confidence)                 |
| Office / alternate phone match | **+15** | Shared office phone (often shared across a team → lower confidence) |
| First name — near              |   +20   | Similarity ≥ 95% (includes nickname matches)                        |
| First name — fuzzy             |   +10   | Similarity ≥ 85%                                                    |
| Last name — near               |   +20   | Similarity ≥ 95%                                                    |
| Last name — fuzzy              |   +10   | Similarity ≥ 85%                                                    |
| Company name — near            |   +15   | Similarity ≥ 90%                                                    |
| Company name — fuzzy           |   +8    | Similarity ≥ 80%                                                    |
| Title — exact                  |   +12   | Identical job titles                                                |
| Title — executive              |   +10   | Executive-level variations (e.g., `CEO` vs `President`)             |
| Title — fuzzy                  |   +8    | Similarity ≥ 85% (e.g., `Senior Manager` vs `Sr. Manager`)          |
| Title — loose                  |   +5    | Similarity ≥ 70% (e.g., `Director` vs `Associate Director`)         |
| Role Code — exact              |   +8    | Identical role codes                                                |
| Role Code — category           |   +6    | Same functional category                                            |
| Role Code — fuzzy              |   +5    | Similarity ≥ 80%                                                    |

#### 🎯 Duplicate Threshold: 70 Points

A pair is flagged as a duplicate when its total score reaches **70 or more**.

**Examples that pass:**

| Combination                                                                                   | Score | Result       |
| :-------------------------------------------------------------------------------------------- | :---: | :----------- |
| Mobile (+30) + First Name near (+20) + Last Name near (+20)                                   |  70   | ✅ Duplicate |
| Exact primary email (+50) + Last Name near (+20)                                              |  70   | ✅ Duplicate |
| Close primary email (+30) + First Name near (+20) + Last Name near (+20) + Company near (+15) |  85   | ✅ Duplicate |

**Examples that are safely rejected:**

| Combination                                                       | Score | Result                                                   |
| :---------------------------------------------------------------- | :---: | :------------------------------------------------------- |
| First Name near (+20) + Last Name near (+20) + Company near (+15) |  55   | ❌ Rejected — same name, same company, no contact signal |
| Office phone (+15) + First Name near (+20)                        |  35   | ❌ Rejected — shared office line, not enough signal      |

---

### 📤 Step 4 — Excel Report

Confirmed pairs are written to `contact_duplicate_report.xlsx`. Every row contains the **original, un-normalized** values for easy review.

| Column                 | Description                                                                |
| :--------------------- | :------------------------------------------------------------------------- |
| `Record1` / `Record2`  | Row positions in the input file (1-based, matches spreadsheet row numbers) |
| `Score`                | Total confidence score                                                     |
| `Link1` / `Link2`      | Salesforce record URLs (clickable hyperlinks)                              |
| `Full Name 1` / `2`    | Original full names                                                        |
| `Title 1` / `2`        | Original job titles                                                        |
| `Account Name 1` / `2` | Original company names                                                     |
| `Email 1` / `2`        | Primary email addresses                                                    |
| `Mobile 1` / `2`       | Mobile numbers                                                             |
| `Notes`                | `"Same Account Name"` or a note flagging a potential company change        |

---

## 🛠️ Usage

File upload and processing is all done through the graphical user interface (GUI).The report is saved to `contact_duplicate_report.xlsx` in the working directory. Both `.csv` and `.xlsx`/`.xlsm` input files are supported.

---
