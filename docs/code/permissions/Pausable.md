# Pausable

## 1. Contract Overview

The `Pausable` contract is a foundational security component that implements a sophisticated pause mechanism for smart contracts in the EigenLayer ecosystem. Its primary purpose is to provide emergency circuit breaker functionality, allowing designated entities to suspend specific contract operations when necessary, such as during upgrades or when security vulnerabilities are discovered.

This abstract contract is designed to be inherited by other contracts in the EigenLayer system that require pausability features. Rather than implementing a simple boolean pause mechanism, it uses a bitmap approach with up to 256 different pause flags, allowing for fine-grained control over which specific functionalities can be paused independently.

The contract leverages several key design patterns:
- **Role-Based Access Control**: Pause/unpause functions are restricted to designated pausers and unpausers
- **Bitmap Flags**: Uses a 256-bit integer to represent multiple pause switches
- **Registry Pattern**: Defers access control decisions to a separate PauserRegistry contract
- **Modifier Pattern**: Provides reusable checks for paused status and authorization

## 2. Contract Interface

### State Variables
- `_paused`: A uint256 bitmap where each bit represents a different pause flag. When a bit is set to 1, the corresponding functionality is paused.
- `pauserRegistry`: An immutable reference to the PauserRegistry contract that manages authorization for pause/unpause operations.

### Public/External Functions
- `pause(uint256 newPausedStatus)`: Allows a pauser to selectively pause functionalities by setting specific bits in the pause bitmap.
- `pauseAll()`: Convenience function that pauses all functionalities by setting all bits to 1.
- `unpause(uint256 newPausedStatus)`: Allows the unpauser to selectively unpause functionalities.
- `paused()`: Returns the complete pause status bitmap.
- `paused(uint8 index)`: Checks if a specific functionality (identified by bit index) is paused.

### Events
- `Paused(address indexed account, uint256 newPausedStatus)`: Emitted when the pause status changes to a new paused state.
- `Unpaused(address indexed account, uint256 newPausedStatus)`: Emitted when the pause status changes to a new unpaused state.

### Modifiers
- `onlyPauser()`: Restricts functions to be callable only by addresses authorized as pausers.
- `onlyUnpauser()`: Restricts functions to be callable only by the designated unpauser.
- `whenNotPaused()`: Requires that the contract is not paused in any way.
- `onlyWhenNotPaused(uint8 index)`: Requires that a specific functionality is not paused.

## 3. Logic Flow

### Pause Mechanism
1. The contract maintains a bitmap (`_paused`) where each bit position represents a specific functionality that can be paused.
2. When a bit is set to 1, the corresponding functionality is considered paused.
3. When a bit is set to 0, the corresponding functionality is considered active.

### Pausing Workflow
1. A designated pauser calls `pause()` with a bitmap indicating which functionalities to pause.
2. The contract verifies the caller is an authorized pauser via the PauserRegistry.
3. The contract ensures the new pause status only adds pauses (it can't unpause anything).
4. The pause status is updated, and the `Paused` event is emitted.

### Unpausing Workflow
1. The designated unpauser calls `unpause()` with a bitmap indicating the new pause status.
2. The contract verifies the caller is the authorized unpauser via the PauserRegistry.
3. The contract ensures the new pause status only removes pauses (it can't pause anything new).
4. The pause status is updated, and the `Unpaused` event is emitted.

### Security Mechanisms
1. **Role Segregation**: Pausing and unpausing permissions are separated between different roles.
2. **External Registry**: Authorization logic is delegated to a separate contract, following separation of concerns.
3. **Bitmap Validation**: The contract ensures that pause operations can only add pauses and unpause operations can only remove pauses.
4. **Storage Gap**: Includes a storage gap to allow for future upgrades without storage collision.

## 4. Visual Representation

```mermaid
flowchart TD
    subgraph Pausable Contract
        constructor[Constructor\nInitializes pauserRegistry]
        pauseMethod[pause(uint256 newPausedStatus)\nOnly pauser can call]
        pauseAllMethod[pauseAll()\nOnly pauser can call]
        unpauseMethod[unpause(uint256 newPausedStatus)\nOnly unpauser can call]
        pausedGetter[paused()\nReturns complete bitmap]
        pausedIndexGetter[paused(uint8 index)\nChecks specific bit]
        pausedStatus[_paused\nBitmap storage]
    end

    subgraph Modifiers
        onlyPauser[onlyPauser\nChecks if caller is pauser]
        onlyUnpauser[onlyUnpauser\nChecks if caller is unpauser]
        whenNotPaused[whenNotPaused\nChecks if fully unpaused]
        onlyWhenNotPaused[onlyWhenNotPaused\nChecks specific bit]
    end

    subgraph External
        PauserRegistry[IPauserRegistry\nManages pause permissions]
        InheritingContract[Contract inheriting Pausable\nImplements pausing logic]
    end
    
    constructor --> PauserRegistry
    pauseMethod --> onlyPauser
    pauseAllMethod --> onlyPauser
    unpauseMethod --> onlyUnpauser
    
    onlyPauser --> PauserRegistry
    onlyUnpauser --> PauserRegistry
    
    pauseMethod --> pausedStatus
    pauseAllMethod --> pausedStatus
    unpauseMethod --> pausedStatus
    
    pausedGetter --> pausedStatus
    pausedIndexGetter --> pausedStatus
    
    InheritingContract --> Pausable
    whenNotPaused --> pausedStatus
    onlyWhenNotPaused --> pausedStatus
```

## 5. Dependencies and Interactions

### External Dependencies
- **IPausable**: Interface that defines the pause-related functions and events.
- **IPauserRegistry**: Interface for the registry contract that manages pauser and unpauser access control.

### Contract Interactions
1. **Interaction with PauserRegistry**:
   - The contract calls `isPauser(address)` to verify if an account is authorized to pause.
   - The contract calls `unpauser()` to verify if an account is authorized to unpause.

2. **Interaction with Inheriting Contracts**:
   - Contracts that inherit from `Pausable` can define specific pause flags for different functionalities.
   - Inheriting contracts should use the provided modifiers (`whenNotPaused`, `onlyWhenNotPaused`) to protect functions that should respect pause status.
   - Inheriting contracts can implement custom pause/unpause functions that target specific bits in the pause bitmap, with appropriate access control.

The pausable pattern implemented in this contract provides an elegant solution for emergency circuit breaking with granular control. By using a bitmap approach rather than a simple boolean, the system can selectively pause specific functions while keeping others operational, minimizing disruption during maintenance or emergency scenarios.