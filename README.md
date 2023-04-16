# Notes on learing about the Ethereum Virtual Machine (EVM)

# Summary

-   [High level overview](#high-level-overview)
-   [Ethereum Transactions](#ethereum-transactions)
-   [Data in the EVM](#data-in-the-evm)
-   [EVM Opcodes](#evm-opcodes)
-   [Creating a Contract with Bytecode and Deploying it to the Blockchain](#creating-a-contract-with-bytecode-and-deploying-it-to-the-blockchain)

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

# Data in the EVM

Stack, Memory, Storage, Code, Call Data & Logs

## Stack

The stack is a temporary storage data location in the EVM. It is a 32 bytes elements array and has a maximum lenght of 1024. For each smart contract call, one stack is created and used to store temporary data that is used during the execution of the smart contract.

EVM Opcodes pop information from the stack and push information to the stack. When pushing to the stack, the stack grows from the bottom to the top. When popping from the stack, the stack grows from the top to the bottom.

## Memory

Memory is a linear memory space that is accessible for the duration of a transaction. It is a 256-bit word array that is initialized empty and can grow as needed.

## Storage

Storage is a persistent memory space that is accessible for the duration of a contract. It is a map of 32 bytes slot to 32 bytes values.

## CallData

CallData is the data field of a transaction. It is the data passed to a smart contract when it is called and it is immutable.

## ReturnData

ReturnData is the data returned by a smart contract when it is called. It is immutable.

## Code

Code is the code of a smart contract, but can also be used for data storage(constants in Solidity in smart contracts, which are stored in it's bitecode).

## Logs

Write-only logger / event output.

# Smart Contracts

## What is a Smart Contract?

A smart contract is a set of instructions. Each instruction is an opcode (with their own handy mnemonic for reference, text representations of their assigned values between 0 and 255). When the EVM executes a smart contract, it reads and executes each instruction sequentially(except for JUMP and JUMPI instructions, which jump to a specific instruction). If an instruction cannot be executed, for instance, if there are not enough values on the stack, or insufficient gas, the execution reverts. In the event of a reverted transaction, any state changes dictated by the transaction instructions are returned to their state before the transaction.

# EVM Opcodes

All the Ethereum Opcodes are listed in the [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) in Appendix G.

OpCodes are 8-bit values that represent operations that can be performed on the EVM. They each have a specific gas cost and can be used to perform a specific operation.

**[See a complete list of EVM Opcodes here](https://github.com/andreitoma8/learn-EVM/blob/master/EVM-Opcodes.md)**, where I explain each one of them, with examples of how they interact with the stack, memory and storage.

# Creating a Contract with Bytecode and Deploying it to the Blockchain

We will go through the process of creating a simple contract that returns my lucky number: 8 and then we will deploy it to the blockchain using Solidity.

Source: [OpenZeppelin - Deconstructing a Solidity Contract](https://blog.openzeppelin.com/deconstructing-a-solidity-contract-part-i-introduction-832efd2d7737/)

## Creating a Contract with Bytecode

In the EVM, the contract creation is made by creating a transaction that has the `to` field empty and the `data` field containing the bytecode of the contract.

When you are doing this in Solidity, the compiler will take the code you wrote and convert it to bytecode.

It's very important to note that the deloyment of a contract has two parts: `Creation` and `Runtime`.

-   `Creation`

    In the first part of the deployment transaction the EVM start executing the bytecode and the executions stops at the first `RETURN`, where the `Runtime bytecode` of the contract needs to be returned. In Solidity this is the part where the `constructor` function is executed.

-   `Runtime`

    The runtime bytecode is the actual code of the contract, the one that is executed when a call is made to it's address.

## Writing the Bytecode

### Runtime Bytecode

First, we will create the runtime bytecode of the contract, which will basically just return the lucky number `8`.

For this we will first need to add the number `8` to memory and then return it.

#### Adding the number `8` to memory

Adding the number `8` to memory we will use `MSTORE(p,v)` opcode, which takes two arguments: the `p` position in memory where we want to store the value and `v` the value itself. The position we will add the value in memory is `0x00` and the value is `0x08`.

So we need our stack to look like:

```
top    = 0x00
bottom = 0x08
```

To do this we will use the opcodes `PUSH1` which will push the value to the stack, and we will use the `MSTORE` opcode to store the value in memory.

```
60 08 60 00 52 // PUSH1 0x08 PUSH1 0x00 MSTORE
```

After the `MSTORE` opcode is executed the stack will be empty and we will have the value `8` stored in memory at position `0x00`.

#### Returning the value from memory

Now we need to return the value from memory, so we will use the `RETURN(p,s)` opcode, which takes two arguments: the `p` position in memory where the value is stored and the `s` the size of the value.

We know that the value is stored at position `0x00` and it's size is `0x20` (32 bytes), which is the default size of a memory word.

To return the value, we need our stack to look like this:

```
top    = 0x00
bottom = 0x20
```

So we will use the `PUSH1` opcode to push the values to the stack, and we will end the bytecode with the `RETURN` opcode.

```
60 20 60 00 f3 // PUSH1 0x20 PUSH1 0x00 RETURN
```

So now we have the runtime bytecode of the contract, which is:

```
60 08 60 00 52 60 20 60 00 f3 // PUSH1 0x08 PUSH1 0x00 MSTORE PUSH1 0x20 PUSH1 0x00 RETURN
```

### Creation Bytecode

Now we need to create the creation bytecode of the contract, which has to return the runtime bytecode of the contract. Similar to the above, we need to first store the runtime bytecode in memory and then return it.

If you want to follow along while reading this, you can use [EVM Codes' Playground](https://www.evm.codes/playground) where you can write the bytecode and see how it interacts with the stack and memory.

#### Adding the runtime bytecode to memory

Adding the runtime bytecode to memory we will use the `CODECOPY(p, o, s)` opcode, which takes three arguments: the `p` position in memory where we want to store the runtime bytecode, the `o` offset in the code where the runtime bytecode starts and the `s` the size of the of the bytes array that we want to copy. The position we will add the value in memory is `0x00`, the offset is still to be determined (we need to finish the creation bytecode first, so we will use a `__` placeholder) and the size is `0x0a` (10 bytes).

We need our stack to look like:

```
top    = 0x00
...    = 0x__
bottom = 0x0a
```

So we will use the `PUSH1` opcode to push the values to the stack, and add the `CODECOPY` opcode at the end of the bytecode to execute it.

```
60 0a 60 __ 60 00 39 // PUSH1 0x0a PUSH1 __ PUSH1 0x00 CODECOPY
```

#### Returning the runtime bytecode from memory

Now we need to return the runtime bytecode from memory, so we will use the `RETURN` opcode, same as explained above.

We know that the value is stored at position `0x00` and it's size is `0x0a` (10 bytes), which is the size of the runtime bytecode.

To return the value, we need our stack to look like this:

```
top    = 0x00
bottom = 0x0a
```

So we will use the `PUSH1` opcode to push the values to the stack, and we will end the bytecode with the `RETURN` opcode.

```
60 0a 60 00 f3 // PUSH1 0x0a PUSH1 0x00 RETURN
```

So now we have the creation bytecode of the contract, which is:

```
60 0a 60 __ 60 00 39 60 0a 60 00 f3 // PUSH1 0x0a PUSH1 __ PUSH1 0x00 CODECOPY PUSH1 0x0a PUSH1 0x00 RETURN
```

### Bytes data for the creation transaction

If we combine the two parts that we created above, we get:

```
 60    0a   60   __   60   00     39     60    0a   60    00    f3    60    08   60    00    52    60    20    60   00    f3
PUSH1 0x0a PUSH1 __ PUSH1 0x00 CODECOPY PUSH1 0x0a PUSH1 0x00 RETURN PUSH1 0x08 PUSH1 0x00 MSTORE PUSH1 0x20 PUSH1 0x00 RETURN
```

The only thing left to do is to replace the `__` placeholder with the offset of the runtime bytecode, which is `0x0c` (12 bytes) from the start of the bytecode.

So the final bytecode is:

```

60    0a   60    0c   60   00     39     60    0a   60    00    f3    60    08   60    00    52    60    20    60   00    f3

```

## But could we optimize this bytecode?

Yes, we can introduce `DUP` to the mix and avoid declarin a extra 3 bytes that `PUSH1` need(it's params).

For the Runtime Bytecode, we PUSH `0x00` two times, so we can use `DUP` to duplicate the value on the top of the stack. But to do this, we need to change the order of adding the values to the stack, so they are not used and basically deleted before we get the chance to duplicate them. We will first build our stack and then use the same `MSTORE` and `RETURN` opcodes, but both at the end. So our stack will need to look like this:

```
top    = 0x00
...    = 0x08
...    = 0x00
bottom = 0x20
```

But instead of using `PUSH1` to add the second `0x00` to the stack, we will use `DUP2` to duplicate the second most recent value on the stack, which is `0x00`. So our new bytecode for the for the runtime bytecode is:

```
60 08 60 00 52 60 20 60 81 f3 // PUSH1 0x08 PUSH1 0x00 MSTORE PUSH1 0x20 PUSH1 DUP2 RETURN
```

For the creation bytecode, we will use the same approach, first adding all the values to the stack and then using `CODECOPY` and `RETURN` at the end, all while using `DUP` to avoid using `PUSH1`. Our stack will need to look like this before performing the operations:

```
top    = 0x00
...    = 0xOfeset
...    = 0x09(runtime bytecode size)
...    = 0x00
bottom = 0x09(runtime bytecode size)
```

As you can see, `0x00` and `0x09` repeat, so we can use `DUP2` and `DUP3` to duplicate them. So our new bytecode for the creation bytecode is:

```
60 09 60 00 81 60 0a 82 39 f3 // PUSH1 0x09 PUSH1 0x00 DUP2 PUSH1 0x0a DUP3 CODECOPY RETURN
```

And the final optimized bytecode is:

```
 60    09   60    00   81   60    0a   82     39      f3    60    08   60    00    52    60    20   60    81    f3
PUSH1 0x09 PUSH1 0x00 DUP2 PUSH1 0x0a DUP3 CODECOPY RETURN PUSH1 0x08 PUSH1 0x00 MSTORE PUSH1 0x20 PUSH1 DUP2 RETURN
```

## Deploying the contract

Now that we have the bytecode of the contract, we can deploy it. You can take the following snippet and run it in Remix to deploy the contract and check the return value of `callDeployedContract()`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BytecodeDeployer {
    address public deployedContract;

    // First we need to deploy the contract
    function deploy() external {
        // Store the bytecode in memory
        bytes memory bytecode = hex"6009600081600a8239f36020600060088152f3";

        // Create a local variable to store the address of the deployed contract
        address addr;

        // Deploy the contract using the CREATE Yul function which takes three arguments:
        // wei value, location of the data in memory and the size of the data (data to be sent as msg.data)
        // We add 0x20 to the location of the data in memory because the first 32 bytes are used to store
        // the size of the data for arrays (bites array in this case)
        assembly {
            addr := create(0, add(bytecode, 0x20), 0x16)
        }

        // Check if the deployment was successful
        require(addr != address(0), "Deployment failed!");

        // Store the address of the deployed contract
        deployedContract = addr;
    }

    // Now we can test the deployed contract
    function callDeployedContract() external view  returns (bytes memory) {
        (, bytes memory response) = deployedContract.staticcall("");

        // And we should receive the value 8
        return response;
    }
}
```
