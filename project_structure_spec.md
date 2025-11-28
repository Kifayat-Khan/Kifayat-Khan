# Windserf Flutter Project Structure Specification

This document defines the required structure for any new Flutter project.  
Any AI agent generating or modifying code must follow this structure exactly.

---

## 1. Root Project Layout

```
project_name/
  pubspec.yaml
  analysis_options.yaml
  README.md
  .gitignore
  lib/
  assets/
  supabase/
```

Tool-generated folders (no app logic allowed):

```
  .dart_tool/
  .idea/ or .vscode/
  .metadata
  pubspec.lock
  *.iml
```

### Rules

- **pubspec.yaml**  
  Declare all dependencies, dev dependencies, assets, and fonts.

- **analysis_options.yaml**  
  Enforce global lint rules; all code must pass.

- **supabase/**  
  Must contain `schema.sql` for database definition.

- **assets/**  
  ```
  assets/icons/
  assets/svg_images/
  assets/fonts/
  ```
  Only static files go here; all must be referenced in `pubspec.yaml`.

---

## 2. Required `lib/` Structure

```
lib/
  main.dart
  bloc_providers.dart
  core/
  domain/
  data/
  presentation/
  files_backup/   (optional)
```

### Layer Responsibilities

- **core**  
  Shared building blocks used across the app.  
  Cannot depend on `presentation`.

- **domain**  
  Business rules and logic.  
  Only depends on `core`.  
  Must not depend on `data` implementations.

- **data**  
  Implements repositories and data sources.  
  Depends on `core` and `domain`.

- **presentation**  
  UI, pages, widgets, BLoCs.  
  Depends on all layers.

---

## 3. Entry Files

### `main.dart`
Responsible for:

- Flutter initialization
- Supabase and MediaKit setup
- App router setup
- App theme setup
- DI initialization via `core/di/injection_container.dart`
- Wrapping the app in `MultiBlocProvider`

Must not contain business logic.

### `bloc_providers.dart`
Contains all global BLoCs and Cubits.  
Feature-specific BLoCs belong in feature folders under `presentation/blocs`.

---

## 4. `core/` Folder Specification

```
lib/core/
  config/
  constants/
  di/
  errors/
  extensions/
  models/
  repositories/
  secrets/
  services/
  theme/
  usecases/
  utils/
  vilidation/
```

### Folder Responsibilities

- **config**  
  Router (`app_router.dart`), theme setup, and global configs.

- **constants**  
  App-wide constants (fonts, sizes, strings).

- **di**  
  Dependency injection configuration (`injection_container.dart`).

- **errors**  
  Base error and failure classes.

- **extensions**  
  Extensions for core types.

- **models**  
  Core models used across features.

- **repositories**  
  Base repository interfaces.

- **secrets**  
  Config wrappers for secret keys. No real keys hardcoded.

- **services**  
  Shared services (logging, analytics, global helpers).

- **theme**  
  Colors, typography, theming.

- **usecases**  
  Core reusable use cases.

- **utils**  
  Generic helpers.

- **vilidation**  
  Shared validation logic.

---

## 5. `domain/` Folder Specification

```
lib/domain/
  config/
  constants/
  errors/
  models/
  repositories/
  services/
  usecases/
  utils/
```

- Models define the business entities.
- Repositories define abstract contracts only.
- Use cases implement business operations.
- No UI code allowed.
- Depends on `core` only.

---

## 6. `data/` Folder Specification

```
lib/data/
  repositories/
  services/
  usecases/
  utils/
```

Responsibilities:

- Implement domain repository interfaces.
- Handle all API, Supabase, or local storage work.
- Map DTOs to domain models (mapping must stay here).
- Register data implementations in DI.

---

## 7. `presentation/` Folder Specification

```
lib/presentation/
  blocs/
  pages/
  widgets/
  config/
  constants/
  errors/
  models/
  repositories/
  services/
  usecases/
  utils/
```

### 7.1 BLoCs

```
lib/presentation/blocs/
  auth/
  user/
  content/
  favorites/
  subscription_package/
  audio_player/
  player/
  category_cubit/
  progress/
  upload_file_cubit/
  update_profile/
  update_notification/
  update_date_cubit/
```

Naming rules:

- `*_bloc.dart`
- `*_cubit.dart`
- `*_state.dart`
- `*_event.dart`

Logic must stay inside these folders, never inside widgets.

### 7.2 Pages

```
lib/presentation/pages/
  home/
  auth/
  subscription/
  player/
```

Rules:

- Every screen file ends with `_page.dart`  
  Examples:  
  - `home_page.dart`  
  - `login_page.dart`  
  - `subscription_page.dart`

### 7.3 Shared Widgets

```
lib/presentation/widgets/
  app_drawer.dart
  audio_player_widget.dart
  auth_text_field.dart
  category_card.dart
  plan_card.dart
```

Widgets must be reusable and as simple as possible.

### 7.4 Presentation Helpers

- `config/` – navigation helpers or page-specific config  
- `constants/`, `errors/`, `models/`, `repositories/`, `services/`, `usecases/`, `utils/`  
  - Contain only presentation-related logic  
  - Not business rules  

---

## 8. Naming Rules

- All Dart files must use `snake_case`.
- Required suffixes:
  - `_page.dart` for screens
  - `_bloc.dart`, `_cubit.dart`, `_state.dart`, `_event.dart`
  - `_model.dart`
  - `_repository.dart` (interfaces)
  - `_repository_impl.dart` (implementations)
  - `_service.dart`
  - `_usecase.dart`

---

## 9. Dependency Direction Rules

Allowed:

```
core → nothing
domain → core
data → domain & core
presentation → data, domain, core
```

Forbidden:

- No imports from `presentation` or `data` inside `domain`.
- No imports from `presentation` inside `core`.
- UI must never reach data sources directly; use DI and use cases.

---

## 10. Dependency Injection Rules

- Use `core/di/injection_container.dart` as the single DI entry point.
- Must register:
  - Repository implementations (from `data/repositories`)
  - Domain use cases
  - Core use cases
  - Shared services
- Feature-level DI should be grouped by feature inside this container.

---

## 11. Asset Rules

- All static files must be inside `assets/`.
- Subfolders:
  ```
  assets/icons/
  assets/svg_images/
  assets/fonts/
  ```
- All assets must be declared in `pubspec.yaml`.

---

## 12. Backup or Experimental Code

```
lib/files_backup/
```
Used only for temporary or old code.  
Production code must not depend on it.

---

## 13. New Feature Checklist

When adding a new feature:

### Domain
- Add or update models
- Add repository interfaces
- Add necessary use cases

### Data
- Implement repository interfaces
- Add DTO mappers
- Register implementations in DI

### Presentation
- Add BLoC/Cubit under `presentation/blocs/<feature_name>`
- Add pages under `presentation/pages/<feature_name>`
- Name screens using `_page.dart`
- Add shared widgets if needed

### Validate Dependencies
- No UI code in domain or data
- No data logic in presentation
- Respect allowed dependency direction

---

This file is the source of truth for structuring any new Flutter project.  
Any generated code or folder must follow these rules exactly.
