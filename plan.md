# iosapp MVP Plan (Microdrama iOS App)

## Project Context
`iosapp` is a consumer video app for short-form microdramas, aimed at women ages 30-60 in the US market.

The product objective is simple:
1. Let users start watching immediately with almost no setup.
2. Keep viewers in a smooth binge flow with vertical swipe playback.
3. Convert engaged viewers to paid subscribers through clear subscription gates.

Business and delivery context:
1. Content is produced by a separate internal studio team.
2. The app team focus is distribution, playback quality, and monetization.
3. MVP must avoid overbuilding and use low-complexity tools.
4. Launch target is a fast MVP window with a clear compliance path for Apple review.

## Summary
Build a native iOS app for short vertical microdramas with a simple binge flow and subscription monetization.

Primary goals:
1. Keep complexity low for a sprint-based MVP.
2. Avoid overbuilding.
3. Reduce App Store rejection risk.
4. Enable simple funnel analytics for optimization.

## Product Scope (Locked)
1. Audience: women 30-60, US-only, English-first, 16+ target.
2. Core UX: vertical swipe feed, full-screen playback, auto-advance episodes.
3. Access: guest-first viewing.
4. Gate: first 5 episodes free, then marked locked episodes.
5. Monetization: subscription-only in v1, coins-compatible model for later.
6. Plans: monthly + yearly, no free trial at launch (trial-capable config retained).
7. Auth target: Sign in with Apple at subscribe/restore boundary (Stage 4 target; can defer if core watch -> gate -> subscribe launch flow is at risk).
8. Content ops: Firebase Console only (no custom admin panel in MVP).
9. Out of scope: comments, downloads, live chat, coin economy UI.

## Technical Decisions
1. App stack: SwiftUI + AVFoundation/AVKit.
2. Backend: Firebase (Firestore + Storage URLs + Remote Config).
3. Billing: StoreKit 2 via RevenueCat.
4. Analytics: Firebase Analytics + RevenueCat integration + BigQuery export.
5. No custom backend or custom BI dashboard in MVP.
6. Environments: `dev` + `prod` only for MVP.

## Implementation Architecture + Build Order
Use this section as the main implementation guide.
Reference docs used by this section:
1. `/Users/jonslimak/Projects/iosapp/analytics-spec.md`
2. `/Users/jonslimak/Projects/iosapp/launch-checklist.md`

### Architecture Overview
1. App Shell Layer
- SwiftUI app entry, navigation, app lifecycle, environment setup.
2. Content Layer
- Firebase-backed read models for series/episodes/config.
- Lock rules and free-episode logic live in one shared rules module.
3. Playback Layer
- Vertical swipe feed UI and full-screen AVPlayer experience.
- Auto-advance and lock interception behavior.
4. Monetization Layer
- RevenueCat + StoreKit 2 entitlement checks.
- Purchase, restore, and guest-to-account upgrade boundaries.
5. User and Account Layer
- Anonymous guest default.
- Sign in with Apple on purchase/restore path.
- Account deletion entry point for signed-in users (if account features ship in MVP).
6. Analytics Layer
- Central analytics dispatcher to avoid event drift.
- Events and properties must follow `/Users/jonslimak/Projects/iosapp/analytics-spec.md`.
7. Release and Risk Layer
- Compliance gates, review package, and final checks from `/Users/jonslimak/Projects/iosapp/launch-checklist.md`.

### Build Order (Execution Sequence)
1. Stage 0: DevOps + Service Onboarding
- Create Apple Developer/App Store Connect setup, app records, bundle IDs, signing assets, sandbox accounts, and IAP products for monthly/yearly plans.
- Create Firebase `dev` and `prod` projects, app registrations, baseline rules, IAM setup, and App Check rollout path.
- Create RevenueCat project/app setup, entitlement mapping, offering setup, and API key handling.
- Enable Firebase -> BigQuery export and validate core MVP events are queryable.
- Establish secrets and access control baseline with least-privilege roles.
- Exit criteria: signed `dev` build succeeds, Firebase config reads in `dev`, RevenueCat purchase/restore mapping is validated, BigQuery receives required MVP events, and no secrets are stored in repo.
2. Stage 1: Foundation
- Set up project, Firebase environments, dependency wiring, core models.
- Exit criteria: app boots, Firebase read works, remote config fetch works.
3. Stage 2: Core Viewing Experience
- Build feed UI, video player, autoplay, and episode transitions.
- Exit criteria: smooth playback flow for unlocked episodes.
4. Stage 3: Locking + Paywall
- Implement first-5-free behavior and marked-lock interception.
- Build paywall UI with legal links and restore access.
- Exit criteria: lock behavior and paywall copy are compliant and testable.
5. Stage 4: Subscription + Account Boundary
- Integrate RevenueCat purchases and restore.
- Add Sign in with Apple when entitlement must persist across devices.
- Scope rule: purchase/restore is must-ship; Sign in with Apple/account deletion can defer if timeline risk threatens core funnel launch.
- Exit criteria: purchase and restore are stable in sandbox/TestFlight; Sign in with Apple boundary and account deletion are complete if included in MVP build.
6. Stage 5: Analytics (Core Funnel Only)
- Implement core conversion analytics event contract and dashboard inputs.
- Exit criteria: required MVP funnel events validated end-to-end.
7. Stage 6: Security + Compliance Hardening
- Finalize rules, App Check enforcement, account deletion flow, review artifacts.
- Exit criteria: all required items in `/Users/jonslimak/Projects/iosapp/launch-checklist.md` pass.
8. Stage 7: Release Readiness Buffer
- Bug fixing, conversion tuning via config, final regression pass, and pre-submit App Store policy/upcoming-requirements re-check.
- Exit criteria: launch checklist is fully green and release candidate is stable.

### MVP Defer List (If Schedule Slips)
1. Push notifications and push-open routing (deferred to post-MVP).
2. Extended analytics events outside conversion core (`episode_impression`, `episode_complete`, `push_open`).
3. Retention/cohort KPI expansion (`3plus_episode_rate`, `early_return_rate`, `repeat_return_rate`).
4. Deep BigQuery slicing and A/B experimentation hooks.

### Sprint Mapping
1. Sprint 0: Stage 0.
2. Sprint 1: Stage 1 + start Stage 2.
3. Sprint 2: finish Stage 2 + start Stage 3.
4. Sprint 3: finish Stage 3 + Stage 4.
5. Sprint 4: Stage 5.
6. Sprint 5: Stage 6.
7. Sprint 6: Stage 7 and submission readiness.
8. Sprint 7 (buffer): final fixes only for critical launch blockers.

## Security Baseline (Required)
1. Firestore security rules must deny open public read/write by default.
2. Storage rules must restrict upload/update paths to authorized app flows only.
3. Add Firebase App Check in monitor mode first, then enforce before launch.
4. Add rule tests with Firebase Emulator before TestFlight release build.
5. Restrict service account access and keep all secrets out of app bundle/repo.

## Media Delivery Contract (MVP Minimum)
1. Every `Episode` record must include: `id`, `seriesId`, `episodeNumber`, `videoUrl`, `durationSec`, `isLocked`, `publishAt`, `posterUrl`.
2. `videoUrl` must point to AVPlayer-compatible media and be reachable from the app without custom headers.
3. If required media metadata is missing or invalid, app must skip playback for that episode, show recoverable UI, and avoid crash.
4. If content/config fetch fails, app must keep safe defaults: first 5 free, no trial, yearly-first paywall order.

## Core Interfaces / Types
1. `Series`: `id`, `title`, `coverUrl`, `genre`, `sortOrder`, `isFeatured`.
2. `Episode`: `id`, `seriesId`, `episodeNumber`, `videoUrl`, `durationSec`, `isLocked`, `publishAt`, `posterUrl`.
3. `PaywallConfig`: `freeEpisodeCount`, `showTrial`, `defaultPlanOrder`, `gateMode`.
4. `EntitlementState`: `userId`, `isSubscribed`, `source`, `expiresAt`, `updatedAt`.
5. `AnalyticsEvent`: `name`, `userId`, `seriesId`, `episodeId`, `isSubscribed`, `timestamp`.

## Launch Risk Gate (Required)
1. Subscription/paywall copy and pricing clarity meets App Store rules.
2. Purchases use App Store-compliant digital purchase flows.
3. Mature-content handling matches rating and policy stance.
4. UX patterns may be familiar, but branding/UI/copy remain distinct.
5. Reviewer notes and demo flow are ready for App Review.
6. Privacy baseline is complete (policy, terms, SDK disclosure).
7. Launch catalog (1-2 flagship series) is fully loaded and QA checked.
8. App Store Review Guidelines and Upcoming Requirements are re-verified at release candidate stage.

## Compliance Must-Haves (Required)
1. If account features ship in MVP, in-app account deletion flow is available and easy to find for signed-in users.
2. Paywall shows plan duration and full billed price clearly (monthly/yearly).
3. Paywall includes links to Terms and Privacy Policy.
4. Restore purchase path is visible and functional; Sign in with Apple path is visible and functional if account features ship in MVP.
5. App Review package includes: reviewer notes, demo/test account flow, and IAP path instructions.

## Funnel Analytics Events (Must-Have)
1. `app_open`
2. `episode_start`
3. `gate_blocked`
4. `paywall_shown`
5. `subscribe_tap`
6. `purchase_success`
7. `purchase_restore`

## Analytics Operating Model
1. One owner is accountable for funnel tracking quality and KPI review.
2. Dashboard reviews happen at sprint checkpoints and launch-gate checkpoints.
3. MVP funnel KPIs: `start_rate`, `paywall_view_rate`, `subscribe_conversion`.
4. Defer KPI expansion (`3plus_episode_rate`, `early_return_rate`, `repeat_return_rate`) to post-MVP.
5. Use BigQuery export for core MVP validation; deep slicing and A/B hooks are post-MVP.

## Test Scenarios
1. Apple Developer/App Store Connect setup supports signed `dev` build and sandbox IAP testing.
2. Firebase `dev` and `prod` projects are configured and app can read from selected environment.
3. RevenueCat entitlements map correctly to monthly/yearly App Store products.
4. BigQuery export receives required MVP events and properties.
5. New guest user can start playback immediately with no onboarding friction.
6. First 5 episodes are playable; locked episode triggers paywall.
7. Monthly and yearly purchases unlock immediately.
8. Restore purchase works after reinstall.
9. Auto-advance stops at lock and opens paywall.
10. Network degradation shows retry and can recover.
11. Core MVP analytics events fire with expected fields.
12. Sandbox subscription renewal behavior is validated in test accounts.
13. Failed billing / interrupted purchase flow is handled without app crash.
14. Refund/cancel behavior updates entitlement state correctly.
15. Different storefront/account states do not break price display or purchase flow.
16. Missing or invalid episode metadata shows recoverable UI and does not crash playback flow.
17. Remote config failure falls back to safe defaults (`freeEpisodeCount=5`, `showTrial=false`, yearly-first plan order).
18. Release candidate checklist includes App Store policy and upcoming requirements re-validation.

## Acceptance Criteria
1. DevOps onboarding is complete across Apple, Firebase, RevenueCat, and BigQuery for `dev` and `prod`.
2. Secrets/access baseline is in place and no secrets are stored in repo.
3. Guest user can watch first 5 episodes without signup friction.
4. Locked episodes always route to compliant paywall with clear pricing and legal links.
5. Monthly/yearly purchase and restore flows succeed in sandbox and TestFlight.
6. If account features ship in MVP, account deletion is accessible and functional for signed-in users.
7. Security baseline items (rules + App Check + tests) are complete.
8. Launch risk gate checklist is fully passed before App Store submission, including release-candidate policy/upcoming-requirements re-check.
9. Core MVP funnel analytics dashboard is live and reviewed at sprint cadence.

## Assumptions and Defaults
1. Minimum iOS target: iOS 15+.
2. Trial is off at launch but can be enabled later via config.
3. Price points are finalized in App Store Connect before release.
4. Content team delivers launch-ready assets on schedule.
5. App Store submission baseline uses Xcode 26+ and iOS 26 SDK+.
6. Environment model for MVP is exactly `dev` + `prod`.
7. Crash monitoring stack can stay optional for MVP; launch owner must monitor TestFlight crash signals manually.
