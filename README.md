# Solidity Audit Checklist

## Optimizations

1. Minimize storage access
2. Prefer using `calldata` instead of `memory`, avoid copying and enjoy slicing:
    - Non-32-bytes fields of `Data calldata data` access introduces ~175 gas costs each, prefer using `uint256`/`bytes32` and downcast.
3. Minimize calldata size (very important for L2):
    - Multiple arrays and nested arrays can be concatenated and sliced by indexes (64 bytes economy per array)
    - Single byte array could be just concatenated to the end of calldata to avoid having offset and length
4. Extenral calls on assembly could save thousands of gas
5. Consider utilizing ["Bit Twidding Hacks"](./solidity-bit-twidding-hacks.md) ideas for 256-bit integers

## Solidity Pitfalls

1. Remember `address.transfer(*)` passes 2300 gas and reverts in case of failure
2. Remember `address.send(*)` passes 2300 gas and returns `false` in case of failure
3. Never use `tx.origin` for authentication, non-legitimate action could be executed even inside spam token transfer.
4. External call may revert in the following cases (sometimes even in `try-catch` block):
    - Not enough smart contract `balance` for passing `value` into the call
    - Not enough `returndatasize()` to abi.decode return types (https://github.com/ethereum/solidity/issues/4116)
    - Not existing or destructed contract when function returns `void` due to `extcodesize` pre-call check. (https://github.com/ethereum/solidity/issues/11373, https://github.com/ethereum/solidity/issues/12725)
5. Arrays pitfalls:
    - Stores length separately and check boundaries with `sload` on each item access
    - Erases items on length shrinking or `pop()`, can be avoided via Yul (Solidity Assembly)

## Yul Pitfalls (Solidity Assembly)

1. Never do assembly `return(*, *)` inside `public` or `internal` methods, due it cause whole external call exit, similar to `revert(*, *)` but with success.
2. Never assume memory at `mload(0x40)` is zeroed, because it could be soiled by previous code.
