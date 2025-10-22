# RemindGo-Pro

    participant U as User
    participant A as RemindGo Pro (App)
    participant OS as OS Location Services
    participant AR as Activity Recognition
    participant S as Server (Manifest + GeoJSON)
    participant G as Geofencing Client
    participant N as Notification Manager

    %% 0) Install → First Launch
    U->>A: Open app (first launch)
    A->>U: Value pitch (why) + minimal perms (Approximate/While Using)
    A->>S: Fetch manifest + latest city datasets (cache)
    S-->>A: Versions + checksums + GeoJSON
    A->>A: Cache last 2 versions (fallback)

    %% 1) "Arm me" moment
    U->>A: Tap "Enable driving reminders"
    A->>U: Explain need for Always+Precise (background)
    A->>OS: Request background + precise location
    OS-->>A: Granted / Denied

    alt Granted
        A->>U: Offer battery optimization exclusion deep link
    else Denied
        A->>U: Suggest Foreground Session Mode for trips
    end

    %% 2) Region detect → Dynamic geofences
    loop On move (X km) or app start/reboot
        A->>OS: Coarse location (balanced power)
        A->>A: Determine active region (city/country)
        A->>S: Get nearest N entry points (precomputed)
        S-->>A: Entry points (lat,lng,radius)
        A->>G: Register geofences (cap-aware: Android ≤100 / iOS ≤20)
        G-->>A: OK / TOO_MANY_GEOFENCES
        opt Cap or error
            A->>A: Trim to nearest subset + retry
        end
    end

    %% 3) In-vehicle gate
    AR-->>A: State update (IN_VEHICLE) / speed > 15 km/h
    A->>A: Refresh geofences if stale/moved far

    %% 4) Boundary entry → Notification
    G-->>A: ENTER transition (zoneId, entry point)
    A->>A: Debounce (1 per zone/day; max 3/day)
    A->>N: Show notification (Pay now • Snooze • Mute)
    N-->>U: Charge/LEZ reminder (with context/landmark)

    %% 5) Fallback for missed triggers
    OS-->>A: Significant location change (Doze wake)
    A->>A: Quick polygon/edge check (are we inside?)
    alt Inside without prior enter
        A->>N: "You're in the zone" catch-up notification
    else Outside
        A->>A: No-op
    end

    %% 6) Foreground Session Mode (if background denied)
    U->>A: Start Driving Session (foreground service)
    loop While moving (30–60s)
        A->>OS: Get location (foreground)
        A->>A: Point-in-polygon / edge proximity check
        alt Entry detected
            A->>N: Heads-up alert with CTAs
        end
    end
    A->>A: Auto-pause after inactivity

    %% 7) Data/version hygiene
    A->>S: Check for dataset updates (manifest)
    S-->>A: New version? (checksum)
    alt New version
        A->>A: Swap to new; keep previous for rollback
    else No change
        A->>A: Continue
    end

    %% 8) Recovery paths
    A->>A: On reboot/app update → reconcile intended vs active geofences
    A->>G: Re-register missing geofences
