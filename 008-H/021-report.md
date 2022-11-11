pashov

high

# Protocol won't work with `USDC` even though it is a token specifically mentioned in the docs

## Summary
The protocol has requirements for values (for example 1e18) that would be too big if used with a 6 decimals token like `USDC` - `USDC` is mentioned as a token that will be used in the docs
## Vulnerability Detail
For the mint functionality, a user has to transfer at least 1e18 tokens so that he can mint pool tokens - `if (amount < 1e18) revert InvalidActionAmount(amount);`. If the `paymentToken` used was `USDC` (as pointed out in docs), this would mean he would have to contribute at least 1e12 USDC tokens (more than a billion) which would be pretty much impossible to do. There is also another such check in `MarketExtended::addPoolToExistingMarket` with `require(initialActualLiquidityForNewPool >= 1e12, "Insufficient market seed");` - both need huge amounts when using a low decimals token like USDC that has 6 decimals.

## Impact
The protocol just wouldn't work at all in its current state when using a lower decimals token. Since such a token was mentioned in the docs I set this as a High severity issue.

## Code Snippet
https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketExtended.sol#L125
https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketCore.sol#L265
## Tool used

Manual Review

## Recommendation
Drastically lower  the `require` checks so they can work with tokens with a low decimals count like `USDC`