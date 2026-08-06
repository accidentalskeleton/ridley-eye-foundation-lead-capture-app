# Setup Guide

This guide covers installation-related setup and submission endpoint configuration for the app.

This project can send submissions to either Google Sheets or Microsoft Excel Online.
GitHub Pages and Electron read their bundled defaults from `config.js` (`site/config.js` source and `docs/config.js` publish output).
The Electron app can override bundled values with its device-local Electron user-data settings.
The spreadsheet side must expose an HTTP endpoint that accepts the payload shape used by the form.

For the public GitHub Pages form, the endpoint must also return a successful CORS-enabled response to the GitHub Pages origin. The form now verifies the response before showing a submission confirmation. If a Google Apps Script or Power Automate endpoint cannot provide CORS headers, place a CORS-enabled proxy or API in front of it; do not use an opaque `no-cors` request because it cannot confirm that a record was saved.

## Table of contents

- [macOS install for unsigned app](#macos-install-for-unsigned-app)
- [Submission payload](#submission-payload)
- [Google Sheets setup](#google-sheets-setup)
- [Google Sheets checklist](#google-sheets-checklist)
- [Google Apps Script proxy setup](#google-apps-script-proxy-setup)
- [Proxy checklist](#proxy-checklist)
- [Microsoft Excel Online setup](#microsoft-excel-online-setup)
- [Excel checklist](#excel-checklist)
- [Updating the configuration](#updating-the-configuration)

## macOS install for unsigned app

If macOS warns that the app is unsigned or blocked, use this immediate unblock step on your Mac:

1. Move the app into Applications first.
2. Open the **Terminal** app (Applications > Utilities > Terminal).
3. Run this command:

   `xattr -dr com.apple.quarantine "/Applications/Ridley Eye Foundation Lead Capture.app"`

4. Try opening the app again.

Optional verification:

- Check Gatekeeper assessment:

   `spctl --assess -vv "/Applications/Ridley Eye Foundation Lead Capture.app"`

- Check signature metadata:

   `codesign -dv --verbose=4 "/Applications/Ridley Eye Foundation Lead Capture.app"`

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
         sheet.getRange(1, 1, 1, HEADERS.length).setFontWeight("bold");
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

## Google Apps Script proxy setup

Use this path when the public GitHub Pages form needs a stable `FLOW_URL`, but the real submission endpoint may change over time.

Recommended pattern:

1. Keep `FLOW_URL` in `site/config.js` pointed at one long-lived Apps Script proxy `/exec` URL.
2. Store the real downstream endpoint in the proxy's Script Properties as `TARGET_FLOW_URL`.
3. Update `TARGET_FLOW_URL` when the destination changes, instead of republishing the GitHub Pages site.

This is useful when:

- the public site should always post to one fixed URL
- the real Google Apps Script or Power Automate endpoint may be rotated later
- you want the browser form to receive a normal JSON success/failure response with CORS headers

Create a dedicated Apps Script project and use a proxy handler like this:

```javascript
function createJsonOutput(payload) {
   return ContentService
      .createTextOutput(JSON.stringify(payload))
      .setMimeType(ContentService.MimeType.JSON);
}

function doGet() {
   return createJsonOutput({ ok: true, proxy: true });
}

function doPost(e) {
   const props = PropertiesService.getScriptProperties();
   const targetUrl = String(props.getProperty('TARGET_FLOW_URL') || '').trim();

   if (!targetUrl) {
      return createJsonOutput({
         ok: false,
         error: 'TARGET_FLOW_URL is not configured'
      });
   }

   const payload = e && e.postData && e.postData.contents
      ? e.postData.contents
      : '{}';

   try {
      const response = UrlFetchApp.fetch(targetUrl, {
         method: 'post',
         contentType: 'application/json',
         payload: payload,
         muteHttpExceptions: true
      });

      const body = response.getContentText();
      const code = response.getResponseCode();

      if (code < 200 || code >= 300) {
         return createJsonOutput({
            ok: false,
            error: 'Proxy target returned an error',
            status: code,
            body: body
         });
      }

      try {
         return createJsonOutput(JSON.parse(body));
      } catch (parseError) {
         return createJsonOutput({
            ok: true,
            proxied: true,
            status: code,
            raw: body
         });
      }
   } catch (error) {
      return createJsonOutput({
         ok: false,
         error: String(error && error.message ? error.message : error)
      });
   }
}
```

Deployment steps:

1. Open **Project Settings** in Apps Script.
2. Add a Script Property named `TARGET_FLOW_URL` with the real downstream URL.
3. Deploy the proxy script as a Web App.
   - Execute as: `Me`
   - Who has access: `Anyone` or `Anyone with the link`
4. Put the proxy `/exec` URL into `FLOW_URL` in `site/config.js`.
5. Test one public GitHub Pages submission.
6. When the real destination changes later, update only `TARGET_FLOW_URL` in Script Properties.

Notes:

- Keep the proxy project long-lived so the public `FLOW_URL` stays stable.
- Do not store secrets in `site/config.js`; it is public.
- If the downstream target requires auth headers or custom routing, move that logic into the proxy rather than exposing it in the site.
- For GitHub Pages, verify the proxy returns a normal JSON response that the form can inspect.

### Proxy checklist

- `FLOW_URL` in `site/config.js` points to the proxy `/exec` URL, not the changing downstream target.
- The proxy Script Property `TARGET_FLOW_URL` is set.
- The proxy Web App is deployed and accessible.
- The downstream target still accepts the app's submission payload.
- A live public Pages submission succeeds after each target rotation.

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

For the public GitHub Pages site, `FLOW_URL` should usually be a stable public endpoint. If the real submission target may change, prefer a proxy URL that stays fixed and forwards to the current destination.

The admin password is created and stored per device in the Electron app. It is not stored in `site/config.js`.

For packaged apps, the admin dialog writes the endpoint and event values to Electron user data for that device.
