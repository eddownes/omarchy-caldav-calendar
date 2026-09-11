# CalDav Calendar (Ed's fork)

Add an event from the Omarchy clock. It shows up on your iPhone, Mac, and everywhere else your CalDAV calendar lives.

iCloud, Nextcloud, Fastmail — same two-way sync. Month, week, and day views, meeting links, and reminders, without moving the clock.

This is [eddownes](https://github.com/eddownes)'s fork of [SirWizardLizard/omarchy-caldav-calendar](https://github.com/SirWizardLizard/omarchy-caldav-calendar), carrying one fix not yet in upstream: **CalDAV discovery against Fastmail**, which 404s on a bare server address until you follow its `/.well-known/caldav` redirect. See [About this fork](#about-this-fork) below for details and how to reapply it on a new machine.

<p align="center">
  <img src="screenshots/month.png" alt="Month view" width="800">
</p>

## Install

```bash
omarchy plugin add https://github.com/eddownes/omarchy-caldav-calendar --enable
```

This replaces the built-in clock. Shortcuts stay put. If Omarchy leaves the center pin on `omarchy.clock`, the plugin retargets that pin to itself on first run. It does not change a custom or empty pin.

The plugin needs Evolution Data Server. If it is not installed, the clock opens a prompt with **Click to install** (opens a terminal for `omarchy pkg add evolution-data-server`) and **SOURCE** (the Arch package page). You do not have to run that command yourself.

## What it does

- Month, week, work week, and day views
- Create, edit, and delete events (including overnight and recurring) — they sync to your phone and other devices
- iCloud, Nextcloud, Fastmail, and other CalDAV servers
- Calendars that live only on this computer
- Join Zoom, Google Meet, or Teams from a link on the event
- Optional desktop reminder 5–30 minutes before timed events

<p align="center">
  <img src="screenshots/week.png" alt="Week view" width="480">
</p>

## Add iCloud

Do not use your Apple ID password. Apple requires an app-specific password.

1. Open [account.apple.com](https://account.apple.com) → **Sign-In and Security → App-Specific Passwords**
2. Generate a password (2FA must be on)
3. Click the clock → **Add calendar**
4. Enter:

   | Field | Value |
   | --- | --- |
   | Display name | `iCloud` |
   | CalDAV URL | `https://caldav.icloud.com/` |
   | Username | your Apple ID email |
   | Password | the app-specific password |

5. **Add CalDAV source**, then **Sync** if events are not there yet

Remove a calendar later from settings.

## Other CalDAV

Same form. Use the provider’s CalDAV URL and an app password when they require one.

- Nextcloud: `https://your-server/remote.php/dav/`
- Fastmail: `https://caldav.fastmail.com/`

## Meetings

Paste a Zoom, Meet, or Teams URL on the event (`meet.google.com/…` is fine). **Join** opens it. The plugin does not sign in to Google, Zoom, or Outlook.

## Reminders

**Settings → Remind me**: off, or 5 / 10 / 15 / 30 minutes before timed events. Click a meeting toast to join.

<p align="center">
  <img src="screenshots/settings.png" alt="Settings" width="800">
</p>

## Not included

Google Calendar and Outlook need OAuth app review. They are not in this plugin. CalDAV servers and local calendars are.

## About this fork

Upstream's own instructions above say to point Fastmail at `https://caldav.fastmail.com/` — but that bare address 404s. Fastmail only reveals where the real CalDAV service lives through a redirect from `/.well-known/caldav` (RFC 6764), and discovery had no code path that tried it, so "Add calendar" always ended in *"Connected, but no calendars were found on that CalDAV server."*

Two commits on top of upstream `main` fix this:

- **Follow the RFC 6764 well-known redirect for a bare CalDAV address** — `discover_caldav_calendars` now PROPFINDs `<origin>/.well-known/caldav` first for any bare server address and, on a redirect, resolves the `Location` header — refused unless it names the same scheme and host the account was configured with, so a server-supplied redirect can never hand credentials to a different origin. Covered by a `--fastmail-like` mode in the test harness that reproduces Fastmail's exact 404-then-redirect behavior, plus unit tests for the pure redirect-resolution logic.
- **Stop the snapshot timeout on its own id, not through root** — an unrelated QML scoping bug (`root.snapshotTimeout.stop()` where `snapshotTimeout.stop()` was meant, as it's called correctly everywhere else in `Service.qml`).

Everything else is untouched upstream code. `helper/omarchy-calendar-helper` keeps its manifest `id` (`sirwizardlizard.calendar`), so installing this fork's URL is a drop-in replacement — it lands in the same plugin folder and keeps your existing bar layout and configured calendars.

### Reapplying this on a new Omarchy install

Just point `omarchy plugin add` at this fork instead of upstream:

```bash
omarchy plugin add https://github.com/eddownes/omarchy-caldav-calendar --enable
```

### Keeping this fork current with upstream

```bash
git remote add upstream https://github.com/SirWizardLizard/omarchy-caldav-calendar.git   # once
git fetch upstream
git merge upstream/main   # or: git rebase upstream/main
git push origin main
```

If upstream ever ships its own well-known-redirect fix, drop the "Follow the RFC 6764..." commit during that merge/rebase and go back to tracking upstream directly.

## Uninstall

```bash
omarchy plugin disable sirwizardlizard.calendar
omarchy plugin remove sirwizardlizard.calendar
```

## Requirements

Omarchy 4, Python 3, and Evolution Data Server. If EDS is missing, the plugin offers to install it. License: MIT.

Credentials stay in the system keyring. They are never written to `shell.json` or this repo.

## Development

```bash
./test/all
omarchy plugin validate .
```
