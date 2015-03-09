---
title: CS 3410 Pipelined MIPS Processor
authors: Luis Gustavo de Medeiros (lsd53), Ajay Gandhi (aag255)
abstract: This document details the design of the MIPS processor, highlighting
details about core logic.
---

# Introduction

This document serves as a detailed explanation of our MIPS processor. It
explains the core logic behind each stage at a low level, preceded by a
high-level view of the design.

The design document is intended as an aid for the simulated Logisim processor.
It clarifies the design of the circuit so that each stage of the processor can
be understood in full. The document assumes a basic knowledge of logic gates and
pieces of a processor, e.g. a register.

The document begins at the top level, describing the processor as a set of
stages. Each stage will then be explained in detail, and important details such
as decoding logic will be covered.

Note that for any $N$-bit binary number, bit 0 refers to the least significant
bit and bit $N-1$ refers to the most significant bit. For example, bit 3 is the
4th least significant bit.

# Overview

The MIPS processor is designed as a 5-stage pipelined processor. Here is a basic
overview of the stages;

1. _Fetch_ The instruction is fetched from the program memory, and the program
   counter is advanced.
2. _Decode_ The instruction from the previous stage is separated into pieces.
   Any values stored in the Register File are fetched and passed on.
3. _Execute_ The pieces from the Decode stage are plexed to select the proper
   values, which are subsequently passed through the ALU or a comparator. The
   result of this operation is passed to the next stage.
4. _Memory_ This stage of the processor was unimplemented for this project.
5. _Writeback_ The values computed in the Execute stage are written back to the
   Register File.

Separating each stage is a set of registers ('pipline registers'). These hold
values between stages, such as the writeback register.

The stages are controlled by the clock. Most stages move forward on the rising
edge of the clock; the Register File writeback is conducted on the falling edge.
Note that values such as the writeback register are passed from the Decode stage
to the Writeback stage without interference. Additionally, any control flow
instructions, such as jumps and branches, are ignored. In other words, a nop is
sent through the processor by simulating the instruction `SLL $r0, $r0, 0`.

# Fetch Stage

This stage controls which instruction in the program to execute. It fetches
instructions from the program ROM, or read-only memory. In order to keep track
of execution of the program, there is a 32-bit register called the Program
Counter (PC). Because there were no jumps or branches for this project, the PC
was controlled solely by the Incrementer.

image: fetch stage

## PC Incrementer

After executing an instruction, the program counter needs to be incremented by
4. This is done using a one-bit adder. In order to prevent using the same adder
4 times in series, the following subcircuit was created:

image: PC+4

By removing the first two bits of the program counter before adding 1, the
program counter is automatically incremented by 4 with just one operation.

The program ROM returns a 32-bit instruction, which is written to the IF/ID
register.

# Decode Stage

The decode stage serves two purposes. It retrieves values from the Register File
and partly decodes the instruction itself.

image: decode stage

## Retrieving Register Values

For every register operation, the register number for A and B live in the
same location. The register number for A is represented by bits 21-25 inclusive,
and the register number for B is represented by bits 16-20, inclusive. Note that
if the instruction requested an operation with an immediate, these 5 bits would
represent which register to write back to. This discrepancy is covered in the
next section, Decoding the Instruction.

## Decoding the Instruction

Although this stage retrieves values from the Register File, it also partly
decodes the instruction. It does so by separating the 32-bit instruction into
smaller sets of bits. Then, it passes each of these sets to the ID/EX pipeline
registers, plexing between several values if necessary.

There are 7 pipeline registers between the Decode and Execute stage. Two of
these registers represent the operands for the operation, A and B (see
Retrieving Register Values above).

The next register contains the immediate value. In immediate operations, this
value is represented by bits 0-15, inclusive, so these bits are sent directly
to this register.

The fourth register down contains the writeback register. If the operation is an
immediate operation, the writeback register is bits 16-20, inclusive, otherwise
the writeback register is bits 11-15. Whether the writeback register is the
first or second value is represented by bit 29 of the instruction, so the
multiplexer takes in this value as the selector.

image: Rw register

Register 5 holds the operation that is being conducted; this is simply bits
31-26, inclusive. These bits can be passed directly to the register.

The next register holds the shift amount for left and right shifts if it is
represented by an immediate. In this case, bits 6-10 represent this value, so
these are passed directly to this register.

Lastly, the ALU function is passed as well; this is represented by bits 0-5.

At this point, there are many redundant values - the register value for B is
fetched as well as the immediate, whereas only one will be used. The logic to
decide between which is used lies in the Execute stage. Additionally, because
each of these fetch/decode operations are conducted in parallel, the performance
of the processor is not relinquished.

# Execute Stage

# Memory Stage

This stage remained unimplemented in this project.

# Writeback Stage

The writeback stage is possibly the fastest stage in the pipeline. During the
writeback stage, the value held by the 32-bit MEM/WB pipeline register is
written to the register denoted by pipeline register Rw.

The register writeback also occurs during the falling edge of the clock cycle.
For more information on why, refer to Resolving Data Hazards below.

# Resolving Data Hazards

Data hazards occur when an instruction attempts to use a register whose value
has not yet been updated. For example:

```mips
ADDI $1, $0, 100
ADDI $2, $1, 1
```

By the time the second instruction is executed, the processor will not have
written the new value to register `$1`. Therefore, in order to account for such
hazards, the proper value from the register must be forwarded to the appropriate
stage.

image: forwarding

With this implementation of forwarding, the values from the following stages
are sent back to the Execute stage. The proper value is chosen via a
multiplexer. The deciding input for the multiplexer is sent from the forwarding
unit:

image: forwarding unit

This unit takes in 4 inputs, each a register number. The registers are compared;
if the register being read by the current instruction matches the register being
written by a previous instruction, there is a data hazard. The forwarding unit
detects this and outputs the proper bits to select the inputs appropriately.

# Summary

Overall, the MIPS processor conducts a wide range of basic instructions. It can
handle data hazards through forwarding logic and conducts many operations
simultaneously due to its 5-stage pipelined design.


