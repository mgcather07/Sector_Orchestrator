# Unresolved Conflicts

Contradictions found during the read-only bootstrap that should be resolved rather than
silently decided. Each records the conflicting evidence, which source currently has
precedence, a likely explanation, the risk, a recommended resolution, and whether
Michael's decision is required.

---

## C1 — Android background geofencing: docs say done, manifest says dormant

- **Conflicting evidence:** `docs/iOS-PARITY-ROADMAP.md` marks Notifications "100% … background
  GeofencingClient"; `docs/PARITY-AUDIT.md` "Shipped this pass" claims background geofencing
  implemented with `ACCESS_BACKGROUND_LOCATION` + receiver. **Current source:** `GeofenceManager`
  exists, but `AndroidManifest.xml` does **not** declare `ACCESS_BACKGROUND_LOCATION`, and
  `GeofenceManager.refresh()` no-ops without it → **foreground-only** alerts.
- **Precedence:** Current source/manifest (primary) over the audit/roadmap (secondary/tertiary).
- **Likely explanation:** The permission/receiver was reverted or never merged after the audit
  wrote its "shipped" note; or Play Store background-location policy blocked it.
- **Risk:** Redzone-entry alerts — the feature's core purpose — do not fire when the app is
  backgrounded on Android, while iOS does. Users miss zone alerts.
- **Recommended resolution:** Bounded verification task — confirm manifest state on current source,
  then either (a) declare the permission + wire the receiver + verify on device (with Play Store
  justification), or (b) record foreground-only as an accepted `adapted` scope with rationale.
- **Michael's decision required:** **Yes** (background-location has Play policy + privacy implications).

## C2 — Trips "local-only" rule vs cloud sync code

- **Conflicting evidence:** Spec `trips.md` and the domain rule say trips are **local-only** (no
  cloud sync/photo upload). The 2026-07-15 audit found `UserDataSyncService.restoreTrips()` (and a
  discovered `pushTrips()`) reading/writing `users/{uid}/trips`. The audit's Phase-log then claims
  both were **removed**.
- **Precedence:** Current source (must be confirmed) over the audit note.
- **Likely explanation:** Trip cloud I/O was added, flagged as a rule violation, then removed —
  needs confirmation it is actually gone in current source.
- **Risk:** Trip data (and photos) leaving the device violates the stated privacy rule; iOS
  reverted this pre-ship due to data-loss bugs.
- **Recommended resolution:** Grep current `sync/UserDataSyncService.kt` for any `trips` read/write;
  confirm removed. If present, remove under an authorized task.
- **Michael's decision required:** Only if reintroducing cloud trips is ever desired (default: keep local-only).

## C3 — No-name fallback: "Member" (iOS) vs "Bowfisher" (Android docs)

- **Conflicting evidence:** iOS `specs/INVENTORY.md` (2026-07-19) sets the no-name display fallback
  to **"Member"**. Android `PARITY-AUDIT.md` and `utils/Names.kt` map blanks to **"Bowfisher"**.
- **Precedence:** Current iOS behavior (the fallback string is a product rule iOS owns).
- **Likely explanation:** iOS changed the fallback after Android implemented "Bowfisher"; Android
  hasn't caught up.
- **Risk:** Cosmetic but user-visible inconsistency across platforms; both are "correct per their
  era," so it is easy to miss.
- **Recommended resolution:** Update Android's `bowfisherName()` fallback to "Member" (small,
  bounded). Confirm Web's fallback too.
- **Michael's decision required:** Confirm "Member" is final, then it's a mechanical fix.

## C4 — RTDB rules drift between the iOS copy and the Web copy

- **Conflicting evidence:** `iOS_Sector/database.rules.json` and `Web_Sector/database.rules.json`
  both define 58 nodes but differ: Web adds **`playAccountTokens`** and **drops top-level
  `offerCodes`**; Web's `entitlements` write also allows `users/{uid}/plan === 'premium'` (iOS
  does not); Web's `registeredTeams` index adds **`seriesTeamId`**.
- **Precedence:** Undetermined — **both repos deploy rules**, so whichever deploys last wins in the
  live database. This is a governance gap, not a clear winner.
- **Likely explanation:** Rules were edited independently in two repos for platform-specific needs
  (Play tokens on Web/Android path; series-team indexing) without a single owner.
- **Risk:** **High.** A deploy from either repo can silently overwrite the other's rules, breaking
  premium writes, Play token claims, or series-team queries in production.
- **Recommended resolution:** Designate **one** repo as the rules owner (open question Q19), unify
  the two copies into that source, and make the other repo's copy read-only/generated. Until then,
  treat any rules deploy as a coordinated, approval-gated change.
- **Michael's decision required:** **Yes** (which repo owns rules + Cloud Functions).
- **RESOLVED (2026-08-23) — rules half.** Michael chose **iOS as the single owner of RTDB
  rules** (see [ADR 0002](../decisions/0002-ios-owns-rtdb-rules.md)). A precise diff that
  day refined the evidence: `entitlements` was in fact **identical**; the real drift was
  Web-only `playAccountTokens`, iOS-only `offerCodes`, `redzoneDrafts` `.write`
  (Web allowed `plan==='premium'`), and `registeredTeams` `.indexOn` (Web added
  `seriesTeamId`). iOS `database.rules.json` was reconciled into a **strict superset**
  (iOS PR #189) containing all of the above, so an iOS deploy can no longer delete a
  Web/Android node. **Still open:** make Web's copy a mirror (or drop it from Web's
  deploy) so a Web deploy can't reintroduce drift, and decide **Cloud Functions**
  ownership (Q19).

## C5 — Android parity docs are stale relative to current source

- **Conflicting evidence:** `PARITY-AUDIT.md` (2026-07-15) lists Chum Bucket, redzone submission +
  admin approval, and `notifyNewRedzone` as **deferred/missing**. Current Android source contains
  `tournament/live/ChumBucket.kt`, `ChumDrawScreen.kt`, `redzones/RedzoneSubmitScreen.kt`,
  `RedzoneDraftService.kt`. iOS handoffs run to 2026-08-06, past the audit date.
- **Precedence:** Current source over the audit.
- **Likely explanation:** Normal doc lag — work shipped after the audit was written.
- **Risk:** Planning off the stale audit under-counts Android maturity and wastes effort re-doing
  done work.
- **Recommended resolution:** Treat `PARITY-AUDIT.md` as historical; verify each "deferred" item
  against current source before scheduling. This orchestrator's
  [`android-parity.md`](android-parity.md) already annotates these as post-audit additions
  needing confirmation.
- **Michael's decision required:** No (process note); just verify before acting.

## C6 — Sponsor model present but apparently unused (iOS)

- **Conflicting evidence:** iOS `Sponsor.swift` and RTDB `sponsors/{id}` exist; `INVENTORY.md`
  flags it as possibly unused in tournament UI. Android added a `Sponsor` model to resolve
  `Redzone.sponsorId`.
- **Precedence:** Inconclusive — needs a code trace of where sponsors render on iOS.
- **Risk:** Low. Android may have built UI for a concept iOS defines but doesn't surface.
- **Recommended resolution:** Trace iOS sponsor usage; record intended behavior in the domain model
  and a redzone feature contract.
- **Michael's decision required:** Clarify intended sponsor UX when the redzone contract is promoted.

---

_Add new conflicts here as they are found. Do not resolve a conflict by editing away one
side of the evidence — record the resolution and its date instead._
