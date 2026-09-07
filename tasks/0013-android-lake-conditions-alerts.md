```yaml
task_id: 0013-android-lake-conditions-alerts
parent_feature: lake-conditions-alerts
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/SwiftData/Models/LakeAlerts/* (LakeAlertsScheduler, LakeAlertsEngine, LakeDigestEngine, LakeAlertConfig); Sector/UI/Settings/Settings.swift; Sector/UI/Tab Bar/TabBar.swift (deep-link)
status: proposed
deployment_authority: none
review_requirement: Michael approves scope, then the branch/PR
```

# 0013 — Android Lake Conditions Alerts (nightly retention loop)

## Scope (PROPOSED — needs a scope decision first)

iOS ships the full **Lake Conditions Alerts** retention loop; Android has **nothing**
(no WorkManager worker, no engine/prefs/log). This task is to bring it to Android **or**
formally declare it iOS-only for now.

iOS behavior to mirror if built: a nightly fire window (iOS uses `BGTaskScheduler` +
an on-open fallback), a `LakeAlertsEngine` that decides which saved lake(s) to notify on
using tonight's scores, quiet-hours handling, a weekly digest, a Settings toggle, and a
notification that deep-links into the lake. Scores come from the shared engine (already on
Android), so the notification/scheduling layer is the new work, not scoring.

**Michael's scope decision needed:** build it on Android now (WorkManager nightly worker
mirroring the trigger rules), or mark `lake-conditions-alerts` Android `deferred`/
`excluded` with a rationale. No implementation until this is decided.

## Out of scope

- The conditions engine / scoring (shared, already on Android).
- iOS behavior (reference only).
- Any RTDB change unless the alert state needs server persistence (iOS keeps it
  device-local; confirm before adding any node).

## Contract references

- [`../parity/ios-delta-since-2026-08-23.md`](../parity/ios-delta-since-2026-08-23.md)
- 2026-09-07 cross-platform audit: `iOS_Sector/docs/audits/2026-09-07-ios-android-parity.md`
  (HIGH finding — Lake Conditions Alerts iOS-only).
- Group F `lake-conditions-alerts` in [`../parity/android-parity.md`](../parity/android-parity.md)
  (which recorded it "absent on both" — **now stale**: iOS shipped it).

## Dependencies

None blocking. Android already scores off the shared engine, so only the
scheduling/notification/prefs layer is new.

## Acceptance criteria (if built)

- A nightly Android worker evaluates saved lakes in the same window and with the same
  trigger rules as iOS, respecting quiet hours.
- A user-facing Settings toggle gates it; a fired notification deep-links to the lake.
- Weekly digest parity.
- Behavior converges with iOS on the same inputs (scores are already shared).

## Verification method

On-device: enable the toggle, force the worker, confirm the notification fires in-window,
respects quiet hours, and deep-links. Read-only/spec pass ⇒ `documented_only` until built.

## Completion record

_(empty — proposed; awaiting Michael's build-vs-iOS-only scope decision. Filed 2026-09-07
from the cross-platform parity audit.)_
