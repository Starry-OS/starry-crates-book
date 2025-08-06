# Development and Testing

> **Relevant source files**
> * [.github/workflows/ci.yml](https://github.com/Starry-OS/axns/blob/622a680e/.github/workflows/ci.yml)
> * [.gitignore](https://github.com/Starry-OS/axns/blob/622a680e/.gitignore)
> * [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

This page provides comprehensive information for developers working on the AXNS resource namespace library itself. It covers the development environment, testing methodology, CI/CD workflow, and guidelines for contributing to the project. For information about using the AXNS library in your applications, please see [Usage Guide](/Starry-OS/axns/5-usage-guide).

## Development Environment

AXNS is developed as a Rust library with minimal dependencies. To work on AXNS, you need:

1. **Rust Toolchain**: The project uses the nightly Rust toolchain for development to leverage advanced features and documentation tools.
2. **Cargo**: For building, testing, and package management.
3. **Git**: For version control.

The repository is organized in a standard Rust project structure:

```mermaid
flowchart TD
A["src/"]
B["lib.rs (Core library code)"]
C[".github/workflows/"]
D["ci.yml (CI pipeline configuration)"]
E["tests/"]
F["all.rs (Test suite)"]
G["Cargo.toml (Dependencies and metadata)"]

A --> B
C --> D
E --> F
```

Sources: [.github/workflows/ci.yml](https://github.com/Starry-OS/axns/blob/622a680e/.github/workflows/ci.yml) [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

## Testing Methodology

AXNS employs a comprehensive testing methodology to ensure correctness and reliability of the namespace system. The test suite in `tests/all.rs` validates the core functionality through various test cases.

### Test Categories

The test suite includes several categories of tests:

```

```

Sources: [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

### Test Case Examples

The test suite validates several key aspects of the AXNS system:

1. **Basic namespace operations** [tests/all.rs(L4 - L25)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L4-L25)

* Creating a namespace
* Defining resources with `def_resource!`
* Getting and modifying resources
2. **Current resource access** [tests/all.rs(L27 - L38)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L27-L38)

* Accessing the current value of a resource
* Modifying resources in the current namespace
3. **Thread-local feature tests** [tests/all.rs(L40 - L159)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L40-L159)

* Resource cleanup and recycling
* Resetting resources
* Sharing resources between namespaces

Sources: [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

## Feature Flag Testing

AXNS uses feature flags to enable optional functionality. The most significant feature is `thread-local`, which enables thread-local namespace support.

### Thread-Local Feature Testing

The thread-local feature is tested in a dedicated module that is only compiled when the feature is enabled:

```mermaid
flowchart TD
subgraph subGraph0["Thread-Local Tests"]
    D["recycle() - Tests resource cleanup in thread-local context"]
    E["reset() - Tests resetting resources in thread-local context"]
    F["clone_from() - Tests sharing resources between thread-local namespaces"]
end
A["Test Suite"]
B["Basic Tests"]
C["Basic Tests + Thread-Local Tests"]

A --> B
A --> C
C --> D
C --> E
C --> F
```

The thread-local tests validate several important aspects:

1. **Resource lifecycle** - Ensuring resources are properly cleaned up when threads terminate
2. **Resource sharing** - Verifying resources can be shared between namespaces
3. **Resource resetting** - Testing the reset functionality in thread-local contexts

Sources: [tests/all.rs(L40 - L159)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L40-L159)

## CI/CD Pipeline

AXNS employs a comprehensive CI/CD pipeline implemented with GitHub Actions to ensure code quality and consistency.

### CI Workflow

```mermaid
flowchart TD
A["Push/PR to main branch"]
B["CI Workflow Trigger"]
C["check job"]
C1["Run cargo fmt"]
C2["Run cargo clippy"]
D["test job"]
D1["Run cargo test"]
D2["Run cargo test with thread-local"]
E["doc job"]
E1["Build documentation"]
F["deploy job"]
F1["Deploy to GitHub Pages"]
G["Successful Check"]
H["Successful Tests"]

A --> B
B --> C
B --> D
B --> E
C --> C1
C --> C2
C1 --> G
C2 --> G
D --> D1
D --> D2
D1 --> H
D2 --> H
E --> E1
E1 --> F
F --> F1
```

The CI pipeline consists of the following jobs:

|Job|Description|Commands|
| --- | --- | --- |
|check|Validates code formatting and checks for linting issues|cargo fmt --all --checkcargo clippy --all-targets --all-features -- -D warnings|
|test|Runs the test suite in both standard and thread-local modes|cargo test --verbosecargo test --verbose -F thread-local|
|doc|Builds the documentation with all features|cargo doc --all-features --no-deps|
|deploy|Deploys the documentation to GitHub Pages|GitHub Actions deployment task|

Sources: [.github/workflows/ci.yml](https://github.com/Starry-OS/axns/blob/622a680e/.github/workflows/ci.yml)

## Test-Driven Development

The AXNS development process follows test-driven development principles:

1. **Write Tests First**: New features should be accompanied by tests that validate their behavior
2. **Validate Core Functionality**: Tests should cover the full range of functionality
3. **Feature Flag Testing**: Both standard and feature-enabled configurations must be tested

### Test to Code Relationship

```mermaid
flowchart TD
subgraph subGraph1["Core Components"]
    B["Namespace Struct"]
    C["Resource and ResWrapper"]
    D["def_resource! Macro"]
    E["Thread-Local Features"]
    F["Resource Lifecycle"]
end
subgraph subGraph0["Test Files"]
    A["tests/all.rs"]
end

A --> B
A --> C
A --> D
A --> E
A --> F
```

Sources: [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

## Testing ResArc Reference Counting

A critical aspect of AXNS is its reference counting mechanism implemented through `ResArc`. The test suite verifies that reference counting works correctly to prevent memory leaks.

```mermaid
sequenceDiagram
    participant Test as Test
    participant Namespace as Namespace
    participant Resource as Resource
    participant ResArc as ResArc

    Test ->> Namespace: Create Namespace
    Test ->> Resource: Define Resource (def_resource!)
    Test ->> Namespace: Get resource (DATA.get_mut())
    Namespace ->> ResArc: Clone ResArc
    Test ->> Namespace: Modify resource
    Note over Test,ResArc: Thread-local tests
    Test ->> ResArc: Share resource (share_from)
    ResArc ->> ResArc: Increment reference count
    Test ->> ResArc: Reset resource (reset)
    ResArc ->> ResArc: Decrement reference count
    Note over Test,ResArc: Verify reference counts
    Test ->> ResArc: Check strong_count matches expectations
```

The thread-local tests specifically validate reference counting by tracking the `strong_count` of `Arc` instances and ensuring they are properly incremented and decremented.

Sources: [tests/all.rs(L40 - L159)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L40-L159)

## Development Guidelines

When developing AXNS, follow these guidelines:

1. **Write Tests**: All new functionality should be accompanied by appropriate tests.
2. **Feature Flags**: When adding features that can be optional, use feature flags and add tests for both configurations.
3. **Documentation**: Document all public APIs with doc comments.
4. **Code Quality**: Ensure code passes `cargo fmt` and `cargo clippy` checks.
5. **Compatibility**: Maintain backward compatibility when possible.

### Adding New Resources

When adding new resource types to the system:

1. Define the resource using the `def_resource!` macro
2. Implement tests that validate the resource behavior in various scenarios
3. Ensure proper cleanup and reference counting

### Testing Thread-Local Features

When working with the thread-local feature:

1. Place thread-local specific tests in the `#[cfg(feature = "thread-local")]` module
2. Verify resources are properly cleaned up when threads terminate
3. Test interactions between thread-local and global namespaces

Sources: [tests/all.rs(L40 - L159)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs#L40-L159)

## Test Code Structure

The test structure in AXNS follows a modular pattern where base functionality is tested first, followed by feature-specific tests:

```mermaid
flowchart TD
A["tests/all.rs"]
B["Base Tests"]
C["#[cfg(feature = thread-local)]"]
B1["ns()"]
B2["current()"]
C1["thread_local! { static NS }"]
C2["CurrentNsImpl struct"]
C3["Feature-specific tests"]
D1["recycle()"]
D2["reset()"]
D3["clone_from()"]

A --> B
A --> C
B --> B1
B --> B2
C --> C1
C --> C2
C --> C3
C3 --> D1
C3 --> D2
C3 --> D3
```

Sources: [tests/all.rs](https://github.com/Starry-OS/axns/blob/622a680e/tests/all.rs)

## Conclusion

The development and testing infrastructure of AXNS is designed to ensure the correctness and reliability of the resource namespace system. By following the guidelines and leveraging the existing test framework, developers can contribute to AXNS while maintaining its quality standards.

For information on using AXNS in your applications, please refer to the [Usage Guide](/Starry-OS/axns/5-usage-guide) section.