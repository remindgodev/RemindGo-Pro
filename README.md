# RemindGo-Pro


    participant U as User
    participant A as App
    participant OS as OS (Location/Perms)
    participant AR as Activity Recognition
    participant S as Server (Manifest + Datasets)
    participant G as Geofencing Client
    participant GJ as GeoJSON Engine (JTS/Turf)
    participant N as Notification Manager

    %% 0) First launch – single consent, then standby
    U->>A: Open app
    A->>U: Single consent (Precise+Always, Activity, Notifications, Battery opt-out)
    A->>OS: Request permissions
    OS-->>A: Granted
    A->>S: Manifest + datasets (entry_points + polygons)
    S-->>A: Cached locally
    A->>AR: Start low-power activity listener
    A->>A: Standby (no geofences, no polling)

    %% 1) Driving detected → choose strategy
    AR-->>A: IN_VEHICLE
    A->>OS: getCurrentLocation (coarse, one-shot)
    OS-->>A: Coarse fix
    A->>A: Determine active city + zone strategy

    alt Tier A (Small / compact)
        A->>G: Register boundary/entry geofences (≤cap)
    else Tier B (Large / irregular e.g., ULEZ)
        A->>G: Register BROAD entry-point geofences (120–200m)
    end

    %% 2) Entry-point geofence path (Tier B)
    G-->>A: ENTER at entry-point
    A->>OS: Start brief HIGH_ACCURACY updates (2–5 min max)
    OS-->>A: Precise fix (t1)
    OS-->>A: Precise fix (t2)
    A->>GJ: Segment/Inside check vs polygon_simplified
    GJ-->>A: CROSSING = true/false

    alt CROSSING = true
        A->>N: Notify (Pay • Snooze • Mute + context/landmark)
        N-->>U: Delivered
    else CROSSING = false (near-miss)
        A->>A: Stop precise updates, keep entry geofence armed
    end

    %% 3) Boundary entry path (Tier A)
    G-->>A: ENTER boundary geofence
    A->>N: Notify immediately (compact zone)
    N-->>U: Delivered

    %% 4) End of drive
    AR-->>A: STILL / ON_FOOT
    A->>G: Unregister geofences
    A->>OS: Stop location updates (if any)
    A->>A: Back to Standby

