# iOS → Android Catch-Up Delta (since 2026-08-23 audit)

**Date compiled:** 2026-09-07 · **Method:** read-only inspection of current iOS
source + trunk git history (`Michael-Master`, PRs #191–#204). Companion to
[`android-parity.md`](android-parity.md), which audited both platforms as of
**2026-08-23**. This file records **only what iOS shipped after that audit** — the
work Android must absorb to stay caught up. Everything below is `implemented` on
iOS (verified by build + on-device this session) and `missing`/`not_evaluated` on
Android unless noted.

> **Zero Realtime Database impact.** Every item here is UI, local device state
> (`UserDefaults`), or an in-memory client cache. No RTDB path, field, rule, or
> Cloud Function changed — so no migration plan is required and the shared data
> contract is untouched. Android work is client-only.

## Executive summary

The post-audit iOS work clusters into **one architecturally important fix** and a
**run of conditions/UI polish**. The important one is the **cross-surface score
consistency fix** (the "Cory bug"): the same lake/coordinate could show three
different tonight-scores on three screens (home gauge 80 / score tab 78 / map pin
62) and disagree between two phones for 5–10 minutes. iOS fixed it with a shared
in-memory per-coordinate cache (`ConditionsMemo`, 10-min TTL) that every score
surface now reads through, plus matching TTLs on the My-Lakes scorer and the
redzone map pin. **Android has the same multiple-score-surface shape and the same
latent divergence risk** — this is the highest-value catch-up item, and it is
purely client-side (both platforms already score off the one shared Cloud Run
Sector_Engine, so the *engine* numbers agree; the divergence was local caching and
rounding, which Android must dedupe the same way).

The rest is conditions-experience parity: **favorite conditions** (star a metric on
its detail sheet → it pins under the home Tonight "Conditions" section), the **wind
detail chart rework** (sustained + gust rendered flush with a drag-to-scrub readout
that shows both values at the now-line), a **fog-aware night banner + honest paywall
CTA**, **My Lakes preloading its scores at app launch** instead of on first open,
and a batch of **cross-screen conditions audit fixes** (graph ranges, copy, colors).
Two items are **iOS-only / excluded for Android**: the iPad star-driven condition
tile grid + larger metric sheet, and the "Built by Rehtac" splash credit (iOS
branding; Android may add its own on its own splash).

## Delta table

| iOS PR(s) | Change | Android status | RTDB | Recommended Android scope | Task |
|---|---|---|---|---|---|
| **#199, #200, #194** | **Cross-surface score consistency** — shared in-memory per-coordinate cache (`ConditionsMemo`, 10-min TTL) that every tonight-score surface reads through; matching TTLs on `LakeScorer` (My Lakes) and the redzone map-pin model so all surfaces for one coordinate return the same cached number within the window. Fixes home/score/map-pin disagreement and 5–10-min cross-device drift. | **done on Android — PR [#20](https://github.com/mgcather07/Android_Sector/pull/20), in review** | none | **required** (client-only) | **0008** |
| **#196** | **My Lakes preload at launch** — saved-lake scores warm at app launch (`LakeScorer.warmAll()` from the root view), **decoupled from the slow conditions snapshot** (score comes from a direct engine `/conditions` call; the snapshot enriches after). My Lakes opens already populated instead of fetching on first tap. | **done on Android — PR [#21](https://github.com/mgcather07/Android_Sector/pull/21), in review** | none | **required** (client-only) | **0009** |
| **#195, #198 (phone part)** | **Favorite conditions** — a star on each metric's detail sheet pins that metric under the home Tonight "Conditions" section, in star order. Per-device UI preference (local only), survives launches, drops unknown metrics on load. Single shared value source (`ConditionReadout`) so a metric can't show two numbers on two screens. | **done on Android — PR [#22](https://github.com/mgcather07/Android_Sector/pull/22), in review** (star on row, not sheet — adapted) | none (local `UserDefaults`) | **required** (phone); iPad tile grid **excluded** | **0010** |
| **#195** | **Wind detail chart rework** — sustained + gust rendered flush (no black gaps between the two smoothed curves); drag-to-scrub readout shows **both** sustained and gust; the scrub dot sits on the last *observed* point and the dashed forecast run starts at a synthetic "now" boundary so solid=observed / dashed=forecast is honest. | partial (Android wind chart exists; scrub/both-series/now-boundary unverified) | none | **adapted** (match behavior; Compose Canvas, not Swift Charts) | 0011 |
| **#193** | **Fog-aware night banner + honest paywall CTA** — the tonight banner accounts for fog; the premium CTA copy no longer overpromises. | missing | none | **required** | 0011 |
| **#194** | **Cross-screen conditions audit fixes** — water-temp chart honors the 3wk/3mo/Season tabs; graph range/tick fixes; copy de-duplication; color/band corrections so every tonight surface reads from the same band scheme. | partial / unverified | none | **required** | 0011 |
| **#191** | Tournament **roster registration-order numbering + sort**, and **host add-team** (host registers a team for people without a smart device; needs a **unique** id per host-add, not the deterministic `{tid}_{uid}` which would overwrite). | in progress | uses existing `registeredTeams` shape | **required** | **already 0006** (Android) / **0007** (Web) |
| **#192** | iOS **Achievements screen** added. | n/a — **iOS caught up to Android** | none | no action | — |
| **#203, #204** | **"Built by Rehtac" splash credit** + 2.5s splash hold. | iOS branding | none | **adapted** (optional — Android may add its own splash credit) | — |
| **#197, #201, #202** | Release bumps (build 26/27; **v4.1.0 build 28** to App Store). | n/a | none | no action | — |

## Detail — the one that matters most

### Cross-surface score consistency (the "Cory bug") — Android task 0008

**Symptom (reported from TestFlight):** the same marker/lake showed a different
tonight-score depending on which screen you opened — home dashboard gauge vs. the
score tab vs. the map pin — and two phones side-by-side disagreed for 5–10 minutes
before converging.

**Root cause (not the engine):** both platforms already compute scores on the one
shared Cloud Run Sector_Engine, so the *server* numbers agree. The divergence was
**purely client-side** — each surface had its own model, its own fetch, its own
cache lifetime, and its own rounding, so within one device they could hold
different snapshots of the same coordinate, and across devices they refreshed on
different clocks.

**iOS fix (the pattern Android should mirror):**
- A process-wide **`ConditionsMemo`** actor: an in-memory map keyed by coordinate
  rounded to `%.3f,%.3f` (~110 m), storing the raw engine response `Data` with a
  **10-minute TTL** (`ConditionsMemo.ttl = 10*60`). The engine client checks the
  memo before the network and stores every 200 response into it, so **all callers
  for the same ~coordinate within 10 minutes get the identical bytes** → identical
  decoded score → identical rounding.
- The **My Lakes scorer** (`LakeScorer`) and the **redzone map-pin model** were
  given the **same TTL guard** (`loadedAt` timestamp compared against
  `ConditionsMemo.ttl`) so they read through the memo instead of re-fetching and
  re-deriving on their own cadence. `invalidate()` on all three clears together on
  an explicit refresh / account switch.

**Android acceptance:** enumerate every surface that shows a tonight-score
(dashboard compact ring, any score/conditions detail, the redzone/lake map pin,
My Lakes cards, Where-to-go). Introduce one shared per-coordinate cache with a
10-minute TTL and a coordinate-rounding key, route every surface through it, and
verify two surfaces + two devices on the same coordinate agree within the window.
This is client-only; **do not** change the engine or any RTDB path.

## Scope notes / exclusions

- **iPad star-driven condition tiles + larger metric sheet (#198):** iOS large-screen
  layout. Per workflow "Declare a feature iOS-only" and the existing
  iPad-layout precedent, **`excluded` on Android** — Android's phone favorite-conditions
  surface (task 0010) is the parity target, not the iPad tile grid.
- **"Built by Rehtac" splash (#203/#204):** iOS branding/cosmetic; no task. If Michael
  wants it on Android it is a trivial `adapted` addition to the Android splash.

## Verification performed (iOS side)

- Built iOS Release/Debug clean this session; on-device (iPhone 17 Pro sim) verified
  My Lakes populates at launch and the wind chart renders flush with a working scrub.
- Score-consistency fix verified by code path (shared memo + matching TTLs) and
  build; full cross-device convergence depends on engine determinism per coordinate
  (server-side) and was reasoned about, not load-tested here.
- **No Android build/device verification** was run — every Android status above is
  read-only (`missing`/`partial`), to be earned on a device pass after implementation.

## Decision history

- **2026-09-07** — Compiled from iOS PRs #191–#204 on Michael's request to "catch
  Android up to iOS." Recorded as a doc-only delta; proposed Android tasks 0008–0011
  (client-only, zero RTDB). Awaiting Michael's scope approval before any Android code.
- **2026-09-07** — Michael approved 0008–0011. **0008 implemented** on Android
  (`ConditionsMemo` in `EngineApiClient`; `LakeScorer` TTL; pull-to-refresh
  invalidation) → PR [Android_Sector#20](https://github.com/mgcather07/Android_Sector/pull/20),
  compiles, in review (`implemented_unverified`). Inspection also confirmed Android
  scores off the shared engine (platform doc's "divergent" note is stale) and
  `EngineApiClient.batch()` is dead. 0009–0011 remain approved, not started.
- **2026-09-07** — **0009 implemented** on Android (launch `warmAll` from
  `MainActivity`; `LakeScorer.request` two-phase score-then-snapshot; sign-out
  clears the per-user store) → PR [Android_Sector#21](https://github.com/mgcather07/Android_Sector/pull/21),
  stacked on #20, compiles, in review (`implemented_unverified`). 0010–0011 remain
  approved, not started.
- **2026-09-07** — **0010 implemented** on Android (star-driven `FavoriteConditionsStore`
  + star on each Conditions row + pinned strip on the Tonight tab) → PR
  [Android_Sector#22](https://github.com/mgcather07/Android_Sector/pull/22), compiles, in
  review (`implemented_unverified`). Finding: Android's old tile-picker
  (`ConditionsTilesStore`/`ShootingConditionsSection`) is parked/unused — iOS moved off
  that model, so the star model was added and the parked store left untouched.
  Adaptation: star on the metric's row, not inside each sheet. 0011 remains approved,
  not started.
