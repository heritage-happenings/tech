# Tech Support Priorities — Setup Guide

This voting app runs entirely on GitHub Pages and uses a **Google Form** to collect votes and a **published Google Sheet** to display results. No server needed.

---

## Step 1: Create the Google Form

1. Go to [Google Forms](https://docs.google.com/forms/) and create a new blank form.
2. Add **exactly 3 questions** in this order:

   | # | Question title | Type          |
   |---|---------------|---------------|
   | 1 | Name          | Short answer  |
   | 2 | Picks         | Short answer  |
   | 3 | Suggestions   | Short answer  |

3. The "Picks" field will receive a JSON array of topic IDs (the app fills this in automatically).
4. The "Suggestions" field will receive any write-in suggestions.

## Step 2: Get the Form Entry IDs

1. Open your Google Form and click the **three-dot menu → Get pre-filled link**.
2. Type a placeholder word in each field (e.g., "test1", "test2", "test3") and click **Get link**.
3. The URL will look like:
   ```
   https://docs.google.com/forms/d/e/FORM_ID/viewform?usp=pp_url&entry.111111=test1&entry.222222=test2&entry.333333=test3
   ```
4. Write down:
   - **FORM_ID**: the long string after `/e/` and before `/viewform`
   - **entry.111111**: the entry ID for Name
   - **entry.222222**: the entry ID for Picks
   - **entry.333333**: the entry ID for Suggestions

## Step 3: Create the Response Spreadsheet

1. In your Google Form, click the **Responses** tab.
2. Click the green Sheets icon to create a linked spreadsheet.
3. Open that spreadsheet. Copy its **SHEET_ID** from the URL:
   ```
   https://docs.google.com/spreadsheets/d/SHEET_ID/edit
   ```
4. Make the sheet viewable: **Share → "Anyone with the link" → Viewer**.

## Step 4: Configure the App

Open `index.html` and find the `CONFIG` object near the top of the `<script>` block:

```js
const CONFIG = {
  FORM_ID:      "YOUR_FORM_ID_HERE",
  ENTRY_NAME:   "entry.XXXXXXX",
  ENTRY_PICKS:  "entry.XXXXXXX",
  ENTRY_CUSTOM: "entry.XXXXXXX",
  SHEET_ID:     "YOUR_SHEET_ID_HERE",
};
```

Replace the Form values with the IDs you collected above.

For `SHEET_ID`, use the ID from your spreadsheet's **edit URL**:
```
https://docs.google.com/spreadsheets/d/THIS_PART_HERE/edit
```

⚠️ **Do NOT use the `2PACX-...` published ID** — that one appears in the "Publish to web" link but doesn't support cross-origin requests from JavaScript.

**Important:** The spreadsheet must be shared so **"Anyone with the link"** can view it. (Go to Share → General access → "Anyone with the link" → Viewer.)

## Step 5: Deploy to GitHub Pages

1. Commit `index.html` (and optionally this README) to your repo.
2. Push to GitHub.
3. Go to **Settings → Pages** in your repo.
4. Set the source to your branch (e.g., `main`) and folder (`/ (root)`).
5. Your app will be live at `https://YOUR_USERNAME.github.io/YOUR_REPO/`.

---

## How It Works

| Action       | Mechanism                                                                  |
|--------------|----------------------------------------------------------------------------|
| **Submit**   | POST to Google Form (`/formResponse`) in no-cors mode                      |
| **Read**     | Fetch Google Visualization API JSON (`/gviz/tq`), parse and tally client-side |

- Each vote is one row: `[Timestamp, Name, Picks JSON, Suggestions]`
- The results page fetches via Google's Visualization API (which supports CORS), parses the JSON, and tallies everything live.
- Multiple people can vote; each submission adds a new row.

## Notes

- **No login required** for voters — anyone with the link can vote.
- **Results are public** via the published sheet. Keep the sheet URL internal if you want privacy.
- If a person votes again, both votes are recorded (you can de-duplicate manually in the sheet).
- The Google Form will also collect a timestamp automatically.
