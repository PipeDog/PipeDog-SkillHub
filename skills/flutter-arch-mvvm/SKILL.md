---
name: flutter-arch-mvvm
description: "Use when designing, implementing, refactoring, or reviewing Flutter projects that use application/business/capability/foundation layering, module-local MVVM, route-based business isolation, ViewModel/Repository/Model boundaries, reusable capability APIs, open-source-grade foundation APIs, UI isolation, state ownership, naming, layout spacing, or architecture-focused code review."
---

# Flutter Arch MVVM

## Overview

Use this skill to design, refactor, implement, or review Flutter projects with four-layer architecture and module-local MVVM. Keep the structure strict enough for team consistency, but avoid abstractions that do not protect a real boundary.

## Core Standard

Default dependency direction:

```text
application -> business -> capability -> foundation
```

Inside each business module:

```text
View -> ViewModel -> Repository -> Model
```

Layer intent:

- `application`: app entry, routing, environment, global configuration, concrete launch tasks, root shell, and cross-module assembly. Launch task ordering belongs in `main` or the chosen launch-task framework, not in the task directory itself.
- `business`: project-specific feature modules and screen flows.
- `capability`: company or product reusable capabilities that may be customized for this project; migration to another company may require adaptation.
- `foundation`: project-independent capabilities that should theoretically be open-source-ready and reusable in any Flutter project.

Mandatory SDK design rule:

- Treat every `foundation` module as an independent public SDK. Design names, APIs, contracts, errors, side effects, and module boundaries from that module's own capability model, not from the current app, project, product, screen, workflow, or business scenario. `foundation` must not contain project-specific naming, product terms, business assumptions, or convenience APIs created for the current project.
- Treat every `capability` module as an independent internal SDK. It may target company/product reusable capabilities, but it must still be designed from the capability module's own stable model instead of the current project's immediate needs. Do not let current project flows, screens, campaigns, or one-off integrations define capability names or public APIs.
- When a current-project need does not fit the SDK's natural boundary, compose existing APIs in `business` or `application` instead of expanding `capability` or `foundation` with project-shaped shortcuts.

`capability` and `foundation` must isolate UI from functional capabilities. Put reusable UI primitives, design-system widgets, or capability-level UI surfaces under explicit `ui/` areas instead of mixing them into modules such as pay, network, db, share, or router.

Read `references/architecture.md` when you need the baseline directory template, migration rules, naming rules, layout spacing rules, review priorities, or code examples. The template is explanatory, not exhaustive; add directories when they have a clear responsibility and obey the layer boundaries.

## Working Modes

### Implement or Refactor

1. Inspect the current Flutter project first: `pubspec.yaml`, `lib/`, routes, state management, existing layers, and module boundaries.
2. Confirm naming intent before changing layer names or directory structure. Current standard is `application -> business -> capability -> foundation`.
3. Classify every new or moved file by layer and by role: functional capability, UI capability, business View, ViewModel, Repository, Model, or application orchestration.
4. Prefer small, stable, composable APIs. Do not add one-off business shortcuts to `capability` or `foundation`.
5. Design `foundation` modules as independent SDKs with names and APIs derived from the module itself, with no company, product, project, screen, workflow, or business assumptions.
6. Design `capability` modules as independent internal SDKs with names and APIs derived from the capability itself; keep project-specific customization behind stable interfaces.
7. Keep ViewModel as the state and page-logic entry point. It must not hold `BuildContext`, create widgets, or directly operate UI.
8. Keep Repository as the data access and aggregation boundary. It must not contain page business decisions.
9. Keep sibling `business` modules isolated. Use routes, explicit application orchestration, or promoted public capabilities.
10. Add or update concise comments for exported APIs, business functions, and non-trivial state or side-effect logic.

### Architecture Review

Lead with findings, ordered by severity, with file and line references when available. Prioritize:

1. Dependency direction violations.
2. `foundation` APIs, names, contracts, or modules that are shaped by the current project instead of an independent public SDK model.
3. `capability` APIs, names, contracts, or modules that are shaped by current project flows instead of an independent internal SDK model.
4. `foundation` containing project, company, product, or business coupling.
5. `capability` and `foundation` mixing UI with functional modules.
6. Direct imports between sibling `business` modules.
7. ViewModel leaking UI concerns: `BuildContext`, widgets, Navigator/Dialog/Toast operations.
8. Repository containing page decisions or UI/display state.
9. Duplicate state, hidden mutable state, or unclear state mutation entry points.
10. Naming, directory, layout spacing, and missing API comments.

If no issues are found, say so clearly and mention remaining verification gaps.

### Architecture Planning

When planning a migration or new module layout, provide:

- Target layer/module placement.
- Public APIs and what they intentionally exclude.
- State ownership and derived-state rules.
- Route or orchestration boundaries between business modules.
- UI isolation plan for `capability/ui` and `foundation/ui`.
- Incremental migration steps that avoid broad unrelated rewrites.
- Verification plan: analyzer, tests, dependency checks, and focused manual UI checks when relevant.

## Placement Rules

Use these quick decisions before creating files:

| Question | Placement |
| --- | --- |
| Is it global startup, routing, environment, theme, root shell, or cross-module orchestration? | `application/` |
| Is it a concrete startup task such as auth restore, cache warmup, remote config, or SDK initialization? | `application/launch_task/` |
| Is it a project-specific feature, page, or screen flow? | `business/` |
| Is it company/product reusable but project-customized, such as pay/share/push/update/analytics? | `capability/` |
| Is it reusable across arbitrary Flutter projects and theoretically open-source-ready? | `foundation/` |
| Is it reusable UI tied to company/product design rules? | `capability/ui/` |
| Is it generic UI infrastructure or primitives with no project/company coupling? | `foundation/ui/` |

If a workflow is one-off, keep it in the business orchestration layer instead of promoting it to a reusable API.

## Finish Check

Before finishing, check:

- Dependency direction: `application -> business -> capability -> foundation`.
- `foundation` modules read as independent public SDKs with no current-project naming, assumptions, or shortcuts.
- `capability` modules read as independent internal SDKs with capability-shaped names, APIs, and contracts.
- UI isolation in both `capability` and `foundation`.
- MVVM role boundaries and state ownership.
- API simplicity, composability, comments, and side-effect clarity.
- Naming and directory conventions.
- Page, section, and scroll-container spacing ownership.
- Analyzer, tests, or a clear reason they were not run.
