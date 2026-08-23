# Contract: Subscriptions & Entitlements

**Status:** Draft. Based on read-only inspection on 2026-08-22.

Sector Premium is a single paid tier unlocked per-user. The **entitlement is the
product concept**; the *purchase channel* differs per platform.

## Current iOS behavior (reference)

- **StoreKit 2, used directly** (`Sector/Store Kit/StoreKitManager.swift`). **RevenueCat
  is not used** (the root `RevenueCat` file is an empty placeholder; it was evaluated and
  rejected).
- **Product IDs:** primary `sector.premium.yearly` (~$9.99/yr); `premiumGrantingProductIDs
  = {sector.premium.yearly, sector.premium.monthly}`; legacy `sector.standard.monthly`/
  `.yearly` also honored as premium. Config `StoreKitConnect.storekit`.
- **Entitlement check:** reads `Transaction.currentEntitlements`. Local plan is only ever
  **upgraded** to `.premium`; it is **never written back to `.free`** (avoids downgrading
  a paying user on a transient/offline read).
- **Sync:** premium truth = StoreKit, mirrored **upgrade-only** to `users/{uid}/plan`
  and, server-side, to `entitlements/{uid}` (+ `appAccountTokens`) via the
  `appStoreNotifications` Cloud Function (App Store Server Notifications webhook).

## Confirmed shared behavior

- **`entitlements/{uid}`** (server-written, master-only, indexed `active`) is the
  canonical entitlement record. `users/{uid}/plan` is a **client-written UI-unlock
  mirror**, explicitly noted on Web as *not* a security boundary.
- **`auth == "master"`** always counts as premium on every platform.
- Premium unlocks (shared intent): tournament/guide/store creation & hosting, redzone
  zone-type + tournament filters, 50-mi location filters, favorites, unlimited markers
  (25→∞) + more colors (3→6), guide/store management, conditions detail sheets.

## Platform differences

| Aspect | iOS | Android | Web |
|---|---|---|---|
| Purchase channel | **StoreKit 2** (direct) | **Google Play Billing 7.1.1** | **None — no web checkout** |
| Product id | `sector.premium.yearly` (+ monthly, legacy standard) | `sector.premium.yearly` | n/a |
| Server webhook → `entitlements` | `appStoreNotifications` | `playNotifications` | (consumes both) |
| `plan` write on purchase | upgrade-only | upgrade-only (`plan=premium`, `dateUpdated`) | never writes plan |
| Entitlement gate | `plan == .premium` (StoreKit truth) | `computeHasPremium` (Play OR master OR plan) | `entitlements.active` OR `plan==='premium'` |
| Payment processing | Apple | Google | **none** — links to App Store / Play |

- **Web sells nothing.** `/premium` markets perks and links to the App Store
  (`id6473187409`) and Play (`io.sector.co`); it only *unlocks* based on the entitlement.
- Entry fees / bookings never process payment in-app on any platform (collected in person).

## Assumptions (to verify)

- Exact set of legacy/active product IDs recognized on Android vs iOS.
- Whether `playAccountTokens` (present in the Web rules copy, absent in the iOS copy) is
  the Android analogue of `appAccountTokens` and how it is populated — see
  [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md).

## Change-control expectations

- Product IDs, tiers, and the entitlement gate are **cross-platform product decisions**.
- Any change to `entitlements/{uid}` or `users/{uid}/plan` shape is an **RTDB contract
  change** with a security-rule review (the Web rules copy grants writes to premium via
  `plan==='premium'`; the iOS copy does not — reconcile before changing).
- Never introduce a web payment processor (Stripe, etc.) without an approved decision.

## Open questions

- Should Android use Play Billing exclusively, or is another provider under
  consideration? ([`../docs/open-questions.md`](../docs/open-questions.md) Q12.)
- What payment approach, if any, will Web ever use? (Q13 — current answer: none.)
- Is `plan` ever meant to be authoritative, or always subordinate to `entitlements`?
