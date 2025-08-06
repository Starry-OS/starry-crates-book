# Starry Crates Book

# weak-map

- [Overview](./weak-map/Overview.md)

- [Core Components](./weak-map/Core_Components.md)
    - [WeakMap and StrongMap](./weak-map/WeakMap_and_StrongMap.md)
    - [Reference Traits](./weak-map/Reference_Traits.md)

- [Usage Guide](./weak-map/Usage_Guide.md)
    - [Basic Usage Examples](./weak-map/Basic_Usage_Examples.md)
    - [Advanced Usage Patterns](./weak-map/Advanced_Usage_Patterns.md)

- [Implementation Details](./weak-map/Implementation_Details.md)
    - [Memory Management](./weak-map/Memory_Management.md)
    - [Performance Considerations](./weak-map/Performance_Considerations.md)

- [Project Information](./weak-map/Project_Information.md)
    - [Contributing Guide](./weak-map/Contributing_Guide.md)
    - [License Information](./weak-map/License_Information.md)

# axns

- [Overview](./axns/Overview.md)

- [Core Concepts](./axns/Core_Concepts.md)
    - [Namespaces](./axns/Namespaces.md)
    - [Resources and ResWrapper](./axns/Resources_and_ResWrapper.md)
    - [The def_resource! Macro](./axns/The_def_resource!_Macro.md)

- [Thread-Local Features](./axns/Thread-Local_Features.md)

- [Resource Lifecycle](./axns/Resource_Lifecycle.md)
    - [Resource Reference Counting](./axns/Resource_Reference_Counting.md)

- [Usage Guide](./axns/Usage_Guide.md)
    - [Basic Resource Access](./axns/Basic_Resource_Access.md)
    - [Sharing and Resetting Resources](./axns/Sharing_and_Resetting_Resources.md)

- [Development and Testing](./axns/Development_and_Testing.md)

# axsignal

- [Overview](./axsignal/Overview.md)

- [Signal Management System](./axsignal/Signal_Management_System.md)
    - [Thread Signal Manager](./axsignal/Thread_Signal_Manager.md)
    - [Process Signal Manager](./axsignal/Process_Signal_Manager.md)
    - [Wait Queue Interface](./axsignal/Wait_Queue_Interface.md)

- [Signal Types and Structures](./axsignal/Signal_Types_and_Structures.md)
    - [Signal Numbers and Sets](./axsignal/Signal_Numbers_and_Sets.md)
    - [Signal Actions and Dispositions](./axsignal/Signal_Actions_and_Dispositions.md)
    - [Pending Signals](./axsignal/Pending_Signals.md)

- [Architecture Support](./axsignal/Architecture_Support.md)
    - [x86_64 Implementation](./axsignal/x86_64_Implementation.md)
    - [ARM64 Implementation](./axsignal/ARM64_Implementation.md)
    - [RISC-V Implementation](./axsignal/RISC-V_Implementation.md)
    - [LoongArch64 Implementation](./axsignal/LoongArch64_Implementation.md)

- [Build Configuration and Dependencies](./axsignal/Build_Configuration_and_Dependencies.md)

# axptr

- [Overview](./axptr/Overview.md)

- [Memory Safety Architecture](./axptr/Memory_Safety_Architecture.md)
    - [User Space Pointers](./axptr/User_Space_Pointers.md)
    - [Address Space Management](./axptr/Address_Space_Management.md)

- [Safety Mechanisms](./axptr/Safety_Mechanisms.md)
    - [Memory Region Checking](./axptr/Memory_Region_Checking.md)
    - [Context-Aware Page Fault Handling](./axptr/Context-Aware_Page_Fault_Handling.md)
    - [Null-Terminated Data Handling](./axptr/Null-Terminated_Data_Handling.md)

- [Integration with Operating System](./axptr/Integration_with_Operating_System.md)

- [API Reference](./axptr/API_Reference.md)
    - [UserPtr API](./axptr/UserPtr_API.md)
    - [UserConstPtr API](./axptr/UserConstPtr_API.md)
    - [Helper Functions](./axptr/Helper_Functions.md)

# axprocess

- [Overview](./axprocess/Overview.md)
    - [Core Architecture](./axprocess/Core_Architecture.md)

- [Process Management](./axprocess/Process_Management.md)
    - [Process Creation and Initialization](./axprocess/Process_Creation_and_Initialization.md)
    - [Process Lifecycle](./axprocess/Process_Lifecycle.md)
    - [Parent-Child Relationships](./axprocess/Parent-Child_Relationships.md)

- [Process Groups and Sessions](./axprocess/Process_Groups_and_Sessions.md)
    - [Process Groups](./axprocess/Process_Groups.md)
    - [Sessions](./axprocess/Sessions.md)
    - [Hierarchy and Movement](./axprocess/Hierarchy_and_Movement.md)

- [Thread Management](./axprocess/Thread_Management.md)
    - [Thread Creation and Builder](./axprocess/Thread_Creation_and_Builder.md)
    - [Thread Lifecycle and Exit](./axprocess/Thread_Lifecycle_and_Exit.md)

- [Memory Management](./axprocess/Memory_Management.md)
    - [Reference Counting and Ownership](./axprocess/Reference_Counting_and_Ownership.md)
    - [Zombie Processes and Cleanup](./axprocess/Zombie_Processes_and_Cleanup.md)

- [Development and Testing](./axprocess/Development_and_Testing.md)
    - [Testing Approach](./axprocess/Testing_Approach.md)
    - [CI/CD Pipeline](./axprocess/CI_CD_Pipeline.md)
