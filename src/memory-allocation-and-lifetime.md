# Memory allocation and lifetime

r[memory.alloc]

r[memory.alloc.intro]
Memory in a Rust program is divided into allocations. Allocations hold a consecutive sequence of 0 or more bytes, starting from an address.

r[memory.alloc.kinds]
There are several kinds of allocations:
* Allocations produced by items, such as `static`s,
* Allocations produced by [variables][memory.variable],
* Allocations produced dynamically on the heap.

r[memory.alloc.properties]
An allocation has a size, an alignment, and a base address. An allocation may be read-only or writable. It is undefined behaviour to modify a read-only allocation.

> [!WARN]
> A writable allocation may not necessarily permit mutation at any given time. 
> For example, a live shared borrow may prevent writing to certain bytes of the allocation, even if writable.

> [!NOTE]
> An allocation may be declared with a type, as a static item or a local.
> However, this type only specifies the size and alignment of the allocation - allocations are untyped in general,
> There are no limits on what data can be stored in the bytes of an allocation, and validity is only checked when a value is read from an allocation as a given type.
> Type-based-alias analysis, as with C, does not apply to Rust allocations. 

r[memory.alloc.address]
The base address of an allocation satisfies all of the following:
* It is non-zero,
* It is a multiple of the alignment of the allocation, and
* The address one-past-the-end of the allocation does not wrap arround the bounds of the address space.

> [!NOTE]
> Other constraints may apply to a particular kind of allocation.

## Items

r[memory.alloc.item]

r[memory.alloc.item.intro]
A static item, as well as const-promoted expressions, produce static allocations. Such allocations outlive the entire duration of the program.

r[memory.alloc.item.static]
An allocation produced in a `static` item has a size and alignment given by the type of the static. The allocation is read-only if the static is not declared `mut` and does not contain an `UnsafeCell`.

r[memory.alloc.item.const-promotion]
Certain expressions that immutably borrow a temporary may cause const-promotion and produce a static allocation. Such allocations have the size and alignment given by the type of the temporary. These allocations are always read-only.

## Variables

r[memory.alloc.variable]

r[memory.alloc.variable.intro]
A local variable (function parameter, binding, or temporary) produces an allocation. The size and alignment of the allocation is given by the type of the variable. Such allocations are always writable

> [!NOTE]
> The allocations are writable as they get initialized at runtime, even when immutable. 


r[memory.alloc.variable.lifetime]
The lifetime of an allocation produced by a variable is bounded by the variables scope. In the case of a temporary variable, this is the temporary scope. In the case of bindings, this is generally the lexical scope in which it is declared. For function parameters, this is the duration of the function call. 

> [!WARN]
> It may not be valid to *use* the value stored in a variable for the entire lifetime, for example, if the variable is moved from.

## Heap Allocations

r[memory.alloc.heap]

r[memory.alloc.heap.intro]
Heap or dynamic allocations may be produced throughout the program to provide dynamically sized memory.

r[memory.alloc.heap.alloc]
Allocation functions such as [`alloc::alloc::alloc`] or types such as [`alloc::boxed::Box<T>`] can produce dynamic allocations. In the case of the former, the size and alignment are given by the `Layout` parameter of the type. In the case of the latter, the size and alignment are given by the type parameter. 

> [!NOTE]
> Other functions may produce dynamic allocations, such as the C function `malloc`.

r[memory.alloc.heap.dealloc]
The lifetime of a heap allocation lasts until it is deallocated. Allocations produced by [`alloc::alloc::alloc`] or by [`Box<T>`][alloc::boxed::Box] may be deallocated by dropping the `Box` or by passing the pointer and the same layout to [`alloc::alloc::dealloc`]. 

> [!NOTE]
> Heap allocations produced by other means, such as `malloc`, will require deallocating through a different function, such as `free`. Mixing allocators for alloc and dealloc is undefined behaviour.
