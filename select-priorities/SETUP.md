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
// ── Handle incoming survey submissions (POST) ──
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();

    var timestamp = data.timestamp || new Date().toISOString();
    var name      = data.name || 'Anonymous';
    var picks     = (data.picks || []).join(' || ');
    var custom    = (data.customSuggestions || []).join(' || ');

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

// ── Return aggregated results (GET) ──
function doGet(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var rows  = sheet.getDataRange().getValues();

    // Skip header row
    var dataRows = rows.slice(1).filter(function(row) {
      return row[0] !== '' && row[0] !== null;
    });

    var totalResponses = dataRows.length;
    var topicCounts = {};
    var customSuggestions = [];

    dataRows.forEach(function(row) {
      var name   = row[1] || 'Anonymous';
      var picks  = (row[2] || '').toString();
      var custom = (row[3] || '').toString();

      // Count each pick
      if (picks) {
        picks.split(' || ').forEach(function(topic) {
          var t = topic.trim();
          if (t) {
            topicCounts[t] = (topicCounts[t] || 0) + 1;
          }
        });
      }

      // Collect custom suggestions
      if (custom) {
        custom.split(' || ').forEach(function(suggestion) {
          var s = suggestion.trim();
          if (s) {
            customSuggestions.push({
              topic: s,
              submittedBy: name
            });
          }
        });
      }
    });

    var result = {
      totalResponses: totalResponses,
      topicCounts: topicCounts,
      customSuggestions: customSuggestions
    };

    return ContentService
      .createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ error: err.toString() }))
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
   - **Description:** Survey endpoint
   - **Execute as:** Me
   - **Who has access:** **Anyone**
4. Click **Deploy**.
5. If prompted, authorize the script (it needs permission to write to your sheet).
6. **Copy the Web app URL** — it looks like:
   ```
   https://script.google.com/macros/s/AKfycbx.../exec
   ```

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

Save the file. Done!

---

## Step 5 — Test It

1. Open `index.html` in a browser (locally or on GitHub Pages).
2. Pick a few topics, enter a name, and click **Submit My Picks**.
3. Check your Google Sheet — a new row should appear within a few seconds.
4. Click **View Results** in the header — you should see a bar chart of all submissions.

You can also link directly to the results view:
```
https://yoursite.github.io/#results
```

---

## How It Works

**Submitting (POST):**
- The survey page sends a `POST` with JSON to your Apps Script.
- The script appends a row: timestamp, name, comma-separated picks, comma-separated custom suggestions.
- We use `mode: 'no-cors'` because Apps Script doesn't support CORS preflight for POST. The data still arrives; we just can't read the response.

**Viewing results (GET):**
- The results page sends a `GET` request to the same URL.
- The `doGet` function reads all rows, tallies votes per topic, collects custom suggestions, and returns JSON.
- The page renders a horizontal bar chart sorted by popularity, plus stats and write-in ideas.

---

## Updating the Script Later

If you change the script code, you must create a **new deployment** (Deploy → New deployment) to get a fresh URL, then update `APPS_SCRIPT_URL` in `index.html`. Editing code alone doesn't update a live deployment.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Submit does nothing | Check browser console (F12). Most likely the URL is wrong or empty. |
| "Setup needed" banner | You haven't pasted the URL into `APPS_SCRIPT_URL` yet. |
| Data not appearing | Make sure script is deployed with **Anyone** access. Re-deploy if unsure. |
| Results won't load | Make sure you deployed the version with **both** `doPost` and `doGet`. A deployment made before adding `doGet` won't have it — create a new deployment. |
| Authorization popup | Click "Advanced" → "Go to [project name] (unsafe)" → Allow. Google warns because the script isn't verified, but it's your own code. |
| CORS error on results | This usually means the URL is wrong or the deployment access isn't set to "Anyone". |
| Old rows show split topics | Data submitted before this version used commas as delimiters. Either re-enter those rows or find-and-replace `, ` with ` \|\| ` in columns C and D of your sheet. |
