# GoodFlip Care Plans — Google Sheet Connected

## Files
- `admin.html` — admin page to create doctor/clinic-specific patient links.
- `index.html` — patient-facing enrollment page.
- `app.js` — care plans + Google Apps Script endpoint.
- `styles.css` — responsive UI.

## Google Sheet integration
The patient form posts directly to the configured Apps Script Web App endpoint and then opens WhatsApp.

Configured endpoint:
`https://script.google.com/macros/s/AKfycbx2yP2QImY0OMxnjxVmSN9EUgouq0HmMf6O4mRLBkynZlAvaXvM0fRD4AobD0D9yJ1v1g/exec`

### IMPORTANT: update Apps Script
The first Apps Script code supplied expected JSON. This website sends normal form fields to avoid browser CORS/preflight problems. Replace your Apps Script code with:

```javascript
const SHEET_ID = '1RJGTRbOzlQWwBr27LgyCoFkpU5K9KA4a30wZGA39OZo';
const SHEET_NAME = 'Sheet1';

function doPost(e) {
  try {
    const p = e.parameter || {};

    const ss = SpreadsheetApp.openById(SHEET_ID);
    const sheet = ss.getSheetByName(SHEET_NAME);

    sheet.appendRow([
      new Date(),
      p.doctor || '',
      p.referrer || '',
      p.patientName || '',
      p.mobile || '',
      p.plan || '',
      p.age || '',
      p.gender || '',
      p.notes || '',
      'New',
      '',
      '',
      p.source || 'GoodFlip Care Link'
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({success:true}))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    return ContentService
      .createTextOutput(JSON.stringify({success:false,error:String(error)}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet() {
  return ContentService
    .createTextOutput('GoodFlip Lead API is working.');
}
```

If your Google Sheet tab is not named `Sheet1`, change `SHEET_NAME`.

After changing the Apps Script:
**Deploy → Manage deployments → Edit → New version → Deploy.**

Keep the same Web App URL if Google gives you the option to update the existing deployment.

## Sheet headers
Use this row 1:

Date & Time | Doctor/Clinic | Referring Doctor | Patient Name | Mobile | Care Plan | Age | Gender | Notes | Lead Status | Amount Collected | Doctor Payout | Source

## Patient flow
GoodFlip QR/link → Patient form → Google Sheet → WhatsApp.

## Deploy the website
Upload all files to GitHub Pages. The admin page will generate patient links containing doctor, WhatsApp routing, referral, photo, and selected care plans.

## Important
This prototype stores lead information in your Google Sheet and sends the same lead details to WhatsApp. Make sure the sheet access and Apps Script permissions are restricted to the intended GoodFlip team.
