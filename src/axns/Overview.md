# Overview

> **Relevant source files**
> * [Cargo.toml](https://github.com/Starry-OS/axns/blob/622a680e/Cargo.toml)
> * [README.md](https://github.com/Starry-OS/axns/blob/622a680e/README.md)
> * [src/lib.rs](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs)

AXNS (Resource Namespace System) is a Rust library providing a unified interface for managing and controlling access to system resources across different deployment scenarios. It enables configurable resource sharing and isolation between processes and threads in various operating system environments, from unikernels with shared resources to monolithic kernels or containerized environments requiring isolation.

For more detailed information about specific components, see [Core Concepts](/Starry-OS/axns/2-core-concepts) and [Thread-Local Features](/Starry-OS/axns/3-thread-local-features).

## Purpose and Scope

AXNS addresses several key requirements for flexible resource management:

* **Unified Resource Access**: Providing a consistent interface to system resources
* **Configurable Isolation**: Supporting varying degrees of resource sharing between threads
* **Deployment Flexibility**: Working effectively in different system architectures
* **Memory Safety**: Ensuring proper resource initialization and cleanup
* **Type Safety**: Providing strongly-typed access to resources

The system manages resources such as virtual address spaces, working directories, file descriptors, and other system facilities that might need to be shared or isolated.

Sources: [README.md(L5 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/README.md#L5-L14)

## Core Architecture

AXNS follows a modular design with several key architectural patterns:

```mermaid
flowchart TD
subgraph subGraph2["Access Patterns"]
    F["Global Namespace"]
    G["Shared resources"]
    H["Thread-local Namespaces"]
    I["Isolated resources"]
end
subgraph subGraph1["Namespace Management Layer"]
    C["Namespace"]
    D["ResArc references"]
    E["Resource instances"]
end
subgraph subGraph0["Resource Definition Layer"]
    A["def_resource! macro"]
    B["Static Resource"]
end
J["Unikernel Mode"]
K["Process/Container Mode"]

A --> B
B --> E
C --> D
D --> E
F --> G
G --> J
H --> I
I --> K
```

The architecture consists of these primary components:

|Component|Description|Role|
| --- | --- | --- |
|Namespace|Container for resources|Stores and provides access to system resources|
|Resource|Resource type metadata|Defines memory layout and lifecycle functions|
|ResWrapper<T>|Static resource handle|Provides the public API for resource access|
|ResArc<T>|Reference-counted pointer|Manages resource lifecycle and memory|
|def_resource!|Resource definition macro|Simplifies creation of new resource types|

Sources: [src/lib.rs(L10 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L10-L14)

## Component Relationships

```mermaid
classDiagram
class Namespace {
    +ptr: NonNull~ResArc~
    +new() Namespace
    +get(Resource) &ResArc
    +get_mut(Resource) &mut ResArc
}

class Resource {
    +layout: Layout
    +init: fn pointer
    +drop: fn pointer
    +index() usize
}

class ResWrapper~T~ {
    +res: &'static Resource
    +current() ResCurrent~T~
    +get(Namespace) &T
    +get_mut(Namespace) &mut T
    +share_from(dst, src)
    +reset(Namespace)
}

class ResArc~T~ {
    +ptr: NonNull~ResInner~
    +as_ref() &T
    +get_mut() Option~&mut T~
}

class CurrentNs {
    <<trait>>
    
    +new() Self
    +as_ref() &Namespace
}

Namespace "1" --> "*" ResArc : contains
ResArc "*" -->  Resource : references
ResWrapper "1" -->  Resource : describes
ResWrapper  ..> "1" Namespace : accesses through
CurrentNs  ..> "1" Namespace : provides context
```

Sources: [src/lib.rs(L10 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L10-L14) [src/lib.rs(L32 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L32-L59)

## Resource Access Flow

Accessing resources in AXNS follows this pattern:

```mermaid
sequenceDiagram
    participant ApplicationCode as "Application Code"
    participant ResWrapperT as "ResWrapper<T>"
    participant Namespace as "Namespace"
    participant ResArcT as "ResArc<T>"

    ApplicationCode ->> ResWrapperT: Define with def_resource!
    ApplicationCode ->> Namespace: Create namespace
    ApplicationCode ->> ResWrapperT: resource.get(&namespace)
    ResWrapperT ->> Namespace: namespace.get(resource)
    Namespace ->> ResArcT: Get ResArc
    ResArcT -->> ApplicationCode: Return reference to T
    ApplicationCode ->> ResWrapperT: resource.get_mut(&mut namespace)
    ResWrapperT ->> Namespace: namespace.get_mut(resource)
    Namespace ->> ResArcT: Get mutable ResArc
    ResArcT -->> ApplicationCode: Return mutable reference to T
    ApplicationCode ->> ResWrapperT: resource.current()
    ResWrapperT ->> Namespace: Get current_ns()
    Note over Namespace: Uses thread-local or global NS
    Namespace -->> ApplicationCode: Access through current namespace
```

Sources: [src/lib.rs(L16 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L16-L59)

## Thread-Local Feature

AXNS provides an optional thread-local feature for fine-grained resource isolation:

```mermaid
stateDiagram-v2
state UseThreadLocalNS {
    [*] --> CheckTLS
    CheckTLS --> InitializeNew : First access
    CheckTLS --> UseExisting : Subsequent access
}
state AccessResources {
    [*] --> GetResource
    GetResource --> ModifyResource : get_mut()
    GetResource --> ShareResource : share_from()
    GetResource --> ResetResource : reset()
}
[*] --> FeatureCheck
FeatureCheck --> UseGlobalNS : thread-local OFF
FeatureCheck --> UseThreadLocalNS : thread-local ON
UseGlobalNS --> AccessResources
UseThreadLocalNS --> AccessResources
```

This feature is controlled by the `thread-local` feature flag in Cargo.toml:

```
[features]
thread-local = ["dep:extern-trait"]
```

When enabled, AXNS uses the `CurrentNs` trait to provide thread-local namespaces. When disabled, all access goes through the global namespace.

Sources: [src/lib.rs(L32 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L32-L59) [Cargo.toml(L14 - L15)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/Cargo.toml#L14-L15)

## Deployment Scenarios

AXNS supports various deployment models by adjusting namespace isolation:

```mermaid
flowchart TD
subgraph subGraph0["Deployment Models"]
    A["Unikernel"]
    D["Single Global Namespace"]
    B["Monolithic Kernel"]
    E["Per-Process Namespaces"]
    C["Container Environment"]
    F["Grouped Namespaces"]
end
G["Shared Resources"]
H["Process-Isolated Resources"]
I["Container-Isolated Resources"]

A --> D
B --> E
C --> F
D --> G
E --> H
F --> I
```

1. **Unikernel Mode**: A single global namespace shared by all threads (default)
2. **Monolithic Kernel Mode**: Each process has its own namespace, with threads in the same process sharing resources
3. **Container Mode**: System resources grouped into namespaces that are shared between specific processes

Sources: [README.md(L5 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/README.md#L5-L14)

## Summary

AXNS provides a flexible, efficient system for managing resource namespaces across different operating system environments. Its architecture balances the need for shared resources with isolation requirements, providing a consistent API regardless of the deployment scenario. The system's design ensures proper resource lifecycle management through reference counting, while the optional thread-local feature provides additional isolation when needed.

For practical guidance on using AXNS, see [Usage Guide](/Starry-OS/axns/5-usage-guide).

Sources: [src/lib.rs(L1 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L1-L59) [README.md(L1 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/README.md#L1-L14)