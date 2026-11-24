# SalatKu: consolidated codebase after M07-M13 (start of M14)

SIF502 Pengembangan Aplikasi Bergerak, Odd 2026/2027. Package `com.example.shalatku`.

This project is SalatKu **as it should look at the end of Meeting 13**, with **HW12 done**, ready for **Meeting 14 (Mini Project Integration)**. M14 has no new script. It only consolidates M07-M13, so there is no M14 code here.

> **Not compiled.** This project was assembled and checked by static review only (no Android SDK or Gradle network access was available). Open it in Android Studio, sync, and run it once before using it in class. Fix any import the IDE complains about with Option+Enter.

## Open and build

1. Install Android Studio and let it install **Android SDK Platform 37** (`compileSdk = 37`) when it asks.
2. **File > Open** and choose this folder (`SalatKu-Project-Template`). Android Studio creates `local.properties` by itself. It is git-ignored, do not share it.
3. Wait for Gradle sync. The wrapper downloads Gradle 9.8.1. Use a JDK 17 or newer (the Android Studio bundled JDK is fine).
4. Run on an emulator or phone (**minSdk 26**, so API 26+). Turn on auto-rotate to check that state survives rotation.

Command line (needs `ANDROID_HOME` or a `local.properties` with `sdk.dir=...`):

```bash
./gradlew assembleDebug testDebugUnitTest
```

Identity: `applicationId` and `namespace` are `com.example.shalatku`. Theme function: `ShalatkuTheme`. ViewModel: `SalatKuViewModel`.

## Versions (frozen, unchanged from the M12 template)

| Item | Version |
|---|---|
| Android Gradle Plugin | 9.4.1 |
| Kotlin | 2.3.21 |
| Compose BOM | 2026.09.00 |
| activity-compose / core-ktx | 1.13.0 / 1.19.1 |
| Gradle wrapper | 9.8.1 |
| compileSdk / targetSdk / minSdk | 37 / 36 / 26 |
| `lifecycle-runtime-ktx`, `lifecycle-viewmodel-compose`, **`lifecycle-runtime-compose` (new in M13)** | 2.11.0, all through `version.ref = "lifecycleRuntimeKtx"` |
| JUnit (tests only) | 4.13.2 |

## Which meeting contributed what

| Meeting | Contribution | Files |
|---|---|---|
| M07 | Compose basics, `PrayerCard` idea, project baseline (BOM, Material 3, activity-compose) | `app/build.gradle.kts`, `PrayerCard` in `HomeScreen.kt` |
| M08 | Static Home layout: `Scaffold`, hero card, prayer rows | `HomeScreen.kt` (`HeroCard`, `PrayerCard`), `MainActivity.kt` (`Scaffold`) |
| M09 | State hoisting, stateless screens, events up / state down | `TasbihScreen.kt`, `TrackerScreen.kt`, `MainActivity.kt` |
| M10 | `LazyColumn` with keys, `LazyRow` date strip, `LazyVerticalGrid` log, SalatKu colour scheme and typography | `HomeScreen.kt`, `DateStrip.kt`, `LogScreen.kt`, `ui/theme/*` |
| M11 | Accessibility semantics, 48dp targets, font-scale / dark / expanded previews, two-pane Home at width >= 840dp | `HomeScreen.kt`, `DateStrip.kt`, `TrackerScreen.kt`, `LogScreen.kt`, `MainActivity.kt` (`EXPANDED_WIDTH_DP`) |
| M12 | `SalatKuViewModel`, `viewModel()` in `ShalatkuApp`, round counter at 33 | `SalatKuViewModel.kt`, `MainActivity.kt` |
| HW12 | Part A: Clear all (`clearPrayers()`, `onClear`). Part B: `roundSize` (default 33), `setRoundSize(size)`, 33 / 99 / 100 chips, resets count and keeps completed rounds | `SalatKuViewModel.kt`, `TrackerScreen.kt`, `TasbihScreen.kt` |
| M13 CP1 | `SalatKuUiState`, `MutableStateFlow` + `update { copy() }`, `collectAsStateWithLifecycle()`, `lifecycle-runtime-compose` dependency | `SalatKuUiState.kt`, `SalatKuViewModel.kt`, `MainActivity.kt`, `gradle/libs.versions.toml`, `app/build.gradle.kts` |
| M13 CP2 | Derived `doneCount` on the state class, shown in the Tracker header (the pitfall demos are **not** in the source) | `SalatKuUiState.kt`, `TrackerScreen.kt`, `MainActivity.kt` |
| M13 CP3 (optional stretch in the lesson) | `SalatKuEvent` and one public `onEvent()` | `SalatKuEvent.kt`, `SalatKuViewModel.kt`, `MainActivity.kt` |
| M14 | No new code. Consolidation only. | (this project) |

The M07-M11 rows are inferred from the plan in `09/SalatKu-Development-Guide.md` and the comments in the code. The M12 template itself does not say which meeting wrote which composable.

Files that are **byte-identical** to the M12 template: `HomeScreen.kt`, `DateStrip.kt`, `LogScreen.kt`, `Prayer.kt`, `ui/theme/Color.kt`, `Theme.kt`, `Type.kt`, all `res/`, `settings.gradle.kts`, root `build.gradle.kts`, `gradle.properties`, the Gradle wrapper.

## Screens and signatures (final)

| Screen | Signature |
|---|---|
| Home | `HomeScreen(prayers, selectedDay, onSelectDay, modifier, expanded = false)` |
| Tracker | `TrackerScreen(prayers, donePrayers, doneCount, onToggle, onClear, modifier)` |
| Tasbih | `TasbihScreen(count, rounds, roundSize, onIncrement, onReset, onSelectRoundSize, modifier)` |
| Log | `LogScreen(cells, modifier)` |

`SalatKuViewModel` public surface: `uiState: StateFlow<SalatKuUiState>` and `onEvent(event: SalatKuEvent)`. The six action functions are `private`.

`ShalatkuApp(viewModel: SalatKuViewModel = viewModel())` is the only composable that knows the ViewModel. The selected bottom-bar tab stays in `ShalatkuApp` with `rememberSaveable` and is **not** in `SalatKuUiState`.

## Assumptions

1. **M13 CP3 is included.** The lesson marks it "optional stretch", but the request was for Checkpoints 1-3, so `SalatKuEvent` and `onEvent()` are in. To get the CP2 shape instead, make the six functions public again, delete `SalatKuEvent.kt` and use `viewModel::togglePrayer` etc. in `ShalatkuApp` (M13-Student.md section 3.4 and 4.3 show that wiring).
2. **HW12 UI is my own.** M12-Student.md section 7 only says "three buttons or chips" and gives no code. I used three Material 3 `FilterChip`s in a `Row`, driven by `private val roundSizeOptions = listOf(33, 99, 100)`. The selected chip is `selected = roundSize == size` (an expression, no `if`). The "Clear all" button is an `OutlinedButton` in the Tracker header item. Parameter names follow M13-Student.md exactly (`roundSize`, `onSelectRoundSize`, `onClear`).
3. **HW13 is not included.** No `PrayerTimesUiState`, `loadPrayerTimes()` or `PrayerTimesSection`. The request stops at M13 and HW12.
4. **The unit tests were migrated.** The M12 tests call functions that are now private, so `SalatKuViewModelTest.kt` reads `vm.uiState.value` and sends events (the M13 troubleshooting table describes this change). Four tests were added for HW12 and `doneCount`.
5. **`SalatKu-Development-Guide.md` (09) disagrees with the real scripts** for M12-M14 (it plans `HomeViewModel`, `HomeUiState`, `nextPrayer()` and `NextPrayer.kt`). The M12 and M13 scripts were followed, as requested, so none of that is here.
6. `kotlinx-coroutines` is not added to the catalog. M13-Student.md says not to add a version yourself, and it arrives transitively through the lifecycle libraries (see "Not fully verified" below).
7. The window-size-class library is still **not** used (same as the M12 template): the breakpoint is a plain `screenWidthDp >= 840` check.

## Known limits

- A ViewModel survives rotation, not process death (expected, M12 and M13).
- Dates in the date strip are fixed text (`Mon 20` to `Sun 26`).
- HW10 (colouring Log cells), HW11 audit fixes, navigation and any data layer are not part of this baseline.

See `NOTES.md` for the instructor notes on the two StateFlow pitfalls.
