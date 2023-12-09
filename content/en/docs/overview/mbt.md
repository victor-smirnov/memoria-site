---
title: "Memoria Build Tool"
description: ""
lead: ""
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 70
toc: true
---

Memoria Build Tool (MBT) is a helper program with the scope of Memoria project (and out of it, if it's fount to be helpful). It basically reads C++ sources (via Clang libs) and can:

1. Generate boilerplate code like bindings,
2. Invoke compiler to do type inference,
3. ...

MBT is integrated with CMake and has a special mode to produce a list of artifacts that will be included into the project's build process.

Currently MBT is used to instantiate containers in libraries, but later this scope will be extended.

TBC ...
