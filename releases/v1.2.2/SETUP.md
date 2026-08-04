# Spreadsheet Setup

This project can send submissions to either Google Sheets or Microsoft Excel Online.
GitHub Pages and Electron read their bundled defaults from `config.js` (`site/config.js` source and `docs/config.js` publish output).
The Electron app can override bundled values with its device-local Electron user-data settings.
The spreadsheet side must expose an HTTP endpoint that accepts the payload shape used by the form.

For the public GitHub Pages form, the endpoint must also return a successful CORS-enabled response to the GitHub Pages origin. The form now verifies the response before showing a submission confirmation. If a Google Apps Script or Power Automate endpoint cannot provide CORS headers, place a CORS-enabled proxy or API in front of it; do not use an opaque `no-cors` request because it cannot confirm that a record was saved.

## Submission payload

The app sends these fields:

- `submissionId`
- `eventId`
- `timestamp`
- `firstName`
- `lastName`
- `organisation`
- `jobTitle`
- `email`
- `phone`
- `interest`
- `source`
- `registrationChannel`
- `comments`
- `gdpr`

For reporting and CSV export, the app uses these columns:

- Submission ID
- Event ID
- Timestamp
- First Name
- Last Name
- Organisation
- Job Title
- Email
- Phone
- Engagement Interest
- Source
- Registration Channel
- Comments
- GDPR Consent

Keep the spreadsheet columns aligned with that order if you want the row mapping to stay simple.

## Google Sheets setup

Use this path when you want submissions stored in a Google Sheet.

1. Create a Google Spreadsheet for the event leads.
2. Open Apps Script from the spreadsheet and paste in a submission handler like this. It writes each submission to a tab named after its `eventId`, creating the tab and its headers on the first submission for that event.

```javascript
const HEADERS = [
   'Submission ID',
   'Event ID',
   'Timestamp',
   'First Name',
   'Last Name',
   'Organisation',
   'Job Title',
   'Email',
   'Phone',
   'Engagement Interest',
   'Source',
   'Registration Channel',
   'Comments',
   'GDPR Consent'
];

function doGet() {
   return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
}

function getEventSheetName(eventId) {
   const cleanedName = eventId
      .replace(/[\\/:?*\[\]\u0000-\u001F\u007F]/g, '-')
      .trim() || 'Event';

   if (cleanedName === eventId && cleanedName.length <= 100) {
      return cleanedName;
   }

   const hash = Utilities.computeDigest(
      Utilities.DigestAlgorithm.SHA_256,
      eventId
   )
      .slice(0, 6)
      .map(function (byte) {
         return ('0' + ((byte + 256) % 256).toString(16)).slice(-2);
      })
      .join('');
   const suffix = '-' + hash;

   return cleanedName.slice(0, 100 - suffix.length) + suffix;
}

function doPost(e) {
   const lock = LockService.getScriptLock();
   lock.waitLock(30000);

   try {
      const payload = JSON.parse(e.postData.contents || '{}');
      const eventId = String(payload.eventId || '').trim();
      const submissionId = String(payload.submissionId || '').trim();

      if (!eventId) {
         throw new Error('eventId is required');
      }

      const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
      const eventSheetName = getEventSheetName(eventId);
      let sheet = spreadsheet.getSheetByName(eventSheetName);

      if (!sheet) {
         sheet = spreadsheet.insertSheet(eventSheetName);
         sheet.appendRow(HEADERS);
      }

      if (!submissionId) {
         throw new Error('Missing submissionId');
      }

      const lastRow = sheet.getLastRow();
      if (lastRow > 1) {
         const existingIds = sheet.getRange(2, 1, lastRow - 1, 1).getValues().flat();
         if (existingIds.includes(submissionId)) {
            return ContentService
               .createTextOutput(JSON.stringify({ ok: true, deduped: true }))
               .setMimeType(ContentService.MimeType.JSON);
         }
      }

      sheet.appendRow([
         submissionId,
         payload.eventId || '',
         payload.timestamp || '',
         payload.firstName || '',
         payload.lastName || '',
         payload.organisation || '',
         payload.jobTitle || '',
         payload.email || '',
         payload.phone || '',
         payload.interest || '',
         payload.source || '',
         payload.registrationChannel || '',
         payload.comments || '',
         payload.gdpr || ''
      ]);

      return ContentService
         .createTextOutput(JSON.stringify({ ok: true, deduped: false }))
         .setMimeType(ContentService.MimeType.JSON);
   } finally {
      lock.releaseLock();
   }

}
```

3. Deploy the script as a Web App.
    - Execute as: `Me`
    - Who has access: `Anyone` or `Anyone with the link`
4. Copy the deployed `/exec` URL into `FLOW_URL` in `site/config.js`.
5. The Electron admin editor stores device-specific overrides in Electron user data.
6. Save the config file and test a form submission.

If you expect retries from the desktop app, make the receiving endpoint idempotent on `submissionId`.
The app sends a unique `submissionId` for each lead, so the handler should ignore duplicates instead of appending the same row again when a submission times out after the server already accepted it. The example above uses a script lock to keep the dedupe check and append operation safe under concurrent retries.

### Google Sheets checklist

- Use the deployed Web App URL, not the editor URL.
- Event IDs are sanitised for tab names: unsupported characters and control characters become `-`; values longer than 100 characters are shortened. A short hash is appended whenever a value changes, preventing clashes such as `event/a` and `event:a`.
- The Event ID column retains the original, decoded event ID. The script creates a new event tab with headers automatically; do not add a conflicting tab with the generated name.
- Confirm the script has permission to write to the target spreadsheet.
- Confirm the script appends the fields in the same order as the app payload.
- Submit one test record and verify a row appears in the correct tab.

## Microsoft Excel Online setup

Use this path when you want submissions stored in Excel Online or SharePoint/OneDrive.

1. Create or open the target Excel workbook in OneDrive or SharePoint.
2. Convert the target range into a table.
3. Add table columns that match the fields listed above.
4. Create a Power Automate flow with an HTTP request trigger.
5. Parse the incoming JSON body so the flow can read the form fields.
6. Add an Excel action such as "Add a row into a table" and map each incoming field to the matching table column.
7. Save the flow and copy the HTTP trigger URL.
8. Put that trigger URL into `FLOW_URL` in `site/config.js`.
9. Save the config file and test a form submission.

### Excel checklist

- The workbook must use a table, not just a plain range.
- The table column names should stay stable once the flow is built.
- If you change the table structure later, update the flow mappings.
- Submit one test record and confirm the row lands in the expected workbook and table.

## Updating the configuration

If you use the in-app admin settings in the Electron build, the same values can be edited from the admin dialog:

- `FLOW_URL`
- `EVENT_ID`

The admin password is created and stored per device in the Electron app. It is not stored in `site/config.js`.

For packaged apps, the admin dialog writes the endpoint and event values to Electron user data for that device.
