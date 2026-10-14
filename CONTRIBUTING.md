# Contributing to Country Info Sync

Thank you for your interest in contributing to Country Info Sync! We welcome contributions from developers of all skill levels. Whether you are fixing a bug, improving documentation, or proposing a new feature, your help is greatly appreciated.

Before you start, these files are worth a read:

- [`AGENTS.md`](AGENTS.md) — the rules this codebase is built and reviewed against.
- [`PROJECT_SPEC.md`](PROJECT_SPEC.md) — what the app is and what it is made of.
- [`TASKS.md`](TASKS.md) — the phase-by-phase plan the app was built from.

---

## 🚀 Getting Started

To set up your local development environment:

1. **Fork the repository** on GitHub.
2. **Clone your fork** locally:

   ```bash
   git clone https://github.com/YOUR_USERNAME/country_info_sync.git
   cd country_info_sync
   ```

3. **Fetch dependencies**:

   ```bash
   flutter pub get
   ```

4. **Verify your environment**:

   ```bash
   flutter doctor
   ```

This project targets **Flutter 3.47.2 (stable)** and **Dart ^3.13.2**.

---

## 🐛 Opening an Issue

Before creating a new issue, please search the existing issues to ensure it hasn't already been reported.

- **Bug Reports**: Include clear steps to reproduce, expected vs. actual behavior, screenshots, and environment details (Flutter version, OS).
- **Feature Requests**: Explain the problem your proposed feature solves, alternative solutions considered, and any relevant context.

Templates are available here:

- [Bug report](.github/ISSUE_TEMPLATE/bug_report.md)
- [Feature request](.github/ISSUE_TEMPLATE/feature_request.md)

---

## 🔀 Submitting a Pull Request (PR)

1. **Create a branch** from the default branch (usually `main`):

   ```bash
   git checkout -b feat/add-country-chart
   # or
   git checkout -b fix/search-case-sensitivity
   ```

2. **Branch Naming Conventions**:
   - `feat/` — New feature
   - `fix/` — Bug fix
   - `docs/` — Documentation changes
   - `refactor/` — Code refactoring
   - `chore/` — Build or toolchain updates

3. **Commit Messages** — Follow [Conventional Commits](https://www.conventionalcommits.org/):
   - `feat: add country comparison export`
   - `fix: resolve search case sensitivity issue`
   - `docs: update CONTRIBUTING.md`

4. **Submit your PR**:
   - Push your branch to your fork and create a Pull Request against the default branch.
   - Fill out the PR template completely, referencing any related issue numbers (e.g., `Closes #12`).

---

## 🎨 Coding Standards

Please ensure your code adheres to standard Dart and Flutter guidelines:

- Follow the official [Effective Dart](https://dart.dev/guides/language/effective-dart) style guide.
- Run static analysis and formatting before committing:

  ```bash
  flutter analyze
  dart format .
  ```

- Keep code clean, readable, and appropriately commented where logic is non-obvious.

### Project-specific rules

This codebase is deliberately minimal — please help keep it that way:

- **No new dependencies unless truly necessary.** See the *Technical Decisions* section of the README for the reasoning behind each choice.
- **No external state management.** Plain `StatefulWidget` + `setState`, plus a single `ValueNotifier<ThemeMode>` for the theme. No Provider, BLoC, or Riverpod.
- **No code generation.** Do not add `build_runner`, `hive_generator`, or `freezed`. The Hive adapter is hand-written in `lib/models/country.dart`.
- **Use the design tokens.** Reach for `AppSpacing` and `AppTypography` instead of hard-coded pixel values, so the UI keeps scaling across screen sizes.
- **Keep the font fallback intact.** Latin text uses Inter; Persian/Arabic glyphs fall back to Vazirmatn. Please do not break mixed-script rendering.
- **Never commit secrets.** `android/key.properties`, `*.jks`, and `*.keystore` are git-ignored. Keep it that way — see the *Release Signing* section of the README.
- **The site root is a hand-written landing page.** `landing/index.html` is plain HTML that search engines can read; the Flutter web build is served from `/app/`. Keep the landing page's copy, `alt` text, and structured data in sync when you add or rename a feature.

---

## 🧪 Running Tests

Always ensure existing tests pass before submitting a Pull Request:

```bash
flutter test
```

If you are adding a new feature, please include corresponding unit or widget tests whenever feasible.

CI already runs `flutter analyze` and `flutter test` on Flutter 3.47.2 for every push and pull request to `main`, so both must be green before a PR can be merged.

---

## 📜 Code of Conduct

Country Info Sync is an open, welcoming community. We expect all contributors to:
- Be respectful, courteous, and constructive in all interactions.
- Focus on what is best for the community and the project.
- Respect differing viewpoints and constructive feedback.

Instances of unacceptable behavior may be reported to the project maintainers.

---

## 📄 License

By contributing, you agree that your contributions are licensed under the [MIT License](LICENSE).
