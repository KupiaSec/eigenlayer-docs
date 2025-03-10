# IStrategy

## 1. Contract Overview

### Purpose and Main Functionality
The `IStrategy` interface defines the standard functionality for strategy contracts within the EigenLayer protocol. These strategies represent different ways users can deploy their assets within the EigenLayer ecosystem. Each strategy handles specific tokens and provides users with shares in proportion to their deposits, allowing them to participate in the protocol while maintaining a claim on their underlying assets.

### System Architecture Context
Within the EigenLayer ecosystem, strategy contracts serve as asset management vehicles that interact primarily with the StrategyManager contract. The StrategyManager tracks user deposits across different strategies and manages the overall delegation of these assets, while the individual strategy contracts implement the specific logic for how those assets are deployed.

### Key Design Patterns
- **Interface Segregation**: The contract separates concerns by splitting functionality into distinct interfaces (`IStrategyErrors`, `IStrategyEvents`, and `ISemVerMixin`).
- **Token-Based Accounting**: Uses an shares-based accounting system where users receive shares proportional to their deposits.
- **Exchange Rate Mechanisms**: Provides functions to convert between shares and underlying tokens.
- **Role-Based Access Control**: Restricts key functions to only be callable by the StrategyManager.

## 2. Contract Interface

### Public/External Functions

#### Deposit/Withdraw Functions
- `deposit(IERC20 token, uint256 amount)`: Allows the StrategyManager to deposit tokens into the strategy, converting the amount to shares.
- `withdraw(address recipient, IERC20 token, uint256 amountShares)`: Withdraws tokens from the strategy to a recipient address based on a specified amount of shares.

#### Conversion Functions
- `sharesToUnderlying(uint256 amountShares)`: Converts shares to the equivalent amount of underlying tokens, may modify state.
- `sharesToUnderlyingView(uint256 amountShares)`: View-only version of the above function.
- `underlyingToShares(uint256 amountUnderlying)`: Converts underlying tokens to the equivalent amount of shares, may modify state.
- `underlyingToSharesView(uint256 amountUnderlying)`: View-only version of the above function.

#### User Information Functions
- `userUnderlying(address user)`: Calculates the underlying value of all a user's shares, may modify state.
- `userUnderlyingView(address user)`: View-only version of the above function.
- `shares(address user)`: Returns the current total shares a user has in this strategy.

#### State Information Functions
- `underlyingToken()`: Returns the ERC20 token that the strategy accepts.
- `totalShares()`: Returns the total number of shares issued by the strategy.
- `explanation()`: Returns a description or link to metadata explaining the strategy's purpose.

### Important Events
- `ExchangeRateEmitted(uint256 rate)`: Emitted when the exchange rate between shares and the underlying token changes.
- `StrategyTokenSet(IERC20 token, uint8 decimals)`: Emitted upon strategy creation to indicate the underlying token and its decimals.

### Key Errors
- `OnlyStrategyManager()`: Thrown when functions restricted to the StrategyManager are called by other addresses.
- `NewSharesZero()`: Thrown when a deposit would result in zero new shares.
- `TotalSharesExceedsMax()`: Thrown when total shares would exceed the maximum allowed.
- `WithdrawalAmountExceedsTotalDeposits()`: Thrown when attempting to withdraw more shares than available.
- `OnlyUnderlyingToken()`: Thrown when attempting actions with tokens not accepted by the strategy.
- `MaxPerDepositExceedsMax()`: Error related to per-deposit limits.
- `BalanceExceedsMaxTotalDeposits()`: Error when a deposit would exceed maximum strategy capacity.

## 3. Logic Flow

### Deposit Workflow
1. The StrategyManager calls the `deposit` function with a token and amount.
2. The strategy verifies the token is the accepted underlying token.
3. The strategy calculates the number of shares to be minted based on the current exchange rate.
4. New shares are minted and recorded in the StrategyManager.
5. The function returns the number of new shares issued.

### Withdrawal Workflow
1. The StrategyManager calls the `withdraw` function with recipient, token, and amount of shares.
2. The strategy verifies the token is the accepted underlying token.
3. The strategy calculates the amount of underlying tokens corresponding to the shares being withdrawn.
4. The calculated amount of underlying tokens is transferred to the recipient.
5. The shares are burned/recorded as withdrawn in the StrategyManager.

### Share/Token Conversion
The contract maintains an exchange rate between shares and the underlying token. This rate may:
- Start at 1:1 (one token = one share)
- Change over time based on strategy performance
- Be calculated differently in various strategy implementations

The conversion functions allow users and the protocol to determine:
- How many tokens they would receive for a given amount of shares
- How many shares they would receive for depositing a given amount of tokens

This exchange rate represents the key mechanism by which users can benefit from strategy performance, as shares may appreciate in value relative to the underlying token.

### Security Mechanisms
- Function access control (only StrategyManager can deposit/withdraw)
- Validation checks for token types
- Checks to prevent withdrawal amounts exceeding available shares
- Limits on total deposits and per-deposit amounts

## 4. Visual Representation

```mermaid
flowchart TD
    User([User]) --> |Deposits tokens| SM[StrategyManager]
    SM --> |deposit()| Strategy[Strategy Contract]
    Strategy --> |Returns shares| SM
    SM --> |Records user shares| SM
    
    User --> |Requests withdrawal| SM
    SM --> |withdraw()| Strategy
    Strategy --> |Sends tokens| User
    
    User --> |Queries position| SM
    SM --> |shares()| Strategy
    Strategy --> |sharesToUnderlyingView()| Strategy
    Strategy --> |Returns token value| User
    
    subgraph Strategy Implementation
        Strategy --> |Manages| UnderlyingTokens[(Underlying Tokens)]
        Strategy --> |Tracks| TotalShares[(Total Shares)]
        Strategy --> |Calculates| ExchangeRate[(Exchange Rate)]
    end
```

## 5. Dependencies and Interactions

### Contract Dependencies
- **@openzeppelin/contracts/token/ERC20/IERC20.sol**: Used for the underlying token interface.
- **../libraries/SlashingLib.sol**: While imported, this library isn't directly used in the interface but is likely used by implementing contracts for slashing logic.
- **./ISemVerMixin.sol**: Provides semantic versioning functionality for the contract.

### External Contract Interactions
- **StrategyManager**: The primary contract that interacts with Strategy contracts. It manages user deposits across different strategies and is the only contract authorized to call the deposit and withdraw functions.
- **DelegationManager**: Referenced in documentation comments, suggesting that users can delegate their strategy shares to operators within the EigenLayer system.

### System Interactions
Strategy contracts serve as the bridge between user assets and whatever protocol or system they deploy those assets to. While this interface doesn't specify implementation details, real-world strategies might:

1. Deploy funds to external protocols like lending markets, DEXes, or yield farms
2. Provide liquidity to various DeFi applications
3. Stake assets in proof-of-stake validators
4. Create derivatives or other financial products

The primary value proposition is allowing users to deposit assets into EigenLayer while still generating yields from other activities, with the exchange rate mechanism enabling users to benefit from that yield.

This interface creates a standardized way for the EigenLayer protocol to interact with diverse strategies while abstracting away the specifics of how each strategy generates returns.