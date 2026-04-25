# SMT Receipt System — Setup Guide

## Files in This Package
```
index.html    ← Admin panel (generate receipts)
receipt.html  ← Receipt viewer (print / share / QR)
verify.html   ← Public verification page
style.css     ← Shared styles
Code.gs       ← Google Apps Script (your backend API)
```

---

## Step 1 — Set Up Google Sheets

1. Go to **sheets.google.com** and create a new spreadsheet
2. Name it anything (e.g. "SMT Receipts Database")
3. **Leave the first sheet blank** — the script will auto-create headers

---

## Step 2 — Deploy Google Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete any existing code in `Code.gs`
3. **Paste the entire contents of `Code.gs`** from this package
4. Click **Save** (Ctrl+S)
5. Click **Deploy → New deployment**
6. Select type: **Web App**
7. Settings:
   - Description: `SMT Receipt API`
   - Execute as: **Me**
   - Who has access: **Anyone**
8. Click **Deploy**
9. **Copy the Web App URL** — it looks like:
   `https://script.google.com/macros/s/AKfycby.../exec`

> ⚠️ Every time you edit Code.gs, you must create a **new deployment** to apply changes.

---

## Step 3 — Configure the Frontend

1. Open `index.html` in a browser
2. Paste your **Web App URL** into the API URL field at the top
3. Click **Save URL**
4. Do the same in `verify.html`

The URL is saved in your browser's localStorage — you only need to do this once per device.

---

## Step 4 — Host the Frontend (Optional)

For public access, host the 4 files (`index.html`, `receipt.html`, `verify.html`, `style.css`) on any static hosting:

- **GitHub Pages** (free) — push files to a repo and enable Pages
- **Netlify** (free) — drag and drop the folder
- **Firebase Hosting** (free tier)
- Any web server

---

## How It Works

### Generating a Receipt (Admin)
1. Open `index.html`
2. Fill in member details and click **Generate Receipt**
3. The system:
   - Creates a unique ID: `SMT-2026-XXXX-YYYY`
   - Sends data to Google Sheets via Apps Script
   - Redirects to `receipt.html` showing the receipt + QR code

### Sharing a Receipt
From `receipt.html`:
- **Print** — browser print dialog (optimized layout)
- **Copy Verify Link** — share the URL directly
- **QR Code** — members can scan to verify themselves

### Verifying a Receipt (Public)
1. Open `verify.html` (or scan the QR code)
2. Enter the Receipt ID
3. The system queries Google Sheets in real time
4. Result: **VALID** (green) or **INVALID** (red)

---

## Receipt ID Format

```
SMT-YYYY-XXXX-RAND
     ↑     ↑    ↑
  Year   Seq  4-char random (alphanumeric)

Example: SMT-2026-4721-B3K9
```

---

## Anti-Fraud Logic

- Receipts can be copied/screenshotted freely — **that's fine by design**
- Only receipts stored in Google Sheets return VALID
- Edited amounts, names, or IDs will fail verification
- Fake IDs return INVALID immediately
- The QR code links directly to the verification page

---

## Troubleshooting

| Problem | Solution |
|---|---|
| "Failed to save" on generate | Check API URL is correct and deployed |
| Verify always returns error | Re-check the Web App URL; ensure "Anyone" access |
| No CORS errors in dev? | Use a local server (`npx serve .`) or host on the web |
| Receipt not found after creating | The POST uses `no-cors` mode; confirm data in Sheet manually |

---

## Important Note on no-cors

The `POST` request from `index.html` uses `mode: 'no-cors'` because Google Apps Script doesn't support CORS preflight for POST. This means:

- The request **is sent** and **does reach** Apps Script
- But the browser cannot read the response
- The system assumes success and redirects to `receipt.html`

**To confirm data was saved:** check your Google Sheet directly after generating a receipt.

For a production system, consider Apps Script's `doPost` JSONP workaround or a proper backend.

---

## Security Checklist

- [x] Unique, unpredictable Receipt IDs
- [x] All receipts stored in centralized database
- [x] Public verification — anyone can check
- [x] Transaction ID required for digital payments
- [x] Timestamp logging (date + time + ISO)
- [x] QR code on every receipt
- [x] Watermark on printed receipts
- [x] Right-click disabled on receipt display
