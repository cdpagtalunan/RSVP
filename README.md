# Christian & Ijane — Wedding Invitation

A single-page wedding invitation site for **January 19, 2027**. It uses pure HTML, CSS, and vanilla JavaScript, with no frameworks and no build step, and is ready for GitHub Pages.

## Project structure

```
index.html     ← everything (HTML + inline CSS + inline JS)
images/        ← put your photos here
music/         ← optional background song
.nojekyll      ← tells GitHub Pages to serve files as-is
```

## Personalize: search `TODO` in index.html

| What | Where |
|---|---|
| Hero background photo | `<!-- TODO: Insert Hero Background Image here -->` inside `.hero__media` |
| Story / venue / gallery photos | `<!-- TODO: Insert Photo Here -->`. Replace the `<span class="photo__placeholder">` with `<img src="images/..." alt="..." loading="lazy">` |
| RSVP endpoint (Apps Script URL) | `data-endpoint="…"` on `<form id="rsvp-form">`. See [RSVP setup](#rsvp-setup-custom-form--google-apps-script) below |
| RSVP guest-count options | `<select id="rsvp-guests">`, set to the number of seats per invitation |
| Ceremony / reception times, schedule | Venue cards + `.timeline` list; also `CONFIG.weddingDate` and `CONFIG.calendar` in the script |
| Dress code, RSVP deadline, hashtag, story text | Marked with `TODO` comments |
| Background music (optional) | Add `music/our-song.mp3` and uncomment the `<source>` in `<audio id="bg-music">` |

## RSVP setup: custom form + Google Apps Script

The RSVP section is a custom HTML form styled to match the site. When a guest submits it, the page sends the answers to a small **Google Apps Script web app**, which adds a row to a **Google Sheet** you own. There is no server to run and it's free.

```
Guest's browser ──POST──▶ Apps Script web app (/exec) ──▶ Google Sheet "RSVP" tab
```

### 1. Create the Google Sheet
1. Go to <https://sheets.new> and name it something like **Wedding RSVPs**.
2. You don't need to add headers. The script creates an **RSVP** tab with headers on the first submission.

### 2. Add the Apps Script
1. In the Sheet, open **Extensions → Apps Script**.
2. Delete everything in `Code.gs` and paste the code below.
3. Optional: set `NOTIFY_EMAIL` to your email address to get an email for every RSVP.
4. Click **Save** (💾).

```javascript
/**
 * Wedding RSVP endpoint — Christian & Ijane
 * Receives POSTs from the website's RSVP form and appends them to the "RSVP" sheet.
 */
const SHEET_NAME   = 'RSVP';
const NOTIFY_EMAIL = '';   // e.g. 'you@example.com' to get an email per RSVP; leave '' to disable

const HEADERS = ['Timestamp', 'Name', 'Email', 'Phone', 'Attending', 'Guests', 'Dietary notes', 'Message'];

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.waitLock(10000); // prevent two simultaneous RSVPs from colliding

  try {
    const p = (e && e.parameter) || {};

    // Honeypot / basic validation
    if (p.website) return json({ result: 'success' });
    if (!p.name || !p.attending) return json({ result: 'error', error: 'Missing required fields' });

    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss.getSheetByName(SHEET_NAME) || ss.insertSheet(SHEET_NAME);
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(HEADERS);
      sheet.getRange(1, 1, 1, HEADERS.length).setFontWeight('bold');
      sheet.setFrozenRows(1);
    }

    const row = [
      new Date(),
      clean(p.name, 120),
      clean(p.email, 160),
      clean(p.phone, 30),
      p.attending === 'yes' ? 'Yes' : 'No',
      p.attending === 'yes' ? Number(p.guests) || 1 : 0,
      clean(p.dietary, 200),
      clean(p.message, 1000)
    ];
    sheet.appendRow(row);

    if (NOTIFY_EMAIL) {
      MailApp.sendEmail(
        NOTIFY_EMAIL,
        `RSVP: ${row[1]} — ${row[4]}${row[4] === 'Yes' ? ' (' + row[5] + ')' : ''}`,
        HEADERS.map((h, i) => `${h}: ${row[i]}`).join('\n')
      );
    }

    return json({ result: 'success' });
  } catch (err) {
    return json({ result: 'error', error: String(err) });
  } finally {
    lock.releaseLock();
  }
}

// Lets you open the /exec URL in a browser to confirm the deployment works
function doGet() {
  return json({ result: 'ok', message: 'RSVP endpoint is live' });
}

// Trim, limit length, and stop spreadsheet formula injection (=, +, -, @)
function clean(value, max) {
  let s = String(value || '').trim().slice(0, max);
  if (/^[=+\-@]/.test(s)) s = "'" + s;
  return s;
}

function json(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
```

### 3. Deploy it as a web app
1. Click **Deploy → New deployment**.
2. Click the gear ⚙ next to "Select type" and choose **Web app**.
3. Fill in the settings:
   - **Description:** `RSVP v1`
   - **Execute as:** **Me**
   - **Who has access:** **Anyone**. Guests aren't signed in to Google, so this setting is required.
4. Click **Deploy**, then **Authorize access**, and choose your Google account.
   - If you see "Google hasn't verified this app", click **Advanced → Go to (project name) (unsafe)**. This is normal for your own scripts.
5. Copy the **Web app URL**. It ends in `/exec`, for example `https://script.google.com/macros/s/AKfycb…/exec`.
6. Optional: open that URL in your browser. You should see `{"result":"ok","message":"RSVP endpoint is live"}`.

### 4. Connect the website
In `index.html`, find the RSVP form and replace the placeholder with your URL:

```html
<form class="rsvp-form" id="rsvp-form" novalidate
      data-endpoint="https://script.google.com/macros/s/AKfycb…/exec">
```

Submit a test RSVP. A new row should appear in the **RSVP** tab within a few seconds.

Until the endpoint is set, the form still shows and validates input, but submitting displays a "not connected yet" notice and logs the data to the browser console instead of sending it.

### Updating the script later
If you edit `Code.gs`, choose **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. This keeps the **same URL**, so you don't need to change `index.html`. Choosing **New deployment** instead creates a *different* URL.

### What the form collects
| Field | Required | Notes |
|---|---|---|
| Full name | Yes | |
| Email | No | Format is checked if filled in |
| Mobile number | No | |
| Will you attend? | Yes | Accepts / Declines |
| Number of guests | — | Hidden when declining; saved as `0` |
| Dietary notes | No | Hidden when declining |
| Message for the couple | No | |

A hidden "honeypot" field blocks simple spam bots.

### Troubleshooting
- **"Something went wrong" on submit:** check that the deployment's access is set to **Anyone**, that the URL ends in `/exec` (not `/dev`), and that you redeployed a **New version** after editing the script.
- **No rows appearing:** open **Apps Script → Executions** to see errors from each request.
- **Form says "not connected yet":** the `data-endpoint` still contains `YOUR_SCRIPT_ID`, or the URL isn't a `script.google.com/macros/s/…/exec` link.

### Photo tips
- Hero: landscape, ~2000px wide, compressed to under 500 KB.
- Others: ~1200px on the long side. Export as `.jpg` or `.webp`.

## Preview locally
Open `index.html` directly in a browser, or run a tiny server:
```
python -m http.server 8000
```
The envelope is skipped on reloads within the same tab. Open a new tab to see it again.

## Deploy to GitHub Pages
1. Create a repo and push these files to the `main` branch.
2. Go to **Settings → Pages → Build and deployment**. Set Source to **Deploy from a branch**, Branch to **main / (root)**.
3. Your site will be live at `https://<username>.github.io/<repo>/` in a minute or two.
