<!-- https://hackmd.io/_ktX_wjcTua_Ios-o8QAnQ?both -->

# zkProver debugging
[TOC]

## Repositories involved
- [zkevm-proverjs](https://github.com/0xPolygonHermez/zkevm-proverjs): prover reference implementation writen in javascript
- [zkevm-prover](https://github.com/0xPolygonHermez/zkevm-prover): prover implementation writen in C
- [zkasm](https://github.com/hermeznetwork/zkasm): compiles .zkasm to a json ready for the zkevm-proverjs
- [pilcom](https://github.com/hermeznetwork/pilcom): Polynomial Identity Language
- [zkvmpil](https://github.com/0xPolygonHermez/zkvmpil): PIL source code for the zkVM (state-machines)
- [zkevm-rom](https://github.com/0xPolygonHermez/zkevm-rom): zkasm source code of the zkEVM
- [zkevm-doc](https://github.com/0xPolygonHermez/zkevm-doc): docs zkevm

## Setup environment
- ideal repository structure:
```
github
    --> zkevm-rom
    --> zkvmpil
    --> zkevm-proverjs
```
- Next steps are required to run the `zkprover:executor`:
```
git clone https://github.com/hermeznetwork/zkevm-rom.git
cd zkevm-rom
npm i && npm run build
cd ..
git clone https://github.com/hermeznetwork/zkvmpil.git
cd zkvmpil
npm i && npm run build
git clone https://github.com/0xPolygonHermez/zkevm-proverjs.git
cd zkevm-proverjs
npm i
```
- Detailed explanation:
  - repository `zkevm-rom`
    - `main/*` : contains assembly code
    - `build`: compiled assembly. code ready to the executor
  - repository `zkvmpil`
    - `src`: state-machines
    - `build`: compiled state-machines. code ready to the executor 
  - repository `zkevm-proverjs`
    - `src/main_executor.js`: cli to run executor easily
      - executor needs files fenerated from `zkevm-rom/build` & `zkvm pil/build`
      - it also needs an `input.json`
      - Examples:
        - [zkevm-rom file](https://github.com/hermeznetwork/zkevm-rom/blob/main/build/rom.json)
        - [zkvmpil file](https://github.com/hermeznetwork/zkvmpil/blob/main/build/zkevm.pil.json)
        - [input file](https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/testvectors/input.json)

- Run executor (in `zkevm-proverjs` repository)
>to just test the executor, the output is not needed
```
node src/main_executor.js ./testvectors/input.json -r ../zkevm-rom/build/rom.json -p ../zkvmpil/build/zkevm.pil.json -o ./testvectors/poly.bin
``` 
## Executor insights
Basically, the executor runs the program that is specified by the ROM.
The program can be seen in the `rom.json` file, which includes some debugging information.
Let's see an example of `assembly code` builded into the `rom.json`:
```
ASSEMBLY: 1 => B
JSON FILE:
{
  "CONST": 1,
  "neg": 0,
  "setB": 1,
  "line": 51,
  "fileName": "../zkevm-rom/main/main.zkasm"
 }
```
All operations are defined in the JSON file, plus `line` & `fileName` where the assembly code is.
This JSON file is ready to be interpreted by the `executor`

## VSCode debugging
In the `zkevm-proverjs` repository you can find an example of `launch.json` to debug the executor code: https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/.vscode/launch.json#L8

## Debugging tips
- Main executor code to debug: https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/src/executor.js#L12
- variable `l` is the rom.json that is going to be executed: https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/src/executor.js#L61
- debug helpers
  - [print registers](https://github.com/0xPolygonHermez/zkevm-proverjs/blob/main/src/executor.js#L1030)
- By monioring `ctx(context)`, registers and `op` you will see all the states changes made by the executor
- `ctx.input` contins all the variables loaded from `input.json`
- `storage` makes refrence to the merkle-tree
- transactions places at `input.json` are pre-processed and store it on `ctx.pTxs`. Besides, `globalHash` is computed given all the `inputs.json` according to [specification (TO_BE_UPDATED)](https://hackmd.io/tEny6MhSQaqPpu4ltUyC_w#validateBatch) 


## Table rom assembly instructions

|   NAME    |     DESCRIPTION      |                                                                                 EXECUTION                                                                                  |
|:---------:|:--------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|   MLOAD   |     memory load      |                                                                               op = mem(addr)                                                                               |
|  MSTORE   |    memory storage    |                                                                               mem(addr) = op                                                                               |
|   SLOAD   |     storage load     |                              op = `storage.get(SR, H[A0 , A1 , A2 , B0 , C0 , C1 , C2 , C3, 0...0]))` where `storage.get(root, key) -> value`                              |
|  SSTORE   |    storage store     | op = `storage.set(SR, (H[A0 , A1 , A2 , B0 , C0 , C1 , C2 , C3, 0...0], D0 + D1 * 2^64 + D2 * 2^128 + D3 * 2^192 )` where `storage.set(oldRoot, key, newValue) -> newRoot` |
|   HASHW   |   hash write bytes   |                                                                        hash[addr].push(op[0..D-1])                                                                         |
|   HASHE   |       hash end       |                                                                              hash[addr].end()                                                                              |
|   HASHR   |      hash read       |                                                                           op = hash[addr].result                                             |
|   ARITH   | arithmetic operation |                                                                              AB + C = D OR op                                                                              |
|    SHL    |      shift left      |                                                                                op = A << D                                                                                 |
|    SHR    |     shift right      |                                                                                op = A >> D                                                                                 |
| ECRECOVER |  signature recover   |                                                                 op = ECRECOVER( A: HASH, B: R, C:S, D: V)                                                                  |
|  ASSERT   |      assertion       |                                                                                   A = op                                                                                   |

## Examples assembly
### MSTORE
- assembly
```javascript=
A                       :MSTORE(sequencerAddr)
```
- rom.json
```json=
{
  "inA": 1,
  "neg": 0,
  "offset": 4,
  "mWR": 1,
  "line": 9,
  "offsetLabel": "sequencerAddr",
  "useCTX": 0,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 }
```
- description:
load `A` register in `op`, write in memory position 4 (`offset`) the `op` value

### MREAD
- assembly:
```javascript=
$ => A          : MLOAD(pendingTxs)
```
- rom.json
```json=
{
  "freeInTag": {
   "op": ""
  },
  "inFREE": 1,
  "neg": 0,
  "setA": 1,
  "offset": 1,
  "mRD": 1,
  "line": 25,
  "offsetLabel": "pendingTxs",
  "useCTX": 0,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 }
```
- description:
load a memory value from position 1 (`offset`) into `op` (action marked by `inFREE`), set `op` in `A` register

### LOAD FROM STORAGE
- assembly
```javascript=
$ => A                          :MLOAD(sequencerAddr)
0 => B,C
$ => A                          :SLOAD 
```
- rom.json
```json=
 {
  "freeInTag": {
   "op": ""
  },
  "inFREE": 1,
  "neg": 0,
  "setA": 1,
  "offset": 4,
  "mRD": 1,
  "line": 47,
  "offsetLabel": "sequencerAddr",
  "useCTX": 0,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 },
 {
  "CONST": 0,
  "neg": 0,
  "setB": 1,
  "setC": 1,
  "line": 48,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 },
 {
  "freeInTag": {
   "op": ""
  },
  "inFREE": 1,
  "neg": 0,
  "setA": 1,
  "sRD": 1,
  "line": 49,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 }
```
- description
  - load from memory position 5 (`sequencerAccValue`) into `op`, store `op` on `D` register
  - load from memory position 4 (`sequencerAddr`) into `op`, store `op` on `A` register
  - load CONST in `op`, store it in registers `B` and `C`
  - Perform SLOAD (reading from merkle-tree) with the follwing key: `storage.get(SR, H[A0 , A1 , A2 , B0 , C0 , C1 , C2 , C3, 0...0]))`
    - `SR` is the current state-root saved in register `SR`
    - `A0, A1, A2` has the sequencer address
    - `B0` is set to `0` pointing out that the `balance` is going to be read
    - `C0,C1,C2,C3` are set to 0 since they are not used when reading balance from merkle-tree
  - merkle-tree value is store in `op` (marked by `inFREE` tag), set `op` to register `A`

### WRITE TO STORAGE
- assembly
```javascript=
$ => A                          :MLOAD(sequencerAddr)
0 => B,C
$ => SR                         :SSTORE
```
- rom.json
```json=
{
  "freeInTag": {
   "op": ""
  },
  "inFREE": 1,
  "neg": 0,
  "setA": 1,
  "offset": 4,
  "mRD": 1,
  "line": 56,
  "offsetLabel": "sequencerAddr",
  "useCTX": 0,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 },
 {
  "CONST": 0,
  "neg": 0,
  "setB": 1,
  "setC": 1,
  "line": 57,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 },
 {
  "freeInTag": {
   "op": ""
  },
  "inFREE": 1,
  "neg": 0,
  "setSR": 1,
  "sWR": 1,
  "line": 58,
  "fileName": ".../zkevm-rom/main/main.zkasm"
 }
```
- description
  - read from memory position 4 (`sequencerAddr`) and store it on `op`, set `op` to register A
  - set CONST to `op`, store `op` in registers `B` and `C`
  - Perform SWRITE (write to merkle-tree) according: `storage.set(SR, (H[A0 , A1 , A2 , B0 , C0 , C1 , C2 , C3, 0...0], D0 + D1 * 2^64 + D2 * 2^128 + D3 * 2^192 )`
    - `SR` is the current state-root saved in register `SR`
    - `A0, A1, A2` has the sequencer address
    - `B0` is set to `0` pointing out that the `balance` is going to be read
    - `C0,C1,C2,C3` are set to 0 since they are not used when reading balance from merkle-tree
    - `D0, D1, D2, D3` is the value writen in the merkle-tree pointed out by `H[A0 , A1 , A2 , B0 , C0 , C1 , C2 , C3, 0...0]` --> in this example register `D` has the balance of the `seqAddr`
  - write merkle-tree state root in `SR` register 