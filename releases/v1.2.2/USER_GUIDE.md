# Ridley Visitor App User Guide

This guide is for day-to-day use at events.
Think of the app like a digital sign-in sheet that can still cope if the internet is patchy.

## What this app does

- Collects visitor details
- Stores records safely on the device
- Sends records to your online sheet/system
- Lets admins export data and manage settings

## Before you start an event

- Open the app
- Check the small status message at the top
- Do one test submission
- Confirm the test appears in your spreadsheet

If you see a warning at the top, open Admin Tools and check settings.

## Start the app (normal mode vs non-kiosk mode)

### What kiosk mode means

Kiosk mode is "lock-down mode".
It keeps the app full-screen and stops accidental closing or switching to other apps during an event.

Use kiosk mode for live visitor collection.
Use non-kiosk mode for setup, testing, and troubleshooting.

### Start in normal kiosk mode

If you are using the installed app:

1. Double-click the app icon as normal.
2. The app starts in kiosk mode automatically.

If your team has provided a desktop shortcut, use that shortcut.

### Start in non-kiosk mode (step by step)

If your team has provided a non-kiosk shortcut:

1. Close the app first.
2. Open the non-kiosk shortcut provided by your coordinator.

If you are using the installed app on Windows via a shortcut:

1. Close the app first.
2. Right-click the app shortcut and click Properties.
3. In the Target box, add a space, then `--nokiosk` at the end.
4. Click Apply, then OK.
5. Start the app from that shortcut.

If you are using the installed app on Mac:

1. Open the **Terminal** app (Applications > Utilities > Terminal).
2. Copy and paste this command, then press Enter:
	`open -a "Ridley Eye Foundation Lead Capture" --args --nokiosk`
3. If your app name is slightly different, type `open -a "` then drag the app from Applications into Terminal, then add `" --args --nokiosk` and press Enter.
4. The app will open in non-kiosk mode.

## How to register a visitor

- Fill in First Name, Last Name, Organisation, and Email (these are required)
- Tick at least one option under how they want to engage
- Tick the Contact Preferences checkbox
- Press Submit

After submit:

- You will see a thank-you message
- The screen returns to the form automatically

## Using the QR code

In the desktop app, a QR code can be shown near the top.
Visitors can scan it to open the public version of the same form on their phone.

## If internet drops or is slow

Do not panic.
The app keeps records locally and retries uploads.

What to do:

- Keep collecting visitor details as normal
- Later, open Admin Tools and check Pending uploads
- Wait for internet to return, or export pending records as backup

## Opening Admin Tools

- Press Ctrl + Shift + E on Windows
- Press Cmd + Shift + E on Mac
- Enter admin password

If no admin password exists yet, you will be asked to create one.

## Admin Tools explained (simple)

### Configuration

- Submission Endpoint: where records are sent
- Event ID: event label attached to each record
- Data Retention (days): how long local visitor data stays on device
- Log Retention (days): how long app logs stay on device
- Save Settings: saves the above values

### Admin password controls

- Update Admin Password: sets or changes the admin password for this device

### Local Data

- Export All Visitors CSV: downloads all local visitor records
- Export Pending Uploads CSV: downloads only records still waiting to upload
- Clear Local Data: permanently deletes local visitor records on this device

### Runtime Logs

- Refresh Logs: reloads latest log text
- Export Logs: downloads logs file
- Clear App Logs: deletes stored logs on this device

## Kiosk mode note

The app normally runs in kiosk mode.

- Exit shortcut: Ctrl/Cmd + Shift + Q
- Non-kiosk launch option is for admin/testing only

## Common problems and quick fixes

### I pressed Submit and nothing seems to happen

- Check required fields are filled
- Make sure Contact Preferences is ticked
- Check internet connection
- Try again

### Pending uploads is not zero

- Wait a few minutes for retries
- Check endpoint and internet
- Export Pending Uploads CSV as backup

### Admin password forgotten

How to start password reset mode from an installed app shortcut on Windows:

1. Close the app.
2. Right-click the app shortcut and click Properties.
3. In the Target box, add a space, then `--reset-admin` at the end.
4. Click Apply, then OK.
5. Start the app from that shortcut.

How to start password reset mode on Mac:

1. Close the app if it is open.
2. Open the **Terminal** app (Applications > Utilities > Terminal).
3. Copy and paste this command, then press Enter:
	`open -a "Ridley Eye Foundation Lead Capture" --args --reset-admin`
4. If your app name is slightly different, type `open -a "` then drag the app from Applications into Terminal, then add `" --args --reset-admin` and press Enter.
5. Once the app opens, continue with the steps below.

After the app is running in reset mode (both Windows and Mac):

1. Open Admin with Ctrl + Shift + E (Windows) or Cmd + Shift + E (Mac).
2. In the Admin Login dialog, click Reset Password.
3. Confirm reset.
4. Set a new admin password (minimum 8 characters) when prompted.
5. Record the new password in your secure team password process.

Important:

- Remove temporary flags from shortcuts after support tasks are complete.
- Use kiosk mode for normal event operation.

## Good operating habit at the end of each day

- Export All Visitors CSV
- Export Pending Uploads CSV
- Confirm central spreadsheet has expected records
- Keep backups with event date in filename

## One-line summary

Collect as normal, submit as normal, and if the network acts up the app will queue and retry while giving admins export and recovery tools.