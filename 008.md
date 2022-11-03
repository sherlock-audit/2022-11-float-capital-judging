ctf_sec

medium

# updateSystemStateUsingValidatedOracleRoundIds in MarketCore can revert because of division by zero error

## Summary

updateSystemStateUsingValidatedOracleRoundIds in MarketCore can revert because of division by zero error

## Vulnerability Detail

When we update the oracle data, we need to call

```solidity
function updateSystemStateUsingValidatedOracleRoundIds(uint80[] memory oracleRoundIdsToExecute) external checkMarketNotDeprecated {
```

which calls

```solidity
      /* i is incremented later in scope*/
      (int256 floatPoolLeverage, ValueChangeAndFunding memory rebalanceParams) = _getValueChangeAndFunding(
        totalEffectiveLiquidityPoolType[LONG_TYPE],
        totalEffectiveLiquidityPoolType[SHORT_TYPE],
        // this is the previous execution price, not the previous oracle update price
        previousPrice,
        epochPrices[i]
      );
```

which calls

```solidity
  function _getValueChangeAndFunding(
    uint256 effectiveValueLong,
    uint256 effectiveValueShort,
    int256 previousPrice,
    int256 currentPrice
  ) internal view returns (int256 floatPoolLeverage, ValueChangeAndFunding memory params) {
    uint256 floatPoolLiquidity = pools[PoolType.FLOAT][0].value;
    // We set the floating tranche leverage to the exact leverage that ensure effectiveValueLong = effectiveValueShort when taking
    //     into the floating liquidity added the underbalanced side.
    floatPoolLeverage = (int256(effectiveValueShort) - int256(effectiveValueLong)).div(int256(floatPoolLiquidity));
```

Well, if floatPoolLiquidity is 0,

```solidity
 floatPoolLeverage = (int256(effectiveValueShort) - int256(effectiveValueLong)).div(int256(floatPoolLiquidity));
```

revert if division by zero error.

## Impact

updateSystemStateUsingValidatedOracleRoundIds in MarketCore can revert because of division by zero error if the floatPoolLiquidity is 0.

## Code Snippet

https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketCore.sol#L201-L210

https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketCore.sol#L69-L79

## Tool used

Manual Review

## Recommendation

We recommend the project add code to handle the case when floatPoolLiquidity is 0 before the division.
