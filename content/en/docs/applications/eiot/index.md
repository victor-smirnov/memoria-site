---
title: "Embedded and IoT Applications"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2060
toc: true
---

## Current Status

Memoria is a pretty heavyweight framework and most likely isn't suitable for embedded and IoT applications running on micro-controllers with little memory onboard. Some parts of the framework, like lightweight Hermes profiles can be scaled down to such MCUs and used for data storage and communication purposes. 

Another limitation, that is not fundamental, is that Memoria assumes 64-bit hardware. Compiling it for 32-bit hardware is possible, but ensuring that everything works correctly and there are no performance regressions is not currently a priority. Anyway, [Hermes](/docs/overview/hermes)-level interoperability with 32-bit platforms is a priority.

We assume that eventually (sooner than later) 64-bit hardware starts prevailing even in the embedded domain, because it's easier to generalize the  software when all essential platforms have the same width.

Memoria currently can be run on top of Linux so any Linux-capable embedded platform is technically capable of running Memoria. We can even prototype computational storage devices out of single-board computers and off-the-shelf SSDs.

## Perspectives

There are several directions where Memoria may be moving in the context of [MAA](/docs/overview/accel).

* HRPC-enabled 64-bit RISC-V MCU, based on MMA's [xPU](/docs/overview/accel/#processing-element).

xPU runs computational _kernels_ -- _code modules_ loadable via HRPC protocol from code loading _endpoints_ (similar to Java's classloaders) available somewhere in HRPC _infrastructure_ the MCU is attached to. HRPC is transport-agnostic, so even a pair of an ordinary serial (SPI?) lines can serve its traffic. 

Such an HRPC-enabled MCU can be attached to other potential HRPC-enabled devices, like [computational storage](/docs/applications/storage) (in a variant suitable for embedded uses) or DRAM modules with in-memory computing capabilities, either local or _remote_.

Reference HDL/HCL implementation for such an MCU is a priority for Memoria.

* MMA accelerator downscaled to requirements embedded system (like, 15W total system power draw). 

There are two directions that are currently in mind for such accelerators. 

**First**, computational storage devices in the format of a single chip. They may provide rich storage and in-storage processing functions together with advanced features like transactions and nearly-perfect power outage resilience. Factoring out storage and data processing functions into a dedicated device may dramatically simplify design of an embedded system at the expense of single additional chip (that is usually an SPI flash anyway). 

**Second**, DSLEngine accelerators for embedded systems. DSLEngine is a database/stream/complex-event-processing engine that can be useful for handling various complex (composite) events in IoT applications and process controllers. RETE algorithm is useful but is pretty memory-hungry, so specialized device capable of running it may be welcome here.
