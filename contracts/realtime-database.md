# Contract: Firebase Realtime Database

**Status:** Draft (initial audit). Becomes canonical only after full field-level
verification against implementation and Michael's approval.
**Backend:** Firebase Realtime Database (RTDB). **Firestore is not used** and must not
be introduced without an approved architectural decision.

Firebase RTDB is the **shared** database for all Sector clients. Every platform reads
and writes the **same paths with the same field names** so the same server-side Cloud
Functions fire regardless of client. This is the single highest-risk shared contract
in the system.

## Instances & environments (confirmed)

- **Project:** `sector-9393c`.
- **Prod instance:** `sector-9393c-default-rtdb`.
- **Dev instance:** `sector-dev-1f4d0`.
- **Switching:** iOS + Android route all access through a `DatabaseProvider` with a
  master-only runtime dev/prod toggle (Debug builds default to dev on iOS; Android has
  a documented env policy). Web selects via `build:dev`/`build:prod`.
- **Persistence:** iOS and Android enable RTDB disk persistence; live updates via
  `child` observers (iOS `FirebaseListener`) / value listeners.
- **Rules:** Checked into **both** iOS (`database.rules.json`) and Web
  (`database.rules.json`) and deployed from there. The two copies have **drifted** —
  see [`../parity/unresolved-conflicts.md`](../parity/unresolved-conflicts.md). Android
  ships no rules copy.
- **Related but separate:** BAA records use a **different** Firebase project
  (`baa-website-908e4`) over REST — not part of this contract.

## Top-level nodes (confirmed from `database.rules.json`)

58 top-level nodes. Root is locked (`.read/.write: false`); each node opens
explicitly. Grouped by domain. Field lists below are **partial** and drawn from rules
+ code evidence; treat un-cited fields as _assumed_ until a full audit.

### Tournaments & competition
| Node | Shape | Notes / access |
|---|---|---|
| `tournaments/{tid}` | event record | Public read; owner/co-host/master write. Indexed on `ownerUserId`. `coHostUserIds` is a map `{uid:true}`. Image at Storage `Tournaments/{id}.jpg`. |
| `series/{sid}` | season series | Indexed `ownerUserId`. |
| `tournamentRSVPs/{tid}/{uid}` | `going`\|`interested`\|`notGoing` | |
| `tournamentComments/{tid}/{cid}` | comment | Indexed `createdAt`; one-level threading. |
| `tournamentCommentLikes/{tid}/{cid}` | likes | |
| `commentReports/{cid}/{uid}`, `postReports/{pid}/{uid}` | moderation reports | |
| `registeredTeams/{teamId}` | team registration | Indexed `tournamentId` (Web copy also `seriesTeamId`). |
| `fish/{fishId}` | bucket weigh-in entry | `teamId`, `tournamentId`, `weightLb`, `count`, `bigFishLb`, `sidePotValues`. Indexed `tournamentId`. |
| `tournamentCheckIns/{tid}` | host roll-call | + public `checkedInCount` mirror. |
| `chumCustomTiles/{tid}` | Chum Bucket tile-draw data | |
| `userSidePots/{ownerId}` | side-pot library (owner-private) | |
| `weighIn`, `weighInPublic/t/{tid}` | admin/host weigh-in surfaces | |

### Redzones / spatial
| Node | Notes |
|---|---|
| `redzones/{rzId}` | Geofenced zones (polygon/line/point). Public read; master or linked-tournament-owner write. |
| `redzoneDrafts/{uid}` | Premium-gated drafts (`entitlements.active` or master; Web copy also allows `plan==='premium'`). Indexed `status`. |
| `redzoneSubmissions/{sid}` | User submissions (submit → master approval → publish to `redzones`). |
| `zones/{id}` | Named geofence zones. |
| `bankSpots/{spotId}` (indexed `state`), `bankSpotReports/{spotId}/{uid}` | Community bank spots. |
| `sharedMarkers/{teamId}/{markerId}` | Team-shared markers (`ownerUid`, `lat`, `lng`). Personal markers are otherwise **local-only** (not synced). Owner-only write, active-member read. |

### Directory content
| Node | Notes |
|---|---|
| `stores/{id}`, `storeReviews/{sid}/{uid}`, `storeDeals/{sid}/{did}`, `storeProducts/{sid}/…` | Store profile/management. Deals/products carry `views`/`clicks`. Reviews indexed `createdAt`. |
| `guides/{gid}` (indexed `ownerUserId`), `guideReviews`, `guideDeals`, `guideTrips`, `availability/{gid}`, `guideAnnouncements`, `guideBookings/{gid}/{bid}`, `userBookings/{uid}` | Guides & booking. Booking status machine pending→confirmed→completed/declined/cancelled. |
| `regulations/{id}`, `states/{id}`, `counties/{id}`, `address/{id}`, `sponsors/{id}`, `formats/{id}` | Reference/lookup content. |

### Users / social / identity
| Node | Notes |
|---|---|
| `users/{uid}` | Private profile. Self-or-master; indexed `auth`. Validates `platform ∈ {ios,android}`; guarded role flags `isGuide`/`isStoreOwner`/`isHostVerified`; `auth` role (`master`/`admin`). Holds `fcmToken`, `plan`, `appMode`, `notificationPrefs`. |
| `publicUsers/{uid}` | Public mirror (world-read, **server-written only** via `mirrorPublicUser`). Contact fields stripped. |
| `follows/{followerUid}`, `followers/{targetUid}/{followerUid}` | Two-sided follow graph. |
| `teams/{teamId}` (+ `members`), `userTeams/{uid}` | Teams. Indexed `ownerUserId`. |
| `roleRequests/{uid}` | Guide/store/host-verification requests → master approval. |
| `presence/{uid}` | Online presence (master-read). Writes `{online,lastSeen,name,imageURL,state,appMode,platform}`; `platform:"android"` from Android. `onDisconnect`. |
| `supportThreads/{uid}/messages` | Support inbox (self-or-master). |
| `activity/{uid}/{notifId}`, `activityMeta/{uid}` | In-app activity feed + read-state. |

### Community feed & records
| Node | Notes |
|---|---|
| `trophyPosts/{pid}` | Community posts. Indexed `createdAt`; paged (`limitToLast`). Nested `likes`, `comments`, `shareCount`. Author server-stamped (anti-impersonation). |
| `recordClaims/{uid}/{recordId}`, `recordClaimsByRecord/{recordId}` | BAA record claims + public mirror + status machine (pending/approved/denied). |

### Subscriptions / billing (server-owned)
| Node | Notes |
|---|---|
| `entitlements/{uid}` | StoreKit/Play-derived entitlement. Indexed `active`. Master-write only (written by Cloud Functions `appStoreNotifications`/`playNotifications`). **Source of truth for premium.** |
| `appAccountTokens/{token}` | Maps StoreKit appAccountToken → uid (self-claimable once). |
| `playAccountTokens/{token}` | Play equivalent — **present in the Web rules copy, absent in the iOS copy** (drift). |
| `offerCodes` | Master-read, no client write. **Present in iOS copy, replaced in Web copy** (drift). |
| `appConfig` | Public read, master write (remote-config-ish). |

See [`subscriptions.md`](subscriptions.md) for the entitlement model.

## Read/write ownership (initial, to verify)

- **Client-writable (auth-gated):** tournaments/series (owner/co-host), redzone drafts
  & submissions (premium), RSVPs, comments/likes/reports, reviews, bookings, teams,
  follows, `users/{uid}` (self), presence (self), support messages, record claims,
  trophy posts, fish/registeredTeams/checkIns (host/team).
- **Server-only writes (Cloud Functions):** `publicUsers` mirror, `entitlements`,
  `activity/*`, author stamps on trophy/comment nodes.
- **Master-only:** `redzones` publish, `offerCodes`, `appConfig`, moderation actions,
  role approvals, subscriber/admin reads.
- **Local-only (not in RTDB):** custom map markers (except `sharedMarkers`), trips,
  boat tracks — stored in SwiftData (iOS) / Room (Android).

## Compatibility expectations

- **Field names and paths must match across platforms exactly.** Never rename a synced
  field; renaming breaks persisted values and cross-client reads.
- Dates: several nodes store **Unix seconds** (e.g. `fish`, series timestamps, support
  messages); some fields use ISO-8601 (`dateUpdated`) and some `ServerValue.TIMESTAMP`
  (announcements). A platform must match the per-field convention iOS uses. _(Full
  per-field date-convention audit pending.)_
- Heavy denormalization and public mirrors (`publicUsers`, `recordClaimsByRecord`,
  public counters) must be kept consistent by whichever writer the contract assigns.

## Security-rule considerations

- Root denied; every node explicitly opened. Premium/master gates encoded in rules
  (`entitlements/{uid}/active`, `users/{uid}/auth === 'master'`, and — in the Web copy
  — `users/{uid}/plan === 'premium'`). `.indexOn` declarations exist for query paths
  (`tournaments.ownerUserId`, `trophyPosts.createdAt`, `bankSpots.state`, etc.).
- **Two divergent rules copies** (iOS vs Web) are a real risk: a deploy from either
  repo can overwrite the other's changes. Resolving ownership is open question Q19.

## Change-control process (required for any RTDB change)

A shared-database change must go through, in order:

1. **Contract update** here (paths/fields/rules).
2. **Impact assessment** across iOS, Android, Web (who reads/writes the path).
3. **Migration plan** when data shape changes — see [`../migrations/TEMPLATE.md`](../migrations/TEMPLATE.md).
4. **Platform implementation tasks** (per authorized repo).
5. **Backward-compatibility review** (old clients keep working during rollout).
6. **Security-rule review** (both rules copies until ownership is unified).
7. **Compatibility verification** (read/write round-trips per platform).
8. **Explicit deployment approval** from Michael before rules/functions ship.

**Do not change Firebase data, rules, or configuration as part of orchestrator work.**

## Open questions

- Full field-level schema per node (types, required, date conventions) — pending audit.
- Which single repo should own and deploy `database.rules.json` and Cloud Functions?
  (Q19) The iOS↔Web drift must be reconciled first.
- Confirm all three clients target the same project/instances in all environments (Q7).
- Exact validation rules per node (many `.validate` blocks were not fully enumerated).
