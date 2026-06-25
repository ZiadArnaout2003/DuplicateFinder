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

| Signal                  | Points  | Condition                                                                 |
| :---------------------- | :-----: | :------------------------------------------------------------------------ |
| Primary email — exact   | **+50** | Identical primary email addresses                                         |
| Primary email — close   | **+30** | Same domain, usernames differ only by `.` or `_`                          |
| Alternate email — exact | **+30** | Identical alternate email addresses                                       |
| Alternate email — close | **+15** | Same domain, usernames differ only by `.` or `_`                          |
| Mobile ↔ Mobile match   | **+30** | Both records share the same mobile number (personal → highest confidence) |
| Mobile ↔ Office match   | **+20** | Mobile on one record matches an office phone on the other                 |
| Office ↔ Office match   | **+10** | Shared office phone only (often shared across a team → lowest confidence) |
| First name — near       |   +20   | Similarity ≥ 95% (includes nickname matches)                              |
| First name — fuzzy      |   +10   | Similarity ≥ 85%                                                          |
| First name — weak       | **−15** | Similarity ≥ 60% — names are different enough to reduce confidence        |
| First name — mismatch   | **−25** | Similarity < 60% — strong signal these are different people               |
| Last name — near        |   +20   | Similarity ≥ 95%                                                          |
| Last name — fuzzy       |   +10   | Similarity ≥ 85%                                                          |
| Company name — near     |   +10   | Similarity ≥ 90%                                                          |
| Company name — fuzzy    |   +5    | Similarity ≥ 80%                                                          |
| Title — exact           |   +10   | Identical job titles                                                      |
| Title — executive       |   +5    | Executive-level variations (e.g., `CEO` vs `President`)                   |
| Title — fuzzy           |   +3    | Similarity ≥ 85% (e.g., `Senior Manager` vs `Sr. Manager`)                |
| Role Code — exact       |   +5    | Identical role codes                                                      |
| Role Code — category    |   +2    | Same functional category                                                  |

> **First name penalties** prevent false positives when two people share a last name, a phone, or an email domain but have clearly different first names. Without penalties, a shared office phone between _Robert_ and _Rachel_ Lee could accumulate enough points to cross the threshold — the penalty pulls the score back down.

#### 🎯 Duplicate Threshold: 70 Points

A pair is flagged as a duplicate when its total score reaches **70 or more**.

**Examples that pass:**

| Combination                                                                                   | Score | Result       |
| :-------------------------------------------------------------------------------------------- | :---: | :----------- |
| Mobile↔Mobile (+30) + First Name near (+20) + Last Name near (+20)                            |  70   | ✅ Duplicate |
| Exact primary email (+50) + Last Name near (+20)                                              |  70   | ✅ Duplicate |
| Close primary email (+30) + First Name near (+20) + Last Name near (+20) + Company near (+10) |  80   | ✅ Duplicate |
| Exact alternate email (+30) + Mobile↔Mobile (+30) + Last Name near (+20)                      |  80   | ✅ Duplicate |

**Examples that are safely rejected:**

| Combination                                                            | Score | Result                                                             |
| :--------------------------------------------------------------------- | :---: | :----------------------------------------------------------------- |
| First Name near (+20) + Last Name near (+20) + Company near (+10)      |  50   | ❌ Rejected — same name and company, no contact signal             |
| Office↔Office (+10) + First Name near (+20) + Last Name near (+20)     |  50   | ❌ Rejected — shared office line not enough on its own             |
| Mobile↔Office (+20) + Last Name near (+20) + First Name mismatch (−25) |  15   | ❌ Rejected — penalty correctly separates _Robert_ vs _Rachel_ Lee |

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

The report is saved to `contact_duplicate_report.xlsx` in the working directory. Both `.csv` and `.xlsx`/`.xlsm` input files are supported.

---
