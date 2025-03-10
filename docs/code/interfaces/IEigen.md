# IEigen

## Contract Overview

IEigen is an interface that extends the standard ERC20 token interface (IERC20) with additional functionality specific to the Eigen token ecosystem. This interface defines the contract interactions for Eigen, which appears to be a governance or utility token within a broader Web3 system.

The interface functions suggest that Eigen implements a token with transfer restrictions, wrapping/unwrapping capabilities (likely for a related token named "bEIGEN"), and voting mechanisms through a checkpoint system. The contract follows a controlled access pattern where certain actions are restricted to specific addresses (likely the contract owner or designated roles).

Based on the CLOCK_MODE and clock functions, Eigen also implements EIP-6372, which standardizes the way on-chain time is handled, specifically using timestamp-based voting mechanisms rather than block numbers.

## Contract Interface

### Public/External Functions

1. **Transfer Restriction Management**
   - `setAllowedFrom(address from, bool isAllowedFrom)`: Allows the owner to specify addresses that can transfer tokens from their accounts.
   - `setAllowedTo(address to, bool isAllowedTo)`: Allows the owner to specify addresses that can receive token transfers.
   - `disableTransferRestrictions()`: Allows the owner to completely remove transfer restrictions from the token.

2. **Token Supply Management**
   - `mint()`: Enables authorized minters to create new tokens, increasing the total supply.

3. **Token Wrapping**
   - `wrap(uint256 amount)`: Allows holders of bEIGEN tokens to convert them to Eigen tokens.
   - `unwrap(uint256 amount)`: Allows Eigen token holders to convert them back to bEIGEN tokens.

4. **Checkpoint and Voting Mechanisms**
   - `clock()`: Returns the current timestamp, used for voting checkpoints.
   - `CLOCK_MODE()`: Returns a string indicating the contract uses timestamps for its clock, complying with EIP-6372.

5. **Inherited ERC20 Functions (from IERC20)**
   - Standard ERC20 functions like `transfer`, `approve`, `transferFrom`, `balanceOf`, and `allowance`

### Key Events
While no events are explicitly defined in the interface, the implementation would typically include:
- Events for successful wrapping/unwrapping operations
- Events for changing allowance statuses
- Events for enabling/disabling transfer restrictions
- Standard ERC20 events like Transfer and Approval (inherited)

## Logic Flow

### Token Transfer Workflow
1. A user initiates a token transfer using ERC20's `transfer` or `transferFrom`
2. The contract checks if the sender is in the allowed "from" addresses list
3. The contract checks if the recipient is in the allowed "to" addresses list
4. If both checks pass (or if restrictions are disabled), the transfer proceeds
5. If either check fails, the transfer is rejected

### Token Wrapping Workflow
1. User approves the Eigen contract to spend their bEIGEN tokens
2. User calls `wrap(amount)` with the desired amount
3. The contract transfers bEIGEN tokens from the user to itself
4. The contract mints an equivalent amount of Eigen tokens to the user

### Token Unwrapping Workflow
1. User calls `unwrap(amount)` with the desired amount
2. The contract burns the specified amount of Eigen tokens from the user
3. The contract transfers an equivalent amount of bEIGEN tokens to the user

### Voting Mechanism
1. Actions requiring governance votes use the `clock()` function to timestamp voting periods
2. The contract creates checkpoints at specific timestamps
3. Voting power is determined based on token balance at these checkpoints
4. The `CLOCK_MODE()` function indicates timestamp-based voting to other contracts

## Visual Representation

```mermaid
flowchart TD
    A[User] -->|transfer| B{Transfer Restrictions}
    B -->|Sender Allowed?| C{Recipient Allowed?}
    B -->|Restrictions Disabled| E[Process Transfer]
    C -->|Yes| E
    C -->|No| F[Reject Transfer]
    
    A -->|wrap| G[Wrap bEIGEN to EIGEN]
    G -->|1. Take bEIGEN| H[Lock bEIGEN in Contract]
    H -->|2. Mint EIGEN| I[Send EIGEN to User]
    
    A -->|unwrap| J[Unwrap EIGEN to bEIGEN]
    J -->|1. Burn EIGEN| K[Reduce EIGEN Supply]
    K -->|2. Release bEIGEN| L[Send bEIGEN to User]
    
    M[Owner] -->|setAllowedFrom| N[Update Allowed Senders]
    M -->|setAllowedTo| O[Update Allowed Recipients]
    M -->|disableTransferRestrictions| P[Remove All Restrictions]
    
    Q[Minter] -->|mint| R[Increase EIGEN Supply]
    
    S[Governance] -->|read clock()| T[Get Current Timestamp]
    S -->|check CLOCK_MODE()| U[Confirm Timestamp-Based]
```

## Dependencies and Interactions

### External Dependencies
- **OpenZeppelin's IERC20**: The interface extends OpenZeppelin's standard IERC20 interface, inheriting all standard token functionalities.
- **EIP-6372 Compliance**: The implementation of `clock()` and `CLOCK_MODE()` indicates compliance with this EIP, which standardizes timestamp handling for on-chain voting.

### Related Contracts
- **bEIGEN Token**: Based on the wrapping and unwrapping functions, this interface implies the existence of another token called bEIGEN, which likely has a 1:1 relationship with Eigen. bEIGEN might be a basic or bonded version of the EIGEN token.
- **Access Control System**: The functions that modify allowed addresses suggest the existence of an access control system, possibly using OpenZeppelin's Ownable pattern or a role-based system.

### Key System Interactions
1. **Token Swapping Mechanism**: The wrap/unwrap functions enable users to convert between two token types (EIGEN and bEIGEN).
2. **Governance Integration**: The timestamp-based checkpointing suggests integration with a governance system where token holders can vote on proposals.
3. **Controlled Token Distribution**: The transfer restrictions and minting capabilities indicate a controlled distribution model for the token.

This interface combines standard ERC20 functionality with enhanced features for governance participation and controlled token distribution, making it suitable for a protocol that requires both economic incentives and decentralized governance.