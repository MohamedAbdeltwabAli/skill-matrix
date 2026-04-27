# Factory Skill Matrix — Test Version

A simple browser-based exam system that automatically detects a worker's department from their SAP number and loads the relevant questions.

---

## Test SAP numbers

Use these to try the exam:

| SAP Number | Department |
|------------|------------|
| 10001 | Welding |
| 10002 | Welding |
| 10003 | Quality Control |
| 10004 | Quality Control |
| 10005 | Maintenance |
| 10006 | Maintenance |
| 10007 | Assembly |
| 10008 | Assembly |

---

## How to deploy on GitHub Pages (step by step)

### Step 1 — Create a GitHub account
Go to [github.com](https://github.com) and sign up for a free account if you don't have one.

### Step 2 — Create a new repository
1. Click the **+** icon (top right) → **New repository**
2. Name it: `skill-matrix` (or anything you like)
3. Set it to **Public**
4. Click **Create repository**

### Step 3 — Upload the file
1. Inside your new repository, click **Add file** → **Upload files**
2. Drag and drop the `index.html` file
3. Scroll down and click **Commit changes**

### Step 4 — Enable GitHub Pages
1. Go to your repository **Settings** tab
2. Scroll down to **Pages** (left sidebar)
3. Under **Source**, select **Deploy from a branch**
4. Choose branch: **main** — folder: **/ (root)**
5. Click **Save**

### Step 5 — Get your live URL
After 1–2 minutes, your page will be live at:
```
https://YOUR-USERNAME.github.io/skill-matrix/
```
Replace `YOUR-USERNAME` with your actual GitHub username.

---

## What this test version does

- Worker enters their name and SAP number
- App automatically identifies their department (no dropdown needed)
- Shows 3 MCQ questions for that department
- Scores the exam and shows a pass/fail result
- In this test version, results are logged to the browser console (press F12 to see them)

## Next steps (after testing)

Once you confirm the flow works, the next version will:
- Connect to Google Sheets to save every result automatically
- Allow HR to update the SAP→Department list from the sheet
- Add a manager dashboard to view all scores
