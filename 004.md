Sm4rty

medium

# Chainlink's latestRoundData might return stale or incorrect results

## Summary
Chainlink's latestRoundData might return stale or incorrect results as there is no check for revert if the function returns stale data.

## Vulnerability Detail
The initializePools Function in MarketExtended.sol Contract calls out to a Chainlink oracle receiving the latestRoundData(). If there is a problem with Chainlink starting a new round and finding consensus on the new value for the oracle (e.g. Chainlink nodes abandon the oracle, chain congestion, vulnerability/attacks on the chainlink system) consumers of this contract may continue using outdated stale or incorrect data (if oracles are unable to submit no new round is started).

## Impact
The contract may continue using outdated stale or incorrect data (if oracles are unable to submit no new round is started).

## Code Snippet
https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/market/template/MarketExtended.sol#L37

## Tool used

Manual Review

## Recommendation
add the following checks to the code.
```solidity
 (uint80 latestRoundId, int256 initialAssetPrice, , , ) = oracleManager.chainlinkOracle().latestRoundData(); 

+ require(rawPrice > 0, "Chainlink price <= 0");
+ require(updateTime != 0, "Incomplete round");
+ require(answeredInRound >= roundId, "Stale price");
```