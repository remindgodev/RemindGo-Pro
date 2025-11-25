# RemindGo Pro – Roadmap

Status legend:
- [ ] Not started
- [~] In progress
- [x] Done

---

## M0 – Project Skeleton & Docs

**Goal:** Clean Android project set up with basic structure + documentation so we always know where things live.

- [ ] Create Android project `RemindGoPro` (Kotlin, minimum SDK as agreed).
- [ ] Set up base package structure:
  - `ui/` (activities/fragments/screens)
  - `core/` (common utils, config)
  - `data/` (datasets, storage)
  - `domain/` (business logic, decision engines)
- [ ] Add `docs/PROJECT_OVERVIEW.md` (problem, solution, principles).
- [ ] Add `docs/ROADMAP.md` (this file).
- [ ] Add `docs/DEVLOG.md` and make first entry.

**Done when:** App builds & runs with a simple “Hello RemindGo Pro” screen and docs folder exists.

---

## M1 – Consent & Permissions Flow

**Goal:** Single, clear flow to request all needed permissions without being annoying or confusing.

- [ ] Design a “Welcome / Setup” screen that explains:
  - What the app does (high level).
  - Why it needs Location, Activity Recognition, Notifications, Battery optimisation ignore (if needed later).
- [ ] Implement stepwise permission requests:
  - [ ] Activity Recognition
  - [ ] Location (foreground + background as required)
  - [ ] Notifications (Android 13+)
- [ ] Handle “deny” and “don’t ask again” gracefully with explanation + retry path.
- [ ] Persist a simple onboarding-completed flag.

**Done when:** New user opens app → goes through setup → permissions granted or clearly declined, no crashes.

---

## M2 – Core Architecture & Interfaces

**Goal:** Define clear abstractions so logic is readable and testable (inversion + extraction).

- [ ] Create interfaces for key services:
  - `LocationProvider`
  - `ActivityStateProvider`
  - `GeofenceClient`
  - `DatasetRepository`
  - `ReminderEngine`
- [ ] Implement simple, real Android-backed versions:
  - Basic location provider (last known location).
  - Basic geofence client wrapper.
- [ ] Use dependency inversion:
  - UI talks to `ViewModel` / use cases.
  - Use cases talk to interfaces, not Android APIs directly.
- [ ] Add clear data classes & enums for decision-making:
  - `DrivingState`
  - `ZoneTier`
  - `ZoneEntryEvent`
  - `ReminderDecision`

**Done when:** We have a small set of interfaces and data classes, and no heavy logic is sitting directly in Activities.

---

## M3 – Activity Recognition & Driving State

**Goal:** Reliably know when the user is driving vs not driving.

- [ ] Hook up Activity Recognition / Activity Transition API (or equivalent).
- [ ] Implement `DrivingStateManager`:
  - [ ] Listen to activity changes (e.g. STILL, WALKING, IN_VEHICLE).
  - [ ] Apply simple rules to set `DrivingState = ACTIVE/INACTIVE`.
  - [ ] Debounce noise (e.g. short spikes not treated as full drives).
- [ ] Add logging for state transitions for debugging.
- [ ] Add a basic debug UI element (or log overlay) showing current driving state.

**Done when:** Starting/stopping a car journey reliably flips state to DRIVING/NOT_DRIVING (as visible in logs or on a debug text label).

---

## M4 – Tile / Dataset Loader (Mock → Real)

**Goal:** Load the right GeoJSON or dataset slices for the user’s rough area (city/region), starting with a mock.

- [ ] Define format & storage for:
  - Entry points (geofences).
  - Polygons (zones).
- [ ] Implement a local mock dataset (e.g. a small set of zones in one city) bundled with the app.
- [ ] Implement `DatasetRepository` with:
  - [ ] `getZonesForTile(tileId)`
  - [ ] `getEntryPointsForTile(tileId)`
- [ ] Implement simple tile computation (e.g. geohash5 or grid) from current location.
- [ ] Log which tile is active and which dataset slice is loaded.

**Done when:** When standing in your test area, the app can compute a tile and load a small set of entry points + polygons from local data.

---

## M5 – Local Storage & Config

**Goal:** Persist lightweight app state and configuration so behaviour is consistent across sessions.

- [ ] Set up local storage (e.g. Room DB or simple key-value + JSON for MVP).
- [ ] Store:
  - [ ] User onboarding state.
  - [ ] User mutes / preferences per zone.
  - [ ] Last known tile / dataset metadata.
  - [ ] History of zone entries (for testing).
- [ ] Create `LocationConfig` and `ReminderConfig` objects/constants:
  - Hysteresis distance/time thresholds.
  - Max number of active geofences.
  - Reminder cooldowns.

**Done when:** Closing/reopening the app preserves user mutes and basic configuration, and config is all in one place (no magic numbers scattered).

---

## M6 – Tier 0: Simple Single Geofence Demo

**Goal:** Prove end-to-end geofence → callback → notification with a hardcoded zone.

- [ ] Implement `GeofenceClient` wrapper (add/remove/list geofences).
- [ ] Register a single test geofence around a fixed location.
- [ ] Implement broadcast receiver / service to handle enter/exit.
- [ ] Show a simple test notification on enter.
- [ ] Add logging around geofence registration and triggers.

**Done when:** Walking/driving into the test geofence reliably triggers a notification.

---

## M7 – Tier A: Entry-Point Geofences from Dataset

**Goal:** Dynamically arm a set of entry-point geofences around the user while driving (no polygons yet).

- [ ] Implement `GeofenceArmer` service:
  - [ ] Given current tile + dataset, choose N closest entry points.
  - [ ] Register geofences for those points.
  - [ ] Clean up old geofences when user moves far enough (hysteresis).
- [ ] Connect `DrivingStateManager` → `GeofenceArmer`:
  - Start arming when state = DRIVING.
  - Stop/remove geofences when state = NOT_DRIVING.
- [ ] Add logs showing:
  - Number of geofences registered.
  - IDs and coordinates.
  - Reason when geofences are refreshed.

**Done when:** On starting a drive in the test city, the app automatically arms entry-point geofences and logs them; entering one triggers a test notification.

---

## M8 – Tier B: Polygon Confirmation & Hysteresis Logic

**Goal:** Confirm that the user truly entered a charge zone polygon before deciding on reminders, using distance/time thresholds.

- [ ] Implement `ZoneEntryEvaluator`:
  - [ ] On entry-point trigger, compute whether user is likely entering a polygon (e.g. based on location + heading + proximity).
  - [ ] Apply hysteresis: only re-fetch/rescan when:
    - Moved ≥ X km, or
    - Time since last evaluation ≥ Y minutes, or
    - Tile changed.
- [ ] Parse and store polygon shapes from dataset.
- [ ] Implement point-in-polygon checks for current location vs zone.
- [ ] Capture a `ZoneEntryEvent` with:
  - Zone ID, entry ID, timestamp, tier, confirmation status.
- [ ] Log decisions with reasons (for debugging).

**Done when:** Entering an entry-point geofence leads to a polygon check and a clear “inside zone / outside zone” decision in logs.

---

## M9 – Reminder Engine (Real-Time + End-of-Drive)

**Goal:** Turn confirmed entries into smart reminders that respect deadlines, schedules, and user preferences.

- [ ] Design `ReminderEngine` with clear API:
  - `onZoneEntry(event: ZoneEntryEvent)`
  - `onDriveEnded()`
- [ ] Implement decision flow (in code, not UI yet):
  - [ ] Is this zone chargeable today?
  - [ ] Has the user muted this zone?
  - [ ] Have we already reminded for this entry?
  - [ ] Is it before/after payment deadline?
- [ ] Implement:
  - [ ] Immediate reminder (on or shortly after entry).
  - [ ] End-of-drive reminder (if configured).
- [ ] Log every reminder decision (Send/Skip) with reason.
- [ ] Basic notifications (no fancy UI yet) with:
  - Zone name.
  - High-level action (e.g. “Check payment for ULEZ”).

**Done when:** Entering a test zone produces reminders that follow the rules, and the engine’s decisions are readable in logs.

---

## M10 – Settings & User Controls

**Goal:** Give users control so the app feels helpful, not naggy.

- [ ] Add a simple Settings screen:
  - [ ] Toggle for real-time reminders.
  - [ ] Toggle for end-of-drive reminders.
  - [ ] Per-zone mute/unmute (list of recently seen zones).
  - [ ] Quiet hours or “Do not disturb while driving” variant if needed.
- [ ] Persist all settings via local storage.
- [ ] Update `ReminderEngine` to respect settings.

**Done when:** User can mute a zone or turn off certain reminder types, and behaviour changes accordingly.

---

## M11 – Debug & Diagnostics Tools

**Goal:** Make it easy to understand what the app is doing during development and early testing.

- [ ] Add a hidden or developer-only Debug screen showing:
  - Current driving state.
  - Current tile ID.
  - Active geofences (IDs + radii).
  - Last few `ZoneEntryEvent`s.
  - Last few `ReminderDecision`s.
- [ ] Add ability to export logs (e.g. to file or share intent).
- [ ] Add a simple “Test notification” button to verify channels.

**Done when:** You can open the debug screen on your device and understand what the app is doing without attaching Android Studio.

---

## M12 – Battery & Background Behaviour Tuning

**Goal:** Ensure app is efficient and reliable in the background.

- [ ] Review and optimise:
  - Activity recognition polling & thresholds.
  - Location update rate while driving.
  - Geofence limits per OS (and our own max).
- [ ] Test:
  - Long drives with screen off.
  - App killed/restarted by OS.
- [ ] Add fallbacks if activity recognition is unavailable.
- [ ] Document any device-specific quirks and workarounds.

**Done when:** App can run for a full day’s normal driving without noticeable battery drain beyond expectations and still triggers reminders reliably.

---

## M13 – Beta Packaging & Internal Testing

**Goal:** Prepare for internal/beta release.

- [ ] Set up app icon, name, and basic branding.
- [ ] Clean up logging (dev vs production levels).
- [ ] Add privacy policy screen + link.
- [ ] Prepare Play Store internal testing release (if using Play Store).
- [ ] Internal test plan:
  - Different routes.
  - Different zones.
  - Mix of mute/unmute, settings, background behaviour.

**Done when:** You (and a small group) can install the app via an internal track and use it on real journeys.

---

## Current Focus

_(Update this section as we go.)_

- M0 – Project Skeleton & Docs

---

## Recently Completed

_(Move milestones here or list completed tasks with dates.)_

- …

