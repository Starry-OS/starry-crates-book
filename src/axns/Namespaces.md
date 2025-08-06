# Namespaces

> **Relevant source files**
> * [src/lib.rs](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs)
> * [src/ns.rs](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs)

## Purpose and Scope

This document provides a detailed explanation of the `Namespace` struct in the AXNS system, which serves as a container for resources. It covers the internal structure, creation, resource access methods, and memory management of namespaces. For information about the resources themselves and how they're wrapped, see [Resources and ResWrapper](/Starry-OS/axns/2.2-resources-and-reswrapper). For details on thread-local namespace features, see [Thread-Local Features](/Starry-OS/axns/3-thread-local-features).

## Namespace Structure

In AXNS, a `Namespace` is a collection of resources, each accessed through a reference-counted pointer (`ResArc`). The `Namespace` struct is defined in `src/ns.rs` and consists of a single pointer field that points to an array of `ResArc` instances.

```mermaid
flowchart TD
subgraph subGraph0["Namespace Structure"]
    Namespace["Namespace {ptr: NonNull}"]
    ResArcArray["Array of ResArcs (size = Resources.len())"]
    ResArc1["ResArc[0]"]
    ResArc2["ResArc[1]"]
    ResArcN["ResArc[n-1]"]
    Resource1["Resource Data 1"]
    Resource2["Resource Data 2"]
    ResourceN["Resource Data n"]
end

Namespace --> ResArcArray
ResArc1 --> Resource1
ResArc2 --> Resource2
ResArcArray --> ResArc1
ResArcArray --> ResArc2
ResArcArray --> ResArcN
ResArcN --> ResourceN
```

Sources: [src/ns.rs(L6 - L13)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L6-L13)

## Namespace Creation and Initialization

When a new `Namespace` is created using `Namespace::new()`, it:

1. Allocates memory for an array of `ResArc` instances (one for each resource in the system)
2. Initializes each `ResArc` with its corresponding resource's default value
3. Returns the constructed `Namespace`

```mermaid
sequenceDiagram
    participant CodecreatingNamespace as "Code creating Namespace"
    participant Namespacenew as "Namespace::new()"
    participant MemoryAllocator as "Memory Allocator"
    participant ResourcesCollection as "Resources Collection"

    CodecreatingNamespace ->> Namespacenew: Call new()
    Namespacenew ->> ResourcesCollection: Get resources count (Resources.len())
    ResourcesCollection -->> Namespacenew: Return count
    Namespacenew ->> MemoryAllocator: Allocate array of ResArc (size)
    MemoryAllocator -->> Namespacenew: Return allocated memory
    loop For each resource in Resources
        Namespacenew ->> ResourcesCollection: Get resource
        ResourcesCollection -->> Namespacenew: Return resource
        Namespacenew ->> Namespacenew: Initialize ResArc for resource
    end
    Namespacenew -->> CodecreatingNamespace: Return new Namespace
```

Sources: [src/ns.rs(L16 - L36)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L16-L36)

## Resource Access

The `Namespace` provides two primary methods for accessing resources:

1. `get(&self, res: &'static Resource) -> &ResArc`: Returns a reference to the `ResArc` for a given resource.
2. `get_mut(&mut self, res: &'static Resource) -> &mut ResArc`: Returns a mutable reference to the `ResArc` for a given resource.

Both methods use the resource's index (obtained via `res.index()`) to locate the corresponding `ResArc` in the array.

|Method|Description|Implementation|
| --- | --- | --- |
|get|Returns a reference to a resource'sResArc|Uses the resource's index to find the correspondingResArcin the array|
|get_mut|Returns a mutable reference to a resource'sResArc|Uses the resource's index to find the correspondingResArcin the array|

Sources: [src/ns.rs(L38 - L46)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L38-L46)

## Global and Thread-Local Namespaces

AXNS supports two namespace access patterns:

1. **Global Namespace**: A singleton namespace accessible from anywhere via the `global_ns()` function
2. **Thread-Local Namespaces**: When the "thread-local" feature is enabled, each thread can have its own namespace

```mermaid
flowchart TD
subgraph subGraph0["Namespace Resolution"]
    A["current_ns()"]
    B["thread-local feature?"]
    C["Thread-Local CurrentNsImpl"]
    D["Global CurrentNsImpl"]
    E["Thread's Namespace"]
    F["Global Namespace (from global_ns())"]
    G["Access Resources"]
end

A --> B
B --> C
B --> D
C --> E
D --> F
E --> G
F --> G
```

Sources: [src/lib.rs(L16 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L16-L59)

## Memory Management

The `Namespace` struct carefully manages memory for all its resources. When a `Namespace` is dropped:

1. It calls `drop_in_place()` on the array of `ResArc` instances, which decrements the reference count for each resource
2. It deallocates the memory used for the array itself

This ensures that resources are properly cleaned up when they're no longer needed.

Sources: [src/ns.rs(L55 - L63)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L55-L63)

## Namespace in the AXNS Architecture

The `Namespace` is a central component in the AXNS system, working closely with other components:

```mermaid
flowchart TD
subgraph subGraph0["AXNS Component Relationships"]
    Namespace["Namespace(Container for resources)"]
    ResArc["ResArc(Reference-counted resource)"]
    Resource["Resource(Resource metadata)"]
    ResWrapper["ResWrapper(Type-safe resource access)"]
    GlobalNs["global_ns()(Returns static Namespace)"]
    CurrentNs["current_ns()(Thread-local or global)"]
end

CurrentNs --> Namespace
GlobalNs --> Namespace
Namespace --> ResArc
ResArc --> Resource
ResWrapper --> Namespace
ResWrapper --> Resource
```

Sources: [src/lib.rs(L10 - L14)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L10-L14) [src/ns.rs(L1 - L4)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L1-L4)

## Implementation Details

The `Namespace` implementation includes several important features:

* **Memory Efficiency**: Uses a single pointer to an array rather than a standard Rust collection to minimize overhead
* **Safety Markers**: Implements `Send` and `Sync` traits to indicate thread safety
* **Default Implementation**: Provides a `Default` implementation that calls `new()`
* **Manual Memory Management**: Performs explicit allocation and deallocation to maintain control over memory layout

### API Summary

|Method|Description|Example Usage|
| --- | --- | --- |
|Namespace::new()|Creates a newNamespacewith default values|let ns = Namespace::new();|
|ns.get(resource)|Gets a reference to a resource|let r = ns.get(&MY_RESOURCE);|
|ns.get_mut(resource)|Gets a mutable reference to a resource|let r = ns.get_mut(&MY_RESOURCE);|
|global_ns()|Gets the global namespace|let ns = global_ns();|
|current_ns()|Gets the current namespace (global or thread-local)|let ns = current_ns();|

Sources: [src/ns.rs(L15 - L63)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/ns.rs#L15-L63) [src/lib.rs(L16 - L59)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/lib.rs#L16-L59)