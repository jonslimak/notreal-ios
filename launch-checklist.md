# iosapp Launch Checklist (MVP)

## Goal
Use this checklist as the final go/no-go gate before App Store submission and public launch.

## 1) Product and Content Readiness
1. At least 1-2 flagship series are fully uploaded and QA reviewed.
2. Episode metadata is complete (title, order, lock state, artwork).
3. First 5 episodes free behavior is correct for each launch series.
4. Locked episodes route to paywall consistently.

## 2) Subscription and Entitlements
1. App Store Connect products are configured for monthly and yearly plans.
2. RevenueCat entitlement mapping is correct for both plans.
3. Purchase flow works in sandbox and TestFlight.
4. Restore purchase flow works on reinstall and new device login.
5. Cancel/refund state updates entitlement correctly.

## 3) Compliance and Legal
1. Paywall clearly shows plan length and full billed price.
2. Paywall includes Terms and Privacy Policy links.
3. In-app account deletion exists and is discoverable for signed-in users.
4. App age rating and content positioning are aligned with 17+ target.
5. Privacy labels and third-party SDK disclosures are complete.

## 4) Security and Platform Safety
1. Firestore rules are not open to public write/read by default.
2. Storage rules are scoped to authorized access paths.
3. Firebase App Check is enabled (monitor mode complete, enforcement ready).
4. Security rules tests pass in emulator or CI checks.
5. Secrets and service credentials are not in app code or repo history.

## 5) Analytics and Monitoring
1. All required events in `/Users/jonslimak/Projects/iosapp/analytics-spec.md` are implemented.
2. Required properties are present for core funnel events.
3. Launch funnel dashboard is available and validated with test data.
4. BigQuery export is active and queryable.
5. Sprint owner and review cadence are assigned.

## 6) QA Regression
1. Guest user can start playback immediately with no onboarding friction.
2. Auto-advance works and stops at locked boundaries.
3. Network errors show retry and recover gracefully.
4. Push notification opens correct in-app destination.
5. No release-blocking crashes in latest TestFlight build.

## 7) App Review Submission Package
1. Reviewer notes include clear purchase and restore test path.
2. Demo/test account details are prepared and validated before submission.
3. Backend services needed for review are active during the review phase.
4. App Store screenshots, description, and support URLs are finalized.
5. Submission metadata does not contradict in-app behavior.

## 8) Launch Operations
1. Assign launch owner for technical decision-making.
2. Assign business owner for KPI and conversion decisions.
3. Define launch check-in sequence (initial, stabilization, and post-launch checkpoints).
4. Prepare rollback path (feature flags, paywall config fallback, hotfix owner).
5. Confirm support response process for billing/access user tickets.

## Exit Rule
All sections must pass with no critical open items before submission. Any failed item becomes a launch blocker or requires explicit signed risk acceptance.
