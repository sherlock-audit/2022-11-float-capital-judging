ctf_sec

medium

# Incompatible with fee-on-transfer token

## Summary

Incompatible with fee-on-transfer token

## Vulnerability Detail

In the function _mint, we use the amount user passed in for accounting, but not check the actual account we received.

```solidity
    IERC20(paymentToken).safeTransferFrom(msg.sender, liquidityManager, amount);

    uint256 fees = _calculateStabilityFees(uint256(amount).mul(int256(pools[poolType][poolTier].fixedConfig.leverage).abs()));
    amount -= uint112(fees);
```

and

```solidity
   /// NOTE: userAction.amount > 0 IFF userAction.correspondingEpoch <= currentEpoch - this check is redundant for safety.
    if (userAction.amount > 0 && userAction.correspondingEpoch < currentEpoch) {
      // This case occurs when a user minted in the previous epoch and upkeep has still not yet
      // occured and therefore this previous order has not been processed.
      // This is likely to happen if the user mints early on in a new epoch when enough time has not
      // passed (see MEWT) for the previous epoch to be executed.
      userAction.nextEpochAmount += amount;
    } else {
      userAction.amount += amount;
      userAction.correspondingEpoch = currentEpoch;
    }
```

## Impact

Some tokens take a transfer fee (e.g. STA, PAXG), some do not currently charge a fee but may do so in the future (e.g. USDT, USDC).

For example, if the token charge 1% transfer fee, the user wants to mint 100 amount, but the contract actually receive 99 amount.

but the code assumes that we receive 100 token and use the number 100 for accounting.

```solidity
userAction.nextEpochAmount += amount;
} else {
userAction.amount += amount;
userAction.correspondingEpoch = currentEpoch;
```

## Code Snippet

https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketCore.sol#L252-L307

## Tool used

Manual Review

## Recommendation

We recommend the project use before and after balance check to check how many token we actually receive instead of assume we received the exact token amount from user's input.

```solidity
    uint256 balanceBefore = IERC20(paymentToken).balanceOf(liquidityManager);
    IERC20(paymentToken).safeTransferFrom(msg.sender, liquidityManager, amount);
    uint256 balanceAfter = IERC20(paymentToken).balanceOf(liquidityManager);
    amount = balanceAfter - balanceBefore;

    uint256 fees = _calculateStabilityFees(uint256(amount).mul(int256(pools[poolType][poolTier].fixedConfig.leverage).abs()));
    amount -= uint112(fees);
```