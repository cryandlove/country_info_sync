# TASKS — country_info_sync

This file follows the task template required by `AGENTS.md` (Section 5).
Each task must be planned, approved, implemented, and tested individually.
Do not start a task until the previous one is tested and confirmed working.

---

## Status

Phases 1–4 below are **implemented and verified**. Phase 5 was added later to
make the project publishable.

| Phase | Scope | Status |
|---|---|---|
| Phase 1 | Favorites — Hive box, service, UI toggle, filter | ✅ Done |
| Phase 2 | Compare — selection mode, button, screen, null handling | ✅ Done |
| Phase 3 | Dark mode — notifier, dark theme, wiring, toggle, persistence | ✅ Done |
| Phase 4 | UNESCO heritage — dataset, asset, model/service, details UI | ✅ Done |
| Phase 5 | Branding, app ID, release signing, CI/CD, docs | ✅ Done — see the end of this file |

Last verified with `flutter analyze` (no issues) and `flutter test`
(all passing) on Flutter 3.47.2.

## Project Facts (verified, not assumed)

- `Country` model has **no ISO code field** (no `cca2`/`cca3`). The only
  stable identifier available is `Country.name`. All tasks below use `name`
  as the identifier for favorites and comparison.
- `CountryAdapter` is **hand-written**, not generated via `build_runner`
  (no `hive_generator`/`build_runner` in `pubspec.yaml`). New Hive data in
  this project should stay on primitive types (`List<String>`, `bool`,
  `String`) to avoid needing a new adapter or new dependency.
- `pubspec.yaml` has **no `provider` package**. State in this project is
  plain `StatefulWidget`/`setState`, plus a single
  `ValueNotifier<ThemeMode>` for the theme. No state-management package is
  used anywhere.
- Existing Hive box: `countries_cache_v4`, typed `Box<Country>`, opened
  inside `CountryService.getCountries()`. (The name was bumped from `v3`
  when the JSON schema changed.)
- Other Hive boxes opened in `main.dart`: `favorites_box` (`Box<List>`),
  `insight_cache`, and `settings_box` (stores the saved theme mode).

---

## PHASE 1 — Favorites

### Task 1.1 — Create the favorites Hive box

**Goal:** Add a persistent, isolated place to store favorite country names.

**Scope:** New box only. No UI. No existing box touched.

**Files affected:**
- `lib/main.dart`

**Required changes:**
- After the existing `Hive.initFlutter()` / adapter registration in
  `main.dart`, add:
  `await Hive.openBox<List>('favorites_box');`
- Do not register a new adapter — `List<String>` is a Hive-native type.

**Expected result:** App starts with a new empty box ready to use.

**Acceptance criteria:**
- App builds and launches without errors.
- No change to existing `countries_cache_v4` behavior.

**Test procedure:**
- Run the app (`flutter run`). Confirm normal startup, country list loads
  as before.

**Risks:** None — additive only.

---

### Task 1.2 — FavoritesService

**Goal:** Provide a small service to read/write favorite country names.

**Scope:** One new file. No UI changes.

**Files affected:**
- `lib/services/favorites_service.dart` (new)

**Required changes:**
- Create `FavoritesService` with:
  - `Future<List<String>> getFavorites()`
  - `Future<void> addFavorite(String name)`
  - `Future<void> removeFavorite(String name)`
  - `Future<bool> isFavorite(String name)`
- Internally, open `Hive.box<List>('favorites_box')` (already opened in
  Task 1.1), read/write a single key, e.g. `'names'`, holding
  `List<String>`.
- Follow the existing pattern in `CountryService`: no external state
  library, plain async methods.

**Expected result:** A working, UI-independent favorites data layer.

**Acceptance criteria:**
- Calling `addFavorite('Italy')` then `isFavorite('Italy')` returns `true`.
- Calling `removeFavorite('Italy')` then `isFavorite('Italy')` returns
  `false`.

**Test procedure:**
- Temporarily call these methods from a debug button or breakpoint in
  `main.dart` and verify behavior via `print()` or debugger before wiring
  to real UI.

**Risks:** Low. Key collisions are impossible (single fixed key).

---

### Task 1.3 — Favorite toggle in the UI

**Goal:** Let the user mark/unmark a country as favorite from the list and
details screen.

**Scope:** UI only, using the service from Task 1.2.

**Files affected:**
- `lib/screens/home_screen.dart`
- `lib/screens/details_screen.dart`

**Required changes:**
- Add a heart icon (`Icons.favorite` / `Icons.favorite_border`) to each
  country card in `home_screen.dart` and to the app bar of
  `details_screen.dart`.
- On tap, call `FavoritesService.addFavorite`/`removeFavorite` using
  `country.name`, then update local widget state (`setState`) to reflect
  the new icon.

**Expected result:** Tapping the heart persists the favorite status.

**Acceptance criteria:**
- Favoriting a country in the list updates the icon immediately.
- Opening `details_screen` for that country shows it as already favorited.
- Fully closing and reopening the app preserves the favorite status.

**Test procedure:**
- Favorite 2–3 countries, force-close the app (not just hot reload),
  reopen, confirm all 2–3 are still marked favorite.

**Risks:** Low. Watch for `setState` calls after the widget is disposed
(e.g., if the user navigates away mid-write) — guard with `mounted` checks.

---

### Task 1.4 — "Favorites only" filter on the home screen

**Goal:** Let the user view only their favorited countries.

**Scope:** UI filter logic in `home_screen.dart` only.

**Files affected:**
- `lib/screens/home_screen.dart`

**Required changes:**
- Add a toggle (e.g., an `IconButton` in the `AppBar`, or a `Switch`) that
  filters the currently displayed list to only countries whose `name` is
  in the favorites list (fetched via `FavoritesService.getFavorites()`).
- This filter must compose with the existing search functionality (i.e.,
  filtering happens on top of search results, not instead of them).

**Expected result:** Toggling the filter shows only favorited countries;
toggling it off restores the full (or searched) list.

**Acceptance criteria:**
- With 0 favorites, the filtered view shows an empty state (not a crash).
- With N favorites, exactly N countries are shown when filtered.
- Search still works while the filter is active.

**Test procedure:**
- Favorite 2 countries, enable the filter, confirm exactly those 2 show.
- Type a search query for a non-favorited country while filter is on,
  confirm it does not appear.

**Risks:** Low. Make sure the filter re-evaluates when a favorite is
added/removed while the screen is open.

---

## PHASE 2 — Compare

### Task 2.1 — Selection mode on the home screen

**Goal:** Let the user long-press to select up to 2 countries for
comparison.

**Scope:** UI state only, no navigation yet.

**Files affected:**
- `lib/screens/home_screen.dart`

**Required changes:**
- Add local state: `List<String> selectedForCompare = [];` (stores
  `Country.name`).
- On long-press of a card: if not already selected and list length < 2,
  add it; if already selected, remove it; if list length is already 2 and
  a third is long-pressed, show a brief message (e.g., a `SnackBar`)
  explaining only 2 can be compared — do not silently replace a selection.
- Visually indicate selected cards (e.g., a border or background tint).

**Expected result:** User can select exactly 0, 1, or 2 cards with clear
visual feedback.

**Acceptance criteria:**
- Selecting a 3rd country shows the explanatory `SnackBar` and does not
  change the current selection.
- Long-pressing a selected card again deselects it.

**Test procedure:**
- Long-press 2 cards, confirm both are visually marked.
- Long-press a 3rd, confirm the `SnackBar` appears and selection stays at
  2.

**Risks:** Low.

---

### Task 2.2 — Compare button

**Goal:** Show a way to navigate to the comparison screen once 2 countries
are selected.

**Scope:** UI only.

**Files affected:**
- `lib/screens/home_screen.dart`

**Required changes:**
- Show a `FloatingActionButton` (or similar) labeled "Compare" **only**
  when `selectedForCompare.length == 2`.
- On tap, navigate to the new `compare_screen.dart` (Task 2.3), passing
  the two `Country` objects.

**Expected result:** Button appears/disappears correctly and navigates
with the right data.

**Acceptance criteria:**
- Button is absent with 0 or 1 selection.
- Button is present with exactly 2 selected.
- Tapping it navigates with both countries passed correctly.

**Test procedure:**
- Select 0, then 1, then 2 countries and confirm button visibility at each
  step.

**Risks:** Low.

---

### Task 2.3 — Compare screen

**Goal:** Show two countries' key stats side by side.

**Scope:** New screen only.

**Files affected:**
- `lib/screens/compare_screen.dart` (new)

**Required changes:**
- Accept two `Country` objects via constructor.
- Display a two-column layout with rows for: `population`, `area`,
  `capital`, `currencies`, `languages`, `subregion`, `continent`,
  `timezones`.
- Use the existing app theme/typography (Inter + Vazirmatn via
  `google_fonts`) for consistency with the rest of the app.

**Expected result:** A readable side-by-side comparison screen.

**Acceptance criteria:**
- All 8 fields render for both countries in aligned rows.
- Long text (e.g., many currencies) wraps instead of overflowing.

**Test procedure:**
- Compare two data-rich countries (e.g., Germany and Brazil) and visually
  confirm alignment and readability on a real device/simulator.

**Risks:** Low. Watch for `FittedBox`/overflow issues with long
`currencies`/`languages` lists, consistent with the adaptive-text pattern
already used elsewhere in the app.

---

### Task 2.4 — Handle missing/null fields gracefully

**Goal:** Prevent crashes or blank rows when a country lacks certain data.

**Scope:** `compare_screen.dart` only.

**Files affected:**
- `lib/screens/compare_screen.dart`

**Required changes:**
- For nullable fields (`capital`, `area`) and empty lists (`currencies`,
  `languages`, `timezones`), display `"—"` instead of `null`, `"null"`, or
  an empty string.

**Expected result:** No crash and no confusing blank/null text regardless
of which two countries are compared.

**Acceptance criteria:**
- Comparing a country with a `null` `capital` or `area` shows `"—"` in
  that row, not an error.

**Test procedure:**
- Find and compare a country in the dataset with a missing `capital` or
  `area` (if none exists in practice, temporarily pass a `Country` object
  with those fields null during testing) against a fully-populated
  country.

**Risks:** Low.

---

## PHASE 3 — Dark Mode

### Task 3.1 — Theme mode notifier

**Goal:** Introduce a way to track and change the current theme mode
without adding a state-management package.

**Scope:** New minimal state holder.

**Files affected:**
- `lib/main.dart`

**Required changes:**
- Add: `final ValueNotifier<ThemeMode> themeNotifier = ValueNotifier(ThemeMode.light);`
  at the top level (or inside the root widget's state), using plain
  Flutter — no new dependency.

**Expected result:** Compiles with no behavior change yet.

**Acceptance criteria:** App builds and runs exactly as before.

**Test procedure:** `flutter run`, confirm no visual or behavioral change.

**Risks:** None.

---

### Task 3.2 — Define the dark theme

**Goal:** Add a dark `ThemeData` alongside the existing light theme.

**Scope:** Theme definitions only.

**Files affected:**
- `lib/main.dart`

**Required changes:**
- Define `darkTheme` using
  `ColorScheme.fromSeed(seedColor: <existing seed>, brightness: Brightness.dark)`,
  reusing the same seed color as the current light theme for visual
  consistency, plus the existing Inter + Vazirmatn text theme.

**Expected result:** A valid dark `ThemeData` object exists but is not yet
wired to the app.

**Acceptance criteria:** Compiles with no runtime change yet.

**Test procedure:**
- Temporarily hardcode `themeMode: ThemeMode.dark` in `MaterialApp` to
  visually check contrast/readability on `home_screen` and
  `details_screen`, then revert this temporary change before Task 3.3.

**Risks:** Low. Check that card backgrounds and flag images still look
correct against a dark background.

---

### Task 3.3 — Wire theme mode to MaterialApp

**Goal:** Make the app actually switch between light/dark based on
`themeNotifier`.

**Scope:** Root widget only.

**Files affected:**
- `lib/main.dart`

**Required changes:**
- Wrap `MaterialApp` in a `ValueListenableBuilder<ThemeMode>` listening to
  `themeNotifier`, passing `theme: lightTheme`, `darkTheme: darkTheme`,
  `themeMode: currentValue`.

**Expected result:** Changing `themeNotifier.value` anywhere in the app
switches the whole app's theme live.

**Acceptance criteria:** With default value `ThemeMode.light`, app looks
identical to before this phase.

**Test procedure:** `flutter run`, confirm app is in light mode by
default.

**Risks:** Low.

---

### Task 3.4 — Theme toggle button

**Goal:** Let the user switch themes from the UI.

**Scope:** `home_screen.dart` only.

**Files affected:**
- `lib/screens/home_screen.dart`

**Required changes:**
- Add a sun/moon `IconButton` in the `AppBar` that toggles
  `themeNotifier.value` between `ThemeMode.light` and `ThemeMode.dark`.

**Expected result:** Tapping the icon instantly switches the whole app's
theme.

**Acceptance criteria:** Toggling affects `home_screen` and
`details_screen` (and `compare_screen`, if Phase 2 is already done)
consistently.

**Test procedure:** Toggle the icon, navigate between screens, confirm the
theme stays consistent everywhere.

**Risks:** Low.

---

### Task 3.5 — Persist theme choice

**Goal:** Remember the user's theme choice across app restarts.

**Scope:** Small addition using existing Hive infrastructure.

**Files affected:**
- `lib/main.dart`
- `lib/screens/home_screen.dart`

**Required changes:**
- Open a small Hive box (e.g., `Hive.openBox('settings_box')`) at startup.
- On toggle (Task 3.4), write the chosen mode as a `String`
  (`'light'`/`'dark'`) to a fixed key, e.g. `'theme_mode'`.
- On app start, read this key before building `MaterialApp` and set
  `themeNotifier`'s initial value accordingly (default to `light` if
  unset).

**Expected result:** Theme choice survives a full app restart.

**Acceptance criteria:** Set dark mode, fully close and reopen the app,
confirm it opens in dark mode.

**Test procedure:** As above — force-close (not hot reload), then reopen.

**Risks:** Low. Keep this box separate from `favorites_box` for clarity,
per the "no unrelated changes" rule.

---

## PHASE 4 — Heritage Sites (text-only, offline-bundled)

### Task 4.1 — Prepare the offline dataset

**Goal:** Create a small, hand-curated JSON file of heritage sites (start
with ~20–30 well-known sites, not the full ~1,273, to keep early testing
fast).

**Scope:** Data file only, no code.

**Files affected:**
- `assets/data/heritage_sites.json` (new)

**Required changes:**
- Create JSON in the form:
  ```json
  [
    {
      "name": "Colosseum",
      "countryName": "Italy",
      "description": "Ancient Roman amphitheater completed in 80 AD.",
      "lat": 41.8902,
      "lng": 12.4922
    }
  ]
  ```
- **Note:** use `countryName` (matching `Country.name`) as the linking
  field, since there is no ISO code in the `Country` model (see Project
  Facts above).

**Expected result:** A valid, small JSON file ready to bundle.

**Acceptance criteria:** File passes JSON validation (e.g., paste into any
online JSON validator, or `python -m json.tool heritage_sites.json`).

**Test procedure:** Validate JSON syntax before proceeding.

**Risks:** None — no app code touched yet.

---

### Task 4.2 — Register the asset

**Goal:** Make the JSON file available to the app at runtime.

**Scope:** `pubspec.yaml` only.

**Files affected:**
- `pubspec.yaml`

**Required changes:**
- Under `flutter: assets:`, add `- assets/data/heritage_sites.json`
  (alongside the existing `- assets/images/` entry).

**Expected result:** Asset is bundled with the app.

**Acceptance criteria:** `flutter pub get` runs clean; app still launches.

**Test procedure:** `flutter pub get && flutter run`, confirm no errors
(the asset isn't used yet, so behavior is unchanged).

**Risks:** None.

---

### Task 4.3 — Model and service

**Goal:** Load and expose heritage site data per country.

**Scope:** Two new files.

**Files affected:**
- `lib/models/heritage_site.dart` (new)
- `lib/services/heritage_service.dart` (new)

**Required changes:**
- `HeritageSite`: plain Dart class (no Hive adapter needed — this is
  static bundled data, not something the app writes) with `name`,
  `countryName`, `description`, `lat`, `lng`.
- `HeritageService.getSitesForCountry(String countryName)`: loads the
  JSON once via `rootBundle.loadString('assets/data/heritage_sites.json')`,
  parses it, caches the parsed list in memory (a simple static/instance
  field) so the file is read only once per app session, then filters by
  `countryName`.

**Expected result:** A working, testable data layer for heritage sites.

**Acceptance criteria:**
- `getSitesForCountry('Italy')` returns the matching site(s) from the
  sample data.
- `getSitesForCountry('NonexistentCountry')` returns an empty list, not an
  error.

**Test procedure:** Call both cases from a temporary debug print/
breakpoint before wiring to UI.

**Risks:** Low. Make sure a malformed JSON entry doesn't crash the whole
parse — wrap parsing in a way that skips a bad entry rather than failing
the entire list (explain this choice if implemented, per AGENTS.md
Section 14 on documenting non-obvious decisions).

---

### Task 4.4 — Display in Details screen

**Goal:** Show heritage sites for the currently viewed country, only when
data exists.

**Scope:** UI addition, conditional.

**Files affected:**
- `lib/screens/details_screen.dart`

**Required changes:**
- Add a "Heritage Sites" section at the end of the details layout that
  calls `HeritageService.getSitesForCountry(country.name)`.
- If the returned list is empty, render nothing (no section, no empty
  placeholder) — do not show a confusing empty state for the ~97% of
  countries without curated data yet.
- If non-empty, list each site's `name` and `description`.

**Expected result:** Countries with data show the section; all others
look exactly as before.

**Acceptance criteria:**
- Opening Italy's details screen shows the Colosseum entry (or whichever
  sample data was added for it).
- Opening a country with no heritage data shows no trace of the section.

**Test procedure:** Open both a country with sample data and one without,
confirm the difference described above.

**Risks:** Low.

---

## Suggested Execution Order

Phase 1 (Favorites) → Phase 2 (Compare) → Phase 3 (Dark Mode) → Phase 4
(Heritage). Do not start a later phase's Task 1 until the previous
phase's final task has been tested and confirmed, per `AGENTS.md`
Section 5 ("One Task at a Time").

---

## PHASE 5 — Branding, Release & Tooling

This phase was not in the original plan; it was required to make the
project safe to publish.

### Task 5.1 — App identity

**Goal:** Replace every `flutter create` placeholder with the real identity.

**Files affected:** `pubspec.yaml`, `android/app/src/main/AndroidManifest.xml`,
`android/app/build.gradle.kts`, `web/index.html`, `web/manifest.json`,
`macos/Runner/Configs/AppInfo.xcconfig`, `windows/runner/Runner.rc`,
`linux/CMakeLists.txt`, the iOS/macOS `project.pbxproj`, and `MainActivity.kt`.

**Result:**
- Display name on every platform: `Country Info Sync`.
- Android / Linux application ID: `io.github.moradzadeh67.country_info_sync`.
- iOS / macOS bundle ID: `io.github.moradzadeh67.countryInfoSync` —
  underscores are not legal in Apple bundle identifiers.
- `MainActivity` moved to the matching package directory.
- No `com.example` string remains anywhere in the repository.

**Breaking change:** the Android application ID changed, so users on `v1.0.x`
must uninstall the old build first — Android treats it as a different app and
will refuse the install otherwise.

### Task 5.2 — Release signing

**Goal:** Ship an APK signed with a real upload key without ever committing it.

**Files affected:** `android/app/build.gradle.kts`, `.gitignore`.

**Result:**
- Keystore lives outside the repo at `~/keystores/country_info_sync.jks`.
- Credentials are read from `android/key.properties`, which is git-ignored,
  along with `*.jks` and `*.keystore`.
- If that file is missing, Gradle falls back to the debug key, so anyone who
  clones the project can still build.

### Task 5.3 — CI and live web demo

**Files affected:** `.github/workflows/ci.yml`, `.github/workflows/deploy-web.yml`.

**Result:**
- `ci.yml` runs `flutter analyze` and `flutter test` on Flutter 3.47.2 for
  every push and PR to `main`.
- `deploy-web.yml` builds the web bundle and publishes it to GitHub Pages at
  <https://moradzadeh67.github.io/country_info_sync/>.

### Task 5.4 — Documentation

**Files affected:** `README.md`, `CONTRIBUTING.md`, `PROJECT_SPEC.md`,
`.github/ISSUE_TEMPLATE/bug_report.md`.

**Result:** the README now documents release signing and CI/CD, and
`CONTRIBUTING.md` gained the project-specific conventions plus links to the
other documents.

### Task 5.5 — Landing page and discoverability

**Goal:** make the published site something a search engine can actually read.

**Files affected:** `landing/index.html` (new), `landing/robots.txt` (new),
`landing/sitemap.xml` (new), `landing/og-image.svg` + `landing/og-image.png`
(new), `.github/workflows/deploy-web.yml`.

**Why:** a Flutter web build renders into a canvas and exposes almost no text,
so a crawler sees only the document title and meta description. The landing
page is the fix.

**Result:**
- The site root now serves a static HTML landing page with a keyword-rich title
  and description, Open Graph and Twitter cards, JSON-LD
  `SoftwareApplication` structured data, feature copy, screenshots with alt
  text, and an FAQ.
- The Flutter app moved to `/app/`, built with
  `--base-href /country_info_sync/app/`.
- `robots.txt`, `sitemap.xml` and a 1200×630 `og-image.png` are published
  alongside it.
- `deploy-web.yml` assembles all of this into `_site/` before upload.
- A 30-second demo GIF (`assets/demo/demo.gif`) was recorded on the Android
  emulator with `adb shell screenrecord` and encoded with `ffmpeg`. It is
  embedded in both the README and the landing page. Note that `screenrecord`
  outputs 720x1280 regardless of the device aspect ratio, so the frame is
  rescaled back to the real proportions during encoding.

---

## Known follow-ups

Found while auditing the docs against the code, not yet fixed:

- `lib/screens/onboarding_screen.dart` is an unused 15-line stub that is not
  referenced from `main.dart`, and it loads `assets/logo.png`, which does not
  exist. The real assets live in `assets/images/`. Using this screen as-is
  would crash.
- `lib/services/country_service.dart` carries the comment "Incremented to v4
  for alpha2Code schema", but `Country` has no `alpha2Code` field. The comment
  is misleading.
