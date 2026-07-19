# PR Review — PR #3: Add payment service

**Repo:** nebius-academy-templates/mobile-pr-review-practice
**PR:** https://github.com/nebius-academy-templates/mobile-pr-review-practice/pull/3
**Files changed:** `src/PaymentService.kt` (new file, +18)

---

### src/PaymentService.kt
[HIGH] Hardcoded live payment-provider API key (`"sk-live-9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c"`) as a source-level property — secrets must not live in source; extractable from the compiled APK
[HIGH] `charge()` builds its own `Retrofit` instance directly and exposes networking as a plain (non-suspend) function instead of going through a single shared API client — violates ADR-002 (async/await/coroutines networking)
[HIGH] `PaymentService` constructs its own network stack rather than being a repository wrapping a single shared API client — violates ADR-003 (Repository Pattern)
[MEDIUM] `charge(amount: Int, token: String)` has no input validation — `amount` isn't checked for positive/non-zero/sane bounds and `token` isn't checked for non-blank/expected format before use
[LOW] `API_KEY` is a non-const instance `val` in UPPER_SNAKE_CASE — should be `lowerCamelCase` per naming conventions (UPPER_SNAKE_CASE is reserved for compile-time constants)

## Summary
3 HIGH, 1 MEDIUM, 1 LOW

**Most important issue:** a live payment-provider API key (`sk-live-...`) is hardcoded directly in client-side source, trivially extractable from the compiled APK — this must be rotated/revoked immediately and charge execution moved server-side; it should never ship in a mobile client.

**Verdict:** Critical security defect plus architecture violations found; this diff must not be merged as-is (this is not an approval).
