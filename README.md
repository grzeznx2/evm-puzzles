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
## Puzzle 2
```
00      34      CALLVALUE
01      38      CODESIZE
02      03      SUB
03      56      JUMP
04      FD      REVERT
05      FD      REVERT
06      5B      JUMPDEST
07      00      STOP
08      FD      REVERT
09      FD      REVERT
```
### Relevant OPCODES
```
CALLVALUE: Get deposited value by the instruction/transaction responsible for this execution
CODESIZE: Get size of code running in current environment
SUB: Subtraction operation
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
Similar to the Puzzle 1, the goal is to JUMP to the JUMPDEST omitting all the REVERT opcodes. So we have to determine the value at the top of the stack when JUMP command is executed. In our case this value is the result of the SUB command, which subracts the value at position 1 from the value at position 0 on the stack. The value at position 0 is equal to the bytecode size of our contract (which is basically the number of OPCODES x 1 byte = 10 bytes). The last thing we have to do is to calculate the CALLVALUE (value at position 1), such that 10 - CALLVALUE = 6, which gives us 4.

```
Execution Order:
CALLVALUE: Pushes 4 to the stack
CODESIZE: Pushes 10 to the stack
SUB: Removes 10 and 4 from the stack, pushes 10-4=6 to the stack
JUMP: Removes 6 from the stack and jumps to the 6th position of our code
JUMPDEST: Marks the jump destination
STOP: Halts Execution
```
### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=4&unit=Wei&callData=&codeType=Bytecode&code=%2734380356FDFD5B00FDFD%27_&fork=merge
