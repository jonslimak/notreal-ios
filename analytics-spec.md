# iosapp Analytics Spec (MVP)

## Purpose
Define a single analytics contract for the MVP so product decisions are based on clean, consistent funnel data.

## Data Stack
1. Event collection: Firebase Analytics.
2. Subscription outcomes: RevenueCat integration.
3. Deeper analysis: BigQuery export from Firebase.

## Naming Rules
1. Event names use `snake_case`.
2. Property names use `snake_case`.
3. Do not rename or duplicate event names after launch without version note.

## Common Event Properties (Required on Every Event)
1. `event_time_ms`: client timestamp in milliseconds.
2. `user_id`: stable ID (anonymous or signed-in).
3. `is_anonymous`: `true` or `false`.
4. `session_id`: current session identifier.
5. `app_version`: app version string.
6. `platform`: always `ios` for this app.

## Content Context Properties (Attach When Relevant)
1. `series_id`
2. `episode_id`
3. `episode_number`
4. `is_locked_episode`
5. `feed_position`

## Monetization Context Properties (Attach When Relevant)
1. `paywall_id`
2. `plan_id` (`monthly` or `yearly`)
3. `price_local`
4. `currency`
5. `is_subscribed`
6. `entitlement_id`

## Push Context Properties (Attach When Relevant)
1. `push_campaign_id`

## Event Catalog
1. `app_open`
- Trigger: app enters foreground with active session.
- Required properties: common event properties.
2. `episode_impression`
- Trigger: an episode card is visible long enough to count as seen.
- Required properties: common + content context.
3. `episode_start`
- Trigger: video playback begins.
- Required properties: common + content context.
4. `episode_complete`
- Trigger: playback reaches completion threshold.
- Required properties: common + content context.
5. `gate_blocked`
- Trigger: user hits a locked episode and cannot continue.
- Required properties: common + content context + `paywall_id`.
6. `paywall_shown`
- Trigger: paywall fully displayed.
- Required properties: common + monetization context.
7. `subscribe_tap`
- Trigger: user taps a subscription CTA.
- Required properties: common + monetization context.
8. `purchase_success`
- Trigger: successful purchase completion.
- Required properties: common + monetization context + `entitlement_id`.
9. `purchase_restore`
- Trigger: restore completes with entitlement.
- Required properties: common + monetization context + `entitlement_id`.
10. `push_open`
- Trigger: user opens app via push notification.
- Required properties: common + `push_campaign_id`.

## KPI Definitions (MVP)
1. `start_rate` = unique users with `episode_start` / unique users with `app_open`.
2. `3plus_episode_rate` = unique users with at least 3 `episode_start` events in first viewing session / unique users with `app_open`.
3. `paywall_view_rate` = unique users with `paywall_shown` / unique users with `gate_blocked`.
4. `subscribe_conversion` = unique users with `purchase_success` / unique users with `paywall_shown`.
5. `early_return_rate` = users active in the first return cohort window / new users in the onboarding cohort.
6. `repeat_return_rate` = users active in the sustained return cohort window / new users in the onboarding cohort.

## Dashboard Views (Required)
1. Launch Funnel Dashboard
- Open -> Start -> Gate -> Paywall -> Subscribe.
2. Content Performance Dashboard
- Episode start/completion by series and episode.
3. Subscription Dashboard
- Monthly vs yearly taps, purchases, restores.
4. Retention Dashboard
- Early and repeat return trends by install cohort.

## Quality and Validation
1. Maintain one source-of-truth event map in code.
2. Verify each event in debug logs before TestFlight.
3. Validate production data in BigQuery after internal launch.
4. Treat missing required properties as release-blocking analytics bug.

## Ownership and Cadence
1. Product owner reviews dashboard at sprint checkpoints.
2. Team reviews funnel in sprint planning and sprint review.
3. Any event contract change must update this file first.
