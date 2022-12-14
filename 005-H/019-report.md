AkshaySrivastav

medium

# Failing Contract Initialization

## Summary
The `MarketLiquidityManagerSimple` smart contract makes incorrect use of `initializer` modifier which results in the contract not getting initialized properly.

## Vulnerability Detail
The `constructor` of the `MarketLiquidityManagerSimple` smart contract invokes the `initializer` modifier.
```solidity
  constructor(address _market, address _paymentToken) initializer {
      ...
  }
```
Due to this the `initializer` modifier gets executed as soon as the `MarketLiquidityManagerSimple` contract gets deployed.

The contract also contains an `initialize` function which uses the `initializer` modifier.
```solidity
  function initialize(address admin) external initializer {
    ...
  }
```
Since the `initializer` modifier can only be executed once, the `initialize` function becomes uninvokable.

## Impact
Due to this issue the `_AccessControlledAndUpgradeable_init`  and other chained internal inherited functions cannot be executed, resulting in improper contract initialization.

Hence, due to this issue `DEFAULT_ADMIN_ROLE`, `ADMIN_ROLE`, `UPGRADER_ROLE` roles are not set up for `MarketLiquidityManagerSimple` contract which results in loss of admin restricted and contract upgrade features.

## Code Snippet
[github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/YieldManagers/MarketLiquidityManagerSimple.sol?plain=1#L40](https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/YieldManagers/MarketLiquidityManagerSimple.sol#L40-L49)


## Tool used
A simple test case invoking `MarketLiquidityManagerSimple.initialize()` can be expected to get reverted with message `Initializable: contract is already initialized`.

## Recommendation
Consider removing `initializer` modifier from the `constructor`.
