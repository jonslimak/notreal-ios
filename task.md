# iosapp MVP Execution Board

## Context Snapshot
- Project: iosapp
- Repo: /Users/jonslimak/Projects/iosapp
- Source of truth PRD: `/Users/jonslimak/Projects/iosapp/prd.json`
- Plan baseline: `/Users/jonslimak/Projects/iosapp/plan.md`
- Current status: Stage 0 onboarding remains in progress (`S0-T6` blocked), but app implementation can proceed in parallel via a `dev`-first path (Stages 1-3) under soft-gate policy.
- Last updated: 2026-02-13
- Session closeout: parallel execution approved. Ops lane continues `S0-T6`/`S0-T7`/`S0-T8`; app lane starts Stage 1 in `dev` while Stage 0 remains a required release gate.

## Scope Guardrails (Must Hold)
- MVP is watch -> gate -> subscribe only.
- No push notifications in MVP.
- Core conversion analytics only in MVP.
- Environments are `dev` + `prod` only.
- No new user-facing scope after Stage 4 starts unless equal-sized scope is removed.
- Core funnel launch path has priority; if schedule risk appears, defer Sign in with Apple/account deletion before core watch -> gate -> subscribe work.

## Task Row Format
- Fields: `ID | Stage | Type | Task | Status | Evidence`
- Status values: `not_started`, `in_progress`, `blocked`, `done`
- Type values: `build`, `test`, `doc`, `release`
- Rule: a stage is complete only when its gate row is `done`.

## Execution Mode (Approved)
- Lane A (App lane): progress Stage 1 -> Stage 2 -> Stage 3 in `dev` first.
- Lane B (Ops lane): continue Stage 0 `S0-T6`, `S0-T7`, and `S0-T8` in parallel.
- Soft-gate rule: Stage 0 does not block Stage 1-3 implementation progress.
- Hard-gate rule: Stage 0 gate (`S0-G1`) is still mandatory before release readiness.
- Dependency rule: do not start live billing implementation (`S4-T1`) until `S0-T6` is unblocked and product status is healthy.

## Stage 0: DevOps + Service Onboarding
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S0-T1 | Stage 0 | build | Create Apple Developer/App Store Connect app records and bundle IDs for `dev` and `prod`. | done | Created bundle IDs `com.notreal.iosapp` and `com.notreal.iosapp.dev`; created App Store Connect apps `notreal` (prod) and `notreal Dev` (dev) on 2026-02-11. |
| S0-T2 | Stage 0 | build | Create signing certificates/profiles and confirm signed `dev` build. | done | Configured automatic signing in Xcode with bundle ID `com.notreal.iosapp.dev` and validated signed `dev` build by running successfully on iPhone Simulator on 2026-02-11. |
| S0-T3 | Stage 0 | build | Configure monthly/yearly IAP products and sandbox tester accounts. | done | Created subscription group `notreal Premium`; added dev products `com.notreal.iosapp.dev.monthly` (49.99) and `com.notreal.iosapp.dev.yearly` (199.99) on 2026-02-11; added equivalent prod products on 2026-02-13 during RevenueCat linking; created 2 sandbox tester accounts. |
| S0-T4 | Stage 0 | build | Create Firebase projects (`dev`, `prod`) and register iOS app in both. | done | Created Firebase projects `notreal-dev` and `notreal-prod` with Analytics enabled; registered iOS apps `com.notreal.iosapp.dev` and `com.notreal.iosapp`; downloaded configs (`GoogleService-Info.plist` for dev and `GoogleService-Info-prod.plist` for prod) and confirmed both apps connected in Firebase console on 2026-02-11. |
| S0-T5 | Stage 0 | build | Enable Firestore, Remote Config, Analytics; set baseline rules and IAM roles. | done | Enabled Firestore (`(default)` DB, `us-central1`, production mode) in both `notreal-dev` and `notreal-prod`; published Remote Config keys `freeEpisodeCount=5`, `showTrial=false`, `defaultPlanOrder=yearly_then_monthly` in both projects; applied restrictive baseline Firestore rules (public read-only path + deny-by-default fallback); completed IAM baseline check for required access only on 2026-02-11. |
| S0-T6 | Stage 0 | build | Configure RevenueCat project/app, products, offerings, and entitlement mapping. | blocked | Started setup: created RevenueCat project `notreal`, added apps `notreal-dev` (`com.notreal.iosapp.dev`) and `notreal-prod` (`com.notreal.iosapp`), created entitlement `premium`, and imported products for both apps. Current blocker: product status now shows `Missing Metadata` on both apps (App Store Connect + RevenueCat). Apple agreement updates were submitted on 2026-02-13 (verification window up to 24h), and required subscription metadata completion is still pending before offering/package mapping can be finalized. |
| S0-T7 | Stage 0 | build | Enable Firebase -> BigQuery export for `dev` and `prod`. | not_started | |
| S0-T8 | Stage 0 | doc | Document secrets/access policy (no repo secrets, least privilege, ownership). | not_started | |
| S0-Q1 | Stage 0 | test | Technical check: Signed `dev` build succeeds end-to-end. | not_started | |
| S0-Q2 | Stage 0 | test | Technical check: App reads Firebase content/config from `dev`. | not_started | |
| S0-Q3 | Stage 0 | test | Manual check: Sandbox purchase + restore works with mapped entitlement. | not_started | |
| S0-Q4 | Stage 0 | test | Technical check: BigQuery query confirms required MVP events and properties. | not_started | |
| S0-G1 | Stage 0 | release | Milestone gate: all Stage 0 tasks/tests complete with evidence. | not_started | |

## Stage 1: Foundation
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S1-T1 | Stage 1 | build | Set up app shell, dependency wiring, and environment configuration. | not_started | |
| S1-T2 | Stage 1 | build | Implement core data models (`Series`, `Episode`, `PaywallConfig`, `EntitlementState`). | not_started | |
| S1-T3 | Stage 1 | build | Implement Firebase read layer for core models and remote config fetch (`*planning needed` for `ContentRepository` + `ConfigProvider` abstraction before implementation). | not_started | |
| S1-T4 | Stage 1 | build | Add analytics dispatcher scaffold (without full event wiring yet) (`*planning needed` for `AnalyticsSink` abstraction before implementation). | not_started | |
| S1-T5 | Stage 1 | doc | Define mini-spec for service abstractions: `ContentRepository`, `EntitlementService`, `ConfigProvider`, `AnalyticsSink` (`*planning needed`). | not_started | |
| S1-Q1 | Stage 1 | test | Technical check: App builds and boots cleanly. | not_started | |
| S1-Q2 | Stage 1 | test | Technical check: Firebase reads and remote config fetch succeed in `dev`. | not_started | |
| S1-Q3 | Stage 1 | test | Manual check: App launch and initial navigation shell are stable. | not_started | |
| S1-G1 | Stage 1 | release | Milestone gate: Stage 1 exit criteria met. | not_started | |

## Stage 2: Core Viewing Experience
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S2-T1 | Stage 2 | build | Implement Player + Home flows aligned to `/Users/jonslimak/Projects/iosapp/mockups/v1-screens.html`. | not_started | |
| S2-T2 | Stage 2 | build | Implement unlocked playback and episode transition behavior. | not_started | |
| S2-T3 | Stage 2 | build | Implement auto-advance for unlocked episodes. | not_started | |
| S2-Q1 | Stage 2 | test | Technical check: Playback and transitions run without blocking errors. | not_started | |
| S2-Q2 | Stage 2 | test | Manual check: Open -> watch -> swipe -> next episode path works smoothly. | not_started | |
| S2-Q3 | Stage 2 | test | Manual check: Player close returns to Home reliably. | not_started | |
| S2-G1 | Stage 2 | release | Milestone gate: Stage 2 exit criteria met. | not_started | |

## Stage 3: Locking + Paywall
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S3-T1 | Stage 3 | build | Implement first-5-free and lock interception rules. | not_started | |
| S3-T2 | Stage 3 | build | Route locked episodes to paywall across all entry paths. | not_started | |
| S3-T3 | Stage 3 | build | Implement compliant paywall content (pricing clarity, legal links, restore visibility). | not_started | |
| S3-Q1 | Stage 3 | test | Technical check: lock rule returns correct results around boundary episodes. | not_started | |
| S3-Q2 | Stage 3 | test | Manual check: locked episodes consistently open paywall. | not_started | |
| S3-Q3 | Stage 3 | test | Manual check: Terms/Privacy links and restore action are visible and functional. | not_started | |
| S3-G1 | Stage 3 | release | Milestone gate: Stage 3 exit criteria met. | not_started | |

## Stage 4: Subscription + Account Boundary
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S4-T1 | Stage 4 | build | Integrate RevenueCat purchase/restore flows for monthly/yearly plans (`*planning needed` for `EntitlementService` live integration details). | not_started | Must wait for `S0-T6` unblock and healthy RevenueCat product status before live wiring. |
| S4-T2 | Stage 4 | build | Implement Sign in with Apple at subscribe/restore boundary only (stretch-capable if core funnel timeline is at risk). | not_started | |
| S4-T3 | Stage 4 | build | Add signed-in account deletion entry path (stretch-capable if core funnel timeline is at risk). | not_started | |
| S4-Q1 | Stage 4 | test | Technical check: entitlement state updates correctly after purchase/restore. | not_started | |
| S4-Q2 | Stage 4 | test | Manual check: sandbox monthly and yearly purchase succeed. | not_started | |
| S4-Q3 | Stage 4 | test | Manual check: reinstall + restore returns access correctly. | not_started | |
| S4-Q4 | Stage 4 | test | Manual check: Sign in with Apple appears only at intended boundary (only if `S4-T2` is in MVP scope). | not_started | |
| S4-G1 | Stage 4 | release | Milestone gate: purchase/restore must pass; SIWA/account deletion pass if included in MVP scope. | not_started | |

## Stage 5: Core Analytics (MVP Required Set Only)
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S5-T1 | Stage 5 | build | Instrument required events: `app_open`, `episode_start`, `gate_blocked`, `paywall_shown`, `subscribe_tap`, `purchase_success`, `purchase_restore`. | not_started | |
| S5-T2 | Stage 5 | build | Enforce required common properties on all required events. | not_started | |
| S5-T3 | Stage 5 | doc | Document MVP event map and owner for analytics quality. | not_started | |
| S5-Q1 | Stage 5 | test | Technical check: debug logs show all required events fire with required properties. | not_started | |
| S5-Q2 | Stage 5 | test | Technical check: required events appear in BigQuery `dev` export. | not_started | Depends on `S0-T7` completion. |
| S5-Q3 | Stage 5 | test | Manual check: one full funnel run captures expected event sequence. | not_started | |
| S5-G1 | Stage 5 | release | Milestone gate: Stage 5 exit criteria met and no deferred events added. | not_started | |

## Stage 6: Security + Compliance Hardening
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S6-T1 | Stage 6 | build | Finalize Firestore/Storage security rules for launch-safe defaults. | not_started | |
| S6-T2 | Stage 6 | build | Complete App Check rollout path (monitor -> enforce before launch). | not_started | |
| S6-T3 | Stage 6 | doc | Finalize App Review package (reviewer notes, demo path, metadata consistency). | not_started | |
| S6-T4 | Stage 6 | doc | Finalize privacy/SDK disclosure and compliance checklist evidence. | not_started | |
| S6-Q1 | Stage 6 | test | Technical check: rules validation/emulator checks pass. | not_started | |
| S6-Q2 | Stage 6 | test | Technical check: repo check confirms no secrets committed. | not_started | |
| S6-Q3 | Stage 6 | test | Manual check: account deletion discoverable for signed-in users (if account features shipped in MVP). | not_started | |
| S6-G1 | Stage 6 | release | Milestone gate: Stage 6 exit criteria and launch checklist criticals pass. | not_started | |

## Stage 7: Release Readiness Buffer
| ID | Stage | Type | Task | Status | Evidence |
|---|---|---|---|---|---|
| S7-T1 | Stage 7 | build | Fix critical launch blockers only (no new features). | not_started | |
| S7-T2 | Stage 7 | test | Run final MVP regression on core funnel and billing boundary flows. | not_started | |
| S7-T3 | Stage 7 | release | Final release candidate sanity pass, submission readiness check, and App Store policy/upcoming-requirements re-check. | not_started | |
| S7-G1 | Stage 7 | release | Milestone gate: release candidate is stable with no critical blockers. | not_started | |

## Deferred Post-MVP (Do Not Execute in MVP)
- Push notifications and push routing.
- Deferred analytics events: `episode_impression`, `episode_complete`, `push_open`.
- Deferred KPIs: `3plus_episode_rate`, `early_return_rate`, `repeat_return_rate`.
- Deep BigQuery slicing and A/B hooks.
- Optional player side actions.
- Free-trial activation work.

## Milestone Test Evidence Log Template
- Stage:
- Date:
- Technical checks executed:
- Manual checks executed:
- Result (`pass`/`fail`):
- Notes and residual risk:
