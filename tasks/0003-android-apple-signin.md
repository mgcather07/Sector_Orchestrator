---
task_id: 0003-android-apple-signin
parent_feature: auth
authorized_repositories:
  - Android_Sector
platform: android
ios_behavior_reference: iOS_Sector/specs/auth.md (Sign in with Apple required)
status: in_review
deployment_authority: none
review_requirement: Michael approves the branch/PR
severity: MEDIUM
approved: 2026-08-24 by Michael
implemented: 2026-08-24 — Android PR #18 (compiles; BLOCKED on Firebase console Apple-provider config)
---

# 0003 — Android: add Sign in with Apple

## Scope

`auth.md` requires **Sign in with Apple**, and it is **entirely absent** on Android (the
prior parity doc wrongly labeled it excluded-by-design — corrected in the 2026-08-23 audit).
Users who created their account with Apple on iOS cannot sign in on Android.

Work in `Android_Sector`:
- Add Sign in with Apple via Firebase Auth's Apple OAuth provider (`apple.com`) using the
  web-based OAuth flow (Apple has no native Android SDK; Firebase `startActivityForSignInWithProvider`
  is the standard path).
- Ensure the resulting account maps to the same Firebase uid / `users/{uid}` record as iOS
  (so a cross-platform user is one account).
- Match the iOS auth UI (Apple button placement, name/email handling on first sign-in).

**Related (note, may be a separate task):** Android's Google Sign-In uses the **deprecated
GMS `GoogleSignIn` API**. Not part of this task's acceptance, but flag for a follow-up to
migrate to Credential Manager / Google Identity — it has been a historical source of Play
Services error 10.

## Out of scope

- The Google Sign-In migration itself (separate follow-up).
- iOS/Web changes.

## Contract references

- `auth.md` (providers required), `authentication.md` (orchestrator contract).

## Dependencies

- Apple Developer service ID + key already configured for the Firebase project (confirm;
  iOS already uses Apple sign-in so the provider is likely enabled).

## Acceptance criteria

- [ ] Sign in with Apple works on an Android device.
- [ ] An Apple account created on iOS signs into the **same** Firebase uid on Android.
- [ ] First-time name/email handled per Apple's one-time disclosure.

## Verification method

- On-device: sign in with Apple on Android; confirm uid matches the iOS account (same
  `users/{uid}`), profile intact.

## Completion record

- **Implemented + merged:** Android PR #18 (2026-08-24), 2 files, +58.
- **Approach:** Android has no native Apple SDK, so this uses Firebase's **OAuth web flow** —
  `FirebaseAuth.startActivityForSignInWithProvider(activity, OAuthProvider("apple.com"))`,
  resuming any `pendingAuthResult`. Success routes through the SAME `ensureUserRecord` heal +
  `navigateHomeAfterAuth` that email/Google login use → a cross-platform Apple account lands
  on the same `/users/{uid}`.
- **Files:** `login/AuthComponents.kt` (`AppleButton`), `login/Login.kt` (`completeAppleSignIn`
  + `startAppleSignIn` + button under Google).
- **Verification done:** `compileDebugKotlin` passes.
- **⚠️ BLOCKED on backend config (Michael) — the reason it's not `done`:** the Apple provider
  must be enabled in the **Firebase console** with an Apple **Services ID + private key +
  return URL** (`https://sector-9393c.firebaseapp.com/__/auth/handler`). iOS uses the NATIVE
  Apple credential path, which does **not** require this web-provider config — so it may be
  unconfigured. Until it's set up, the button starts the flow but Firebase rejects it with
  `operation-not-allowed`. Steps: Firebase console → Authentication → Sign-in method → Apple →
  enable + fill Services ID/key/redirect; Apple Developer → the Services ID's Return URLs must
  include the handler above.
- **Then verify on-device:** tap Sign in with Apple → Apple web sheet → authorize → lands on
  Home as the same account; a second sign-in reuses the same uid.
- **Follow-up (optional):** the Signup screen only offers email/Google — add `AppleButton`
  there too for symmetry; also migrate Google Sign-In off the deprecated GMS API (noted in
  scope).
