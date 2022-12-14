imare

medium

# wrong return parameter used in ``validateAndReturnMissedEpochInformation`` for validating oracle rounds

## Summary

As stated in the [chainlink documents](https://docs.chain.link/docs/data-feeds/price-feeds/historical-data/) :
```text
To check the round, validate that the timestamp on that round is not 0.
```

## Vulnerability Detail
This timestamp is specified as the 4th return parameter in calling ``getRoundData``. Look at the first solidity example in the documentation.

But the following three lines are using the 3rd return parameter:

* https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/oracles/OracleManager.sol#L103
* https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/oracles/OracleManager.sol#L106
* https://github.com/sherlock-audit/2022-11-float-capital/blob/main/contracts/oracles/OracleManager.sol#L118

As a note. The mentioned variables have ``update`` string inside their name but they are from the ``start``-TimeStamp parameter.

## Impact
The call ``validateAndReturnMissedEpochInformation`` is used to move the protocol epochs forward. And this is the single most important part of the protocol . If there is something wrong in the worst case we need to migrate to a new implementation with ``deprecateMarketNoOracleUpdates``.
So it is better to follow documentation for the correct validations of oracle rounds.

## Code Snippet
One example of wrong validation:
In case we found the phaseId has increased we are validating the next possible round by watching the wrong return parameter:

```solidity
        while (previousOracleUpdateTimestamp == 0) {
          // NOTE: re-using this variable to keep gas costs low for this edge case.
          latestExecutedOracleRoundId = (((latestExecutedOracleRoundId >> 64) + 1) << 64) | uint64(oracleRoundIdsToExecute[i] - 1);

          (, , previousOracleUpdateTimestamp, , ) = chainlinkOracle.getRoundData(latestExecutedOracleRoundId);
        }
```

Also the check if we are in the correct timestamp window is using the wrong return parameter for validation:

```solidity
  if (
        previousOracleUpdateTimestamp >= relevantEpochStartTimestampWithMEWT ||
        currentOracleUpdateTimestamp < relevantEpochStartTimestampWithMEWT ||
        currentOracleUpdateTimestamp >= relevantEpochStartTimestampWithMEWT + EPOCH_LENGTH
      ) revert InvalidOracleExecutionRoundId({oracleRoundId: oracleRoundIdsToExecute[i]});
```

## Tool used
Manual Review

## Recommendation
From the ``getRoundData`` [documentation](https://docs.chain.link/docs/data-feeds/price-feeds/api-reference/#getrounddata):

In all validations we should use ``updatedAt`` return parameter instead of ``startedAt`` one.