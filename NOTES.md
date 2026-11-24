# Instructor notes: StateFlow pitfalls (M13 section 4)

Reference only. **No runnable code for these pitfalls is in this project.** The throwaway demo functions from M13 sections 4.1 and 4.2 (`togglePrayerBad` and the read-then-write `incrementTasbih`) are classroom-only and were left out of the source on purpose. Source: `14/M13-Student.md` sections 2.5, 4.1, 4.2, 4.5.

## Pitfall 1: changing a list in place (section 4.1)

**Setup shown in class.** The state class is temporarily changed so that `donePrayers` is a `MutableList<String>` (with a `mutableListOf()` default). A "bad" ViewModel function adds to that list directly and is called from the Tracker checkbox. Students predict what happens first.

**What happens.** Nothing changes on screen, even though the list really did change.

**Why.** `_uiState.value` is still the *same object*, so `StateFlow` sees an equal value and does not emit. Compose never hears about the change. Writing `_uiState.value = _uiState.value.copy(donePrayers = sameList)` fails for the same reason: the list inside is the same instance, so `equals` returns `true`. (General rule from section 2.5: `StateFlow` skips equal values.)

**Fix.** Go back to a read-only `List<String>` and build a **new** list inside `update { }`, for example `state.donePrayers + name` or `state.donePrayers - name`, as `SalatKuViewModel.togglePrayer` does.

**Rule to say out loud.** State classes use `val` and `List`, never `var` and `MutableList`.

## Pitfall 2: read, then write, without `update` (section 4.2)

**The shape of the mistake.** The function reads `_uiState.value` into a local variable, then assigns `_uiState.value = current.copy(...)` computed from that local.

**Why it is wrong.** With one tap on one thread it works. If two updates run at nearly the same time, both can read the same old value and one overwrites the other (a lost update). `update { }` does read, change and write as one safe step.

**Rule to say out loud.** Use `update { }` every time the new value depends on the old one. In this project every state change in `SalatKuViewModel` goes through `update { }`.

## Related traps from the same lesson (not asked for, kept for completeness)

- **Coroutines in `init` (section 4.5).** Do not start work in `init { viewModelScope.launch { ... } }`. It is hard to test, you do not control when it starts, and it can read properties declared below the `init` block before they are initialised. Write a named function (HW13 uses `loadPrayerTimes()`) and call it on purpose.
- **Process death (section 2.5).** A ViewModel survives rotation, not process death. `rememberSaveable` or `SavedStateHandle` solve that. M13 only promises rotation.
- **`stateIn` (section 4.4).** For streams (repository, database), not for counting a list. `doneCount` stays a computed property on `SalatKuUiState`.

## How to demonstrate in class

Do the demos in a scratch branch, or undo them afterwards (M13-Student.md CP2 says to remove the demo code). Starting from this project, the demo for Pitfall 1 needs the state class change and a temporary extra ViewModel function, and the Tracker call site must be pointed at it. Because the ViewModel's functions are private after CP3, the demo function also has to be reachable (for example through a temporary `onEvent` branch). Reset to the committed state when done.
