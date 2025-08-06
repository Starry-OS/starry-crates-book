# Project Information

> **Relevant source files**
> * [.github/workflows/ci.yml](https://github.com/Starry-OS/weak-map/blob/b19a081d/.github/workflows/ci.yml)
> * [README.md](https://github.com/Starry-OS/weak-map/blob/b19a081d/README.md)

This document provides essential information about the weak-map project structure, development workflow, contribution guidelines, and licensing. For detailed technical information about the implementation, please refer to [Core Components](/Starry-OS/weak-map/2-core-components) and [Implementation Details](/Starry-OS/weak-map/4-implementation-details).

## Project Overview

The weak-map repository provides a Rust implementation of `WeakMap` - a B-Tree map data structure that stores weak references to values, automatically removing entries when referenced values are dropped. It is hosted on GitHub at [https://github.com/Starry-OS/weak-map](https://github.com/Starry-OS/weak-map) and published as a crate on crates.io.

```mermaid
flowchart TD
A["weak-map Repository"]
B["Source Code"]
C["Project Metadata"]
D["CI Configuration"]
E["lib.rs"]
F["map.rs"]
G["traits.rs"]
H["README.md"]
I["Cargo.toml"]
J["License Files"]
K[".github/workflows/ci.yml"]

A --> B
A --> C
A --> D
B --> E
B --> F
B --> G
C --> H
C --> I
C --> J
D --> K
```

Sources: README.md, .github/workflows/ci.yml

## Repository Structure

The weak-map project follows a standard Rust crate organization with a clean separation between the core implementations and trait definitions.

```mermaid
classDiagram
class SourceCode {
    src/lib.rs
    src/map.rs
    src/traits.rs
    
}

class Implementation {
    WeakMap
    StrongMap
    
}

class Traits {
    StrongRef
    WeakRef
    
}

class ProjectFiles {
    README.md
    Cargo.toml
    LICENSE-MIT
    LICENSE-APACHE-2.0
    
}

class CIConfig {
    .github/workflows/ci.yml
    
}

SourceCode  -->  Implementation : contains
SourceCode  -->  Traits : contains
Implementation  ..>  Traits : uses
```

Sources: README.md

## Development Workflow

The weak-map project employs GitHub Actions for continuous integration to ensure code quality and test coverage.

### CI Process

The CI workflow runs automatically on:

* Push to the main branch
* Pull requests targeting the main branch

```mermaid
flowchart TD
A["Push/PR to main branch"]
B["CI Workflow Triggered"]
C["Matrix Setup"]
D["Rust Toolchain Versions"]
E["stable"]
F["nightly"]
G["nightly-2025-01-18"]
H["Run Clippy"]
I["Run Tests"]
J["Build Success/Failure Report"]

A --> B
B --> C
C --> D
D --> E
D --> F
D --> G
E --> H
F --> H
G --> H
H --> I
I --> J
```

Sources: [.github/workflows/ci.yml(L3 - L10)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/.github/workflows/ci.yml#L3-L10)

### CI Actions

The CI performs these specific checks:

1. **Clippy Linting**: Runs with all features and targets, with warnings treated as errors:

```
cargo clippy --all-features --all-targets -- -Dwarnings
```
2. **Comprehensive Testing**: Runs all tests with all features enabled:

```
cargo test --all-features
```

Sources: [.github/workflows/ci.yml(L28 - L31)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/.github/workflows/ci.yml#L28-L31)

## Contributing Guidelines

Contributions to the weak-map project are welcome. Based on the repository structure and CI configuration, here are the recommended steps for contributing:

1. **Fork the repository** on GitHub
2. **Create a feature branch** for your changes
3. **Make your changes** following the code style of the project
4. **Add tests** for your changes to ensure they work correctly
5. **Run the checks locally** that will be performed by CI:
```
cargo clippy --all-features --all-targets -- -Dwarnings
cargo test --all-features
```
6. **Submit a pull request** to the main branch

```mermaid
sequenceDiagram
    participant Developer as Developer
    participant Repository as Repository
    participant CI as CI

    Developer ->> Repository: Fork repository
    Developer ->> Developer: Create feature branch
    Developer ->> Developer: Make changes
    Developer ->> Developer: Add tests
    Developer ->> Developer: Run local checks
    Developer ->> Repository: Submit pull request
    Repository ->> CI: Trigger CI checks
    CI ->> Repository: Report results
    alt Tests Pass
        Repository ->> Developer: Approve and merge
    else Tests Fail
        Repository ->> Developer: Request changes
    end
```

Sources: [.github/workflows/ci.yml(L14 - L31)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/.github/workflows/ci.yml#L14-L31)

## License Information

The weak-map project is dual-licensed under both the MIT License and the Apache License 2.0, allowing users to choose the license that best suits their needs.

### Dual License Approach

```mermaid
flowchart TD
A["weak-map Project"]
B["MIT License"]
C["Apache License 2.0"]
D["Simple, permissive license"]
E["Includes explicit patent grants"]
F["Users can choose either license"]

A --> B
A --> C
B --> D
C --> E
D --> F
E --> F
```

### License Usage

* **MIT License**: A permissive license that allows users to do almost anything with the code, including using it in proprietary software, as long as they provide attribution.
* **Apache License 2.0**: Also permissive, but includes explicit patent grants and more detailed terms around trademark usage.

The license files (LICENSE-MIT and LICENSE-APACHE-2.0) are included in the repository root directory, as indicated by project structure diagrams.

Sources: README.md

## Package Information

The weak-map package is published on [crates.io](https://crates.io/crates/weak-map) and documentation is available on [docs.rs](https://docs.rs/weak-map).

```mermaid
flowchart TD
A["weak-map Package"]
B["crates.io"]
C["docs.rs"]
D["Rust dependency management"]
E["API documentation"]
F["Your Project"]
G["Add dependency in Cargo.toml"]

A --> B
A --> C
B --> D
C --> E
F --> G
G --> B
```

### Project Origins

As noted in the README, weak-map is "similar to and inspired by [weak-table](https://github.com/Starry-OS/weak-map/blob/b19a081d/weak-table) but using `BTreeMap` as underlying implementation."

Sources: [README.md(L6)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/README.md#L6-L6)