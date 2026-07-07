# Google Sheets Backend Setup

Every survey submission from any driver on any device flows automatically into one Google Sheet — same pattern as the league registration form.

**Requires:** A Google account (Gmail)

---

## One-time setup (~5 minutes)

### Step 1 — Create the Google Sheet

1. Go to [sheets.google.com](https://sheets.google.com) and create a new blank spreadsheet
2. Name it **OpMo Driver Car Survey**

### Step 2 — Open Apps Script

1. In the spreadsheet, click **Extensions → Apps Script**
2. Delete any existing code in the editor
3. Paste the following script:

```javascript
function doPost(e) {
  var ss    = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getActiveSheet();

  if (sheet.getLastRow() === 0) {
    sheet.appendRow(['First Name','Last Name','iRacing ID','Cars','Submitted']);
    sheet.getRange(1,1,1,5).setFontWeight('bold').setBackground('#c4a87a').setFontColor('#0d1117');
    sheet.setFrozenRows(1);
  }

  var d = e.parameter;
  sheet.appendRow([
    d.firstName || '',
    d.lastName  || '',
    d.iracingId || '',
    d.cars      || '',
    new Date().toLocaleString('en-US')
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. Click **Save** (disk icon) and name the project **OpMo Driver Survey**

### Step 3 — Deploy as a Web App

1. Click **Deploy → New deployment**
2. Click the gear icon next to "Select type" → choose **Web app**
3. Set:
   - **Execute as**: Me
   - **Who has access**: Anyone
4. Click **Deploy**
5. When prompted, click **Authorise access** → sign in with your Google account → click **Advanced → Go to OpMo Driver Survey (unsafe)** → **Allow**
6. **Copy the Web App URL** — it looks like:
   `https://script.google.com/macros/s/ABC.../exec`

### Step 4 — Add the URL to the form

Open `index.html` and find this line near the top of the `<script>` block:

```javascript
const APPS_SCRIPT_URL = '';
```

Paste your URL inside the quotes:

```javascript
const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_ID_HERE/exec';
```

Commit and push — every submission now writes a permanent row to your Google Sheet.

---

## The data you get

One row per submission:

| First Name | Last Name | iRacing ID | Cars | Submitted |
|---|---|---|---|---|
| Ricky | Bobby | 123456 | GT3, LMP2, Miatas | 7/6/2026, 8:12:33 PM |

- **Cars** is a comma-separated list of everything the driver ticked
  (GTP, LMP2, LMP3, GT3, GT4, TCR, GR Cup, Miatas, BMW M Cup, Porsche Cup, Ferrari Cup)
- To answer "who can drive GT3?", use **Data → Create a filter** on the Cars column,
  filter by **Text contains → GT3**
- Duplicate submissions from the same driver just add a newer row — when in doubt,
  trust the latest row for that iRacing ID

## Changing the car list

The pill grid is built from the `CAR_CLASSES` array at the top of the `<script>`
block in `index.html` — add or remove a line there and the form updates. The
spreadsheet needs no change (the Cars column stores whatever the form sends).

## Exporting to Excel

In Google Sheets go to **File → Download → Microsoft Excel (.xlsx)** — done.
