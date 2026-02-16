+++
title = "Multimodule Clean Architecture"
description = "Exploring clean architecture patterns in multi-module Android projects"
date = 2025-01-15

[taxonomies]
tags = ["kotlin", "android", "architecture", "clean-architecture"]

[extra]
draft = false
+++

**Premise:**

The general guidelines for "clean architecture" neglect the complexities of a multi-module environment. Therefore, I've found each multi-module project attempts its own flavour, many of which reveal common pitfalls. This document outlines some of the mistakes I've personally made over the years and my present best advice.

## Quick Reference: Gradle Module Dependency Graph

A detailed structural hierarchy & description of module contents is provided at the bottom of this document.

```
flowchart LR
    app(":app") --> screen1 & data-data

    subgraph core
      core-util(":core:util")
      core-domain(":core:domain")
      core-theme(":core:theme")
      core-ui(":core:ui")
      core-xml(":core:xml")

      core-domain --> core-util
      core-theme --> core-domain
      core-ui & core-xml --> core-theme
    end

    subgraph screen1
      screen1-ui(":screen1:ui")
      screen1-domain(":screen1:domain")

      screen1-ui --> screen1-domain
      screen1-ui --> core-ui
      screen1-ui --> core-xml
      screen1-domain --> core-domain
    end

    subgraph data
      data-data(":data")
      data-network(":data:network")
      data-database(":data:database")
      data-datastore(":data:datastore")

      data-data --> data-network & data-database & data-datastore & core-domain & screen1-domain
    end
```

## Benefits

The benefits of Gradle enforced clean architecture have significant and immediate advantages for a project (i.e. creating separate submodules for `:data`, `:domain` and `:ui` (a.k.a. `:presentation`). Strict arch-layering via Gradle modularisation:

- Makes it impossible to violate module encapsulation without first consuming that module as a dependency
- Prevents the strong coupling and god-objects across arch-layers in new code
- Helps guide junior members to develop good "architecture sense" & code that properly separates concerns
- Makes the `internal` visibility modifier a viable encapsulation option

## Issues

However, there is a wrong way to do it, and this can cause a number of issues:

- Implementing strict clean architecture together with feature-modularisation has a multiplier effect on the number of Gradle modules.

    ":app + 11 features x 3 clean architecture layers = 34 gradle modules"

- Too many modules can cause obscure bugs. For example, a private codebase experienced memory issues where "GitHub's Linux CI kept running out of memory on its first run until the total memory allocated between all app modules, daemons, and the convention plugins subproject was reduced to less than 7GB."
- This issue rarely affected local builds; firstly because laptops had more RAM available, and secondly due to Gradle's module optimisations and caching. However clean builds would frequently fail because no submodules had been cached yet.
- It becomes incredibly difficult to disambiguate the different `build.gradle.kts` files, because they are all named the same thing, and searching often doesn't prioritise comments.
- Individual Gradle module configuration is greatly simplified by the use of convention plugins, but remains tedious.

## Advice!

**Follow the "rule of locality"! Related things should always be placed together in what is called a "vertical".** In practice, this means that your top-level should always subdivide based on the vertical first, and then by architecture structure. Getting this subdivision the wrong way around (architecture first, then vertical) will still cause growing pains.

The rule of locality applies to both Gradle modules and package structures. Whether it's:
- Sorting code by locality ("Where do i place this?", "Does it deserve its own package?")
- Avoiding conflicting changes in a team
- Refactoring to expand/reduce a vertical ("Is this big enough to deserve its own module?")

The rule of locality is always the best structure!

You should never attempt a pure "horizontal" divided into multiple modules (a.k.a. an architecture-first structure, wherein classes that fulfill the same purpose are grouped together, e.g. putting all Screens in the same `/screens` package; all Models in the same `/models` package; and all Repositories in the same `/repositories` package). While it seems logical to not repeat package names for architecture layers, this is a fundamentally flawed approach that completely hamstrings expansion efforts and makes cooperations more difficult.

**Verticals differ above and below `:domain`!**

- `:ui` & `:domain` verticals should be based on group of screens or related UI, sometimes called a "feature".
- `:data`, other than implementing the `:domain`, should instead base its verticals by data source library. This means grouping all API code in `:network`; all database code in `:database` etc.

**Do not create a separate dependency injection module!** This adds unnecessary bloat and excessive maintenance burden due to the submodule multiplier effect. Instead, a specialised `/di` package or file should be included in the root of each submodule, and its sole responsibility should be the dependency injection for all implementation classes inside that same module. This also increases mobility / reduces coupling. Alternative implementations of interfaces can be provided via the dependency injection framework or a Factory pattern.

As for the catch-all for **"non-vertical" code** (i.e. code without any clear relation to a subdivided featureset), usually named `util`, it's recommended to create `/util` packages first as siblings to the code that uses them. If a file obtains a wider breadth of use, it can be "promoted" to a higher position in the structure. The goal is to be as local as possible, even if that means creating many different `/util` packages at various depths or branches of the package structure.

The exception to the local `util` rule, is the `:core:util` package, which contains globally useful non-vertical utilities, and is kept separate from the vertical `:core:domain` for clarity & reusability.

**Use convention plugins to maintain a consistent submodule configuration.** Convention plugins drastically reduce the maintenance burden brought about by so many Gradle modules.

---

## An aside on Presentation-Layer Architecture

Clean architecture is not in conflict and should be used in tandem with presentation-layer architecture patterns, such as MVI or MVVM etc.!

Nowadays, it's common to see a mix of partially migrated MVI and MVVM in Android apps. True MVI can be difficult to achieve without a solid understanding, but it should be considered preferable over MVVM. MVVM worked well for separating concerns in the imperative XML but for declarative UI libraries like Compose, MVI takes the cake.

1. MVI recommends aggregating all screen parameters into a UiState data class for that screen. This significantly reduces the number of arguments passed around and thus reduces the maintenance surface area & burden (i.e. since composables call each other, this means avoiding potentially updating multiple method definitions each time a change is made to those parameters)
2. UiState classes mean the entire UI can be updated at once, avoiding undefined in-between states
3. MVI's pattern of unidirectional data flows greatly simplified the model and examination of state changes

In other words, MVI makes maintenance & debugging significantly easier!

### Composables

Composables can generally be divided into the following groups:

1. **screens** - bind viewmodels, callbacks, & events; converts complex types into simple types for layouts (2)
2. **layouts** - logical controllers that reroute the renderer into buckets based on the ui state; keeping layout state code separate from the groups (3) and atoms (4) vastly reduces cognitive complexity; fullscreen layouts, specifically, should be used for previews and testing in place of screens (1)
3. **groups** - aggregates of atoms (4) arranged in a unique identifiable block with actual UI logic
4. **atoms** - the smallest uniquely identifiable unit
5. **utils** - general purpose utilities

Generally, only utils (5) and standard, theme-specific atoms (4) should be shared in common. Screens (1), layouts (2), groups (3), and most atoms (4) need to be unique implementations per vertical to prevent coupling issues. Even if two composables are the same now they may not be the same in the future, and unique composables will make your life a lot easier and avoid unpredictable changes.

**Lastly:**

1. Always provide a default parameter for lambdas, and any other parameters where a default makes sense. This reduces the arguments necessary for previews, which in turn reduces the maintenance surface area & burden
2. Always use named parameters when calling compose functions (and generally). It's easy for compose parameters to change order, and with the proliferation of primitive types, relying solely on positional input is a recipe for disaster. `String`, `Int`, and `Double`, in particular, are overused; but one can alleviate this issue by strongly-typing their domain with `@JvmInline value class`

### Navigation Parameters

Including the class definition for the navigation arguments class of each screen inside the `:domain` submodule is the only reasonable way to enable type-safe, cross-module Fragment navigation. This is a fatal flaw of the `:ui` layer XML navigation graph and would otherwise require serialisation & URI encoding the class to use for deeplinking due to circular dependencies / encapsulation withholding visibility.

Instead, `:app` should associate a map of unique navigation argument class names to Fragments KClass references, and then instantiate the value with FragmentManager. This allows you to "cross-pollinate" the `:ui` submodules with the `:domain` of other verticals, avoiding the circular dependency / visibility trap. Alternatively, you can host all navigation arguments in a `:nav` module, however this is not recommended if you want to navigate with complex types like domain models, as you will need to consume those domains anyway.

---

## Vertical-First Multimodule Clean Architecture

**The answer is always verticals.**

### Notes:

1. `:data` consumes all `:domain` modules in this example. It's important to remember that `:domain` modules should only contain model and interface definitions (and maybe some use cases), not logic, so the actual impact of this "bloat" should be minimal. It's recommended to start here and only splice verticals (verticals make splicing modules easy!) into separate data-layer submodules only if necessary (at which point, you may also want to relocate data-source modules to top-level or somewhere else for clarity).

2. DataSource & Data Transfer Object (DTO) class names should differ based on source, both for easy identification/clarity and so that they do not conflict in the repository implementation that maps them to the domain model. Only domain models have the right to not include a suffix. E.g.:
    - `:domain`: `XXRepository`, `XX` (domain model)
    - `:network`: `XXNetworkSource` (data source), `XXRequest` (upload DTO), `XXResponse`, (download DTO)
    - `:database`: `XXDatabaseSource` (data source), `XXEntity` (database entity DTO / table)

### Directory Structure (approx.)

```
Project Root
├── :app (top-level module)
├── :core (top-level module)
│   ├── :core:theme (theme & standard components)
│   ├── :core:xml (shared xml-related code & wrapper views for :core:theme)
│   ├── :core:ui (multi-screen compose ui verticals)
│   │   ├── :core:ui/di/* (same module di only)
│   │   ├── :core:ui/composables/*
│   │   ├── :core:ui/Screen1Fragment.kt
│   │   ├── :core:ui/Screen1ViewModel.kt
│   │   └── :core:ui/Screen1.kt
│   ├── :core:domain (multi-screen domain verticals)
│   │   ├── :core:domain/di/* (usecase di only)
│   │   ├── :core:domain/models/* (domain models)
│   │   ├── :core:domain/repositories/* (interfaces only)
│   │   └── :core:domain/usecases/* (multi-repo only)
│   └── :core:utils
│       └── :core:utils/* (common non-verticals)
├── :screen1 (top-level module - duplicate this structural pattern for screen2, ui-feat3 etc.)
│   ├── :screen1:ui
│   │   ├── :screen1:ui/di/* (usecase di only)
│   │   ├── :screen1:ui/composables/*
│   │   ├── :screen1:ui/Screen1Fragment.kt
│   │   ├── :screen1:ui/Screen1ViewModel.kt
│   │   └── :screen1:ui/Screen1.kt
│   └── :screen1:domain
│       ├── :screen1:domain/di/* (usecase di only)
│       ├── :screen1:domain/models/* (domain models)
│       ├── :screen1:domain/nav/* (*navigation parameter models)
│       ├── :screen1:domain/repositories/* (interfaces only)
│       └── :screen1:domain/usecases/* (multi-repo only)
└── :data (top-level module - consumes all nested modules)
    ├── :data:database (database data source implementation verticals)
    │   ├── :data:database/di/* (database datasource di only)
    │   ├── :data:database/vertical1/models/* (database entity DTOs)
    │   └── :data:database/vertical1/Vertical1DatabaseSource.kt
    ├── :data:datastore (datastore data source implementation verticals)
    │   ├── :data:datastore/di/* (datastore datasource di only)
    │   ├── :data:datastore/vertical1/models/* (datastore DTOs)
    │   └── :data:datastore/vertical1/Vertical1DatastoreSource.kt
    ├── :data:network (network data source implementation verticals)
    │   ├── :data:network/di/* (network datasource di only)
    │   ├── :data:network/vertical1/models/* (API Resource & Response DTOs)
    │   └── :data:network/vertical1/Vertical1NetworkSource.kt
    ├── :data/di/* (repository di only)
    └── :data/Vertical1RepoImpl.kt (get from data sources, map to domain models)
```
