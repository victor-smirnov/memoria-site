---
title: "Why C++"
description: "Why Memoria is using C++"
date: 2020-10-06T08:49:31+00:00
lastmod: 2020-10-06T08:49:31+00:00
draft: false
images: []
menu:
  docs:
    parent: "overview"
weight: 30
toc: true
---

Memoria is using C++ as its main development language and it's a deliberate decision. This document explains what C++ gives to data platforms and how optimal language and run-time environment might look like.

From set-theoretic perspective there are much more data structures than algorithms, this is the reason why generic programming (GP) support is important for any modern programming language. The same linear search algorithm can be applied to the great variety of physical data layouts and types. Generic programming hides this variety from the programmer. 

## Two Types of Generic Programming ##

There are two main ways how generic programming can be implemented. **Polymorphic** GP is based on the fact that all objects allocated on the heap have common layout and access pattern. They are long-lived and referenced via their addresses in the heap. Primitive data types may also be wrapped into an object (boxed). In this case it's possible to compile only one instance of generic procedure and then use virtual dispatch to select possibly different execution paths for different types. Polymorphic GP-supporting compiler just checks generic type substitution rules at compile-type, then erase all type-specific information on generic variables because it's not actually needed at run-time. This type of GP is the most scalable, providing that the programmer is happy with heap allocation and primitive data types boxing.

As a specific optimization, some polymorphic GP compilers can do whole or partial program monomorphisation, when they generate type-specific instances of procedures. Note that from the programmer's perspective, this optimization is a black box, there is no flexible way to mange it. Yet it's a cheap way to increase performance of generic code in some cases.

Compilers supporting **Monomorphic** GP produce specialized instance of generic procedure for each set of types it is used with in the program. This type of GP produces highly data-specialized and potentially the most performant code but it's not scalable. Monomorphic GP code is not modularizable, there is no separate compilation of generic libraries possible. In some cases it's possible to hide generic code behind [quasi-generic interfaces](https://bitbucket.org/vsmirnov/memoria/wiki/TemplatePimpl), compiled separately from application code using it. Nevertheless, if definition of a generic procedure or datatype is changed, all code using it directly must be recompiled.

Monomorphic GP is not limited to a specific heap data layout scheme, it doesn't depend on objects and virtual dispatch, it's truly generic. From other side, polymorphic GP is limited to heap objects but allows separate compilation of generic code because only one type of generic procedure is instantiated. Unfortunately, there is no cheap and elegant way to have both types of GP in a single language. So, practical way is to stick with only one type of GP: Java, C# and other languages with managed heaps support polymorphic GP. C++, D and Rust support mononorphic GP.

Note also that while some languages like C# (and Java in a distant future) support elements of monomorphic GP like generics on primitive data types, this should not be considered fully featured monomorphic GP, because such schemes are not Turing-complete and hence can't be considered a programming. More on this below.

By and large, the following is true for both types of GP with minor exceptions. Monomorphic GP is focused on Turing-complete type construction and specialization. The result is a great variety of different physical data layouts in the memory and layout-specific operations on that data, that may lead to unnecessary increase in machine code size.

Polymorphic GP is built around specific object layouts in the heap allowing to compile only one instance of a generic procedure for all types. This type of GP does not usually affect run-time code in any way. Instead it allows a programmer to define a set of type substitution rules, which are enforced at compile time. Polymorphic GP just guarantees that actual type substitution in the program is correct. 

Though monomorphic GP languages may also provide type substitution checking, rules are much more complex in this case, and there is ongoing debate around how to implement this feature efficiently. Rust has some form of substitution checking, but C++ doesn't (yet).

## Generic Programming for Data Structures ##

So, polymorphic GP is scalable in terms of generic code side but limited to object heaps with predefined data layouts. This is fine until our data structures are well-reducible to object graphs (linked lists) without introducing much overhead. Many practically important lightweight in-memory data structures like arrays, maps, trees, generic graphs and so on are of this kind. When specialization of such simple data structure on a primitive data types is desirable, restricted form of monomorphisation like in C# may be provided.

Though linked list is pretty generic data structure by itself and suitable to be a foundation for many practical data structures, object heaps are not always optimal when we need more precise (bit-grade) control on physical data layout in main memory. For example, linked list-based implementations of suffix trees, used in bioinformatics, have around 40x space overhead over raw unstructured genome data. But custom schemes have much more improved space properties.

In order to be efficient for generic data structures implementation, any programming language must provide generic configurable mapping from logical data space (ex.: objects) to physical data layout in main and external memory. C++ by itself together with its GP capabilities is well suitable for this task.

## C++ for Data Structures ##

First, C++ is a multi-paradigm language with very cheap abstractions which do not impose much overhead at run-time. In the simplest case C++ uses zero-cost object mapping to a raw memory location. That means,  information other than object properties is put into an object by the compiler. Important details of this mapping like alignment can be controlled via compiler attributes and pragma-directives.

Second, C++ has generic value types when a value is physically put inside specified context withing existing memory mapping:
```c++
template <typename T>
struct IntContextMapping {
    int prefix_; // Some data fields before.
    T value_     // value_ mapping will be in-lined between prefix_ 
                 // and suffix_ using default alignment.
    int suffix_; // Some data fields after.
};
```

Third, C++ allows building almost arbitrary types using template metaprogramming, and hence, arbitrary memory mappings. The following example demonstrates how different classes can be mapped linearly to a memory region with specified order via linear class inheritance hierarchy generator: 

```c++
// Forward declaration for linear hierarchy generator helper class. The
// helper takes list of mapping members which are classes with one template
// parameter, that is a chained base class.
template <template <typename Base> class C...> struct LinearHierarchy; 

// Main case. Split the list into head and tail. Instantiate Head as a superclass
// with LinearHierarchy<Tail...> as a parameter.
template <
    template <typename> class Head,
    template <typename> class Tail...
> 
struct LinearHierarchy: Head<LinearHierarchy<Tail...>> {};

// There are no more members in the list. Stop the generator. 
template <> struct LinearHierarchy {
    void visit() {} // do nothing here
};


// First example class for our linear mapping. 
template <typename Base>
struct Member1: Base {
    int value1_ = 1; // simple data member 

    // Function member that may call functions in other layout
    // members upward the hierarchy providing their names are known.
    void visit() 
    {
        Base::visit();
        std::cout << value1_ << " ";
    }
};

// Second example class for our linear mapping. 
template <typename Base>
struct Member2: Base {
    int value2_ = 2;
    void visit() {
        Base::visit();
        std::cout << value2_ << " ";
    }
};

// Fird example class for our linear mapping. 
template <typename Base>
struct Member3: Base {
    int value3_ = 3;
    void visit() {
        Base::visit();
        std::cout << value3_ << " ";
    }
};

void do_something() 
{
    LinearHierarchy<Member1, Member2, Member3> m;
    m.visit(); // prints 3 2 1
}
```

In other words we can build arbitrary type-to-memory mappings using basic C++ rules for POD types and template metaprogramming. In Memoria complex high-level data structures like relational tables are split into reusable fragments, which are combined by the metaprogramming framework into compound structures and mapped to raw memory blocks. Building a complex data structure from simple blocks, compiler can solve combinatorial optimization problems by selecting fragments most suitable for the specific context. 

