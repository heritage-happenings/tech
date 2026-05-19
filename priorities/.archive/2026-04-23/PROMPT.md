# Prompt: Build the HotM Tech Support Priorities Survey

Build a complete priority-selection survey tool for the **HotM Tech Support Subcommittee**. The deliverable is two things:

1. A single self-contained `index.html` file — no dependencies, no build step, no server
2. A Google Apps Script that acts as the backend, wired to a Google Sheet

---

## What the tool does

Participants open the page, browse a categorised list of tech support topics, tap up to **10** they want covered, optionally add write-in suggestions, enter their name, and hit Submit. Their picks are stored in a Google Sheet. A Results tab on the same page shows a live bar chart of all submissions, fetched directly from the sheet.

---

## index.html — full specification

### Layout & views

The page has two views toggled by a pill-style button pair in the header:
- **Take the Survey** — the selection interface
- **View Results** — the aggregated bar chart

A sticky counter bar sits below the header showing how many topics have been chosen (e.g. "3 of 10 chosen"), a row of 10 animated dots filling as topics are selected, and a dynamic hint that reads "Tap topics to choose" → "7 more to go" → "All 10 chosen!" when the limit is reached. The counter bar is hidden when the Results view is active.

### Survey view

At the top, a brief instruction box explains the task.

If `APPS_SCRIPT_URL` is empty, show a dismissible banner prompting setup.

Topics are rendered from a JavaScript data array (see below) into category cards. Each card has a coloured top border, a category header with icon, and subcategory labels above a responsive grid of topic chips. Each chip is a `div` with `role="checkbox"`, `aria-checked`, and `tabindex="0"` so it is keyboard and screen-reader accessible. Clicking a chip toggles selection. Once 10 are chosen, unselected chips become visually disabled and unclickable.

Below the topic cards is a **"Something Missing?"** section with a text input and Add button (also triggered by Enter key) for write-in suggestions. Write-ins count toward the 10-topic limit and appear as removable tags. The remove button on each tag must be a real `<button>` element with an `aria-label`.

Once at least one topic is selected, a **summary panel** slides into view showing a numbered list of picks with their category. Below it: a name field (required), a Submit button, and a Copy to Clipboard button.

On submit, the page shows a **thank-you screen** with a ✓ icon, a note that picks have been recorded, and a small caveat: *"Don't see your name in results? The network may have dropped — feel free to resubmit."* (This is important because POST uses `mode: 'no-cors'` and failures are silent.) Two buttons offer "Submit Another Response" and "View Results".

Draft selections persist in `localStorage` so the page can be refreshed without losing picks.

### Results view

On first load, fetch results from the Apps Script URL (GET request, no query parameters needed). Cache the response in memory. Switching back to Results re-uses the cache — no repeat network call. A **↻ Refresh Results** button forces a fresh fetch (`force = true` bypasses the cache). A successful submit also clears the cache so results are fresh after your own submission.

Display three stat cards (total responses, topics chosen, write-in ideas), a horizontal bar chart sorted by vote count, and a list of write-in suggestions with the name of who submitted each.

Bars animate in (width transitions from 0) using `requestAnimationFrame`. Each bar is coloured by category class.

### Design

- **Fonts:** Sora (headings) and Plus Jakarta Sans (body) — load both in a single Google Fonts `<link>` tag
- **Palette:**
  - Background: `#F4F4F0`
  - Surface: `#FFFFFF`
  - Ink: `#111122`
  - Primary accent (indigo): `#5B5BD6`
  - Hardware accent (teal): `#14B8A6`
  - Software accent (indigo): `#5B5BD6`
  - Security accent (amber): `#F59E0B`
  - Accessibility accent (violet): `#8B5CF6`
  - Error/warning accent (peach): `#F97066`
- **Header:** Dark (`#111122`) with a subtle indigo radial glow, bouncing 💡 icon, title, subtitle, and the view toggle
- **Cards:** White, soft shadow, 16px radius, 3px coloured top border per category
- **Mobile:** Single-column topic grid below 600px, stacked buttons, hidden progress dots

### CSS variable rules

Define all variables used in JavaScript feedback calls. The correct variables are:
- `var(--peach)` for error/warning states — **not** `--coral`
- `var(--teal-dark)` for success states — **not** `--sage-dark`
- `var(--ink-muted)` for muted text — **not** `--bark-muted`

### Topic data

```javascript
const DATA = [
  {
    name: "Hardware", icon: "🖥️", barClass: "hw",
    subcategories: [
      { label: "Computers", items: ["Laptops", "Desktops", "Peripherals (keyboards, mice, monitors)"] },
      { label: "Printing", items: ["Printing Devices"] },
      { label: "Mobile Devices", items: ["Smartphones", "Tablets"] },
      { label: "Internet Connectivity", items: ["Wi-Fi setup & troubleshooting", "Modem / router configuration"] },
      { label: "Entertainment Systems", items: ["Smart TVs", "Streaming devices (Roku, Apple TV, etc.)", "Audio systems (speakers, headphones)"] },
      { label: "Assistive Technology", items: ["Hearing aids", "Screen readers", "Mobility devices"] },
      { label: "Home Automation", items: ["Smart home devices (thermostats, lights, security)"] },
      { label: "Wearables", items: ["Fitness trackers", "Smartwatches", "VR headsets"] }
    ]
  },
  {
    name: "Software", icon: "💾", barClass: "sw",
    subcategories: [
      { label: "Operating Systems", items: ["Windows", "macOS", "Linux", "iOS", "Android"] },
      { label: "Productivity", items: ["Microsoft Office", "Google Workspace", "Email clients (Outlook, Gmail)"] },
      { label: "Communication", items: ["Video calls (Zoom, Skype, FaceTime)", "Messaging apps (WhatsApp, Messenger)"] },
      { label: "Social Media", items: ["Facebook", "Twitter / X", "Instagram"] },
      { label: "Online Services", items: ["Online banking", "E-commerce (Amazon, eBay)", "Government portals (SSA, healthcare)"] },
      { label: "Entertainment", items: ["Streaming (Netflix, Hulu)", "Music (Spotify, Apple Music)"] },
      { label: "Learning", items: ["Online courses (Coursera, Udemy)", "E-books & audiobooks"] }
    ]
  },
  {
    name: "Security & Privacy", icon: "🔒", barClass: "sec",
    subcategories: [
      { label: "Protection", items: ["Antivirus programs", "Password managers", "VPN services"] },
      { label: "Digital Literacy", items: ["Internet safety & privacy", "Recognizing scams & phishing"] }
    ]
  },
  {
    name: "Accessibility & Customization", icon: "♿", barClass: "acc",
    subcategories: [
      { label: "Accessibility Tools", items: ["Screen magnifiers", "Voice recognition software", "Text-to-speech"] },
      { label: "Customization", items: ["Personalizing settings & preferences", "Using accessibility features effectively"] },
      { label: "Maintenance", items: ["Software updates", "Troubleshooting common issues", "Cloud storage & backups", "File organization"] }
    ]
  }
];
```

---

## Google Apps Script — full specification

### Google Sheet setup

Create a new Google Sheet named **Tech Support Survey Responses**. In Row 1 add these headers:

| A | B | C | D |
|---|---|---|---|
| Timestamp | Name | Picks | Custom Suggestions |

Optionally rename the sheet tab to `Responses`.

### Apps Script

Go to **Extensions → Apps Script**, delete any existing code, and paste the following:

```javascript
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

function doGet(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var rows  = sheet.getDataRange().getValues();

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

      if (picks) {
        picks.split(' || ').forEach(function(topic) {
          var t = topic.trim();
          if (t) topicCounts[t] = (topicCounts[t] || 0) + 1;
        });
      }

      if (custom) {
        custom.split(' || ').forEach(function(suggestion) {
          var s = suggestion.trim();
          if (s) customSuggestions.push({ topic: s, submittedBy: name });
        });
      }
    });

    return ContentService
      .createTextOutput(JSON.stringify({ totalResponses, topicCounts, customSuggestions }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

### Deployment

1. Click **Deploy → New deployment**
2. Click the gear icon → choose **Web app**
3. Set **Execute as:** Me, **Who has access:** Anyone
4. Click Deploy and authorise when prompted
5. Copy the resulting URL (format: `https://script.google.com/macros/s/.../exec`)

### Wiring it up

In `index.html`, find:
```javascript
const APPS_SCRIPT_URL = '';
```
Paste the URL between the quotes and save.

### Important deployment note

If you ever change the script code, you must create a **new deployment** to get an updated URL — editing code alone does not update a live deployment.

---

## How POST submissions work

The survey sends a JSON payload via `fetch` with `method: 'POST'` and `mode: 'no-cors'`. Apps Script does not support CORS preflight for POST, so `no-cors` is required. The trade-off is that the response body is opaque — the page cannot confirm success. The thank-you screen always shows regardless of outcome, hence the caveat note to resubmit if a name doesn't appear in results.

The payload shape is:
```json
{
  "name": "Jane Smith",
  "timestamp": "2026-03-25T10:00:00.000Z",
  "picks": ["Laptops", "Wi-Fi setup & troubleshooting", "Password managers"],
  "customSuggestions": ["Electric wheelchair charging"]
}
```

Picks and custom suggestions are stored as `||`-delimited strings in the sheet (e.g. `Laptops || Wi-Fi setup & troubleshooting`).
