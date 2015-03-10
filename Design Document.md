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
stages. Each stage will then be explained in detail, and each subcircuit, such
as the program counter incrementer, will be covered.

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
   values, which are subsequently passed through the ALU or similar subcircuit.
   The result of this operation is passed to the next stage.
4. _Memory_ This stage of the processor was unimplemented for this project.
5. _Writeback_ The values computed in the Execute stage are written back to the
   Register File.

Separating each stage is a set of registers ('pipline registers'). These hold
values between stages, such as the writeback register.

The stages are controlled by the clock. Most stages move forward on the rising
edge of the clock; the Register File writeback is conducted on the falling edge.
Note that values such as the writeback register are passed from the Decode stage
to the Writeback stage without interference. Additionally, any control flow
instructions, such as jumps and branches, are ignored. This is simulated by
disabling the writeback stage (no register is overwritten).

# Fetch Stage

This stage controls which instruction in the program to execute. It fetches
instructions from the program ROM, or read-only memory. In order to keep track
of execution of the program, there is a 32-bit register called the Program
Counter (PC). Because there were no jumps or branches for this project, the PC
was controlled solely by the Incrementer.

!(./images/fetch.png)[The Fetch stage]

## PC Incrementer

After executing an instruction, the program counter needs to be incremented by
4. This is done using a one-bit adder. In order to prevent using the same adder
4 times in series, the following subcircuit was created:

!(./images/pcplus.png)[The Program Counter incrementer]

By removing the first two bits of the program counter before adding 1, the
program counter is automatically incremented by 4 with just one operation. The
two bits that were initially removed are then appended back on.

The program ROM returns a 32-bit instruction, which is written to the IF/ID
register.

# Decode Stage

The decode stage serves two purposes. It retrieves values from the Register File
and partly decodes the instruction itself.

image: decode stage

## Retrieving Register Values

The portion of the instruction carrying the register number for A and B remains
consistent for every register operation. The register number for A is always
represented by bits 21-25 inclusive, and the register number for B is
represented by bits 16-20, inclusive. Note that if the instruction requested an
operation with an immediate, bits 16-20 would instead represent which register
to write back to. This discrepancy is covered in the next section, Decoding the
Instruction.

## Decoding the Instruction

Although the Fetch stage retrieves values from the Register File, it also partly
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
the writeback register is bits 11-15. By looking carefully at the opcodes for
the immediate and register operations, it becomes apparent that immediate
operations always have a 1 in bit 29 of the instruction, and register operations
always have a 0. Therefore, this bit is sent to the multiplexer as the selector.

image: Rw register

Register 5 holds the opcode for the current operation; this is simply bits
31-26, inclusive. These bits can be passed directly to the register.

The next register holds the shift amount for left and right shifts if it is
represented by an immediate. If it is, bits 6-10 represent this value, so these
are passed directly to this register.

Lastly, the ALU function is passed as well; this is represented by bits 0-5.

At this point, there are many redundant values - the register value for B could
be sent to both pipeline register B as well as the writeback register, whereas
only one will be used. The logic to decide between which is used lies in the
Execute stage. Additionally, because each of these fetch/decode operations are
conducted in parallel, the performance of the processor is not relinquished.

# Execute Stage

The Execute stage is where instructions are evaluated and computed. This stage
contains the ALU, or arithmetic logic unit, which computes a variety of
operations including logical shifts and sums.

During this stage, the pipeline registers between the Decode and Execute stage
are multiplexed so that the correct inputs are sent to the ALU. The selectors
for these multiplexers were chosen by identifying patterns in the opcodes and
`func` register, then sending specific bits from these numbers. These patterns
are identified in detail below. 

## Choosing an Operation Subcircuit

Not every instruction from the ROM uses the ALU. For this reason, there are
several computations that occur in parallel with the ALU; the proper output from
these subcircuits is then chosen. All of the operations can be divided into the
following subsections:

Operation | Subcircuit
----------|---------------------
ANDI      | ALU
ADDIU     | ALU
ORI       | ALU
XORI      | ALU
ADDU      | ALU
SUBU      | ALU
AND       | ALU
OR        | ALU
XOR       | ALU
NOR       | ALU
SLL       | ALU
SRL       | ALU
SRA       | ALU
SLLV      | ALU
SRLV      | ALU
SRAV      | ALU
SLT       | Signed comparator
SLTI      | Signed comparator
STLU      | Unsigned comparator
STLIU     | Unsigned comparator
MOVZ      | Equality
MOVN      | Equality

As can be seen, the majority of the operations do utilize the ALU.

### How to decide which uses ALU

# Memory Stage

This stage remained unimplemented in this project. However, pipeline registers
were still inserted to simulate this fourth stage in the processor.

# Writeback Stage

The writeback stage is possibly the fastest stage in the pipeline. During the
writeback stage, the value held by the 32-bit MEM/WB pipeline register B is
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

!(./images/forwarding.png)[Forwarding logic]

With this implementation of forwarding, the values from the following stages
are sent back to the Execute stage. The proper value is chosen via a
multiplexer. The deciding input for the multiplexer is sent from the forwarding
unit:

image: forwarding unit

This unit takes in 3 inputs, each a register number. The registers are compared;
if the register being read by the current instruction matches the register being
written by a previous instruction, there is a data hazard. The forwarding unit
detects this and outputs a 1 for that bit, a 0 for the other output. This bit
can then be sent directly to a multiplexer to select the proper output.

# Summary

Overall, the MIPS processor conducts a wide range of basic instructions. It can
handle data hazards through forwarding logic and conducts many operations
simultaneously due to its 5-stage pipelined design.


