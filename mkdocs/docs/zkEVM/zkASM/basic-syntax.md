This section is devoted to explain the basic syntax of zkASM from a high-level point of view. Advanced syntax is totally dependendant of the use case (e.g. the design of a zkEVM) and will be explained in more detail with more complete examples in a latter section.

It is important to remark that each instruction of the zkASM is executed sequentially (the exception being the execution of a jump) one after the other. Instructions are depicted line by line and are divided in two parts. The left side part includes the part of the code that is actually gets executed in the corresponding file, while the right part is related to the execution of opcodes, jumps and subrutines, which is indicated by the colon "$:$" symbol.

## Comments and Modules

Comments are made with the semicolon "$;$" symbol.

```
; This a totally useful comment
```

At this moment, only one-line comments are available.

One can subdivide the zkASM code into multiple files and import code with the `INCLUDE` keyword. This is what we refer to as the **modularity** of the zkASM.

```
; File: main.zkasm

INCLUDE "utils.zkasm"
INCLUDE "constants.zkasm"
; -- code --
```

## Storing Values on Registers

There are many ways in which values can be stored into registers:

1. Assign a constant into one or more registers is made using the arrow operator "=>".

```
0 => A,B
```

2. Similarly, we can store the value of a register into other registers.

```
A => B,C
```

More generally, we can store the value of a function $f$ of registers.

```
f(A,B) => C,D
```

3. We can also store a global variable into some register.

```
%GLOBAL_VAR => A,B
```

4. The result of executing an executor method can also be stored into one or more registers. The indication of such an execution is done with the dollar "$" sign, which should be treated as a free input.

```
${ExecutorMethod(params)} => A,B
```

Notice that the method `ExecutorMethod` does not necessarily depends on the registers. An good example of such a method is `SHA256`.

5. If a method gets executed (with the dollar sign) by its own, its main purpose is generating log information.

```
${ExecutorMethod(params)}
```

6. Apart from executor methods, one can also use inline functions. This functions, which are also instantiated by the executor, are simply "short" and non-reused executor methods.

```
${A >> 2} => B
${A & 0x03} => C
```

## Introducing Opcodes

Until this point, every instruction consisted in a direct interaction with the registers. Now, we move one step forward and we obtain interaction with other parts of the ROM thank to the introduction of the zkEVM opcodes.

To assign the output of a zkEVM opcode into some register we use the following syntax:

```
$ => A,B    :OPCODE(param)
```

A clear example of such situation is when using the memory load opcode:

```
$ => A,B    :MLOAD(param)
```

When a registers appear at the side of an opcode, it is typically used to indicate that the value of the register `A` is the input of the memory store opcode:

```
A   :MSTORE(param)
```

Similarly, we can assign a free input into a register and later on execute several zkEVM opcodes using the following syntax:

```
${ExecutorMethod(params)} => A      :OPCODE1
                                    :OPCODE2
                                    :OPCODE3
                                    ...
```

When a executor method with a register to store its result gets combined with a jump opcode is typically to handle some unexpected situation, such as running out of gas:

```
${ExecutorMethod(params)} => A :JMP(param)
```

It is also typicall to encounter negative jumps to check appropiate situations in which carry on forthcoming operations:

```
SP - 2  :JMPN(stackUnderflow)
```

## Code Injection

Inline javascript-based instruction can be injected in plain by using the doble dollar "$" symbol.

```
$${CODE}
```

The main difference between the single dollar sign and the doble dollar sign is that while the methods inside the single dollar sign come from the Executor, the doble dollar ones do not: its is plain javascript code that is executed by the ROM.

## Asserts

Asserts work by comparing what is being asserting with what the value on register `A`. So, for instance, the following instructions compares the value inside register `B` with the value inside register `A`:

```
B    :ASSERT
```
