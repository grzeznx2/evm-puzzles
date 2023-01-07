# EVM puzzles

A collection of EVM puzzles. Each puzzle consists on sending a successful transaction to a contract. The bytecode of the contract is provided, and you need to fill the transaction data that won't revert the execution.

## How to play

Clone this repository and install its dependencies (`npm install` or `yarn`). Then run:

```
npx hardhat play
```

And the game will start.

In some puzzles you only need to provide the value that will be sent to the contract, in others the calldata, and in others both values.

You can use [`evm.codes`](https://www.evm.codes/)'s reference and playground to work through this.

## Puzzle 1
```
00      34      CALLVALUE
01      56      JUMP
02      FD      REVERT
03      FD      REVERT
04      FD      REVERT
05      FD      REVERT
06      FD      REVERT
07      FD      REVERT
08      5B      JUMPDEST
09      00      STOP
```
### Relevant OPCODES
```
CALLVALUE: Get deposited value by the instruction/transaction responsible for this execution
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
The goal is to JUMP to the JUMPDEST omitting all the REVERT opcodes. JUMP takes the first value from the stack and that's the destination to jump to. In our case there's only one value on the stack determined by CALLVALUE (which is equal to msg.value). In this case the mentioned value must be equal to 8, because JUMPDEST = 8.

```
Execution Order:
CALLVALUE: Pushes 8 to the stack
JUMP: Removes 8 from the stack and jumps to the 8th position of our code
JUMPDEST: Marks the jump destination
STOP: Halts Execution
```
### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=8&unit=Wei&callData=&codeType=Bytecode&code='3456FDFDFDFDFDFD5B00'_
