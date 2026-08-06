# Changelog

All notable changes to this project will be documented in this file.

## 1.3.1 - Runtime UX, CI simplification, and Pages config hardening

### Added

* Add optional Electron window sizing via `--resolution WIDTHxHEIGHT`
* Add transient submission status messages and online/offline icon indicators in the status bar
* Add setup guidance for a stable Google Apps Script proxy `FLOW_URL`

### Changed

* Refine QR panel positioning and status layout for non-kiosk operation
* Update PR validation to run lightweight app checks (`npm ci`, syntax checks, and tests) while keeping installer packaging in release-only workflows
* Update release and setup documentation to reflect current CI, runtime, and public configuration behavior
* Move GitHub Pages build configuration to repository secrets and stop tracking generated `docs/` output in source control

### Fixed

* Make Node test scripts CI-portable by using explicit test file paths
* Rebuild generated `docs/` output during smoke parity tests now that `docs/` is not tracked in source control

## 1.3.0 - Admin workflow and release documentation updates

### Added

* Add application version to the status banner
* Add nested local data sub-tabs for local records and pending uploads

### Changed

* Rework the admin dialog into tabbed settings, export, and logs views
* Make the local data viewers and action controls easier to use in the export tab
* Resize and reposition the QR widget for non-kiosk use on 2560x1664 displays
* Update the release architecture and user documentation for the current public release flow

## 1.2.1 - Documentation and channel tracking updates

### Added

* Add `USER_GUIDE.md` with non-technical operating instructions for event staff
* Add `registrationChannel` to submission payloads and CSV exports using values `Event` and `QR Scan`

### Changed

* Consolidate runtime configuration to a single `config.js` file for site, docs, and Electron defaults
* Update favicon/build icon references and replace legacy app icon assets
* Refresh `site/assets/nepal.jpg` for the latest event visual asset

## 1.2.0 - Reliability and event flow hardening

### Added

* Retry policy module and unit coverage for queued lead submission backoff behavior
* Event-specific Google Sheets setup guidance for `submissionId`-based deduplication

### Changed

* Rename the visitor consent prompt from `GDPR Consent` to `Contact Preferences` across the public form and CSV exports
* Speed up PR packaging validation for Windows and macOS while preserving required check names for branch protection
* Update workflow action dependencies (`actions/checkout`, `actions/labeler`, `actions/github-script`)

### Fixed

* Admin password dialog now accepts `Enter` to submit login/password setup
* Electron runtime settings now define default retention constants to prevent startup `ReferenceError`
* Public QR-driven registrations now preserve the event ID from `eventId` and `event_id` URL query parameters
* Saving admin configuration in the Electron app now refreshes the QR code immediately and preserves keyboard focus in the admin form
* Persist Electron runtime endpoint and event overrides in user data for packaged apps
* Bound lead submission request duration to prevent hung requests from stalling queued processing
* Improve public form submission verification reliability
* Queue retry backoff now uses the current time when rescheduling timed-out and failed submissions, avoiding immediate retries after slow responses

## 1.1.1 - Release pipeline reliability fix

### Fixed

* Prevent electron-builder implicit publish during tag builds by passing `--publish never` in CI build steps
* Keep GitHub release publication owned by the dedicated publish job in the release workflow

## 1.1.0 - Operational hardening and release automation

### Added

* Startup health checks and runtime status visibility in the application UI
* Structured runtime logging and admin log management actions
* PR packaging validation workflow for Windows and macOS installers
* Automated PR and issue triage labeling workflow
* Repository governance templates and maintenance documentation
* Changelog-driven GitHub release notes generation

### Improved

* Release workflow guardrails with stricter artifact and version validation
* Build and temporary-artifact hygiene for local and CI environments
* Dependabot and GitHub Actions dependency posture for workflow reliability

## 1.0.0 - Initial Release

### Added

* Electron desktop application wrapper
* Windows and macOS build support using electron-builder
* Local HTTP server for serving the application UI
* Secure Electron IPC bridge using `preload.js`
* Context isolation and sandboxed renderer process
* Kiosk-style exhibition mode
* Password-protected admin console
* Visitor lead capture form
* GDPR consent capture
* Multiple engagement selections
* Event ID tracking
* Unique submission ID generation
* Timestamp tracking
* Offline submission queue
* Local visitor archive
* CSV export and recovery tools
* Google Sheets integration via Google Apps Script
* Microsoft 365 Excel integration via Power Automate
* Ridley Eye Foundation branding support
* Application favicon and installer icon support

### Supported Platforms

* Windows installer (`.exe`)
* macOS application bundle (`.app`)
* macOS installer (`.dmg`)

### Security

* Renderer process isolation enabled
* Node.js integration disabled
* Secure preload bridge used for application functions
* Local configuration separated from source control

### Deployment Notes

Before an event deployment:

* Confirm the correct `EVENT_ID` is configured
* Confirm the submission endpoint works
* Test offline behaviour
* Test CSV export
* Confirm admin password is configured
* Verify kiosk exit shortcut
* Confirm laptop power and network availability
