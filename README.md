# Solidity Audit Checklist

## Optimizations

1. Minimize storage access
2. Prefer using `calldata` instead of `memory` and avoid copying
3. Minimize calldata size (very important for L2)
4. Extenral calls on assembly could save thousands of gas
5. Consider utilizing ["Bit Twidding Hacks"](./solidity-bit-twidding-hacks.md) ideas for 256-bit integers

## Solidity Pitfalls

1. Remember `address.transfer(*)` passes 2300 gas and reverts in case of failure
2. Remember `address.send(*)` passes 2300 gas and returns `false` in case of failure
3. External call may revert in the following cases (sometimes even in `try-catch` block):
    - Not enough smart contract `balance` for passing `value` into the call
    - Not enough `returndatasize()` to abi.decode return types (https://github.com/ethereum/solidity/issues/4116)
    - Not existing or destructed contract when function returns `void` due to `extcodesize` pre-call check. (https://github.com/ethereum/solidity/issues/11373, https://github.com/ethereum/solidity/issues/12725)
4. Arrays pitfalls:
    - Stores length separately and check boundaries with `sload` on each item access
    - Erases items on length shrinking or `pop()`, can be avoided via Yul (Solidity Assembly)

## Yul (Solidity Assembly)

1. Never do assembly `return(*, *)` inside `public` or `internal` methods, due it cause whole external call exit, similar to `revert(*, *)` but with success.
2. Never assume memory at `mload(0x40)` is zeroed, because it could be soiled by previous code.
