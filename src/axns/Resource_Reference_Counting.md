# Resource Reference Counting

> **Relevant source files**
> * [src/arc.rs](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs)
> * [src/res.rs](https://github.com/Starry-OS/axns/blob/622a680e/src/res.rs)

## Purpose and Scope

This document details how AXNS implements reference counting for resources through the `ResArc` type, which provides memory management and safe resource sharing between namespaces. This page covers the internal memory layout of resources, the reference counting mechanism, and the resource lifecycle from allocation to deallocation.

For information about the broader resource lifecycle, see [Resource Lifecycle](/Starry-OS/axns/4-resource-lifecycle). For details on how resources are defined, see [Resources and ResWrapper](/Starry-OS/axns/2.2-resources-and-reswrapper).

## Memory Layout

The AXNS resource system uses a custom memory layout that combines metadata with the actual resource data in a contiguous memory block.

### Resource Memory Structure

```mermaid
flowchart TD
subgraph subGraph1["ResInner Structure"]
    C["res: &'static Resource"]
    D["strong: AtomicUsize"]
end
subgraph subGraph0["Memory Block"]
    A["ResInner (Metadata)"]
    B["Resource Data"]
end

A --> B
A --> C
A --> D
```

The memory layout consists of two key sections:

1. **Metadata Section (ResInner)**: Contains:

* A reference to the static resource descriptor
* An atomic counter for tracking references
2. **Resource Data Section**: Contains the actual data of the resource, with its layout defined during resource creation

Sources: [src/arc.rs(L17 - L21)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L17-L21) [src/arc.rs(L23 - L27)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L23-L27)

### Memory Allocation Process

```mermaid
sequenceDiagram
    participant ClientCode as "Client Code"
    participant ResArcnew as "ResArc::new"
    participant MemoryAllocator as "Memory Allocator"
    participant ResInner as "ResInner"

    ClientCode ->> ResArcnew: "new(&'static Resource)"
    ResArcnew ->> ResArcnew: "Calculate layout requirements"
    ResArcnew ->> MemoryAllocator: "alloc(layout)"
    MemoryAllocator -->> ResArcnew: "raw memory pointer"
    ResArcnew ->> ResInner: "Initialize metadata"
    ResArcnew ->> ResInner: "Initialize resource data (res.init)"
    ResArcnew -->> ClientCode: "Return ResArc instance"
```

When a resource is created, `ResArc::new`:

1. Calculates the combined layout of the metadata and resource data
2. Allocates a single memory block
3. Initializes the metadata section with a reference count of 1
4. Calls the resource's init function to initialize the data section

Sources: [src/arc.rs(L57 - L72)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L57-L72)

## Reference Counting Implementation

The `ResArc` type implements a custom atomic reference counting mechanism to track resource usage.

### ResArc Architecture

```mermaid
classDiagram
class ResArc {
    +NonNull~ResInner~ ptr
    +new(Resource) Self
    +get_mut() Option~&mut T~
    +as_ref() &T
    +clone() Self
}

class ResInner {
    +&'static Resource res
    +AtomicUsize strong
    +body() NonNull~() ~
}

ResArc  -->  ResInner : references
```

The reference counting is implemented through the `AtomicUsize strong` field in `ResInner`, which ensures thread-safe operations on the reference count.

Sources: [src/arc.rs(L49 - L51)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L49-L51) [src/arc.rs(L18 - L21)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L18-L21)

### Reference Counter Operations

```mermaid
flowchart TD
A["Start"]
B["ResArc::clone()"]
C["Atomically increment strong count"]
D["Count > MAX_REFCOUNT?"]
E["Panic: Counter overflow"]
F["Return new ResArc with same ptr"]
G["ResArc::drop()"]
H["Atomically decrement strong count"]
I["Count == 0?"]
J["Return - Resource still in use"]
K["Call resource drop function"]
L["Deallocate memory"]

A --> B
A --> G
B --> C
C --> D
D --> E
D --> F
G --> H
H --> I
I --> J
I --> K
K --> L
```

Key aspects of the reference counting implementation:

1. **Incrementing**: When `clone()` is called, the strong count is atomically incremented with a relaxed ordering
2. **Overflow Protection**: Checks ensure the counter doesn't exceed `MAX_REFCOUNT` (isize::MAX)
3. **Decrementing**: When `drop()` is called, the strong count is atomically decremented
4. **Cleanup**: When the count reaches zero, the resource's drop function is called and memory is deallocated

Sources: [src/arc.rs(L95 - L102)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L95-L102) [src/arc.rs(L104 - L120)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L104-L120)

## Thread Safety

`ResArc` implements both `Send` and `Sync` traits, allowing it to be safely shared between threads. The atomic operations ensure that reference counting works correctly in a multi-threaded environment.

```mermaid
flowchart TD
subgraph subGraph1["Ordering Used"]
    E["Relaxed: For increment"]
    F["Release: For decrement"]
    G["Acquire: For get_mut and drop fence"]
end
subgraph subGraph0["Thread Safety Mechanisms"]
    A["impl Send for ResArc"]
    B["impl Sync for ResArc"]
    C["AtomicUsize for reference counting"]
    D["Memory ordering guarantees"]
end

C --> E
C --> F
C --> G
```

The implementation uses specific memory orderings to balance performance and correctness:

* `Relaxed` ordering for incrementing the counter (lighter weight)
* `Release` ordering when decrementing to ensure visibility of all previous operations
* `Acquire` fence after the last reference is dropped to ensure all operations complete before deallocation

Sources: [src/arc.rs(L53 - L54)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L53-L54) [src/arc.rs(L5 - L9)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L5-L9) [src/arc.rs(L95 - L102)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L95-L102) [src/arc.rs(L104 - L120)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L104-L120)

## Resource Access Patterns

### Accessing and Mutating Resources

```mermaid
flowchart TD
A["Start"]
B["ResArc::as_ref()"]
C["Access resource data immutably"]
D["ResArc::get_mut()"]
E["strong count == 1?"]
F["Return mutable reference to resource data"]
G["Return None - Resource is shared"]

A --> B
A --> D
B --> C
D --> E
E --> F
E --> G
```

`ResArc` provides two primary access patterns:

1. **Immutable access** (`as_ref()`): Always available, returns a reference to the resource data
2. **Mutable access** (`get_mut()`): Only available when the reference count is exactly 1, ensuring exclusive access

Sources: [src/arc.rs(L79 - L85)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L79-L85) [src/arc.rs(L88 - L93)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L88-L93)

## Integration with Namespace System

The reference counting mechanism integrates with the namespace system to enable safe resource sharing between namespaces.

```mermaid
sequenceDiagram
    participant ClientCode as "Client Code"
    participant ResWrapper as "ResWrapper"
    participant SourceNamespace as "Source Namespace"
    participant DestinationNamespace as "Destination Namespace"
    participant ResArc as "ResArc"

    ClientCode ->> ResWrapper: "share_from(dst, src)"
    ResWrapper ->> SourceNamespace: "get(resource)"
    SourceNamespace -->> ResWrapper: "ResArc reference"
    ResWrapper ->> ResArc: "clone()"
    ResArc -->> ResWrapper: "New ResArc with incremented count"
    ResWrapper ->> DestinationNamespace: "Update with cloned ResArc"
    Note over SourceNamespace,DestinationNamespace: Both namespaces now share the same resource instance
```

When a resource is shared between namespaces:

1. The `share_from` method obtains a reference to the resource in the source namespace
2. This reference is cloned, incrementing the strong count
3. The destination namespace's reference is replaced with the cloned reference
4. Both namespaces now point to the same underlying resource data

Sources: [src/res.rs(L94 - L98)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/res.rs#L94-L98)

## Resource Reset and Memory Management

```mermaid
flowchart TD
A["Start"]
B["ResWrapper::reset()"]
C["Get mutable reference to ResArc in namespace"]
D["Create new ResArc instance"]
E["Replace old ResArc with new one"]
F["Old ResArc is dropped"]
G["Was this the last reference?"]
H["Resource memory is deallocated"]
I["Resource continues to exist for other namespaces"]

A --> B
B --> C
C --> D
D --> E
E --> F
F --> G
G --> H
G --> I
```

When a resource is reset in a namespace:

1. A new `ResArc` instance is created with a fresh copy of the resource
2. The old `ResArc` in the namespace is replaced and dropped
3. If this was the last reference to the old resource, its memory is deallocated
4. Otherwise, the resource continues to exist for other namespaces that share it

Sources: [src/res.rs(L100 - L104)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/res.rs#L100-L104)

## Technical Limitations and Safeguards

The ResArc implementation includes several important safeguards:

1. **Reference Count Maximum**: Limited to isize::MAX to prevent overflow
2. **Mutable Access Safety**: Mutable access is only granted when the reference count is exactly 1
3. **Memory Layout Handling**: Carefully manages the layout and offset calculations for resource data
4. **Drop Sequence**: Ensures proper ordering of deallocation operations

Sources: [src/arc.rs(L14 - L15)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L14-L15) [src/arc.rs(L79 - L85)&emsp;](https://github.com/Starry-OS/axns/blob/622a680e/src/arc.rs#L79-L85)

## Summary

The resource reference counting system in AXNS provides:

1. **Memory Safety**: Ensuring resources are only deallocated when all references are dropped
2. **Thread Safety**: Allowing resources to be safely shared between threads
3. **Efficient Sharing**: Enabling namespaces to share resources without unnecessary duplication
4. **Controlled Mutability**: Preventing data races by restricting mutable access to unshared resources

This system forms the foundation for the resource lifecycle management in AXNS, balancing safety, performance, and flexibility.