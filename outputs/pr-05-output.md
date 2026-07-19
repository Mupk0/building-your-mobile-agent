# PR Review — PR #5: Add checkout flow

**Repo:** nebius-academy-templates/mobile-pr-review-practice
**PR:** https://github.com/nebius-academy-templates/mobile-pr-review-practice/pull/5
**Files changed:** `src/CheckoutViewModel.kt` (new file), `src/CouponView.swift` (new file)

---

### src/CheckoutViewModel.kt
[HIGH] Hardcoded live secret `CLIENT_SECRET = "sk-live-1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d"` embedded in source — extractable from the compiled APK
[HIGH] `CLIENT_SECRET` is also sent in the POST body of `submitOrder` on every order submission — a client-side secret being transmitted over the wire, confirming it should be a server-side-only credential
[HIGH] `CheckoutViewModel` calls `apiClient.post(...)` directly instead of going through a repository — violates ADR-003 (Repository Pattern); "A ViewModel must not import or reference `APIClient`"
[MEDIUM] `GlobalScope.launch` used instead of `viewModelScope` — coroutines convention requires launching from a lifecycle-aware scope
[MEDIUM] `orderId` interpolated directly into the request path with no validation (format/charset/length/encoding) — missing input validation on user-supplied data
[LOW] `CLIENT_SECRET` is a non-const instance `val` in UPPER_SNAKE_CASE — should be `lowerCamelCase` per naming conventions

### src/CouponView.swift
[HIGH] `CouponView` (a SwiftUI `View`) defines `ApplyCoupon`, a method implementing business/validation logic directly in the View — violates ADR-001 (MVVM); the View "must not contain business logic," which belongs in a ViewModel
[MEDIUM] `ApplyCoupon`'s validation only checks for a non-empty string — no length/charset/format validation before the value is presumably used further
[LOW] `ApplyCoupon` uses a completion-handler callback instead of `async`/`await` — directionally against ADR-002's async/await convention, though ADR-002's text is scoped to networking/`APIClient` and this method performs no network call, so this citation is advisory rather than a strict violation
[LOW] Function name `ApplyCoupon` is UpperCamelCase; should be `applyCoupon` (lowerCamelCase) — swift-conventions naming rule

## Summary
4 HIGH, 3 MEDIUM, 2 LOW

**Most important issue:** a live `sk-live-...` client secret is hardcoded in `CheckoutViewModel.kt` and additionally transmitted from the client on every order submission — a directly exploitable credential leak that must be rotated and moved server-side immediately.

**Verdict:** Critical security defect plus multiple architecture violations across both files; this diff must not be merged as-is (this is not an approval).
