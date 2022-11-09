neila

medium

# Use safe version ERC20

## Summary
Use safe version ERC20
Found by: @Tomosuke0930

## Vulnerability Detail
The `shiftOrder` function performs an ERC20 transfer `IPoolToken(token).transferFrom(msg.sender, address(this), _amountOfPoolToken)`; but does not check the return value, nor does it work with all legacy tokens.

Some tokens (like USDT) don't correctly implement the EIP20 standard and their `transfer/transferFrom` function return `void` instead of a success boolean. Calling these functions with the correct EIP20 function signatures will always revert.

The `ERC20.transfer()` and `ERC20.transferFrom()` functions return a boolean value indicating success. This parameter needs to be checked for success. Some tokens do not revert if the transfer failed but return `false` instead.

## Impact
Tokens that don't actually perform the transfer and return false are still counted as a correct transfer and tokens that don't correctly implement the latest EIP20 spec, like USDT, will be unusable in the protocol as they revert the transaction because of the missing return value.

## Code Snippet
https://github.com/unchain-dev/2022-11-float-capital-UNCHAIN/blob/e167ac24613e02748aad72c1d216283225733355/contracts/shifting/Shifting.sol#L134
```solidity
function shiftOrder(
    uint112 _amountOfPoolToken,
    address _marketFrom,
    IMarketCommon.PoolType _poolTypeFrom,
    uint8 _poolTierFrom,
    address _marketTo,
    IMarketCommon.PoolType _poolTypeTo,
    uint8 _poolTierTo
  ) external {
		/* ~~~ */
    //slither-disable-next-line unchecked-transfer
    IPoolToken(token).transferFrom(msg.sender, address(this), _amountOfPoolToken)
```

## Tool used

Manual Review

## Recommendation
I recommend using OpenZeppelin’s `SafeERC20Upgradable` versions with the `safeTransfer`
 and `safeTransferFrom` functions that handle the return value check as well as non-standard-compliant tokens.
https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/token/ERC20/utils/SafeERC20Upgradeable.sol
