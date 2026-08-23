# Platform: Web

- **Repository:** https://github.com/mgcather07/Web_Sector.git
- **Local path:** `/Users/mcather/Desktop/Development/Web/Sector-Web`
- **Product role:** Web app; **serves both customers and administrators/master**
  (marketing landing when signed out, dashboard + full features when signed in).
- **Source-of-truth responsibility:** Source of truth for the **current Web
  implementation only** — not for shared product intent.
- **Current status:** Existing and **mature** (~8,900 LOC pages + ~3,800 LOC lib, 33
  routes).

> The root `README.md` calls this an "admin redzone tool." That framing is **stale** —
> the app is a full multi-role customer + admin web app. Trust the code.

## Technology (confirmed 2026-08-22)

- **Next.js 14.2** (App Router; every page `"use client"`), **React 18.3**,
  **TypeScript 5.5**, **Tailwind 3.4**.
- **firebase v11** — Auth, Realtime Database, Storage, Analytics (GA4).
- **Mapbox GL 3.8** + Mapbox GL Draw + Mapbox Geocoding (NOT Google Maps / Leaflet).
- **Static export** (`output: "export"` → `out/`), deployed to **Firebase Hosting** with
  `prod` + `dev` targets. `xlsx` for spreadsheet export.
- State: React Context for auth (`src/lib/useAuth.tsx`) + `onValue` RTDB hooks in
  `src/lib/data.ts`. No Redux/Zustand.

## Architecture

- App Router under `src/app/`; root layout wraps `AuthProvider` + `ShellFrame` +
  `AnalyticsTracker`. `BARE_ROUTES` (`/login`, `/signup`, `/live`) render chromeless.
- Domain/data layer in `src/lib/` (`firebase.ts`, `data.ts`, `redzones.ts`,
  `registration.ts`, `social.ts`, `weighIn.ts`, `teams.ts`, `trophy.ts`, `owner.ts`,
  `roles.ts`, `conditions.ts`, `types.ts`, `states.ts`).
- Components under `src/components/` (`site/`, `tournament/`, `weigh-in/`, `social/`,
  `owner/`, plus `PublicMap.tsx`, `cards.tsx`, `AddressSearch.tsx`).

## Roles & feature coverage (confirmed)

- **Roles:** customer/member, premium member (creation tools), owner (guide/store
  managers), **admin/master** (`isAdmin = role=="admin" || auth=="master"`;
  `isMaster = auth=="master"`).
- **Implemented:** tournaments (browse/detail/create), redzone map + **creation/approval**
  (Mapbox GL Draw; `/redzones`, `/redzone-requests`, `/review`), stores, guides + bookings,
  regulations (read-only), trophy feed + social, teams, live weigh-in + results, admin
  (`/admin`, `/admin/users`), premium (display/unlock only), auth.
- **Partial/indirect:** personal markers (type + `sharedMarkers` node referenced; no
  confirmed dedicated marker-editor page).
- **Auth providers:** email/password, Google, **Apple** (full, popup-based).

## Capabilities & constraints

- **No web payments** — subscriptions are mobile-store only; Web *unlocks* premium via
  `entitlements/{uid}` or `users/{uid}/plan`. `/premium` links to App Store / Play.
- **Background geofencing is not a reliable Web capability** — treat as `excluded` for the
  geofence feature contract.
- Ships its **own** `database.rules.json` (deployed prod+dev) and a `functions/` codebase.
  This rules copy has **drifted** from the iOS copy (conflict C4).
- Cloud Functions here: `appStoreNotifications`, `playNotifications`,
  `notifyRedzoneSubmissionDev/Prod`, `adminListUsers`, `clearPastTournamentRedzoneLinks`,
  `syncBaaRegistrations`, and the `conditions` engine (`WhereToLookEngine`).

## Build & test (confirmed)

- `npm run dev`, `next build`, `build:dev` / `build:prod`, `deploy:dev` / `deploy:prod`,
  `next lint`.
- **No automated test runner** (no Jest/Vitest/Playwright). Confidence: high.

## Distribution / deployment

- Firebase Hosting: prod `www.sectorbowfishing.com` / `sector-9393c.web.app`; dev
  `sector-9393c-dev.web.app`. Project `sector-9393c`; dev RTDB `sector-dev-1f4d0`, prod
  `sector-9393c-default-rtdb`.

## Ownership

- Maintained through **approved development tasks**. Michael reviews.

## Open questions

- Does Web serve customers, admins, or both? **Answer (confirmed): both.** (Q15)
- Should Web write a `platform` value / participate in presence (currently `platform ∈
  {ios,android}`)?
- Should Web own the RTDB rules + Cloud Functions, given it deploys them? (Q19 / C4)
- Which iOS features are intentionally Web-excluded (background geofencing, personal
  marker capture, etc.) vs adapted?
- Refresh the stale `README.md`.
