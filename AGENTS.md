# General Guidelines

This is a Flutter and Dart mobile app project, the architecture is still ongoing, and many features are not implemented yet
**ALWAYS** use "question" or "askQuestions" tools if available, to ask the user questions

## App Architecture

This app uses a modularized monorepo architecture.
Each module is package on the packages/core/ or packages/features/ folder.

### Project Structure

```markdown
my-sample-app/                                  # Root workspace of the monorepo
├── packages/                                   # All modularized, independent Dart packages
│   ├── core/                                   # Foundational, domain-agnostic utilities (network, storage, navigation)
│   │   ├── <package_a>/
│   │   │   └── pubspec.yaml
│   │   ├── <package_b>/...
│   │   └── ui_kit/                             # Shared design system, themes, and reusable UI components
│   │       └── pubspec.yaml
│   │
│   └── features/                               # Vertical domain slices containing business logic and UI flows
│       ├── <feature_a>/                        # A standalone feature cluster containing its contracts and implementations
│       │   ├── <feature_a>_api/                # Optional - Exposed public interfaces, DTOs, and route definitions for cross-feature use
│       │   │   └── pubspec.yaml
│       │   ├── <feature_a>_internal_api/       # Optional - Shared logic, DI, and models strictly internal to this feature cluster
│       │   │   └── pubspec.yaml
│       │   ├── <feature_a>_<sub_feature_a>/    # Independent Clean Architecture sub-feature implementation packages
│       │   │   └── pubspec.yaml
│       │   └── <feature_a>_<sub_feature_b>/
│       │       └── pubspec.yaml
│       ├── <feature_b>/...
│       └── <feature_c>/...
│   
├── apps/                                       # Runnable application targets that assemble the packages
├   └── mobile_app/                             # The main Flutter app entry point that wires up dependencies and routing
│       ├── lib/
│       ├── test/
│       ├── android/
│       ├── ios/
│       └── pubspec.yaml
│
├── docs/                                       # Project documentation and architectural guidelines
│   └── specs/                                  # Features and requirements specs
│   └── adrs/                                   # Architecture Decision Records logging significant technical choices
│
├── tools/                                      # Custom helper scripts, CI/CD utilities
│
├── AGENTS.md                                   # AI assistant prompts and contributor guidelines
├── melos.yaml                                  # Melos workspace configuration for managing monorepo scripts and linking
├── analysis_options.yaml                       # Shared Dart linter rules enforced strictly across the entire workspace
└── pubspec.yaml                                # Root metadata and workspace-level tooling dependencies
```

### Feature Packages

Feature packages are placed at the packages/features/ folder, and may contain the **API** (api), **Internal API** (internal_api) and **Sub-Feature Implementation** packages.

Folder structure:

```markdown
my-sample-app/
└── packages/
    └── features/
        └── <feature_a>/
            ├── <feature_a>_api/
            ├── <feature_a>_internal_api/
            ├── <feature_a>_<sub_feature_a>/
            ├── <feature_a>_<sub_feature_b>/
            └── <feature_a>_<sub_feature_c>/
```

#### API Feature Packages

**API Packages**: Optional package - Defines the public contracts, interfaces, and models that allow a feature to interact with another feature without tight coupling.

```markdown
my-sample-app/packages/features/
└── <feature_a>_api/
    ├── lib/
    │   ├── <barrel_file_a>.dart
    │   ├── <barrel_file_b>.dart
    │   └── src/
    │       ├── models/
    │       ├── interfaces/
    │       ├── exceptions/
    │       └── routes/
    │
    └── pubspec.yaml
```

#### Internal API Feature Packages

**Internal API Packages**: Optional paclage - Internal cluster contracts, interfaces and models exclusively used across the sub-feature implementation packages within it's specific feature cluster.
They can only depend on it's own cluster api and general core packages.

```markdown
<feature_a>_internal_api/
├── lib/
│   ├── <barrel_file_a>.dart
│   ├── <barrel_file_b>.dart
│   └── src/
│       ├── models/
│       ├── interfaces/
│       ├── exceptions/
│       └── routes/
│
└── pubspec.yaml
```

#### Sub-Feature Implementation Feature Packages

**Sub-Feature Implementation Packages**: Sub-feature packages are the vertical, domain-specific slices of this app.
Each sub-feature package consists of isolated Clean Architecture layers (presentation, domain, data, infrastructure) that execute the business logic, state management, and UI for a specific feature or sub-feature.

They can only depend on other feature's APIs, core packages or it's own cluster internal API.

Each sub-feature package must also implement the Module abstract class (<feature_name>_module.dart file).

```markdown
<feature_a>/
└── <feature_a>_<sub_feature_a>/
    ├── lib/
    │   └── src/
    │       ├── data/
    │       │   ├── models/
    │       │   │   ├── request/
    │       │   │   ├── response/
    │       │   │   └── mappers/
    │       │   ├── datasources/
    │       │   │   ├── remote/
    │       │   │   └── local/
    │       │   └── repositories
    │       ├── domain/
    │       │   ├── entities/
    │       │   ├── usecases/
    │       │   ├── repositories/
    │       │   ├── validators/
    │       │   ├── value_objects/
    │       │   └── exceptions/
    │       ├── presentation/
    │       │   ├── widgets/
    │       │   │   └── entrypoints/
    │       │   ├── screens/
    │       │   ├── state/
    │       │   ├── formatters/
    │       │   └── routes/
    │       ├── infrastructure/
    │       │   ├── utils/
    │       │   ├── navigation/
    │       │   ├── di/
    │       │   ├── localization/
    │       │   └── analytics/
    │       └── <feature_a>_module.dart
    │       └── <feature_a>.dart                # barrel file, only exports <feature_a>_module.dart
    └── pubspec.yaml
```

### Core Packages

Core packages are horizontal, domain-agnostic foundational layers (like networking and storage) that provide shared infrastructure to the entire application without containing business logic or depending on feature packages.
You can create one or more barrel files, so the imports can be more specific.

Core packages can only import other core packages, as long as you avoid:

- Circular dependencies (e.g., core_a → core_b → core_a)
- Deep dependency chains (e.g., core_a → core_b → core_c → core_d → …)

```markdown
my-sample-app/packages/core/
└── <core_package>/
    ├── lib/
    │   ├── <barrel_file_a>.dart
    │   ├── <barrel_file_b>.dart
    │   └── src/
    │       └── ...
    └── pubspec.yaml
```

### Dependency Import Rules

- core → can import other core packages (no circular deps)
- \<feature>_api → can import core + \<feature>_apis
- \<feature>_internal_api → can import core + \<feature>_apis
- \<sub_feature> → can import core + \<feature>_apis + \<feature>_internal_apis
- Features never import other features — communicate via events, bus, streams
- core never imports any feature
- No circular dependencies at any level

## Development Guidelines

- Flutter 3.41 and Dart 3.11 (Search online for update documentation)
- BLoC for state management
- Do not use code generation - except for equatable and serializable (json_serializable) classes, avoid freezed, auto_route, injectable packages
- Follow D.R.Y., K.I.S.S., and Clean code patterns
- Do not use `flutter create` command
- Create packages manually
- Do not create ios android and other unnecessary folders for the packages
- Try to follow the suggested app architecture
- Mono-repo with multiple flutter projects with Melos

### Barrel files

- Every folder gets a barrel file
- Parent barrels composes the children
- Sub-feature Implementation Packages: the root barrel exposes only the module file (<feature_a>_module.dart)
- Core Packages: may use multi-entrypoint package barrels (scoped imports) to expose topic specific entrypoints or slices

### Imports

- Use direct style imports internally
- Use Package style imports for external packages`

## Modules System

ONGOING

## Dependency Injection and Registries

ONGOING

## App Startup

TODO

## Navigation

TODO

## Core packages

TODO

## Networking

TODO

## Authentication

TODO

## Automation, Workflows and Melos

TODO

## Design System

TODO

## Tests

TODO
