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

For every register operation, the register number for $A$ and $B$ live in the
same location:

## Decoding the Instruction

grab from registerfile

decode into immediate, func, etc.

# Execute Stage

# Memory Stage

# Writeback Stage

# Summary
