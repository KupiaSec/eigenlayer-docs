# IEigenPodManager

## Contract Overview

The `IEigenPodManager` interface defines a system for managing Ethereum validators that want to participate in EigenLayer, a protocol for restaking ETH. This contract is critical in the EigenLayer architecture as it allows ETH validators to create special pods (EigenPods) that point their withdrawal credentials to EigenLayer, enabling them to restake their staked ETH and receive additional rewards.

The primary purpose of this contract is to create, track, and manage EigenPod instances, which are individual smart contracts that represent Ethereum validators participating in EigenLayer. It allows users to create pods, stake ETH to become validators, track balances, handle slashing events, and manage withdrawals.

The contract uses a factory pattern to deploy EigenPods for users, a proxy pattern (via beacon proxies) for upgradability, and implements a sophisticated share accounting system to track restaked ETH. It also features safety mechanisms like pausability and slashing factor tracking.

## Contract Interface

### Key Functions

- **`createPod()`**: Creates an EigenPod for the caller, allowing them to participate in EigenLayer with their validators.
  
- **`stake(bytes calldata pubkey, bytes calldata signature, bytes32 depositDataRoot)`**: Stakes 32 ETH for a new beacon chain validator on the caller's EigenPod and creates a pod if they don't have one.

- **`recordBeaconChainETHBalanceUpdate(address podOwner, uint256 prevRestakedBalanceWei, int256 balanceDeltaWei)`**: Updates the record of how much ETH a pod owner has staked, handling both increases and decreases in balance.

- **`getPod(address podOwner)`**: Retrieves the EigenPod address for a given pod owner.

- **`hasPod(address podOwner)`**: Checks if a given address has created an EigenPod.

### Important Events

- **`PodDeployed`**: Emitted when a new EigenPod is created for a user.
  
- **`BeaconChainETHDeposited`**: Emitted when a deposit of beacon chain ETH is recorded in the strategy manager.

- **`PodSharesUpdated`**: Emitted when an EigenPod's balance is updated.

- **`BeaconChainETHWithdrawalCompleted`**: Emitted when a withdrawal of beacon chain ETH is completed.

- **`BeaconChainSlashingFactorDecreased`**: Emitted when a staker's slashing factor is updated due to slashing events.

### Key State Variables

- **`ownerToPod`**: Maps user addresses to their EigenPod contract addresses.

- **`podOwnerDepositShares`**: Tracks the number of shares a pod owner has in the virtual beacon chain ETH strategy.

- **`beaconChainSlashingFactor`**: Records the historical balance decreases a pod owner has experienced due to slashing.

- **`burnableETHShares`**: Accumulated amount of ETH shares that can be burned (likely from slashing events).

## Logic Flow

The main workflow in this contract follows these steps:

1. **Pod Creation**: A user calls `createPod()` to deploy their own EigenPod contract, which will serve as their interface to the EigenLayer system.

2. **Staking**: Through the `stake()` function, users can deposit 32 ETH to become Ethereum validators, with withdrawal credentials pointed to their EigenPod.

3. **Balance Tracking**: The EigenPod monitors the validator's beacon chain balance and reports changes through `recordBeaconChainETHBalanceUpdate()`.
   
4. **Share Accounting**:
   - When balances increase, shares are added to the pod owner's account.
   - When balances decrease (e.g., due to slashing), the system updates the slashing factor.
   - The system keeps track of shares using a sophisticated accounting mechanism that can handle both positive and negative balances.

5. **Withdrawals**: When withdrawals occur, the contract emits the `BeaconChainETHWithdrawalCompleted` event and updates accounting accordingly.

The contract also maintains a "slashing factor" for each staker, which represents their historical experience with slashing on the beacon chain. This factor decreases when slashing occurs and affects the amount of shares they have delegated.

An innovative aspect of the design is the handling of negative share balances. This accommodates scenarios where a pod owner's beacon chain ETH shares decrease between queuing and completing a withdrawal.

## Visual Representation

```mermaid
flowchart TD
    U[User] -->|createPod()| EPM[EigenPodManager]
    EPM -->|deploys| EP[EigenPod]
    U -->|stake()| EPM
    EPM -->|forwards stake request| EP
    EP -->|deposits| ETHPoS[ETH PoS Deposit Contract]
    ETHPoS -->|creates validator| BC[Beacon Chain]
    BC -->|balance updates| EP
    EP -->|recordBeaconChainETHBalanceUpdate()| EPM
    EPM -->|updates| Shares[Share Accounting]
    EPM -->|may update| SlashingFactor[Slashing Factor]
    U -->|withdrawal request| EP
    EP -->|completes withdrawal| EPM
    EPM -->|emits| WithdrawalEvent[BeaconChainETHWithdrawalCompleted]
```

## Dependencies and Interactions

The `IEigenPodManager` interacts with several other contracts in the EigenLayer ecosystem:

1. **IEigenPod**: Individual pods created for users to interact with the Ethereum validator system.

2. **IETHPOSDeposit**: The official Ethereum 2.0 deposit contract for becoming a validator.

3. **IStrategyManager**: Likely manages the different strategies for restaking, including the virtual beacon chain ETH strategy.

4. **IBeacon**: The OpenZeppelin beacon contract used for the proxy pattern, allowing for upgradability of the EigenPods.

5. **IShareManager**: Interface for managing shares in the system.

6. **IStrategy**: Interface for the virtual beacon chain ETH strategy that represents restaked ETH.

The contract also depends on OpenZeppelin's contracts for the proxy/beacon pattern and implements a semver interface for versioning, as well as a pausable interface for emergency control.

The design shows careful consideration of the complexities involved in validator slashing and withdrawal processes. By tracking share balances that can go negative, the system can handle the asynchronous nature of the Ethereum beacon chain operations, ensuring that users cannot withdraw more than they are entitled to, even if slashing events occur during withdrawal processes.