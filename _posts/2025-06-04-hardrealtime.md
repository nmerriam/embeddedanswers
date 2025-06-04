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
the overall goal is high throughput and not predictability.

### Pipeline

### Cache

#### Burst Memory Accesses

### Multi-core and DMA

### Branch Prediction

## Approaches for Mitigation