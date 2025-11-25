### Feature: [short title, e.g. “Driving Mode Detection (IN_VEHICLE)”]

**Goal (1–2 sentences):**  
What this feature should do for the user / app.

**User story:**  
“As a driver, when I start driving, the app should quietly arm itself so it can warn me if I enter a charge zone.”

**Inputs & triggers:**
- Activity Recognition state changes: e.g. STILL → IN_VEHICLE
- App foreground/background state (if relevant)

**Outputs:**
- Internal state: `DrivingState = ACTIVE`
- Geofence arming: call `GeofenceArmer.armNearestEntryPoints()`
- No user-facing notification yet

**Logic rules (bullet list):**
- Only arm if last known location is available.
- On transition OUT of IN_VEHICLE → stop geofences and evaluate reminders.

**Data needed:**
- Access to local dataset of entry points.
- Config: number of geofences to arm (e.g. 60–80).

**Done when:**
- I can see logs showing state changing to DRIVING when I start moving.
- Geofences are registered/unregistered correctly in logs.
