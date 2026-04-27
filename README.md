# Skill Matrix — Technical Documentation

> Internal exam system for Arab Group technicians.  
> Built by **Mohamed Abdeltwab Ali** — Process Engineer — WM Group  
> Contact: [mmesba01@elaraby.com](mailto:mmesba01@elaraby.com)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Architecture](#2-architecture)
3. [Google Spreadsheet Structure](#3-google-spreadsheet-structure)
4. [Apps Script Backend](#4-apps-script-backend)
5. [Frontend Configuration](#5-frontend-configuration)
6. [How to Add Employees](#6-how-to-add-employees)
7. [How to Add Questions](#7-how-to-add-questions)
8. [Security Model](#8-security-model)
9. [Deploying Updates](#9-deploying-updates)
10. [Performance](#10-performance)
11. [Known Limitations](#11-known-limitations)

---

## 1. Project Overview

The Skill Matrix is a web-based exam tool that:

- Authenticates employees via **Google Sign-In**
- Looks up their **department** based on their SAP number
- Serves a randomized **exam** fetched from Google Sheets
- Validates answers **server-side** (answers never reach the browser)
- Records results automatically to a Google Sheet
- Prevents re-taking the exam via email-based anti-cheat

---

## 2. Architecture

```
Employee Browser (GitHub Pages)
        │
        │ 1. GET ?action=getEmployees
        │ 2. GET ?action=getQuestions&department=...
        │ 3. POST { action: "scoreExam", responses: [...] }
        ▼
Google Apps Script (Web App)
        │
        ├── CacheService (6-hour in-memory cache)
        │
        └── Google Spreadsheet
                ├── Employees  (SAP lookup + passwords)
                ├── Questions  (question bank, answers server-side only)
                └── Results    (exam results log)
```

### Technology Stack

| Layer | Technology |
|---|---|
| Frontend | HTML + Vanilla JS + CSS |
| Hosting | GitHub Pages (free) |
| Backend | Google Apps Script |
| Database | Google Sheets |
| Auth | Google Sign-In (OAuth 2.0) |

---

## 3. Google Spreadsheet Structure

The spreadsheet has **3 tabs**:

### Tab 1: `Results` — Exam Results Log

Automatically populated when an employee submits an exam.

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| Timestamp | SAP | Name | Email | Department | Score | Percent |
| 27/4/2026 | 10001 | محمد أحمد | m@gmail.com | البلاستيك | 7/10 | 70% |

> ⚠️ **Do not edit column D (Email)** — it is used by the anti-cheat system to detect repeat submissions.

---

### Tab 2: `Employees` — Employee Registry

Managed by you. Add one row per employee.

| A | B | C | D |
|---|---|---|---|
| **Name** | **SAP** | **Department** | **Password** |
| محمد أحمد | 10001 | البلاستيك | AB12 |
| علي محمود | 10002 | البلاستيك | XK87 |
| سامي حسن | 20001 | الكلينشينج | ZP34 |

**Rules:**
- SAP numbers must be unique
- Passwords are case-sensitive — keep them short and simple (e.g. `AB12`, `X9K3`)
- Department name must **exactly match** the department name used in the Questions sheet
- Changes take effect within **6 hours** (cache expiry) or immediately after a script redeploy

---

### Tab 3: `Questions` — Question Bank

Managed by you. Add as many questions as needed per department.

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| **Department** | **Type** | **Question** | **Opt A** | **Opt B** | **Opt C** | **Opt D** | **Answer** |
| البلاستيك | mcq | ما هي درجة الحرارة...؟ | 50-100 | 150-300 | 300-500 | أقل من 50 | B |
| البلاستيك | tf | يجب فحص القالب... | | | | | 1 |

**MCQ rules:**
- `Type` = `mcq`
- Fill columns D–G with the four answer options
- `Answer` = `A`, `B`, `C`, or `D` (capital letter, matching the correct option column)

**True/False rules:**
- `Type` = `tf`
- Leave columns D–G empty
- `Answer` = `1` (True / صح) or `0` (False / خطأ)

> 🔒 **The Answer column (H) is never sent to the browser.** Scoring happens entirely on the server.

---

## 4. Apps Script Backend

**File:** `appsscript.gs`

### Actions

| Action | Method | Description |
|---|---|---|
| `getEmployees` | GET | Returns `{ SAP: { name, department, password } }` |
| `getQuestions` | GET | Returns shuffled questions **without answers** |
| `scoreExam` | POST | Validates answers server-side, saves result, returns score |
| `check` | GET | Checks if an email has already submitted (anti-cheat) |

### Constants (top of file)

```javascript
const SHEET_NAME    = "Results";    // Results tab name
const EMPLOYEES_TAB = "Employees";  // Employees tab name
const QUESTIONS_TAB = "Questions";  // Questions tab name
const MAX_QUESTIONS = 10;           // Questions shown per exam session
```

### Caching

Both `getEmployees` and `getQuestions` use `CacheService` (6-hour TTL).  
The first request reads from Google Sheets; all subsequent requests are served from memory.

To **force-clear the cache** after updating the sheet:
1. Open Apps Script editor
2. Run this in the console:
```javascript
CacheService.getScriptCache().removeAll(["employees", "questions_البلاستيك", "questions_الكلينشينج"]);
```
Or simply wait 6 hours for automatic expiry.

---

## 5. Frontend Configuration

**File:** `index.html`

The only values you need to update are at the top of the `<script>` section:

```javascript
// Line ~809
const APPS_SCRIPT_URL = "https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec";
const EXAM_DURATION_SECONDS = 15 * 60; // 15 minutes
```

**Current live URL:**
```
https://script.google.com/macros/s/AKfycbw98FQ4UG4uZtuWAjYQmc_ar31T4Y_GbLrnxzvFW9qBk8LeNP4j-Ho5qz2-e3xqO1ev/exec
```

**OAuth Client ID** (do not change unless re-registering the app):
```
345260370366-fv81sbdbl7ts16j20rqn2vpjecsje3he.apps.googleusercontent.com
```

---

## 6. How to Add Employees

1. Open the Google Spreadsheet → `Employees` tab
2. Add a new row:

| Name | SAP | Department | Password |
|---|---|---|---|
| اسم الموظف | رقم SAP | اسم القسم | كلمة مرور قصيرة |

3. Send the employee their **SAP number** and **password** privately (e.g. via WhatsApp)
4. No code changes needed — the app reads the sheet automatically

> ⏳ Changes appear after the 6-hour cache expires, or after a new Apps Script deployment.

---

## 7. How to Add Questions

1. Open the Google Spreadsheet → `Questions` tab
2. Add rows following the format above
3. You can add as many questions as you want — the app randomly selects `MAX_QUESTIONS` (currently 10) per session

**Example rows:**

```
| البلاستيك | mcq | ما هي درجة الحرارة المناسبة لحقن البلاستيك؟ | 50-100 | 150-300 | 300-500 | أقل من 50 | B |
| البلاستيك | tf  | يجب فحص القالب قبل بدء الإنتاج              |        |         |         |           | 1 |
```

**Tips:**
- Add at least 15–20 questions per department so different employees get different question sets
- The shuffle happens server-side — each exam session is unique
- To change the number of questions shown, update `MAX_QUESTIONS` in `appsscript.gs` and redeploy

---

## 8. Security Model

### 3-Layer Identity Verification
```
Layer 1: Google Sign-In  → employee must sign in with their real Google account
Layer 2: SAP number      → must exist in the Employees sheet
Layer 3: Personal password → must match the password set for that SAP
```

### Anti-Cheat: No Repeat Submissions
- On sign-in, the system checks if the employee's **email already exists** in the Results sheet
- If yes → exam is blocked immediately
- On submission, the server **checks again** before saving (protects against JS manipulation)

### Answers Never Leave the Server
- The `Questions` sheet's Answer column (H) is **never included** in any response to the browser
- Employee responses (selected options) are sent to Apps Script, which looks up the correct answers from the sheet and scores them server-side
- Inspecting network traffic or page source reveals **zero answers**

---

## 9. Deploying Updates

### Updating the Frontend (index.html)
1. Make changes to `index.html`
2. Commit and push to GitHub
3. Changes are live immediately on GitHub Pages

### Updating the Backend (appsscript.gs)
1. Open [script.google.com](https://script.google.com)
2. Open the Skill Matrix project
3. Paste the updated content from `appsscript.gs`
4. Click **Deploy → Manage deployments**
5. Click the ✏️ edit icon → set **Version: New version** → **Deploy**
6. The URL **does not change** between versions
7. Update `APPS_SCRIPT_URL` in `index.html` **only if** you created a brand-new deployment (not a new version)

> ✅ Best practice: always use "New version" on the existing deployment — never create a new deployment unless necessary.

---

## 10. Performance

### What's fast
| Action | Speed |
|---|---|
| Exam start (after SAP entry) | ~0 sec — questions pre-fetched during password typing |
| 2nd+ concurrent user, same dept | ~0.3 sec — Apps Script cache hit |
| Score calculation | ~1–2 sec — server validates + saves |

### What's slow
| Action | Speed | Reason |
|---|---|---|
| First page load | 2–4 sec | Apps Script cold start |
| First user of the day | Slightly slower | Cache is empty, sheet must be read |

### To eliminate cold starts (optional)
Add a **warm-up trigger** in Apps Script:
1. Add this function to `appsscript.gs`:
```javascript
function keepWarm() {} // empty — just wakes the script
```
2. In Apps Script editor: **Triggers → Add Trigger**
3. Function: `keepWarm` | Time-driven | Every 5 minutes
4. This keeps the script warm during working hours at no cost

---

## 11. Known Limitations

| Limitation | Detail |
|---|---|
| **No re-take** | Once an email submits, it's permanently blocked. To reset, delete the row from the Results sheet. |
| **Cache delay** | Adding new employees/questions may take up to 6 hours to appear without a redeploy. |
| **Apps Script quotas** | Free Google accounts: 90 min execution/day. Google Workspace: 6 hours/day. Sufficient for hundreds of exams. |
| **Password security** | Passwords are stored in plain text in the Employees sheet. Do not use sensitive passwords. |
| **Single attempt only** | By design — one exam per Google account. This is intentional. |
| **Internet required** | The app requires internet to load employees, questions, and submit results. Offline use is not supported. |
