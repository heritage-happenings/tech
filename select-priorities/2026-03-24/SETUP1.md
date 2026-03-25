# Setup Guide: Saving Survey Responses to Google Sheets

This takes about 5 minutes. You'll create a Google Sheet, add a small script to it, and paste one URL into `index.html`. No server needed.

---

## Step 1 — Create the Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a new blank spreadsheet.
2. Name it something like **Tech Support Survey Responses**.
3. In **Row 1**, add these column headers:

| A | B | C | D |
|---|---|---|---|
| **Timestamp** | **Name** | **Picks** | **Custom Suggestions** |

4. (Optional) Rename the sheet tab at the bottom to `Responses`.

---

## Step 2 — Add the Apps Script

1. In your spreadsheet, go to **Extensions → Apps Script**.
2. Delete any code already in the editor.
3. Paste in this entire script:

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);

    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    var timestamp = data.timestamp || new Date().toISOString();
    var name = data.name || 'Anonymous';
    var picks = (data.picks || []).join(', ');
    var custom = (data.customSuggestions || []).join(', ');

    sheet.appendRow([timestamp, name, picks, custom]);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

4. Click the **Save** icon (or Ctrl+S / Cmd+S).
5. Name the project something like "Survey Handler".

---

## Step 3 — Deploy as a Web App

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Set these options:
   - **Description:** Survey endpoint (or anything you like)
   - **Execute as:** Me
   - **Who has access:** **Anyone**
4. Click **Deploy**.
5. If prompted, authorize the script (it needs permission to write to your sheet).
6. **Copy the Web app URL** — it looks like:
   ```
   https://script.google.com/macros/s/AKfycbx.../exec
   ```

```
https://script.google.com/macros/s/AKfycbx_ibn8wRFSsnvwNg3K_WhH2qiSwgzEyPjm_Ba12vHAnNmYtGazWnJS_fA4wiywj8eu2A/exec

---

## Step 4 — Paste the URL into index.html

Open `index.html` and find this line near the top of the `<script>` block:

```javascript
const APPS_SCRIPT_URL = '';
```

Paste your URL between the quotes:

```javascript
const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbx.../exec';
```

Save the file. That's it!

---

## Step 5 — Test It

1. Open `index.html` in a browser (locally or on GitHub Pages).
2. Pick a few topics, enter a name, and click **Submit My Picks**.
3. Check your Google Sheet — a new row should appear within a few seconds.

---

## How It Works

- The survey page sends a `POST` request with JSON data to your Apps Script URL.
- The script parses the JSON and appends a row to your spreadsheet.
- Because we use `mode: 'no-cors'` in the fetch call, the browser can't read the response — but the data still arrives. That's why the page shows a "Thank You" screen optimistically.
- Each row contains: timestamp, respondent name, comma-separated list of picks, and any custom suggestions.

---

## Updating the Script Later

If you change the script code, you need to create a **new deployment** (Deploy → New deployment) to get a fresh URL, then update `index.html` with the new URL. Editing the code alone doesn't update a live deployment.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Submit button does nothing | Open browser console (F12) — look for errors. Most likely the URL is wrong or empty. |
| "Setup needed" banner shows | You haven't pasted the URL into `APPS_SCRIPT_URL` yet. |
| Data not appearing in Sheet | Make sure the script is deployed with "Anyone" access. Re-deploy if in doubt. |
| Authorization popup won't go away | Click "Advanced" → "Go to [project name] (unsafe)" → Allow. Google shows a warning because the script isn't verified, but it's your own script. |
