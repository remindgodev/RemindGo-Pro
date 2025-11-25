2. How to ask me for help (so I can be precise)

When you message me about the app, use this pattern:

Start with a tag:

[PLAN] big-picture / architecture / roadmap

[CODE] “help me write this feature”

[DEBUG] “this crashes / doesn’t work”

[REFACTOR] “make this logic clearer / extract stuff”

[DOC] “help me update docs / comments”

Give minimal context:

Platform: Android / Kotlin

Feature or file: e.g. GeofenceManager.kt or DrivingModeState

Current behaviour vs what you want.

Example:

[CODE] Android/Kotlin. I’m building the permission screen.
Current: single MainActivity with a button.
Goal: step-by-step flow asking for Activity Recognition, Location, then Notifications in the order we decided.

Paste only the relevant code (not the whole project):

The class or function that’s the problem.

Any error message text.

If needed, also your manifest or Gradle snippet.

One main topic per message
Don’t mix “fix this crash” + “also design future iOS architecture” in one go.
Instead:

First: debug the crash.

Later: new message for architecture.

This makes it way easier for me to give you crisp, correct answers.
