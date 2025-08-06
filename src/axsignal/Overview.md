# Overview

> **Relevant source files**
> * [Cargo.toml](https://github.com/Starry-OS/axsignal/blob/b5b6089c/Cargo.toml)
> * [src/lib.rs](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs)

## Purpose and Scope

The `axsignal` crate implements a Unix-like signal handling system for ArceOS. It provides a comprehensive framework for managing signals at both process and thread levels, supporting standard operations such as sending, blocking, and handling signals. This crate enables applications running on ArceOS to use familiar signal handling patterns similar to those found in POSIX systems.

For detailed explanations of specific components, see [Signal Management System](/Starry-OS/axsignal/2-signal-management-system), [Signal Types and Structures](/Starry-OS/axsignal/3-signal-types-and-structures), and [Architecture Support](/Starry-OS/axsignal/4-architecture-support).

Sources: [src/lib.rs(L1 - L16)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L1-L16) [Cargo.toml(L1 - L31)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/Cargo.toml#L1-L31)

## High-Level Architecture

The `axsignal` crate is organized into several interconnected modules that together form a complete signal handling system.

```mermaid
flowchart TD
subgraph subGraph2["axsignal Crate"]
    lib["lib.rs"]
    subgraph subGraph1["Architecture Support"]
        x86_64["arch/x86_64.rs"]
        aarch64["arch/aarch64.rs"]
        riscv["arch/riscv.rs"]
        loongarch64["arch/loongarch64.rs"]
    end
    subgraph subGraph0["Core Modules"]
        action["action.rs"]
        types["types.rs"]
        pending["pending.rs"]
        api["api.rs"]
        arch["arch/mod.rs"]
    end
end

arch --> aarch64
arch --> loongarch64
arch --> riscv
arch --> x86_64
lib --> action
lib --> api
lib --> arch
lib --> pending
lib --> types
```

**High-Level Architecture of axsignal**

Sources: [src/lib.rs(L7 - L15)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L7-L15)

## Key Components

The `axsignal` crate consists of several key components that work together to provide signal handling functionality:

### Signal Types

```mermaid
classDiagram
class Signo {
    +value: u8
    +const SIGHUP, SIGINT, SIGQUIT, etc.
    +is_standard()
    +is_realtime()
}

class SignalSet {
    +bits: u64
    +new()
    +add_signal(Signo)
    +del_signal(Signo)
    +contains(Signo)
    +is_empty()
}

class SignalInfo {
    +signo: Signo
    +code: i32
    +errno: i32
    +fields: SignalFields
    
}

class SignalStack {
    +ss_sp: usize
    +ss_flags: i32
    +ss_size: usize
    
}

SignalInfo  -->  Signo
```

**Signal Type Components**

### Signal Managers

```mermaid
classDiagram
class ProcessSignalManager {
    +pending: Mutex
    +actions: Arc~
    +send_signal(sig: SignalInfo)
    +dequeue_signal(mask: SignalSet)
    +pending()
    +wait_signal()
}

class ThreadSignalManager {
    -proc: Arc
    -pending: Mutex
    -blocked: Mutex
    -stack: Mutex
    +send_signal(sig: SignalInfo)
    +dequeue_signal(mask: SignalSet)
    +check_signals(tf, restore_blocked)
    +handle_signal(tf, restore_blocked, sig, action)
}

class PendingSignals {
    +set: SignalSet
    +info_std: Option[32]
    +info_rt: VecDeque
    +put_signal(sig: SignalInfo)
    +dequeue_signal(mask: SignalSet)
}

ThreadSignalManager  -->  ProcessSignalManager
ProcessSignalManager  -->  PendingSignals
ThreadSignalManager  -->  PendingSignals
```

**Signal Management Components**

Sources: [src/lib.rs(L7 - L15)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L7-L15)

## Signal Processing Flow

The signal handling process in `axsignal` follows a well-defined flow from generation to handling:

```mermaid
flowchart TD
subgraph subGraph2["Signal Handling"]
    CheckAction["Check SignalAction"]
    Default["Execute Default Action"]
    Ignore["Ignore Signal"]
    Handler["Execute Custom Handler"]
    SaveContext["Save CPU Context"]
    ExecuteHandler["Run Handler Function"]
    RestoreContext["Restore CPU Context"]
end
subgraph subGraph1["Signal Queuing"]
    CheckBlocked["Is Signal Blocked?"]
    Queue["Add to PendingSignals"]
    Deliver["Deliver Immediately"]
end
subgraph subGraph0["Signal Generation"]
    Start["Signal Generated"]
    SendToProcess["ProcessSignalManager.send_signal()"]
    SendToThread["ThreadSignalManager.send_signal()"]
end
CheckPending["check_signals()"]

CheckAction --> Default
CheckAction --> Handler
CheckAction --> Ignore
CheckBlocked --> Deliver
CheckBlocked --> Queue
CheckPending --> Deliver
Deliver --> CheckAction
ExecuteHandler --> RestoreContext
Handler --> SaveContext
Queue --> CheckPending
SaveContext --> ExecuteHandler
SendToProcess --> CheckBlocked
SendToThread --> CheckBlocked
Start --> SendToProcess
Start --> SendToThread
```

**Signal Processing Flow**

Sources: [src/lib.rs(L7 - L15)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L7-L15)

## System Dependencies and Integration

The `axsignal` crate integrates with several core components of ArceOS to provide its functionality:

```mermaid
flowchart TD
subgraph subGraph1["External Dependencies"]
    axerrno["axerrno - Error Codes"]
    bitflags["bitflags - Bit Flag Manipulation"]
    log["log - Logging Infrastructure"]
end
subgraph subGraph0["ArceOS Core Components"]
    axconfig["axconfig - System Configuration"]
    axhal["axhal - Hardware Abstraction Layer"]
    axtask["axtask - Task Management"]
end
axsignal["axsignal"]

axsignal --> axconfig
axsignal --> axerrno
axsignal --> axhal
axsignal --> axtask
axsignal --> bitflags
axsignal --> log
```

**Dependencies and Integration**

The crate relies on:

* `axconfig`: For system configuration parameters
* `axhal`: For hardware abstraction with userspace support
* `axtask`: For multitasking integration
* `axerrno`: For error handling
* `bitflags`: For efficient signal set implementation
* Additional utilities for logging and synchronization

Sources: [Cargo.toml(L6 - L26)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/Cargo.toml#L6-L26)

## Architecture Support

The `axsignal` crate provides platform-specific implementations for different CPU architectures, ensuring proper signal context management:

```mermaid
flowchart TD
subgraph subGraph2["Architecture-Specific Features"]
    context["Machine Context Management"]
    trampoline["Signal Trampoline Code"]
    setup["Signal Handler Setup"]
    restore["Context Restoration"]
end
subgraph subGraph1["Architecture Support"]
    arch_mod["arch/mod.rs - Common Interface"]
    subgraph subGraph0["Platform-Specific Implementations"]
        x86_64["arch/x86_64.rs"]
        aarch64["arch/aarch64.rs"]
        riscv["arch/riscv.rs"]
        loongarch64["arch/loongarch64.rs"]
    end
end

aarch64 --> context
aarch64 --> restore
aarch64 --> setup
aarch64 --> trampoline
arch_mod --> aarch64
arch_mod --> loongarch64
arch_mod --> riscv
arch_mod --> x86_64
loongarch64 --> context
loongarch64 --> restore
loongarch64 --> setup
loongarch64 --> trampoline
riscv --> context
riscv --> restore
riscv --> setup
riscv --> trampoline
```

**Architecture Support System**

Each architecture implementation provides specialized functionality for:

* Saving and restoring CPU registers during signal handling
* Setting up signal trampolines (code that transfers control to user-defined handlers)
* Managing signal stacks
* Handling architecture-specific signal delivery requirements

Sources: [src/lib.rs(L8 - L9)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L8-L9)

## Summary

The `axsignal` crate provides a comprehensive signal handling system for ArceOS, implementing familiar Unix-like functionality across multiple processor architectures. It manages signals at both process and thread levels, supports standard and real-time signals, and offers a flexible framework for defining custom signal actions.

Key features include:

* Process and thread level signal management
* Support for multiple architectures (x86_64, AArch64, RISC-V, LoongArch64)
* Signal masking and prioritization
* Custom signal handlers with context management
* Integration with ArceOS task management

For detailed information about specific components, refer to the dedicated pages on [Signal Management System](/Starry-OS/axsignal/2-signal-management-system) and [Signal Types and Structures](/Starry-OS/axsignal/3-signal-types-and-structures).

Sources: [src/lib.rs(L1 - L16)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/src/lib.rs#L1-L16) [Cargo.toml(L1 - L31)&emsp;](https://github.com/Starry-OS/axsignal/blob/b5b6089c/Cargo.toml#L1-L31)