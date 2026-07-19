---
name: kotlin-conventions
description: "Kotlin (Android) style and convention rules for the mobile team. Load ONLY for Kotlin PRs — i.e. when the diff contains one or more .kt files; do not load for Swift-only PRs. Covers naming, coroutines, and Jetpack/lifecycle usage. Used by the style-reviewer sub-agent."
---

# Kotlin Conventions

Apply these rules to every `.kt` file in the diff. Report only issues introduced or
touched by the diff. For each finding, emit one line:

```
[SEVERITY] <short description> — <the rule it breaks>
```

## Rules

### Naming — `[LOW]`
Classes, objects, and interfaces use `UpperCamelCase`. Functions, properties, and
parameters use `lowerCamelCase`. Compile-time constants (`const val`, top-level `val`)
use `UPPER_SNAKE_CASE`. No snake_case functions or properties. Names must be
descriptive — avoid `getData`, `tmp`, `flag`.
- Violation: `class feed_adapter`, `fun GetData()`, `val Badge_Count`.

### Coroutines — `[MEDIUM]`
Asynchronous work runs in coroutines launched from a lifecycle-aware scope
(`viewModelScope`, `lifecycleScope`) — not raw `Thread`, `AsyncTask`, or a bare
`GlobalScope.launch`. Do blocking/IO work on `Dispatchers.IO`; never block the main
thread. Suspend functions must be main-safe and must not swallow `CancellationException`.
- Violation: `GlobalScope.launch { ... }`, `Thread { repo.load() }.start()`, or a
  network call on `Dispatchers.Main`.

### Jetpack / lifecycle — `[MEDIUM]`
ViewModels hold state and never reference a `Context`, `Activity`, `View`, or `Fragment`
(prevents leaks; per ADR-001). Expose immutable state (`StateFlow`/`LiveData`) and keep
the mutable backing field private. Observe with the lifecycle owner
(`viewLifecycleOwner`) and cancel work in `onCleared`/`onDestroy` — do not leak
listeners or scopes.
- Violation: `class ProfileViewModel(val activity: Activity)`, or a public `MutableStateFlow`.

## Template

**Compliant**

```kotlin
class ProfileViewModel(
    private val repository: ProfileRepository,
) : ViewModel() {

    private val _state = MutableStateFlow<ProfileState>(ProfileState.Loading)
    val state: StateFlow<ProfileState> = _state.asStateFlow()

    fun loadProfile(userId: String) {
        viewModelScope.launch {
            _state.value = withContext(Dispatchers.IO) {
                ProfileState.Loaded(repository.fetchProfile(userId))
            }
        }
    }
}
```

**Non-compliant**

```kotlin
class ProfileViewModel(val activity: Activity) : ViewModel() {   // [MEDIUM] holds a Context/Activity

    val State = MutableStateFlow<Any?>(null)                     // [LOW] naming + public mutable state

    fun GetData(userId: String) {                                // [LOW] function in UpperCamelCase
        GlobalScope.launch {                                     // [MEDIUM] not lifecycle-scoped
            val p = repository.fetchProfile(userId)              // blocking call, wrong dispatcher
            activity.runOnUiThread { State.value = p }
        }
    }
}
```
