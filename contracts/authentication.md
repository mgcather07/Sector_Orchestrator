# Contract: Authentication

**Status:** Draft. Based on read-only inspection of iOS/Android/Web on 2026-08-22.

Sector uses **Firebase Authentication** across all platforms, backed by the shared
`sector-9393c` project, so a user has one identity everywhere.

## Current iOS behavior (reference)

Handled in `Sector/UI/UserAuth/` (`Login.swift`, `SignUp.swift`, `UserService.swift`,
`CurrentUser.swift`). Providers:

- **Email/password** — `Auth.auth().signIn(withEmail:)`, `createUser(withEmail:)`.
- **Google Sign-In** — `GIDSignIn` credential → Firebase.
- **Apple Sign-In** — `ASAuthorizationAppleIDCredential` + `OAuthProvider`, nonce via
  CryptoKit SHA-256.
- **Guest / skip-login** — persisted app-locally; browse-only, cannot save. **Not**
  Firebase anonymous auth.

Lifecycle: forgot-password (reset email); logout clears Firebase + Google + UserDefaults
+ FCM topics; delete-account ordered **Auth → RTDB → local**. `CurrentUser` keeps a live
`users/{uid}` listener for role/capability sync (a live listener is the iOS pattern).

## Confirmed shared behavior

- Firebase Auth, same accounts across platforms.
- **Email/password** and **Google** on all three platforms.
- On sign-in, profile is provisioned/backfilled in `users/{uid}` and mirrored to
  `publicUsers/{uid}` (server function `mirrorPublicUser`). Blank-field self-heal only.
- Delete-account mirrors the Auth-first ordering on iOS and Web.
- Roles derive from `users/{uid}`: `auth ∈ {master, admin}`, plus capability flags
  `isGuide`/`isStoreOwner`/`isHostVerified`. **`auth == "master"`** is the single
  master/admin identity (documented as Michael's account).

## Platform differences

| Provider / behavior | iOS | Android | Web |
|---|---|---|---|
| Email/password | ✅ | ✅ | ✅ |
| Google | ✅ | ✅ | ✅ (popup) |
| **Apple** | ✅ | **N/A (intentional)** | ✅ (popup) |
| Guest / skip | ✅ | ✅ | ✅ (signed-out browse; `/` marketing landing) |
| Live role listener on `users/{uid}` | ✅ | ✅ (added post-audit; earlier was one-shot) | ✅ |

- **Apple Sign-In on Android is a deliberate non-gap**, not a missing feature (Android
  has no Apple provider by design). Record any equivalent requirement as `excluded` with
  this rationale in the relevant feature contract.
- Web additionally writes `users/{uid}/lastLoginAt` for the admin user list.

## Assumptions (to verify)

- That Web account creation writes the same `users/{uid}` shape iOS expects (fields,
  `platform` value — Web may not set `platform`, which is validated `∈ {ios,android}`).
- That guest→login marker/data-save prompts behave equivalently where markers exist.

## Change-control expectations

- Adding/removing an auth provider is a **cross-platform product decision** — record it
  as a feature-contract scope change, not an incidental edit.
- Changes to the `users/{uid}` shape, role flags, or `auth` semantics are **RTDB
  contract changes** (see [`realtime-database.md`](realtime-database.md)) and must go
  through change control, including a security-rule review (roles are enforced in rules).
- Never store credentials or tokens in this repo or any platform change.

## Open questions

- Does Web set a `platform` value, and should `users` validation admit a `web` value? (Q)
- Which Firebase Auth capabilities beyond email/Google/Apple are configured in the
  console (e.g. email-link, MFA)? Not verifiable read-only — see
  [`../docs/open-questions.md`](../docs/open-questions.md) Q9.
