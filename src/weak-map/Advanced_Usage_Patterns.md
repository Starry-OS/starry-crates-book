# Advanced Usage Patterns

> **Relevant source files**
> * [src/map.rs](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs)
> * [src/traits.rs](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs)

This page explores complex usage patterns, optimization techniques, and best practices for the `WeakMap` implementation. While [Basic Usage Examples](/Starry-OS/weak-map/3.1-basic-usage-examples) covers fundamental operations, this section focuses on advanced scenarios that leverage the full potential of weak references in map data structures.

## Understanding the Cleanup Mechanism

The `WeakMap` implementation includes an automatic cleanup mechanism that purges expired weak references. Understanding this mechanism is crucial for optimizing performance.

```mermaid
flowchart TD
OP["Operation on WeakMap"]
BUMP["Counter.bump()"]
CHECK["reach_threshold() check"]
CLEAN["cleanup()"]
RETAIN["retain(!is_expired())"]
RESET["Counter.reset()"]
EXIT["Continue"]

BUMP --> CHECK
CHECK --> CLEAN
CHECK --> EXIT
CLEAN --> RESET
CLEAN --> RETAIN
OP --> BUMP
```

By default, cleanup occurs after `OPS_THRESHOLD` (1000) operations on the map. This value is defined as a constant in [src/map.rs(L16)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L16-L16)

Sources: [src/map.rs(L14 - L47)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L14-L47) [src/map.rs(L158 - L169)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L158-L169)

### Examining Cleanup Performance

Understanding the performance characteristics of the cleanup process is important for applications with stringent timing requirements:

1. **Cleanup Complexity**: The cleanup operation is O(n) as it iterates through all entries in the map.
2. **Lazy Cleanup**: Entries are only removed during cleanup operations, not immediately when they expire.
3. **Actual vs. Raw Length**: The `len()` method reports only valid entries, while `raw_len()` includes expired entries.

```mermaid
sequenceDiagram
    participant Client as Client
    participant WeakMap as WeakMap
    participant OpsCounter as OpsCounter
    participant BTreeMap as BTreeMap

    Client ->> WeakMap: Multiple operations
    loop Each operation
        WeakMap ->> OpsCounter: bump()
    end
    WeakMap ->> OpsCounter: reach_threshold()?
    OpsCounter -->> WeakMap: true
    WeakMap ->> WeakMap: cleanup()
    WeakMap ->> BTreeMap: retain(!is_expired())
    WeakMap ->> OpsCounter: reset()
    Note over Client,BTreeMap: After cleanup
    Client ->> WeakMap: raw_len()
    WeakMap ->> BTreeMap: len()
    BTreeMap -->> Client: Count of all entries
    Client ->> WeakMap: len()
    WeakMap ->> WeakMap: iter().count()
    WeakMap -->> Client: Count of valid entries only
```

Sources: [src/map.rs(L158 - L169)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L158-L169) [src/map.rs(L113 - L115)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L113-L115) [src/map.rs(L171 - L185)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L171-L185)

## Custom Reference Types

The `WeakMap` can work with any reference type that implements the `WeakRef` trait, while the values must be from types implementing the `StrongRef` trait.

```mermaid
classDiagram
class StrongRef {
    <<trait>>
    type Weak
    downgrade() -~ Self::Weak
    ptr_eq(other: &Self) -~ bool
}

class WeakRef {
    <<trait>>
    type Strong
    upgrade() -~ Option
    is_expired() -~ bool
}

class CustomStrongRef {
    data: T
    downgrade() -~ CustomWeakRef
    ptr_eq(other: &Self) -~ bool
}

class CustomWeakRef {
    reference: WeakInner~T~
    upgrade() -~ Option~CustomStrongRef~
    is_expired() -~ bool
}

StrongRef  ..|>  CustomStrongRef : implements
WeakRef  ..|>  CustomWeakRef : implements
CustomStrongRef  -->  CustomWeakRef : creates
CustomWeakRef  -->  CustomStrongRef : may upgrade to
```

Implementing these traits for custom reference types allows you to integrate them with `WeakMap`:

```mermaid
flowchart TD
A["Custom Type Conversion"]
B["Implement StrongRef for CustomStrong"]
C["Implement WeakRef for CustomWeak"]
D["Use in WeakMapUnsupported markdown: delT~~"]

A --> B
A --> C
B --> D
C --> D
```

Sources: [src/traits.rs(L3 - L19)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L3-L19) [src/traits.rs(L21 - L40)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L21-L40)

## Advanced Conversion Operations

### Converting Between WeakMap and StrongMap

The `WeakMap` implementation provides methods for converting between weak and strong maps:

```mermaid
flowchart TD
A["WeakMapUnsupported markdown: del"]
B["StrongMapUnsupported markdown: del"]

A --> B
B --> A
```

The `upgrade()` method creates a new `StrongMap` containing only the valid entries:

Sources: [src/map.rs(L296 - L306)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L296-L306) [src/map.rs(L368 - L380)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L368-L380)

### Working with Iterators

`WeakMap` provides various iterator types to work with different aspects of the map:

|Iterator Type|Description|Returns|Implementation|
| --- | --- | --- | --- |
|Iter|References to entries|(&'a K, V::Strong)|src/map.rs382-430|
|Keys|References to keys|&'a K|src/map.rs444-485|
|Values|Valid values|V::Strong|src/map.rs487-528|
|IntoIter|Owned entries|(K, V::Strong)|src/map.rs530-571|
|IntoKeys|Owned keys|K|src/map.rs573-597|
|IntoValues|Owned values|V::Strong|src/map.rs599-623|

Note that all iterators automatically filter out expired references, so you only get valid entries.

```mermaid
flowchart TD
WM["WeakMapUnsupported markdown: del"]
Iter["IterUnsupported markdown: del"]
Keys["KeysUnsupported markdown: del"]
Values["ValuesUnsupported markdown: del"]
IntoIter["IntoIterUnsupported markdown: del"]
IntoKeys["IntoKeysUnsupported markdown: del"]
IntoValues["IntoValuesUnsupported markdown: del"]
EntryRef["(&'a K, V::Strong)"]
KeyRef["&'a K"]
Value["V::Strong"]
Entry["(K, V::Strong)"]
Key["K"]
OwnedValue["V::Strong"]

IntoIter --> Entry
IntoKeys --> Key
IntoValues --> OwnedValue
Iter --> EntryRef
Keys --> KeyRef
Values --> Value
WM --> IntoIter
WM --> IntoKeys
WM --> IntoValues
WM --> Iter
WM --> Keys
WM --> Values
```

Sources: [src/map.rs(L119 - L149)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L119-L149) [src/map.rs(L382 - L623)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L382-L623)

## Memory Management Strategies

### Minimizing Memory Overhead

When working with `WeakMap`, consider these strategies to minimize memory overhead:

1. **Preemptive Cleanup**: For large maps, consider manually triggering cleanup before critical operations.
2. **Monitoring Raw Size**: Use `raw_len()` to monitor the total size including expired entries.
3. **Strategic Insert/Remove**: Batch insertions and removals to minimize cleanup frequency.

```mermaid
flowchart TD
A["Memory Optimization"]
B["Preemptive Cleanup"]
C["Size Monitoring"]
D["Batch Operations"]
B1["Call retain() manually"]
C1["raw_len() vs len()"]
D1["Insert/remove in batches"]

A --> B
A --> C
A --> D
B --> B1
C --> C1
D --> D1
```

Sources: [src/map.rs(L113 - L115)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L113-L115) [src/map.rs(L158 - L169)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L158-L169) [src/map.rs(L187 - L201)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L187-L201)

## Thread Safety Considerations

The `WeakMap` can be used with both single-threaded (`Rc`/`Weak`) and thread-safe (`Arc`/`Weak`) reference types.

```mermaid
flowchart TD
A["Reference Type Selection"]
B["Single-Threaded"]
C["Multi-Threaded"]
B1["RcUnsupported markdown: del / WeakUnsupported markdown: del"]
B2["Implements StrongRef/WeakRef"]
B3["Use in WeakMapUnsupported markdown: delT~~"]
C1["ArcUnsupported markdown: del / WeakUnsupported markdown: del"]
C2["Implements StrongRef/WeakRef"]
C3["Use in WeakMapUnsupported markdown: delT~~"]

A --> B
A --> C
B --> B1
B --> B2
B --> B3
C --> C1
C --> C2
C --> C3
```

Selection depends on your concurrency requirements:

|Reference Type|Thread-Safe|Use Case|
| --- | --- | --- |
|Rc/Weak|No|Single-threaded applications, better performance|
|Arc/Weak|Yes|Multi-threaded applications, safe concurrent access|

Sources: [src/traits.rs(L42 - L64)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L42-L64) [src/traits.rs(L66 - L88)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L66-L88)

## Advanced Usage Patterns

### Caching with Automatic Cleanup

`WeakMap` is particularly well-suited for implementing caches that automatically evict entries when they are no longer used elsewhere:

```mermaid
flowchart TD
Client["Client"]
Cache["Cache System"]
WeakMap["WeakMap"]
Compute["Compute Value"]
StoreRef["Store in Application"]
AppData["Application Data"]
Expire["Entry Expires"]
NextCleanup["Next Cleanup"]

AppData --> Expire
Cache --> WeakMap
Client --> Cache
Compute --> StoreRef
Compute --> WeakMap
Expire --> NextCleanup
NextCleanup --> WeakMap
StoreRef --> AppData
WeakMap --> Client
WeakMap --> Compute
```

Sources: [src/map.rs(L203 - L214)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L203-L214) [src/map.rs(L258 - L263)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L258-L263)

### Observer Pattern Implementation

`WeakMap` can be used to implement observer patterns without memory leaks:

```mermaid
flowchart TD
Subject["Observable Subject"]
Observer1["Observer 1"]
Observer2["Observer 2"]
ObserverN["Observer N"]
WeakMap["WeakMapUnsupported markdown: delObserver~~"]
Expired["Reference Expired"]
Removed["Entry Removed"]

Expired --> Removed
Observer2 --> Expired
Subject --> WeakMap
WeakMap --> Observer1
WeakMap --> Observer2
WeakMap --> ObserverN
```

Sources: [src/map.rs(L203 - L214)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L203-L214) [src/map.rs(L258 - L263)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L258-L263)

### Breaking Reference Cycles

`WeakMap` is ideal for breaking reference cycles in complex data structures:

```mermaid
flowchart TD
ParentNode["Parent Node (Strong)"]
ChildNodes["Child Nodes (Strong)"]
ParentRefs["Parent References (Weak)"]

ChildNodes --> ParentRefs
ParentNode --> ChildNodes
ParentRefs --> ParentNode
```

This pattern avoids memory leaks while maintaining bidirectional relationships.

Sources: [src/traits.rs(L3 - L19)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L3-L19) [src/traits.rs(L21 - L40)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/traits.rs#L21-L40)

## Performance Optimizations

### Choosing the Right Cleanup Strategy

The default cleanup strategy may not be optimal for all use cases:

|Usage Pattern|Recommended Approach|
| --- | --- |
|High churn (many entries added/removed)|LowerOPS_THRESHOLDor manual cleanup|
|Mostly static data with few expirations|Default cleanup is adequate|
|Memory-constrained environments|Preemptive cleanup after critical operations|
|Performance-critical code paths|Consider manual cleanup during idle periods|

### Optimizing Map Operations

For performance-critical applications, consider these strategies:

1. **Pre-sizing**: If approximate size is known, create with appropriate capacity
2. **Batch Processing**: Group insertions and retrievals to minimize cleanup overhead
3. **Strategic Cleanup**: Trigger cleanup during low-activity periods
4. **Monitoring**: Track `raw_len()` vs. `len()` to gauge cleanup effectiveness

Sources: [src/map.rs(L158 - L169)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L158-L169) [src/map.rs(L113 - L115)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L113-L115) [src/map.rs(L171 - L185)&emsp;](https://github.com/Starry-OS/weak-map/blob/b19a081d/src/map.rs#L171-L185)

## Conclusion

Advanced usage of `WeakMap` requires understanding its internal cleanup mechanism, reference type interactions, and memory management characteristics. By applying the patterns and strategies outlined in this document, you can leverage `WeakMap` effectively in complex applications while maintaining optimal performance.