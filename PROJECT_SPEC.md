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
- **Framework**: Flutter (v3.13.2+)
- **Language**: Dart (v3.x)
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
- **Performance**: Fast loading, offline resilience, and smooth 60fps scrolling across Android, iOS, macOS, and Web.
