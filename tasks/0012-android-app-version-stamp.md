```yaml
task_id: 0012-android-app-version-stamp
parent_feature: admin-tools (Members / Subscribers screen)
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: Sector/UI/UserAuth/CurrentUser.swift (writes users/{uid}/appVersion + appBuild); Sector/Utils/SubscribersView.swift (reads them)
status: in_review
deployment_authority: none
review_requirement: Michael approves the branch/PR
```

# 0012 — Android writes appVersion + appBuild (iOS Members screen)

## Scope

On each sign-in, Android writes `users/{uid}/appVersion` and `users/{uid}/appBuild`
so the iOS master **Members / Subscribers** admin screen shows which build an Android
account is on (it already shows the Android platform glyph). iOS's `CurrentUser` stamps
both on sign-in and `SubscribersView` renders `v{appVersion} ({appBuild})`; its code
comment states the field names are a **shared iOS/Android contract** and "Android writes
the same paths." Android previously wrote only `platform`.

Both are written as **strings** (iOS reads `u["appBuild"] as? String`): `appVersion` =
`versionName`, `appBuild` = `versionCode`.

## Out of scope

- Any new RTDB field, rule, or Function — `appVersion`/`appBuild` already exist in the
  iOS-owned contract; this only populates them from Android.
- The iOS Members/Subscribers screen itself (reference only; iOS not modified).

## Contract references

- [`../contracts/realtime-database.md`](../contracts/realtime-database.md) — `users/{uid}`
  node (existing `appVersion`/`appBuild` fields; no change).
- Group H `admin-tools` in [`../parity/android-parity.md`](../parity/android-parity.md).

## Dependencies

None. Client-only. Discovered outside the 0008–0011 delta (from Michael asking why
Android builds don't show on his iOS phone).

## Acceptance criteria

- After an Android sign-in, `users/{uid}/appVersion` = the app's `versionName` and
  `users/{uid}/appBuild` = its `versionCode`, both strings.
- The iOS Members/Subscribers screen shows the Android account's version+build instead of
  blank.
- No other `users/{uid}` field is altered; `platform` still written.

## Verification method

`./gradlew compileDebugKotlin` (done — clean). On-device: sign in on Android, open the
iOS master Members/Subscribers screen, confirm the Android row shows `v<version>
(<build>)`. Read-only pass ⇒ `implemented_unverified`.

## Completion record

**2026-09-07 — implemented, in review.** Branch `feat/0012-android-appversion` off
`Michael-Master` → **PR [Android_Sector#24](https://github.com/mgcather07/Android_Sector/pull/24)**.
One block in `MainActivity` next to the existing `platform` write, reading the build via
`PackageManager` (this module doesn't generate `BuildConfig`), wrapped in `runCatching`
on the login path. Compiles clean (exit 0). Client-only; no schema/RTDB-rule/deployment
change. `implemented_unverified` pending the on-device Members-screen check. Not merged.
