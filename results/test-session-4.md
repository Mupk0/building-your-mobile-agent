# Test session 4

## Command

```
/review-pr 3
```

## Transcript

**PR lookup**

The PR was not found in the repo inferred from the local git remote
(`Mupk0/building-your-mobile-agent` — 404, no PRs exist there). Asked the user
where to look; user pointed to
`https://github.com/nebius-academy-templates/mobile-pr-review-practice`.

Fetched PR #3 from `nebius-academy-templates/mobile-pr-review-practice`:

- **Title:** Add payment service
- **Body:** Adds the payment service that charges a rider's saved card via the payments API.
- **Branch:** `pr-03-payment-apikey` → `main`
- **Files changed:** 1 (`src/PaymentService.kt`, +18/-0, new file)

Diff:

```diff
diff --git a/src/PaymentService.kt b/src/PaymentService.kt
new file mode 100644
index 0000000..14d1281
--- /dev/null
+++ b/src/PaymentService.kt
@@ -0,0 +1,18 @@
+package com.rideapp.payments
+
+import retrofit2.Retrofit
+import retrofit2.converter.gson.GsonConverterFactory
+
+class PaymentService {
+
+    val API_KEY = "sk-live-9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c"
+
+    private val retrofit = Retrofit.Builder()
+        .baseUrl("https://api.rideapp.com/payments/")
+        .addConverterFactory(GsonConverterFactory.create())
+        .build()
+
+    fun charge(amount: Int, token: String) {
+        // Sends the charge request with API_KEY in the Authorization header.
+    }
+}
```

**Routing:** Kotlin-only diff (one `.kt` file) → all three sub-agents dispatched in parallel.

### style-reviewer output

```
### PaymentService.kt

[LOW] Property `API_KEY` uses UPPER_SNAKE_CASE but is a regular instance `val`, not a `const val` or top-level constant — kotlin-conventions naming rule (UPPER_SNAKE_CASE reserved for compile-time constants; instance properties should be `lowerCamelCase`, e.g. `apiKey`)
```
Note from sub-agent: hardcoding the API key value itself is a security concern, out of scope for this style-only review.

### security-reviewer output

```
### src/PaymentService.kt
[HIGH] Hardcoded live payment API key `sk-live-9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c` in source — secrets must not be hardcoded in source
[MEDIUM] `charge(amount: Int, token: String)` performs no validation of `amount` (e.g. positive, non-zero, bounds) or `token` (format/non-empty) before use — missing input validation on user-supplied data
```

### architecture-reviewer output

Reasoning: `PaymentService` is not a ViewModel and does not call `APIClient` directly,
so ADR-003 (repository pattern) can't be confirmed as violated — no ViewModel/repository
layer is shown in this diff, and flagging it would be speculative. ADR-001 (MVVM
boundaries) doesn't apply — no View/ViewModel code in this diff. ADR-002 (async/await
networking) is confirmable on two counts: `PaymentService` builds its own `Retrofit`
instance instead of going through the single `APIClient`, and `charge()` is a
synchronous, non-suspend function performing a network call with no coroutine usage.

```
### src/PaymentService.kt
[MEDIUM] `PaymentService` builds its own `Retrofit` instance instead of going through the single `APIClient` — violates ADR-002 (async/await networking stack), which states `APIClient` is the only type permitted to perform network requests.
[MEDIUM] `charge()` is a synchronous, non-suspend function performing a network request with no async/await (coroutine) usage — violates ADR-002 (async/await networking).
```

Relevant ADR files consulted:
- `docs/adr/ADR-001-mvvm-architecture.md`
- `docs/adr/ADR-002-networking-stack.md`
- `docs/adr/ADR-003-repository-pattern.md`

## Consolidated review (as presented to the user)

### PaymentService.kt

```
[HIGH] Hardcoded live payment API key (`sk-live-9f8a7b6c5d4e3f2a1b0c9d8e7f6a5b4c`) in source — secrets must not live in source
[MEDIUM] `charge(amount, token)` performs no validation of `amount` (positive/non-zero/bounds) or `token` (format/non-empty) before use — missing input validation on user-supplied data
[MEDIUM] `PaymentService` builds its own `Retrofit` instance instead of going through the single `APIClient` — violates ADR-002 (async/await networking stack)
[MEDIUM] `charge()` is a synchronous, non-suspend function performing a network request with no coroutine/async usage — violates ADR-002 (async/await networking)
[LOW] `API_KEY` uses UPPER_SNAKE_CASE but is a regular instance `val`, not a `const val`/top-level constant — kotlin-conventions naming rule (instance properties should be `lowerCamelCase`, e.g. `apiKey`)
```

## Summary

**1 HIGH, 3 MEDIUM, 1 LOW**

Most important issue: the hardcoded live payment API key committed directly in source
(`API_KEY = "sk-live-..."`) — an exploitable secret exposure in a payments-handling
class; should block merge until rotated and removed from history.

**Verdict:** Not mergeable as-is — one exploitable security defect plus architecture
and validation gaps need to be addressed before this can go in.

## Post-review status

Review was presented in chat per the confirmation gate. **Not posted to GitHub** —
no `yes`/`post` confirmation was given in this session as of this transcript capture.
