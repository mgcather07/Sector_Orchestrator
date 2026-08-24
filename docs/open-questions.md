# Open Questions

Unresolved decisions for Michael. Recommended defaults are provided where useful, but a
**recommendation is not a decision** — nothing here is settled until Michael confirms. Some
answers were discovered during the bootstrap and are marked **[answered]** with evidence;
verify before relying on them.

## Repositories & environment

1. **What local paths will the repositories use on the Mac mini?**
   Current (this machine): iOS `Desktop/Development/iOS/Sector`, Android
   `Desktop/Development/Android/SectorAndroid`, Web `Desktop/Development/Web/Sector-Web`,
   Orchestrator `Desktop/Development/Orchestrator/Sector`. _Recommend recording the canonical
   Mac mini paths in [`../stack/inventory.md`](../stack/inventory.md)._

2. **Where are the existing Features & Specs folders?** **[answered]** iOS `specs/` (~57
   files, source) and Android `specs/` (55, a generated mirror via `ops/sync-specs.sh`);
   Android parity docs at `docs/PARITY-AUDIT.md` + `docs/iOS-PARITY-ROADMAP.md`; iOS
   handoffs in `docs/handoff/`. Web has `docs/redzone-approval-contract.md` (no specs dir).

## Android

3. **What is currently implemented in Android?** **[answered, high level]** Substantial
   (~83k LOC, near-parity). See [`../parity/android-parity.md`](../parity/android-parity.md).
   Field-for-field/runtime parity still needs on-device verification.

4. **Which Android technologies and architecture are in use?** **[answered]** Kotlin/Jetpack
   Compose, MVVM + manual DI, Room + Firebase, Google Play Billing, Google Maps. See
   [`../platforms/android.md`](../platforms/android.md).

## Web

5. **What is currently implemented in Web?** **[answered, high level]** Mature multi-role
   app (tournaments, redzones + approval, stores, guides+bookings, regulations, feed,
   teams, live weigh-in, admin/master). See [`../platforms/web.md`](../platforms/web.md).

6. **Which Web framework and architecture are in use?** **[answered]** Next.js 14 App Router,
   TypeScript, Tailwind, Mapbox GL, static export → Firebase Hosting. No test runner.

## Backend / Firebase

7. **Do all platforms use the same Firebase project and database instance?** **[likely]**
   All observed config points to project `sector-9393c` with prod `sector-9393c-default-rtdb`
   and dev `sector-dev-1f4d0`. _Confirm no platform diverges in any environment._

8. **How are dev / staging / prod separated?** **[partial]** Two RTDB instances (prod/dev);
   iOS/Android runtime master-only switch (Debug defaults to dev on iOS); Web `build:dev`/
   `build:prod`. _Is there a distinct staging tier, or just dev + prod?_

9. **Which Firebase Authentication capabilities are used?** **[partial]** Email/password,
   Google, Apple (iOS+Web), guest. _Any email-link, MFA, phone auth, or others configured in
   the console? Not verifiable read-only._

10. **Are Storage, Cloud Messaging, or Cloud Functions used?** **[answered]** Yes to all —
    Storage (image paths), FCM (topics + tokens), ~30 Cloud Functions (iOS `functions/`) +
    a Web `functions/` codebase. _Confirm the full function inventory + which repo owns each._

11. **Which platform owns each kind of database write?** **[partial]** Initial ownership
    mapped in [`../contracts/realtime-database.md`](../contracts/realtime-database.md).
    _Needs a full per-node writer audit._

## Product & monetization

12. **Which subscription provider will Android use?** **[answered]** Google Play Billing
    (`sector.premium.yearly`). _Confirm this is final; no RevenueCat._

13. **Which payment approach will Web use?** **[answered: none]** Web processes no payments;
    premium is unlocked via `entitlements`/`plan`, purchases happen in the mobile stores.
    _Confirm Web will never sell subscriptions directly._

14. **Which features are intentionally mobile-only?** Candidates: background geofencing
    (Web-excluded), personal-marker capture, boat-track recording. _Confirm the list and
    record as `excluded`/`adapted` in the relevant contracts._

15. **Does Web serve customers, administrators, or both?** **[answered: both]** Marketing
    landing + full customer app when signed in, plus admin/master tooling.

16. **What parity level defines the first Android release?** _Undecided. Recommend defining
    a minimum feature set (a scoped contract subset) rather than "all of iOS."_

17. **What parity level defines the first Web release?** _Undecided (Web already runs in
    production for specific events). Recommend clarifying customer vs admin scope per feature._

## Governance

18. **Who approves production deployments?** _Assumed Michael for all (App Store, Play,
    Firebase Hosting, RTDB rules, Cloud Functions). Confirm._

19. **Which repository owns Firebase rules and Cloud Functions?** **[answered for rules;
    functions still open]** **Rules:** Michael chose **iOS** as the single owner
    (2026-08-23, [ADR 0002](../decisions/0002-ios-owns-rtdb-rules.md)). iOS
    `database.rules.json` is now the canonical **superset** (iOS PR #189); rules are edited
    in iOS and deployed from iOS. _Remaining for rules:_ make Web's copy a mirror (or drop
    it from Web's deploy) so a Web deploy can't reintroduce drift (conflict C4).
    **Cloud Functions:** still open — iOS `functions/` and Web `functions/` hold *different*
    function sets (iOS: notifications + `redeem`; Web: appstore/play notifications,
    conditions proxy, admin utilities), so this needs its own ownership/mirroring decision.

20. **Which existing parity documents remain current?** **[partial]** `PARITY-AUDIT.md`
    (2026-07-15) is partly stale (source moved past it); `iOS-PARITY-ROADMAP.md`
    (2026-05-29) is largely historical; iOS handoffs to 2026-08-06 are recent. _Treat audits
    as historical; verify against source._

21. **Are there approved cases where Android or Web should intentionally differ from iOS?**
    **[some known]** Google Maps on Android; Mapbox GL on Web; no Apple sign-in on Android;
    no web payments; Web-excluded background geofencing; Android's simplified conditions
    engine. _Confirm each as an approved `adapted`/`excluded` decision in its contract._

## Terminology

22. **Is "Member" the final no-name display fallback?** iOS uses "Member" (2026-07-19);
    Android still uses "Bowfisher" (conflict C3). _Confirm "Member" so Android can be updated._
