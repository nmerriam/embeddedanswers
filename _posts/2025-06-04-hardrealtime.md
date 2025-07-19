---
layout: post
title:  "Why hard real-time is hard"
author: nick
categories: [ C, tutorial ]
# tags: [red, yellow]
image: assets/images/hardTime.png
description: "Why hard real-time is hard"
featured: true
hidden: true
comments: false
---

Hard real-time means software running in a system that only functions correctly
if results are delivered before hard deadlines. This system is a combination of
hardware and software. If we are writing software, we can control the software
itself, and the configuration of the hardware, to some extent. When we look at any
of the popular architectures for embedded real-time systems
(Renesas RH850, NXP MPC5xxx, Arm Cortex or Infineon TriCore), we find many hardware
acceleration features that work against the construction of a hard real-time system.

## The Mismatch

Most hardware acceleration features have evolved from desktop/server designs, where
optimization has largely focussed on throughput and not predictability, as can be
seen in the following examples.

### Pipeline

Executing one instruction can be broken down into multiple steps. Different
architectures identify different steps, but the following make up a representative
example:
- Fetch instruction
- Decode instruction
- Fetch data from memory (if required)
- Compute instruction
- Write data to memory (if required)

Rather obviously, the processor does not have to wait until one instruction has
finished all of its steps before starting to fetch the next instruction. It is not
entirely straightforward, since branch prediction might fail, and our speculative
fetch of what was predicted to be the next instruction might turn out to be incorrect.
Nevertheless, it is clear that we can make a much faster processor if we can overcome
these challenges and implement a
[pipeline](https://en.wikipedia.org/wiki/Instruction_pipelining).

### Cache

A modern processor can execute instructions faster than they can be fetched
main code memory and faster than data can be fetched from main data memory.
To avoid the processor spending most of its time stalled, waiting for memory,
instruction and data caches can be provided that do not help with the first
fetch but greatly accelerate repeated accesses to the same address. If we
have a loop, the first iteration of the loop might be very slow, but every
subsequent iteration finds the instructions for the loop already in fast
cache memory.

#### Burst Memory Accesses

### Multi-core and DMA

### Branch Prediction

## Approaches for Mitigation