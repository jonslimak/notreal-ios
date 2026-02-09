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
1. Audience: women 30-60, US-only, English-first, 17+ target.
2. Core UX: vertical swipe feed, full-screen playback, auto-advance episodes.
3. Access: guest-first viewing.
4. Gate: first 5 episodes free, then marked locked episodes.
5. Monetization: subscription-only in v1, coins-compatible model for later.
6. Plans: monthly + yearly, no free trial at launch (trial-capable config retained).
7. Auth: Sign in with Apple at subscribe/restore boundary.
8. Content ops: Firebase Console only (no custom admin panel in MVP).
9. Out of scope: comments, downloads, live chat, coin economy UI.

## Technical Decisions
1. App stack: SwiftUI + AVFoundation/AVKit.
2. Backend: Firebase (Firestore + Storage URLs + Remote Config + FCM).
3. Billing: StoreKit 2 via RevenueCat.
4. Analytics: Firebase Analytics + RevenueCat integration + BigQuery export.
5. No custom backend or custom BI dashboard in MVP.

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
- Account deletion entry point for signed-in users.
6. Analytics Layer
- Central analytics dispatcher to avoid event drift.
- Events and properties must follow `/Users/jonslimak/Projects/iosapp/analytics-spec.md`.
7. Release and Risk Layer
- Compliance gates, review package, and final checks from `/Users/jonslimak/Projects/iosapp/launch-checklist.md`.

### Build Order (Execution Sequence)
1. Stage 1: Foundation
- Set up project, Firebase environments, dependency wiring, core models.
- Exit criteria: app boots, Firebase read works, remote config fetch works.
2. Stage 2: Core Viewing Experience
- Build feed UI, video player, autoplay, and episode transitions.
- Exit criteria: smooth playback flow for unlocked episodes.
3. Stage 3: Locking + Paywall
- Implement first-5-free behavior and marked-lock interception.
- Build paywall UI with legal links and restore access.
- Exit criteria: lock behavior and paywall copy are compliant and testable.
4. Stage 4: Subscription + Account Boundary
- Integrate RevenueCat purchases and restore.
- Add Sign in with Apple when entitlement must persist across devices.
- Exit criteria: purchase and restore are stable in sandbox/TestFlight.
5. Stage 5: Analytics + Notifications
- Implement analytics event contract and KPI dashboard inputs.
- Add basic push notifications and deep-link handling.
- Exit criteria: funnel events validated end-to-end; push open routes correctly.
6. Stage 6: Security + Compliance Hardening
- Finalize rules, App Check enforcement, account deletion flow, review artifacts.
- Exit criteria: all required items in `/Users/jonslimak/Projects/iosapp/launch-checklist.md` pass.
7. Stage 7: Release Readiness Buffer
- Bug fixing, conversion tuning via config, final regression pass.
- Exit criteria: launch checklist is fully green and release candidate is stable.

### Sprint Mapping
1. Sprint 1: Stage 1 + start Stage 2.
2. Sprint 2: finish Stage 2 + start Stage 3.
3. Sprint 3: finish Stage 3 + Stage 4.
4. Sprint 4: Stage 5.
5. Sprint 5: Stage 6.
6. Sprint 6: Stage 7 and submission readiness.
7. Sprint 7 (buffer): final fixes only for critical launch blockers.

## Security Baseline (Required)
1. Firestore security rules must deny open public read/write by default.
2. Storage rules must restrict upload/update paths to authorized app flows only.
3. Add Firebase App Check in monitor mode first, then enforce before launch.
4. Add rule tests with Firebase Emulator before TestFlight release build.
5. Restrict service account access and keep all secrets out of app bundle/repo.

## Core Interfaces / Types
1. `Series`: `id`, `title`, `coverUrl`, `genre`, `sortOrder`, `isFeatured`.
2. `Episode`: `id`, `seriesId`, `episodeNumber`, `videoUrl`, `durationSec`, `isLocked`, `publishAt`.
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

## Compliance Must-Haves (Required)
1. In-app account deletion flow is available and easy to find for signed-in users.
2. Paywall shows plan duration and full billed price clearly (monthly/yearly).
3. Paywall includes links to Terms and Privacy Policy.
4. Restore purchase and Sign in with Apple paths are visible and functional.
5. App Review package includes: reviewer notes, demo/test account flow, and IAP path instructions.

## Funnel Analytics Events (Must-Have)
1. `app_open`
2. `episode_impression`
3. `episode_start`
4. `episode_complete`
5. `gate_blocked`
6. `paywall_shown`
7. `subscribe_tap`
8. `purchase_success`
9. `purchase_restore`
10. `push_open`

## Analytics Operating Model
1. One owner is accountable for funnel tracking quality and KPI review.
2. Dashboard reviews happen at sprint checkpoints and launch-gate checkpoints.
3. Funnel KPIs: `start_rate`, `3plus_episode_rate`, `paywall_view_rate`, `subscribe_conversion`, `early_return_rate`, `repeat_return_rate`.
4. Use BigQuery export for deep funnel slices (by series, episode, source, entitlement state).
5. If running Firebase A/B tests later, mirror purchase/entitlement outcomes into Firebase events used by experiments.

## Test Scenarios
1. New guest user can start playback immediately with no onboarding friction.
2. First 5 episodes are playable; locked episode triggers paywall.
3. Monthly and yearly purchases unlock immediately.
4. Restore purchase works after reinstall.
5. Auto-advance stops at lock and opens paywall.
6. Network degradation shows retry and can recover.
7. Push deep-link opens correct show/episode.
8. Core analytics events fire with expected fields.
9. Sandbox subscription renewal behavior is validated in test accounts.
10. Failed billing / interrupted purchase flow is handled without app crash.
11. Refund/cancel behavior updates entitlement state correctly.
12. Different storefront/account states do not break price display or purchase flow.

## Acceptance Criteria
1. Guest user can watch first 5 episodes without signup friction.
2. Locked episodes always route to compliant paywall with clear pricing and legal links.
3. Monthly/yearly purchase and restore flows succeed in sandbox and TestFlight.
4. Account deletion is accessible and functional for signed-in users.
5. Security baseline items (rules + App Check + tests) are complete.
6. Launch risk gate checklist is fully passed before App Store submission.
7. Funnel analytics dashboard is live and reviewed at sprint cadence.

## Assumptions and Defaults
1. Minimum iOS target: iOS 15.1+.
2. Trial is off at launch but can be enabled later via config.
3. Price points are finalized in App Store Connect before release.
4. Content team delivers launch-ready assets on schedule.
