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
seen in the following examples:
- *Pipeline* helps to parallelize instructions, but makes the execution time for a
sequence of instructions dependent on the pipeline state.
- *Cache* helps to hide the delays of fetching code and data from slow memory, but
it is hard to predict cache misses, when the full delay will be experienced.
- *Burst Memory Accesses* work with cache to hide the delay of fetching especially
code from slow memory, but are affected by varying code alignment and cause
other users of shared memory to be blocked for longer.
- *Multi-core and DMA* add processing power but create unpredictable contention for
shared resources, such as shared memory.
- *Branch Prediction* usually helps to prefetch code to cache before the delays of
slow memory stall the pipeline, but hardware branch prediction creates unpredictable
behavior.

## Two Approaches for Hard Real-Time Systems

I am aware of two, quite different, approaches for trying to build
predictable, hard real-time systems on unpredictable hardware, discounting
the approach of simply ignoring the problem and hoping that everything will
be okay.

### Worst-case Timing Analysis

With worst-case timing analysis, the system is modelled in a way that captures
the longest possible delays for each unpredictable feature. If a branch can be
mis-predicted, then we assume it will be mis-predicted. If a cache miss
could possibly occur, then we will assume it does occur. Imagine we have a loop
that performs 100 iterations of straight-line code (the only conditional branch
is the loop termination condition). We assume that the first iteration fetches
the loop instructions to cache and that we get the maximum number of interrupts
that can possibly occur in the 100 iterations and that each interrupt evicts all
the loop instruction cache lines.

The concept is that the system might perform better than the worst-case,
but that will just lead to some harmless under-utilization of hardware.

This concept worked relatively well with simple, single-core processors. With
more complex pipelines and multi-core contention for shared resources, the
worst-case approach often leads to extreme under-utilization of hardware. The
system is safe, but no longer cost-effective. This problem has driven a search
for an alternative.

### Average Timing Analysis

With average timing analysis, the unpredictability of the system is not only
accepted, it is embraced as a feature. Rather than trying to accommodate a
worst-case that is incredibly unlikely, we build a system that tolerates
very rare timing failures, and rather make the most of the probabilistic
behavior. If a rare branch prediction failure occurs, we *expect*
(but do not require) that a rare
cache miss will not occur at exactly the same time. And when they do both
occur at the same time, the system fails in a way that is acceptable at
very low frequency.

Although it has some attractions, the average approach creates an obvious
problem. We know that the system will experience failures that must be
tested, but the failures occur too rarely to be explored in a normal system test.
Somehow, we want to be able to provoke those failures under test conditions
but never in the deployed system.