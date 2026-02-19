# Move+ Subscriptions (Android / Google Play)

This document defines how subscriptions work in Move+, including UI flow, tier structure, billing integration, and required backend verification.
Subscriptions are implemented using **Google Play Billing** via Flutter’s `in_app_purchase` package.

---

# 1. Current App Implementation

## Paywall Screen

* File: `lib/screens/subscription_screen.dart`
* Route: `/subscription`
* Registered in: `lib/main.dart`

### Required Google-compliant elements included:

* Clear pricing (Monthly / Yearly toggle)
* Tier benefits explanation
* Restore Purchase button
* Terms of Service link
* Privacy Policy link
* Auto-renew disclosure
* “Cancel anytime in Google Play” notice

---

## Discovery & Conversion Points

Users can access the paywall from:

### 1️⃣ Hamburger Menu

* `Subscription` item above `Leaderboard`
* Implemented in `AppDrawer` (`lib/screens/home_screen.dart`)

### 2️⃣ At Limitation (High Conversion Triggers)

* When daily earning limit is reached
  → Snackbar with **Upgrade** action
  → Opens `/subscription`
  (`lib/screens/run_walk_screen.dart`)

* When Free user taps Leaderboard
  → Upgrade prompt (Subscribe / Not Now)
  (`lib/screens/home_screen.dart`)

* After Free-user activity summary
  → CTA: **“Subscribe to earn ENERGY”**
  (`lib/screens/activity_summary_screen.dart`)

This ensures both discoverability and conversion timing.

---

# 2. Tier Ladder

Subscription model:

Free → Starter → Pro → Elite

---

## 🟢 Free – ₱0

* Daily tracking up to 5 km
* Earn raffle entries only
* Join leaderboard
* Ads enabled
* Online map only

Free users participate in community and prize draws but do not earn direct ENERGY rewards.

---

## 🟢 Starter – ₱56 / month

* Daily max earn distance: 15 km
* Offline map access

Designed for users building daily habits.

---

## 🟢 Pro – ₱250 / month (Most Popular)

* Daily max earn distance: 25 km
* Ad-free experience
* Offline map access
* Faster daily recovery

Designed for serious walkers and runners.

---

## 🟢 Elite – ₱499 / month

* Daily max earn distance: 40 km
* Ad-free experience
* Offline map access
* Fastest recovery
* Priority leaderboard visibility

Designed for high-output users (athletes, delivery riders).

---

# 3. Google Play Product IDs

The following subscription IDs must exist in Play Console and match the app:

Starter:

* `moveplus_starter_monthly`
* `moveplus_starter_yearly`

Pro:

* `moveplus_pro_monthly`
* `moveplus_pro_yearly`

Elite:

* `moveplus_elite_monthly`
* `moveplus_elite_yearly`

If IDs change in Play Console, update them inside:

`lib/screens/subscription_screen.dart` (`_plans` list)

---

# 4. Billing Setup

### Flutter dependency

`in_app_purchase` (declared in `pubspec.yaml`)

### Android permission

`android/app/src/main/AndroidManifest.xml`

```xml
<uses-permission android:name="com.android.vending.BILLING" />
```

---

# 5. Purchase Flow

## Subscribe

1. User taps Subscribe
2. Google Play purchase sheet opens
3. Purchase update received via:
   `InAppPurchase.instance.purchaseStream`
4. App calls backend:
   `verify-play-subscription`
5. Premium unlock occurs only if backend returns:

```json
{ "ok": true }
```

---

## Restore Purchase

1. User taps Restore Purchase
2. `restorePurchases()` is called
3. Restored purchases follow the same backend verification flow

---

# 6. Security Rule (Critical)

The client must NEVER unlock premium based only on local purchase callbacks.

Premium is activated only after:

* Server-side validation of purchase token
* Subscription confirmed active and not expired

This prevents client-side tampering.

---

# 7. Backend Verification

## Edge Function: `verify-play-subscription`

### Request payload

```json
{
  "platform": "android",
  "productId": "moveplus_pro_monthly",
  "purchaseId": "...",
  "source": "google_play",
  "verificationData": "..."
}
```

`verificationData` is the purchase token.

---

## Backend Responsibilities

1. Validate purchase token using Google Play Developer API
2. Confirm:

   * Product matches `productId`
   * Subscription is active
   * `expiryTimeMillis` is in the future
   * Not canceled or revoked
3. Update database
4. Return:

Success:

```json
{ "ok": true }
```

Failure:

```json
{ "ok": false, "error": "reason" }
```

---

# 8. Database Schema (Minimum Required)

Store at least:

* `product_id`
* `purchase_token`
* `expiry_date`
* `is_active`
* `platform`
* `created_at`
* `updated_at`

User-facing entitlement fields:

* `users.is_subscribed`
* `users.subscription_tier` (`free` / `starter` / `pro` / `elite`)
* `users.subscription_expiry`

---

## Product → Tier Mapping

* `moveplus_starter_*` → `starter`
* `moveplus_pro_*` → `pro`
* `moveplus_elite_*` → `elite`

---

# 9. Feature Gating Rules

ENERGY earning:

* Only verified active subscribers can earn ENERGY
* Free users earn raffle entries only

Offline Maps:

* Must be truly usable offline
* Do not advertise if tile caching/preloading is incomplete

Ads:

* Removed only for Pro and Elite tiers

---

# 10. Testing Checklist

Before production release:

* Purchase works on test account
* Restore works on second device
* Invalid tokens rejected
* Expired subscription removes entitlement
* Tier mapping correct
* ENERGY earning locked to verified subscribers only
* Offline maps gated properly

---

Bro — this is now production-level documentation.

Clear for:

* Future developers
* Auditors
* Investors
* Your future self

If you want, next we can write:

* `RAFFLE_SYSTEM.md`
* `ENERGY_ECONOMY.md`
* or `WEB3_INTEGRATION.md`

Build it like a real company from day one.
