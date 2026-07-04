# Architecture — Personal Holistic Health App

**Version:** 1.1 (v1 scope)

## Related documents
- `PRD.md` — product scope and modules.
- `DATA_MODEL.md` — the schema stored in IndexedDB.
- `DESIGN_SYSTEM.md` — UI layer, navigation pattern, responsive breakpoints.

---

## 1. App shape

Single self-contained web app, opened in the browser and added to the home screen for full-screen, app-like launch. Runs on both phone and laptop from the same codebase. No backend, no accounts, no server-side logic. After first load it runs fully offline.

## 2. Hosting & delivery

The app's code is served as static files from a static host (e.g. GitHub Pages). The user opens it once via that URL on each device; the app is then cached and launches offline thereafter.

Hosting serves only the application code. No user data is ever sent to or stored on the host. A static URL (rather than a raw local file) is required so the service worker and "Add to Home Screen" install behave reliably.

## 3. Offline

A service worker caches the app shell on first load so the app opens with no network connection on subsequent launches. The app must be fully functional offline; there are no runtime network dependencies.

## 4. Storage & persistence

- All user data is stored in **IndexedDB** on the device.
- Data persists across app close, browser restart, and device reboot. It is lost only if the user clears browser data, removes the browser, or changes devices — which is what iCloud sync (§6) and export (§7) protect against.
- No data is transmitted to any server operated by this app under any circumstances.
- The schema is defined in `DATA_MODEL.md`.

## 5. State layer

A single storage module is the only code that reads from or writes to IndexedDB; all features access data through it. This boundary is deliberate: it isolates all persistence, keeps the sync layer (§6) decoupled from features, and makes future storage changes a swap rather than a rewrite.

## 6. Sync — file in iCloud Drive

The app supports using the same data across the user's phone and laptop via a single data file kept in the user's own iCloud Drive. This is local-first and server-free: the app has no backend, and data lives only on the user's devices and in the user's own iCloud account.

- **Model:** the app's working data lives in IndexedDB (§4). The authoritative portable copy is a single `.json` data file the user keeps in an iCloud Drive folder. Apple's iCloud syncs that folder across the user's devices; the app does not sync anything itself.
- **Save to iCloud:** the user saves the current data to the file in their iCloud Drive folder via the browser's file interface.
- **Load from iCloud:** on another device, the user opens/imports that same file to bring its data into local IndexedDB.
- **Not silent auto-sync:** because a browser app cannot silently read or write arbitrary iCloud files, switching devices involves an explicit save-then-load step. This is an accepted tradeoff to keep the app server-free and the data in the user's own account.
- **Conflict rule:** last-write-wins at the file level. The user should save from the device they were last using before loading on another, to avoid overwriting newer data. The file carries a schema version and a last-modified timestamp (§8) so the app can warn if the incoming file is older than local data.

## 7. Backup & restore

- **Export:** a one-tap action serializes the entire database to a single `.json` file. This is the same file format used for iCloud sync (§6); saving to an iCloud Drive folder doubles as backup.
- **Import:** the user selects a previously exported `.json` file to restore all data.
- Backup/save is manual, with no reminders (per `PRD.md`).

## 8. Data integrity & versioning

- The IndexedDB schema is versioned. On app load, migrations bring older local data up to the current schema without loss.
- Data files include the schema version and a last-modified timestamp, so a file from an older app version can be imported into a newer one, and stale-file warnings (§6) are possible.

## 9. Responsive layout

One codebase serves phone and laptop, switching layout by viewport breakpoint (see `DESIGN_SYSTEM.md` for the breakpoints and navigation pattern):
- **Phone:** primary navigation is a bottom tab bar; content is single-column.
- **Laptop:** primary navigation becomes a left sidebar; content area widens and multi-column layouts are used where useful (e.g. dashboard cards and Insights charts side-by-side).
- Layout is the only thing that changes across devices; features and data model are identical.

## 10. Deferred but designed-for

- **Notifications/reminders** (e.g. hydration prompts, log reminders).
- **Silent automatic multi-device sync** — would require a backend or a native app with direct iCloud access; explicitly out of scope to keep the app server-free.
- **Pattern-learning cycle prediction** (see `CYCLE_AND_TCM.md`).

## 11. Tech posture

Dependencies are kept minimal so the app can run unattended for years. **Recommendation: build vanilla (plain JavaScript, no UI framework)** — the app is single-user and long-lived, and fewer dependencies means less to break or update over time. IndexedDB access may use one small, well-established wrapper library for ergonomics; no larger framework is warranted for v1.
