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
## Puzzle 3
```
00      36      CALLDATASIZE
01      56      JUMP
02      FD      REVERT
03      FD      REVERT
04      5B      JUMPDEST
05      00      STOP
```
### Relevant OPCODES
```
CALLDATASIZE: Get size of input data in current environment
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
The goal is to JUMP to the JUMPDEST omitting all the REVERT opcodes. JUMP takes the first value from the stack and that's the destination to jump to. In our case there's only one value on the stack determined by CALLDATASIZE (which is equal to the byte size of the calldata). In this case the mentioned value must be equal to 4 (for example 0x1234ABCDE), because JUMPDEST = 4.

```
Execution Order:
CALLDATASIZE: Pushes 4 to the stack
JUMP: Removes 4 from the stack and jumps to the 4th position of our code
JUMPDEST: Marks the jump destination
STOP: Halts Execution
```
### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x1234ABCD&codeType=Bytecode&code=%273656FDFD5B00%27_&fork=merge
## Puzzle 4
```
00      34      CALLVALUE
01      38      CODESIZE
02      18      XOR
03      56      JUMP
04      FD      REVERT
05      FD      REVERT
06      FD      REVERT
07      FD      REVERT
08      FD      REVERT
09      FD      REVERT
0A      5B      JUMPDEST
0B      00      STOP
```
### Relevant OPCODES
```
CALLVALUE: Get deposited value by the instruction/transaction responsible for this execution
CALLDATASIZE: Get size of input data in current environment
XOR: Bitwise XOR operation
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
CALLVALUE pushes some X value to the stack. Next, CODESIZE pushes 12 to the stack. XOR performs bitwise operation between CODESIZE and CALLVALUE and we know that the result must be equal to 10, since this is the JUMPDEST.

XOR explanation:

In the context of computing, the XOR operator performs a logical operation called exclusive or on two operands. It returns a value of true if either of the operands is true, but not both.
```
1 1 0 0 = 12 in binary
1 0 1 0 = 10 in binary
| | | |
| | | |
v v v v
0 1 1 0 = 6 in binary

Operand 1 (CODESIZE = 12) | OPERAND 2 (CODEVALUE = ?) | XOR = 10
1                          0                            1
1                          1                            0
0                          1                            1
0                          0                            0
```


```
Execution Order:
CALLVALUE: Pushes 6 to the stack
CALLDATASIZE: Pushes 12 to the stack
XOR: Removes 12 and 6 from the stack, performs XOR(12,6) and pushes 10 to the stack
JUMP: Removes 10 from the stack and jumps to the 10th position of our code
JUMPDEST: Marks the jump destination
STOP: Halts Execution
```
### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=6&unit=Wei&callData=&codeType=Bytecode&code=%2734381856FDFDFDFDFDFD5B00%27_&fork=merge

## Puzzle 5
```
00      34          CALLVALUE
01      80          DUP1
02      02          MUL
03      610100      PUSH2 0100
06      14          EQ
07      600C        PUSH1 0C
09      57          JUMPI
0A      FD          REVERT
0B      FD          REVERT
0C      5B          JUMPDEST
0D      00          STOP
0E      FD          REVERT
0F      FD          REVERT
```
### Relevant OPCODES
```
CALLDATASIZE: Get size of input data in current environment
DUP1: Duplicate 1st stack item
MUL: Multiplication operation
PUSH1: Place 1 byte item on stack
PUSH2: Place 2 byte item on stack
EQ: Equality comparison
JUMPI: Conditionally alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
```
CALLDATA pushes some X to the stack at position 0
DUP1 pushes the same X to the stack at position 1
MUL removes items from position 0 and 1 and pushes result of multiplication to the stack: X * X
PUSH2 0100 pushes 0100 (256) to the stack
EQ removes items from positon 0 and 1 and pushes result of comparison to the stack: 1 if X * X === 0100 or 0 if X * X !== 0
PUSH1 0C pushes 0x0c (12) to the stack
JUMPI removes items from position 0 and 1 and jumps to 0x0c (12) if X*X === 0100 (256)
Now we know that X = 16, because 16 * 16 = 256
```

```
Execution Order:
CALLDATA: pushes 16 to the stack
DUP1: duplicates 16 and pushes it to the stack
MUL: removes 16 and 16 from the stack and pushes 16*16 (0100) to the stack
PUSH2 0100: pushes 0100 (256) to the stack
EQ: removes 0100 and 0100 from the stack and pushes 1 to the stack (because 0100 === 0100)
PUSH1 0C: pushes 0x0c (12) to the stack
JUMPI: removes 1 and 0x0c from the stack and jumps to 0x0c (12) position
```
### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=16&unit=Wei&callData=&codeType=Bytecode&code=%2734800261010014600C57FDFD5B00FDFD%27_&fork=merge

## Puzzle 6
```
00      6000      PUSH1 00
02      35        CALLDATALOAD
03      56        JUMP
04      FD        REVERT
05      FD        REVERT
06      FD        REVERT
07      FD        REVERT
08      FD        REVERT
09      FD        REVERT
0A      5B        JUMPDEST
0B      00        STOP
```
### Relevant OPCODES
```
PUSH1: Place 1 byte item on stack
CALLDATALOAD: Get input data of current environment
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```
### Solution
We need to include in the calldata a value that will allow us to go to position 0A in the calldata when we use a byte offset index of 0 in the CALLDATA. The solution is in our case 0x000000000000000000000000000000000000000000000000000000000000000A.

### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x000000000000000000000000000000000000000000000000000000000000000A&codeType=Bytecode&code=%2760003556FDFDFDFDFDFD5B00%27_&fork=merge

## Puzzle 7
```
00      36        CALLDATASIZE
01      6000      PUSH1 00
03      80        DUP1
04      37        CALLDATACOPY
05      36        CALLDATASIZE
06      6000      PUSH1 00
08      6000      PUSH1 00
0A      F0        CREATE
0B      3B        EXTCODESIZE
0C      6001      PUSH1 01
0E      14        EQ
0F      6013      PUSH1 13
11      57        JUMPI
12      FD        REVERT
13      5B        JUMPDEST
14      00        STOP
```
### Relevant OPCODES
```
CALLDATASIZE: Get size of input data in current environment
CALLDATACOPY: Copy input data in current environment to memory
PUSH1: Place 1 byte item on stack
DUP1: Duplicate 1st stack item
CREATE: Create a new account with associated code
EXTCODESIZE: Get size of an account’s code
EQ: Equality comparison
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```

```
Execution Order:
CALLDATASIZE: pushes the size X of calldata to the tack
PUSH1 00: pushes 0 to the stack
DUP1: Duplicate 1st stack item (0 in our case).

Currently stack looks like this:
0
0
X

CALLDATACOPY: removes 3 items from the stack and use them as arguments for copying to the memory:
(0: byte offset in memory - where to copy the data, 0: byte offset in calldata - where to copy the data from, X: byte size from calldata - how many bytes to copy)
Here all the data from calldata is being copied to the memory.
After this operation stack is empty.

CALLDATASIZE: pushes the size X of calldata to the stack
PUSH1 00: pushes 0 to the stack
PUSH1 00: pushes 0 to the stack

Currently stack looks like this:
0
0
X

CREATE: removes 3 items form the stack and use them and as arguments for creating new account:
(0: amount of WEI sent to the new account, 0: byte offset in memory - where to copy the data needed to create new account from, X: byte size from memory - how many bytes to copy) and pushes 1 item (newly created account) to the stack.

Currently stack looks like this:
Y (20 bytes long address)

EXTCODESIZE: removes 1 item from the stack (account address) and pushes byte size of the accounts code

Currently stack looks like this:
Y': (code size of Y account)

PUSH1 01: pushes 1 to the stack

Currently stack looks like this:
1
Y': (code size of Y account)

EQ: removes 2 bytes from the stack, compares them and pushes result of the comparison to the stack
PUSH1 13: pushes 13 to the stack

Currently stack looks like this:
13
0 or 1: result of the EQ operation

JUMPI: removes two items from the stack and jumps to position 13 if second item = 1. So in order to make proper JUMP, the result of EQ operation must be equal to 1, which means that the Y' must also be equal to 0.
The example of such code is: 600160005360016000F3
JUMPDEST: destination of the jump

```

### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x600160005360016000F3&codeType=Bytecode&code=%2736%7E0803736%7E0%7E0F03B%7E114601357FD5B00%27%7E600%01%7E_&fork=merge

## Puzzle 8
```
00      36        CALLDATASIZE
01      6000      PUSH1 00
03      80        DUP1
04      37        CALLDATACOPY
05      36        CALLDATASIZE
06      6000      PUSH1 00
08      6000      PUSH1 00
0A      F0        CREATE
0B      6000      PUSH1 00
0D      80        DUP1
0E      80        DUP1
0F      80        DUP1
10      80        DUP1
11      94        SWAP5
12      5A        GAS
13      F1        CALL
14      6000      PUSH1 00
16      14        EQ
17      601B      PUSH1 1B
19      57        JUMPI
1A      FD        REVERT
1B      5B        JUMPDEST
1C      00        STOP
```
### Relevant OPCODES
```
CALLDATASIZE: Get size of input data in current environment
CALLDATACOPY: Copy input data in current environment to memory
PUSH1: Place 1 byte item on stack
DUP1: Duplicate 1st stack item
SWAP5: Exchange 1st and 6th stack items
GAS: Get the amount of available gas, including the corresponding reduction for the cost of this instruction
CALL: Message-call into an account
CREATE: Create a new account with associated code
EXTCODESIZE: Get size of an account’s code
EQ: Equality comparison
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```

```Execution Order:
Execution Order:
CALLDATASIZE: pushes the size X of calldata to the stack
PUSH1 00: pushes 0 to the stack
DUP1: Duplicate 1st stack item (0 in our case).

Currently stack looks like this:
0
0
X

CALLDATACOPY: removes 3 items from the stack and use them as arguments for copying to the memory:
(0: byte offset in memory - where to copy the data, 0: byte offset in calldata - where to copy the data from, X: byte size from calldata - how many bytes to copy)
Here all the data from calldata is being copied to the memory.
After this operation stack is empty.

CALLDATASIZE: pushes the size X of calldata to the stack
PUSH1 00: pushes 0 to the stack
PUSH1 00: pushes 0 to the stack

Currently stack looks like this:
0
0
X

CREATE: removes 3 items form the stack and use them and as arguments for creating new account:
(0: amount of WEI sent to the new account, 0: byte offset in memory - where to copy the data needed to create new account from, X: byte size from memory - how many bytes to copy) and pushes 1 item (newly created account) to the stack.

Currently stack looks like this:
Y (20 bytes long address)

PUSH1 00: pushes 0 to the stack
DUP1: Duplicate 1st stack item (0 in our case).
DUP1: Duplicate 1st stack item (0 in our case).
DUP1: Duplicate 1st stack item (0 in our case).
DUP1: Duplicate 1st stack item (0 in our case).


Currently stack looks like this:
0
0
0
0
0
Y (20 bytes long address)

SWAP5: swaps 0th and 5th item:

Currently stack looks like this:
Y (20 bytes long address)
0
0
0
0
0

GAS: pushes to the stack the remaining gas after this operation

Currently stack looks like this:
Z: remaining gas
Y (20 bytes long address)
0
0
0
0
0

CALL: Creates a new sub context and execute the code of the given account using 7 items from stack as arguments:
Z: amount of gas to send to the sub context to execute. The gas that is not used by the sub context is returned to this one.
Y: the account which context to execute.
0: value in wei to send to the account.
0: byte offset in the memory in bytes, the calldata of the sub context.
0: byte size to copy (size of the calldata).
0: byte offset in the memory in bytes, where to store the return data of the sub context.
0: byte size to copy (size of the return data).

If the call to the external account was successfull, CALL pushes 1 to the stack, otherwise 0.

Currently stack looks like this:
0 or 1: result of the CALL opcode

PUSH1 00: pushes 0 to the stack
EQ: removes 2 items, pushes 1 if they are equal, 0 otherwise. In our case, to make the next JUMP successfull it has to return 1, which means that the result of CALL opcode must be 0 (so the CALL must fail)

Now the question is what code can we send in order to make the CALL fail? We know that the RETURN value of the bytecode that is run on deployment becomes the bytecode for the newly created contract. That's why we need to pass in calldata with a bytecode sequence such that the return value of the sequence causes a REVERT when run.
Such example is 60026000526001601ff3 (returns 02 - MUL). When MUL is being called there are no items on the stack, this way the call will be reverted.


```

### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=0&unit=Wei&callData=0x60FD60025360016000F3&codeType=Bytecode&code=%2736600080373660006000F0600080808080945AF1600014601B57FD5B00%27_&fork=merge

## Puzzle 9
```
00      36        CALLDATASIZE
01      6003      PUSH1 03
03      10        LT
04      6009      PUSH1 09
06      57        JUMPI
07      FD        REVERT
08      FD        REVERT
09      5B        JUMPDEST
0A      34        CALLVALUE
0B      36        CALLDATASIZE
0C      02        MUL
0D      6008      PUSH1 08
0F      14        EQ
10      6014      PUSH1 14
12      57        JUMPI
13      FD        REVERT
14      5B        JUMPDEST
15      00        STOP
```
### Relevant OPCODES
```
CALLDATASIZE: Get size of input data in current environment
PUSH1: Place 1 byte item on stack
DUP1: Duplicate 1st stack item
MUL: Multiplication operation
LT: Less-than comparison
EQ: Equality comparison
JUMP: Alter the program counter
JUMPDEST: Mark a valid destination for jumps
REVERT: Halt execution reverting state changes but returning data and remaining gas
STOP: Halts execution
```

```
Execution Order:
CALLDATASIZE: pushes the size X of calldata to the stack
PUSH1 03: pushes 3 to the stack
LT: removes 2 items from the stack, pushes 1 if first item < second item, 0 otherwise
PUSH1 09: pushes 9 to the stack

Currently stack looks like this:
0 or 1
9

JUMPI: Jumps to the position 9 if first item = 1. So now we know, that the result of LT must be equal to 1, which means that CALLDATASIZE must be greater than 3.
JUMPDEST: Jump destination
CALLVALUE: pushes value Y (call value) to the stack
CALLDATASIZE: pushes the size X of calldata to the stack
MUL: removes 2 items from the stack and pushes result of multiplication

Currently stack looks like this:
X * Y: result of multiplication

Execution the next opcodes we see that the result of the above multiplication must be equal to 8. We also know that calldatasize must be greater than 3. 
In order to solve the puzzle we can send 0x00000000 (4 bytes) as the calldata and 2 as the callvalue or 0x0000000000000000 (8 bytes) as the calldata and 1 as the callvalue

``

### Solution in the EVM Playground
https://www.evm.codes/playground?callValue=1&unit=Wei&callData=0x00000000&codeType=Bytecode&code=%2736600310600957FDFD5B343602600814601457FD5B00%27_&fork=merge
