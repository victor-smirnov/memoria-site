---
title: "Computational Storage"
description: ""
lead: ""
draft: false
menu: 
  docs:
    parent: "apps"
weight: 2020
toc: true
---

## Basic Information

Computational storage device or **CSD** is a computer device which primary function is data storage, but that can also perform certain data processing functions. Functions can be either pre-configured or user-provided, both offline and online.

There are two main reasons one may need a CSD.

1. It can significantly reduce computational system design complexity by eliminating dependency from CPU and operating system. CSD may perform all essential storage functions like object space management, concurrency and transactions on-device. Accelerators may access CSDs directly over PCIe using database-like message-based protocols instead of transferring raw blocks of data (as in case of block storage devises).
1. It can significantly improve power outage and other types of failures handling by tightly coupling essential low-low level storage functions with device hardware. By controlling essential functions, device manufactures may _co-desing_ those functions and storage hardware.

Counter-intuitively, *speed* isn't considered here to be the main argument for using a CSD. Tight coupling of memory and compute will always give speed advantage. The problem is that storage media like NAND Flash doesn't like _heat_ produced by compute hardware. Putting massive radiators on devices will decrease storage density. As a result, intensive compute functions have to be disaggregated from CSDs into separate storage accelerators which are easier to cool.

Fortunately, one of the most important class of queries that benefit from using CDS, _point-like queries_, do not produce a lot of heat. 

Note that NAND Flash has pretty high access latency, that is much higher than PCIe latency. On the random object access CSD may show only a little performance improvements over a conventional block-based SSD.

## Simple Example: Persistent Queue


## SWMRStore
## 


