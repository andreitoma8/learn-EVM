# Notes on learing about the Ethereum Virtual Machine (EVM)

# High level overview

## What is a Virtual Machine?

A virtual machine is a software program that simulates a computer, allowing you to run multiple operating systems on a single physical device.

Virtual Machines pretend to be a Physical Machine(coputer) to the benefit of taking code that was written for one machine and running it on another.

## What is the Ethereum Network?

The Ethereum Network is composed of multiple Nodes that each run Clients. Each client has an instance of the Ethereum Virtual Machine (EVM) that runs the code of the smart contracts. The EVM is a stack-based virtual machine that executes bytecode. The bytecode is generated from the Solidity/Vyper/Yul/Huff source code.

The EVM is a stack-based virtual machine that executes bytecode. The bytecode is generated from the Solidity/Vyper/Yul/Huff source code.

# Ethereum Transactions

### Transaction fiels

-   `Nonce`: The number of transactions sent by the sender. Starts at 0.
-   `Gas Price`: The price of gas for this transaction in wei.
-   `Gas Limit`: The maximum amount of gas that should be used in this transaction.
    -   For value transfers, this is set to 21,000(the fixed cost of a ETH transfer).
-   `To`: The 20-byte address of the message call's recipient.
    -   For a contract creation transaction, this must be left empty.
-   `Value`: The value in wei to be transferred to the message call's recipient or in the case of a contract creation, as an endowment to the newly created account.
-   `Data`: The data sent with the transfer.
    -   In case of `Value transfers`, this is left empty.
    -   For `Smart Contract deployments`, this contains the compiled code of a contract.
    -   And for `Smart Contract calls`, it contains the hash of the invoked method signature and encoded parameters(by Solidity standards).
    -   `v`, `r`, `s`: The components of the transaction signature.

# Data locations in the EVM

Stack, Memory, Storage, Code, Call Data & Logs

## Stack

The stack is a temporary storage data location in the EVM. It is a 256-bit word array.

EVM Opcodes pop information from the stack and push information to the stack.

When pushing to the stack, the stack grows from the bottom to the top. When popping from the stack, the stack grows from the top to the bottom.

## CallData

CallData is the data field of a transaction. It is the data passed to a smart contract when it is called.

## Memory

Memory is a linear memory space that is accessible for the duration of a transaction. It is a 256-bit word array.

## Storage

Storage is a persistent memory space that is accessible for the duration of a contract. It is a 256-bit word array.

## Code

Code is the code of a smart contract, but can also be used for data storage(constants in smart contracts, which are stored in it's bitecode).

## Logs

Write-only logger / event output.

# EVM Opcodes

All the Ethereum Opcodes are listed in the [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) in Appendix G.

OpCodes are 8-bit values that represent operations that can be performed on the EVM. They each have a specific gas cost and can be used to perform a specific operation.

## STOP

-   Hex: `0x00`
-   Gas used: `0`
-   Description: Halts execution.

## ADD

-   Hex: `0x01`
-   Gas used: `3`
-   Description: Adds the top-most two values on the stack and pushes the result to the stack.

```
Stack representation:
Before          After

top    = a      top    = a + b
...    = b      bottom = c
bottom = c
```

## MUL

-   Hex: `0x02`
-   Gas used: `5`
-   Description: Multiplies the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a * b
...    = b      bottom = c
bottom = c
```

## SUB

-   Hex: `0x03`
-   Gas used: `3`
-   Description: Subtracts the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a - b
...    = b      bottom = c
bottom = c
```

## DIV

-   Hex: `0x04`
-   Gas used: `5`
-   Description: Divides the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a / b
...    = b      bottom = c
bottom = c
```

## SDIV

-   Hex: `0x05`
-   Gas used: `5`
-   Description: Signed integer division of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a / b
...    = b      bottom = c
bottom = c
```

## MOD

-   Hex: `0x06`
-   Gas used: `5`
-   Description: Modulo of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a % b
...    = b      bottom = c
bottom = c
```

## SMOD

-   Hex: `0x07`
-   Gas used: `5`
-   Description: Signed integer modulo of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a % b
...    = b      bottom = c
bottom = c
```

## ADDMOD

-   Hex: `0x08`
-   Gas used: `8`
-   Description: Modulo of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = (a + b) % c
...    = b      bottom = d
...    = c
bottom = d
```

## MULMOD

-   Hex: `0x09`
-   Gas used: `8`
-   Description: Modulo of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = (a * b) % c
...    = b      bottom = d
...    = c
bottom = d
```

## EXP

-   Hex: `0x0a`
-   Gas used: `10 + 50 * byte_len_exponent` (exponent is b in the reference implementation)
-   Description: Exponentiation of the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a ** b
...    = b      bottom = c
bottom = c
```

## SIGNEXTEND

-   Hex: `0x0b`
-   Gas used: `5`
-   Description: Extend the length of the signed integer in the top stack item to the given number of bytes.

```
Before          After

top    = a      top    = signextend(b, a)
...    = b      bottom = c
bottom = c
```

## LT

-   Hex: `0x10`
-   Gas used: `3`
-   Description: Returns 1 if the top-most value is lower than the second top-most value, otherwise 0.

```
Before          After

top    = a      top    = a < b
...    = b      bottom = c
bottom = c
```

## GT

-   Hex: `0x11`
-   Gas used: `3`
-   Description: Returns 1 if the top-most value is greater than the second top-most value, otherwise 0.

```
Before          After

top    = a      top    = a > b
...    = b      bottom = c
bottom = c
```

## SLT

-   Hex: `0x12`
-   Gas used: `3`
-   Description: Same as LT, but treats the values as signed integers.

```
Before          After

top    = a      top    = a < b
...    = b      bottom = c
bottom = c
```

## SGT

-   Hex: `0x13`
-   Gas used: `3`
-   Description: Same as GT, but treats the values as signed integers.

```
Before          After

top    = a      top    = a > b
...    = b      bottom = c
bottom = c
```

## EQ

-   Hex: `0x14`
-   Gas used: `3`
-   Description: Returns 1 if the top-most value is equal to the second top-most value, otherwise 0.

```
Before          After

top    = a      top    = a == b

...    = b      bottom = c
bottom = c
```

## ISZERO

-   Hex: `0x15`
-   Gas used: `3`
-   Description: Returns 1 if the top-most value is zero, otherwise 0.

```
Before          After

top    = a      top    = a == 0
bottom = b
```

## AND

-   Hex: `0x16`
-   Gas used: `3`
-   Description: Bitwise AND operation on the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a & b
...    = b      bottom = c
bottom = c
```

## OR

-   Hex: `0x17`
-   Gas used: `3`
-   Description: Bitwise OR operation on the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a | b
...    = b      bottom = c
bottom = c
```

## XOR

-   Hex: `0x18`
-   Gas used: `3`
-   Description: Bitwise XOR operation on the top-most two values on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = a ^ b
...    = b      bottom = c
bottom = c
```

## NOT

-   Hex: `0x19`
-   Gas used: `3`
-   Description: Bitwise NOT operation on the top-most value on the stack and pushes the result to the stack.

```
Before          After

top    = a      top    = ~a
bottom = b
```

## BYTE

-   Hex: `0x1a`
-   Gas used: `3`
-   Description: Returns the byte at position `i` in the second top-most value on the stack(`i` is the top-most value on the stack).

```
Before          After

top    = a      top    = (b >> (248 - a * 8)) && 0xFF
...    = b      bottom = c
bottom = c
```

## SHL

-   Hex: `0x1b`
-   Gas used: `3`
-   Description: Shifts the second top-most value on the stack left by the number of bits given by the top-most value on the stack.

```
Before          After

top    = a      top    = b << a
...    = b      bottom = c
bottom = c
```

## SHR

-   Hex: `0x1c`
-   Gas used: `3`
-   Description: Shifts the second top-most value on the stack right by the number of bits given by the top-most value on the stack.

```
Before          After

top    = a      top    = b >> a
...    = b      bottom = c
bottom = c
```

## SAR

-   Hex: `0x1d`
-   Gas used: `3`
-   Description: Arithmetic shifts the second top-most value on the stack right by the number of bits given by the top-most value on the stack, preserving the sign.

```
Before          After

top    = a      top    = b >> a
...    = b      bottom = c
bottom = c
```

## SHA3

-   Hex: `0x20`
-   Gas used: `30 + 6 * byte_len_data`
-   Description: Pushes the Keccak-256 hash of the data in memory starting at the top-most value on the stack and with the length given by the second top-most value on the stack.

```
Before          After

top    = a      top    = keccak256(memory[a:a+b])
...    = b      bottom = c
bottom = c
```

## ADDRESS

-   Hex: `0x30`
-   Gas used: `2`
-   Description: Pushes the address of the current contract to the stack.

```
Before          After

top    = a      top    = address(this)
bottom = b      ...    = a
                bottom = b
```

## BALANCE

-   Hex: `0x31`
-   Gas used: `100/2600` See more [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a5-balance-extcodesize-extcodehash).
-   Description: Pushes the balance of the given address to the stack.

```
Before          After

top    = a      top    = balance(a)
bottom = b      bottom = b
```

## ORIGIN

-   Hex: `0x32`
-   Gas used: `2`
-   Description: Pushes the address of the original transaction sender to the stack.

```
Before          After

top    = a      top    = tx.origin
bottom = b      ...    = a
                bottom = b
```

## CALLER

-   Hex: `0x33`
-   Gas used: `2`
-   Description: Pushes the address of the caller of the current contract to the stack.

```
Before          After

top    = a      top    = msg.sender
bottom = b      ...    = a
                bottom = b
```

## CALLVALUE

-   Hex: `0x34`
-   Gas used: `2`
-   Description: Pushes the value sent with the current call to the stack.

```
Before          After

top    = a      top    = msg.value
bottom = b      ...    = a
                bottom = b
```

## CALLDATALOAD

-   Hex: `0x35`
-   Gas used: `3`
-   Description: Pushes the data in the call data starting at the top-most value on the stack and with the length of 32 bytes to the stack.

```
Before          After

top    = a      top    = calldata[a:a+32]
bottom = b      bottom = b
```

## CALLDATASIZE

-   Hex: `0x36`
-   Gas used: `2`
-   Description: Pushes the size of the call data to the stack.

```
Before          After

top    = a      top    = len(msg.data)
bottom = b      ...    = a
                bottom = b
```

## CALLDATACOPY

-   Hex: `0x37`
-   Gas used: `3 + 3 * data_size_words(len in bytes) + mem_expansion_cost` (mem_expansion_cost - more [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a0-1-memory-expansion))
-   Description: Copies the data in the call data to memory.

```
Memory: [a:a+c] = msg.data[b:b+c]

Before          After

top    = a      top    = d
...    = b
...    = c
bottom = d
```

## CODESIZE

-   Hex: `0x38`
-   Gas used: `2`
-   Description: Pushes the size of the code of the current contract to the stack.

```
Before          After

top    = a      top    = len(this.code)
bottom = b      ...    = a
                bottom = b
```

## CODECOPY

-   Hex: `0x39`
-   Gas used: `3 + 3 * data_size_words(len in bytes) + mem_expansion_cost`
-   Description: Copies the code of the current contract to memory.

```
Memory: [a:a+c] = this.code[b:b+c]

Before          After

top    = a      top    = d
...    = b
...    = c
bottom = d
```

## GASPRICE

-   Hex: `0x3a`
-   Gas used: `2`
-   Description: Pushes the gas price of the current transaction to the stack.

```
Before          After

top    = a      top    = tx.gasprice
bottom = b      ...    = a
                bottom = b
```

## EXTCODESIZE

-   Hex: `0x3b`
-   Gas used: `100/2600`
-   Description: Pushes the size of the code of the given address to the stack.

```
Before          After

top    = a      top    = len(code(a))
bottom = b      bottom = b
```

## EXTCODECOPY

-   Hex: `0x3c`
-   Gas used: `3 + 3 * data_size_words(len in bytes) + mem_expansion_cost`
-   Description: Copies the code of a given address to memory.

```
Memory: [b:b+d] = code(a)[c:c+d]

Before          After

top    = a      top    = e
...    = b
...    = c
...    = d
bottom = e
```

## RETURNDATASIZE

-   Hex: `0x3d`
-   Gas used: `2`
-   Description: Pushes the size of the return data of the current call to the stack.

```
Before          After

top    = a      top    = len(returndata)
bottom = b      ...    = a
                bottom = b
```

## RETURNDATACOPY

-   Hex: `0x3e`
-   Gas used: `3 + 3 * data_size_words(len in bytes) + mem_expansion_cost`
-   Description: Copies the return data of the current call to memory.

```
Memory: [a:a+c] = returndata[b:b+c]

Before          After

top    = a      top    = d
...    = b
...    = c
bottom = d
```

## EXTCODEHASH

-   Hex: `0x3f`
-   Gas used: `100/2600`
-   Description: Pushes the hash of the code of the given address to the stack.

```
Before          After

top    = a      top    = address ? keccak256(code(a)) : 0
bottom = b      bottom = b
```

## BLOCKHASH

-   Hex: `0x40`
-   Gas used: `20`
-   Description: Pushes the hash of the block with the given number to the stack.

```
Before          After

top    = a      top    = blockhash(a)
bottom = b      bottom = b
```

## COINBASE

-   Hex: `0x41`
-   Gas used: `2`
-   Description: Pushes the address of the current block's beneficiary to the stack.

```
Before          After

top    = a      top    = block.coinbase
bottom = b      ...    = a
                bottom = b
```

## TIMESTAMP

-   Hex: `0x42`
-   Gas used: `2`
-   Description: Pushes the timestamp of the current block to the stack.

```
Before          After

top    = a      top    = block.timestamp
bottom = b      ...    = a
                bottom = b
```

## NUMBER

-   Hex: `0x43`
-   Gas used: `2`
-   Description: Pushes the number of the current block to the stack.

```
Before          After

top    = a      top    = block.number
bottom = b      ...    = a
                bottom = b
```

## DIFFICULTY

-   Hex: `0x44`
-   Gas used: `2`
-   Description: Pushes the difficulty of the current block to the stack.

```
Before          After

top    = a      top    = block.difficulty
bottom = b      ...    = a
                bottom = b
```

## GASLIMIT

-   Hex: `0x45`
-   Gas used: `2`
-   Description: Pushes the gas limit of the current block to the stack.

```
Before          After

top    = a      top    = block.gaslimit
bottom = b      ...    = a
                bottom = b
```

## CHAINID

-   Hex: `0x46`
-   Gas used: `2`
-   Description: Pushes the chain ID of the current chain to the stack.

```
Before          After

top    = a      top    = chainid
bottom = b      ...    = a
                bottom = b
```

## SELFBALANCE

-   Hex: `0x47`
-   Gas used: `5`
-   Description: Pushes the balance of the current contract to the stack.

```
Before          After

top    = a      top    = address(this).balance
bottom = b      ...    = a
                bottom = b
```

## POP

-   Hex: `0x50`
-   Gas used: `2`
-   Description: Removes the top element from the stack.

```
Before          After

top    = a      top    = b
...    = b      bottom = c
bottom = c
```

## MLOAD

-   Hex: `0x51`
-   Gas used: `3`
-   Description: Pushes 32 bytes from the memory starting at the given position to the stack.

```
Before          After

top    = a      top    = mem[a:a+32]
bottom = b      bottom = b
```

## MSTORE

-   Hex: `0x52`
-   Gas used: `3 + mem_expansion_cost`
-   Description: Stores 32 bytes from the stack to the memory starting at the given position.

```
Memory: [a:a+32] = b

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## MSTORE8

-   Hex: `0x53`
-   Gas used: `3 + mem_expansion_cost`
-   Description: Stores 1 byte from the stack to the memory starting at the given position.

```
Memory: [a:a+1] = b

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## SLOAD

-   Hex: `0x54`
-   Gas used: `100 or 2100` see more details [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a0-1-memory-expansion)
-   Description: Pushes the value of the storage slot with the given index to the stack.

```
Before          After

top    = a      top    = storage[a]
bottom = b      bottom = b
```

## SSTORE

-   Hex: `0x55`
-   Gas used: See more details [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a0-1-memory-expansion)
-   Description: Stores the value from the stack to the storage slot with the given index.

```
Storage: storage[a] = b

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## JUMP

-   Hex: `0x56`
-   Gas used: `8`
-   Description: Performs an unconditional jump to a new location in the byte code. (The destination has to be a valid jump destination.)

```
Jump at a'th byte in the byte code

Before          After

top    = a      top    = b
...    = b      bottom = c
bottom = c
```

## JUMPI

-   Hex: `0x57`
-   Gas used: `10`
-   Description: Performs a jump to a new location in the byte code if the second top-most element of the stack is not zero. (The destination has to be a valid jump destination.)

```
Jump at a'th byte in the byte code if b != 0

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## PC

-   Hex: `0x58`
-   Gas used: `2`
-   Description: Pushes the current position in the byte code to the stack.

```
Before          After

top    = a      top    = pc
bottom = b      ...    = a
                bottom = b
```

## MSIZE

-   Hex: `0x59`
-   Gas used: `2`
-   Description: Pushes the current size of the memory to the stack.

```
Before          After

top    = a      top    = len(memory)
bottom = b      ...    = a
                bottom = b
```

## GAS

-   Hex: `0x5a`
-   Gas used: `2`
-   Description: Pushes the amount of gas left to the stack.

```
Before          After

top    = a      top    = gas
bottom = b      ...    = a
                bottom = b
```

## JUMPDEST

-   Hex: `0x5b`
-   Gas used: `1`
-   Description: Marks a valid destination for jumps.

## PUSH1 to PUSH32

-   Hex: `0x60` - `0x7f`
-   Gas used: `3`
-   Description: Pushes a given number of bytes from the byte code to the stack(1 to 32 bytes).

```
Bytecode: 60 08  // PUSH1 0x08

Before          After

top    = a      top    = 0x08
bottom = b      ...    = a
                bottom = b
```

## DUP1 to DUP16

-   Hex: `0x80` - `0x8f`
-   Gas used: `3`
-   Description: Duplicates the 1-16th element of the stack and pushes it to the stack.

```
Before          After

top    = a      top    = a
bottom = b      ...    = a
                bottom = b
```

## SWAP1 to SWAP16

-   Hex: `0x90` - `0x9f`
-   Gas used: `3`
-   Description: Swaps the top element on the stack with the 2-17th element of the stack with the top element of the stack.

```
Before          After

top    = a      top    = b
...    = b      ...    = a
bottom = c      bottom = c
```

## LOG0

-   Hex: `0xa0`
-   Gas used: `375 + 8 * data_size + mem_expansion_cost`
-   Description: Creates a log entry with no topics with data from the memory starting at the given position and with the given length.

```
Log: Memory[a:a+b]

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## LOG1 to LOG4

-   Hex: `0xa1` - `0xa4`
-   Gas used: `375 + 375 * num_topics + 8 * data_size + mem_expansion_cost`
-   Description: Creates a log entry with n topics with data from the memory starting at the given position and with the given length.

```
Log: Memory[a:a+b] with topics c

Before          After

top    = a      top    = d
...    = b
...    = c
bottom = d
```

## CREATE

-   Hex: `0xf0`
-   Gas used: `32000 + 700 * mem_expansion_cost` + `code_deposit_cost` (More on code deposit cost [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a9-create-operations)
-   Description: Creates a new contract with the given code and sends the given amount of ether to it and pushes the address of the new contract to the stack.

```
Create contract with code Memory[a:a+b] and send b ether

Before          After

top    = a      top    = address(new_contract)
...    = b      bottom = c
bottom = c
```

## CALL

-   Hex: `0xf1`
-   Gas used: [Call opperations costs](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#aa-call-operations)
-   Description: Calls the given address with the given amount of ether and the given input data and pushes the result to the stack.

```
call(gas: a, to: b, value: c, inputData: memory[d:d+e], outputData: memory[f:f+g])

Before          After

top    = a      top    = result(0/1)
...    = b      bottom    = h
...    = c
...    = d
...    = e
...    = f
...    = g
bottom = h
```

## CALLCODE

-   Hex: `0xf2`
-   Gas used: [Call opperations costs](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#aa-call-operations)
-   Description: same as DELEGATECALL, but does not propagate original msg.sender and msg.value.

```
call(gas: a, to: b, value: c, inputData: memory[d:d+e], outputData: memory[f:f+g])

Before          After

top    = a      top    = result(0/1)
...    = b      bottom    = h
...    = c
...    = d
...    = e
...    = f
...    = g
bottom = h
```

## RETURN

-   Hex: `0xf3`
-   Gas used: `0` or `mem_expansion_cost` if memory is expanded
-   Description: Stops execution and returns data from the memory starting at the given position and with the given length.

```
Return Memory[a:a+b]

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## DELEGATECALL

-   Hex: `0xf4`
-   Gas used: [Call opperations costs](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#aa-call-operations)
-   Description: The code of the called contract is executed on the current contract's context(storage, msg.sedner and msg.value).

```
call(gas: a, to: b, inputData: memory[c:c+d], outputData: memory[e:e+f])

Before          After

top    = a      top    = result(0/1)
...    = b      bottom    = g
...    = c
...    = d
...    = e
...    = f
bottom = g
```

## CREATE2

-   Hex: `0xf5`
-   Gas used: `32000 + 700 * mem_expansion_cost`+ `code_deposit_cost` (More on code deposit cost [here](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a9-create-operations))
-   Description: Creates a new contract with the given code and sends the given amount of ether to it and pushes the address of the new contract to the stack.

```
create(value: a, memStart: b, memLength: c, salt: d)

Before          After

top    = a      top    = address(new_contract)
...    = b      bottom = e
...    = c
...    = d
bottom = e
```

## STATICCALL

-   Hex: `0xfa`
-   Gas used: [Call opperations costs](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#aa-call-operations)
-   Description: Calls the given address like call, but guarantees that the called contract cannot modify the state.

```
call(gas: a, to: b, inputData: memory[c:c+d], outputData: memory[e:e+f])

Before          After

top    = a      top    = result(0/1)
...    = b      bottom    = g
...    = c
...    = d
...    = e
...    = f
bottom = g
```

## REVERT

-   Hex: `0xfd`
-   Gas used: `0` or `mem_expansion_cost` if memory is expanded
-   Description: Stops execution and reverts state changes, but unlike RETURN it also returns data from the memory starting at the given position and with the given length.

```
Revert memory[a:a+b]

Before          After

top    = a      top    = c
...    = b
bottom = c
```

## SELFDESTRUCT

-   Hex: `0xff`
-   Gas used: `5000` + `ETH transfer cost`
-   Description: Stops execution, destroys the current contract and sends its funds to the given address.

```
Selfdestruct to address a

Before          After

top    = a      top    = b
bottom = b
```

## Invalid opcodes

The invalid opcodes are: `0x0c - 0x0f`, `0x1e-0x1f`, `0x49-0x4f`, `0x5c-0x5f`, `0xa5-0xef`, `0xf6-0xf9`, `0xfb-0xfc` and `0xfe`.

On execution of any invalid operation, whether the designated INVALID opcode or simply an undefined opcode, all remaining gas is consumed and the state is reverted to the point immediately prior to the beginning of the current execution context.
