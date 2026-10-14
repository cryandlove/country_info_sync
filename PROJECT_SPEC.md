# Project Specification: Country Info Sync

## 1. Overview
Country Info Sync is a professional, offline-first mobile and desktop application built with Flutter that provides users with up-to-date global country information, Wikipedia summaries, and curated UNESCO World Heritage sites.

## 2. Core Features
- **Fetch Country List**: Retrieve country data from a public API (`https://countries.dev/countries`).
- **Country Details**: Display detailed information for each country (Flag, Population, Capital, Continent, Languages, Currencies, Area, Timezones, Calling Code, Borders).
- **Offline-First Caching**: Store fetched data locally using Hive to ensure full availability without an internet connection, supported by a bundled asset fallback (`countries.json`).
- **Smart Search & Filtering**: Case-insensitive search by country name or capital city, combined with a persistent "Favorites Only" list toggle.
- **Side-by-Side Comparison**: Select up to 2 countries via long-press to compare key metrics side by side on a dedicated screen.
- **UNESCO World Heritage Sites**: View curated UNESCO sites per country with site descriptions loaded from an offline JSON dataset (`heritage_sites.json`).
- **Wikipedia Summaries**: Rich "About <country>" cards powered by the Wikipedia REST API with offline summary caching.
- **Dark & Light Theme**: Live theme switching with persistent storage across app restarts.
- **Localization & Font Fallbacks**: Latin text uses **Inter**; Persian/Arabic glyphs automatically fall back to **Vazirmatn** for flawless mixed-string rendering.

## 3. Technology Stack
- **Framework**: Flutter (v3.47.2, stable channel)
- **Language**: Dart (^3.13.2)
- **Networking**: `http` package
- **Local Storage**: `hive` and `hive_flutter`
- **Fonts**: `google_fonts` (Inter + Vazirmatn)
- **Image Caching**: `cached_network_image`
- **External Links**: `url_launcher`
- **Architecture**: Plain `StatefulWidget` + `setState` and `ValueNotifier<ThemeMode>` (Zero-boilerplate minimal architecture following `AGENTS.md`).

## 4. User Interface (UI) & Design System
- **Home Screen**: List of countries with flags, capitals, smart search, favorite filter, theme toggle, and long-press selection for comparison.
- **Details Screen**: Multi-section view (General Info, Communication, Detailed Records, Wikipedia Summary, UNESCO Heritage Sites) with Hero flag animations.
- **Compare Screen**: Two-column side-by-side comparison matrix with adaptive text wrapping and graceful null handling.
- **Adaptive Design System**: Responsive spacing derived proportionally via `AppSpacing` and `MediaQuery`.
- **Error & Loading States**: Shimmer placeholders and empty state views.

## 5. Non-Functional Requirements
- **Simplicity**: Avoid over-engineering. Zero unnecessary abstractions or code generation.
- **Maintainability**: Clean, readable, hand-written Hive adapters and modular services.
- **Performance**: Fast loading, offline resilience, and smooth 60fps scrolling on Android, iOS, macOS, and Web.
- **Security**: No API keys are required to run the app. Signing material (`android/key.properties`, `*.jks`, `*.keystore`) is git-ignored and never committed.

## 6. Identity & Platforms

- **Display name (all platforms)**: `Country Info Sync`
- **Android / Linux application ID**: `io.github.moradzadeh67.country_info_sync`
- **iOS / macOS bundle ID**: `io.github.moradzadeh67.countryInfoSync` (Apple bundle IDs cannot contain underscores)
- **Current version**: `1.1.0+2`

| Platform | Status |
|---|---|
| Android | ✅ Tested — `minSdk 24`, `targetSdk 36` |
| iOS | ✅ Tested |
| macOS | ✅ Tested — Apple Silicon |
| Web | ✅ Tested |
| Windows | ⚠️ Untested — build available |
| Linux | ⚠️ Untested — build available |

## 7. Release & CI/CD

- **Signing**: release builds are signed with an upload key kept outside the repository at `~/keystores/country_info_sync.jks`. Credentials are read from `android/key.properties`, which is git-ignored. If that file is absent, Gradle falls back to the debug key so a fresh clone still builds.
- **CI** (`.github/workflows/ci.yml`): runs `flutter analyze` and `flutter test` on Flutter 3.47.2 for every push and PR to `main`.
- **Published site** (`.github/workflows/deploy-web.yml`): assembles and publishes the site to GitHub Pages — <https://moradzadeh67.github.io/country_info_sync/>
- **Repository**: <https://github.com/moradzadeh67/country_info_sync> — MIT licensed, public.

## 8. Web Presence & Discoverability

The published GitHub Pages site is split in two so that search engines have something to read:

- `/` — a static, crawlable HTML landing page (`landing/index.html`) with a keyword-rich title and meta description, Open Graph and Twitter cards, JSON-LD `SoftwareApplication` structured data, feature copy, screenshots with alt text, and an FAQ. A Flutter web build renders into a canvas and exposes almost no text to crawlers, so this page is the part that gets indexed.
- `/app/` — the Flutter web app, built with `--base-href /country_info_sync/app/`.

`robots.txt`, `sitemap.xml` and a 1200×630 `og-image.png` are published alongside them. A 30-second demo GIF (`assets/demo/demo.gif`), recorded on an Android emulator, is embedded in both the README and the landing page.

Metadata that lives on GitHub rather than in the repository: the repository description, the topic list, and the social preview image (Settings → Social preview).

## 9. Related Documents

- [`AGENTS.md`](AGENTS.md) — the rules the codebase is built and reviewed against.
- [`TASKS.md`](TASKS.md) — the task-by-task plan, phase by phase.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — how to set up and contribute.
- [`README.md`](README.md) — user-facing overview, screenshots, and setup.
