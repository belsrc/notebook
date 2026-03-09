---
tags:
  - comp-sci
  - memory
gardening: 🌳
date: 2025-03-08
reference:
  - https://en.wikipedia.org/wiki/Call_stack
  - https://en.wikipedia.org/wiki/Memory_management#HEAP
---
## Process Memory Layout

Before examining the stack and heap individually, it helps to understand where they live within a process's virtual address space. On a typical 64-bit Linux system the kernel maps a process into a canonical layout:

![](../../images/comp-sci/proc-memory-layout.png)
<!--
High address  (0x7FFF FFFF FFFF)
┌──────────────────────────────────┐
│  kernel space (not user-visible) │
├──────────────────────────────────┤
│  stack          ↓ grows down     │  thread stacks, one per thread
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤
│  (unmapped gap / ASLR variance)  │
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤
│  memory-mapped region            │  mmap(), shared libs (.so / .dylib)
├ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤
│  heap           ↑ grows up       │  malloc / new / Box<T>
├──────────────────────────────────┤
│  BSS segment                     │  zero-initialized globals
├──────────────────────────────────┤
│  data segment                    │  initialized globals and statics
├──────────────────────────────────┤
│  text segment                    │  executable instructions (read-only)
└──────────────────────────────────┘
Low address   (0x0000 0000 0000)
-->

The stack and heap grow toward each other. If they meet (an extreme stack overflow or exhausted virtual memory), the OS raises a fault. **Address Space Layout Randomization (ASLR)** randomizes the base addresses of the stack, heap, and shared libraries at each process launch to make memory-corruption exploits less predictable.

Each thread has its own stack (commonly allocated via `mmap` by the threading library) but shares the heap and all other segments with the process.

**Note:** diagrams in this document are schematic. Actual address ranges and stack growth direction are architecture- and OS-dependent, but the logical structure is the same.
## Stack

A **call stack** consists of **stack frames** (also known as _activation records_ or _activation frames_). These data structures are machine-dependent and application binary interface (ABI)-dependent, containing state information for subroutines. Each stack frame corresponds to a call to a subroutine that has not yet returned.

The stack frame at the top of the stack belongs to the currently executing routine, which can access information within its frame (such as parameters or local variables) in any order. A typical stack frame includes at least the following items: the arguments (parameter values) passed to the routine, the return address back to the caller, and space for the routine's local variables.

Since the call stack is organized like a stack, the caller pushes the return address onto it. When the called subroutine finishes executing, it pops the return address off the call stack and transfers control back to that address.

```c
void displayDoubled(int value) {
  int doubled = value * 2; // local variable
  printf("doubled = %d\n", doubled);
}

int main() {
  int myInt = 10;        // local variable in main
  displayDoubled(myInt);  // call to functionA
  return 0;
}
```

![](../../images/comp-sci/call-stack-simple-dark.png)
<!--
 main Frame                         main Frame
 ╔═══════════════╗                  ╔═══════════════╗
 ║ ┌───────────┐ ║                  ║ ┌───────────┐ ║
 ║ │ myInt: 10 │ ║                  ║ │ myInt: 10 │ ║
 ║ └───────────┘ ║                  ║ └───────────┘ ║
 ╚═══════╦═══════╝                  ╚═══════════════╝
         ║                          ╔════════════════════╗
         ║                          ║ ┌────────────────┐ ║
         ║                          ║ │ value: 10      │ ║
         ║                          ║ └────────────────┘ ║
         ║                          ║ ┌────────────────┐ ║
         ╚══════════════════════════╣ │ Return Address │ ║
          main calls displayDouble  ║ └────────────────┘ ║
                                    ║ ┌────────────────┐ ║
                                    ║ │ doubled: 20    │ ║
                                    ║ └────────────────┘ ║
                                    ╚════════════════════╝
-->

If a called subroutine invokes another subroutine, it will push another return address onto the call stack, and this process continues, stacking information as the program dictates.

```c
int factorial(int n) {
  if (n <= 1) {
    return 1;
  }
  else {
    return n * factorial(n - 1);
  }
}

int main() {
  int result = factorial(5);
  printf("Factorial of 5 is %d\n", result);
  return 0;
}
```

![](../../images/comp-sci/call-stack-recursive-dark.png)
<!--
      ╔══════════════════════╗
      ║ ┌──────────────────┐ ║
      ║ │ factorial(n = 5) │ ║
    ┌▶║ └──────────────────┘ ║─┐
    │ ╚══════════════════════╝ │
    │ ╔══════════════════════╗ │
    └─║ ┌──────────────────┐ ║◀┘
      ║ │ factorial(n = 4) │ ║
    ┌▶║ └──────────────────┘ ║─┐  E
    │ ╚══════════════════════╝ │  x
 R  │ ╔══════════════════════╗ │  e
 e  └─║ ┌──────────────────┐ ║◀┘  c
 t    ║ │ factorial(n = 3) │ ║    u
 u  ┌▶║ └──────────────────┘ ║─┐  t
 r  │ ╚══════════════════════╝ │  i
 n  │ ╔══════════════════════╗ │  o
    └─║ ┌──────────────────┐ ║◀┘  n
      ║ │ factorial(n = 2) │ ║
    ┌▶║ └──────────────────┘ ║─┐
    │ ╚══════════════════════╝ │
    │ ╔══════════════════════╗ │
    └─║ ┌──────────────────┐ ║◀┘
      ║ │ factorial(n = 1) │ ║
      ║ └──────────────────┘ ║
      ╚══════════════════════╝
-->

If the stack grows to consume all the allocated space, an error known as a **stack overflow** occurs, often resulting in a program crash. Adding a subroutine's entry to the call stack is sometimes referred to as "winding," while removing entries is known as "unwinding."

### Stack Size Limits

The stack does not grow without bound. The OS allocates a fixed virtual region for it per thread and places a **guard page**, an unmapped page, immediately below the bottom of that region. Any write into the guard page triggers a segmentation fault, which the OS surfaces to the program as a stack overflow signal (`SIGSEGV` on Linux, `EXCEPTION_STACK_OVERFLOW` on Windows).

Default stack sizes (main thread, platform-dependent):

|Platform|Default stack size|
|---|---|
|Linux|8 MiB (`ulimit -s`)|
|macOS|8 MiB|
|Windows|1 MiB|
|pthreads (new thread)|2–8 MiB (implementation-defined)|

Stack size can be changed at process launch or per thread:

```c
// POSIX: set stack size when creating a thread
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setstacksize(&attr, 4 * 1024 * 1024);  // 4 MiB
pthread_create(&tid, &attr, thread_fn, NULL);
pthread_attr_destroy(&attr);
```

The shallow default is intentional. Because the stack is allocated up front in the virtual address space, a large default would exhaust the 64-bit address space quickly in programs with thousands of threads (common in server workloads). Runtimes that need deep recursion either increase the limit explicitly or, as Go and Java do, implement **growable stacks** that copy-and-extend the stack segment on overflow rather than crashing.

The return address is saved upon entering the subroutine. Having such a field located in a known position within the stack frame allows code to access each frame sequentially below the currently executing routine's frame. It also facilitates restoring the frame pointer to the _caller’s_ frame just before the routine returns. See [CPU Function Execution](../CPU%20Function%20Execution.md) for how return addresses are managed via `CALL`/`RET`.

### Stack vs. Heap Allocation

The compiler decides at compile time where a variable lives. The decision follows two primary criteria:

**Lifetime:** a variable whose lifetime is strictly bounded by its enclosing scope can live on the stack; it is automatically destroyed when the function returns. A variable that must outlive its creating scope (returned pointer, stored in a global, passed across threads) must live on the heap.

**Size:** stack allocation requires the size to be statically known. Variable-length arrays and values whose size is only known at runtime must go to the heap.

```c
void example(int n) {
  int fixed[4];           // stack: size known, lifetime = this call
  int *dynamic = malloc(n * sizeof(int)); // heap: size unknown at compile time

  char *msg = "hello";    // stack: pointer is on stack; string literal is in .rodata
  char *copy = strdup(msg); // heap: must survive past this scope if returned

  free(dynamic);
  free(copy);
}
```

Languages with borrow checkers (Rust) or escape analysis (Go, Java HotSpot) formalize this decision. In Rust, `Box<T>` is the explicit heap allocation; the absence of `Box` means stack. In Go, the compiler runs **escape analysis** to detect whether a variable's address escapes the function; if it does, the variable is heap-allocated even if the source code looks stack-like:

```go
func newInt() *int {
  x := 42        // x escapes to heap because its address is returned
  return &x
}
```

The practical consequence: stack allocation is essentially free (a single `sub rsp, N` in the prologue) and has excellent cache locality since all locals are contiguous. Heap allocation involves an allocator call, possible lock contention, and the allocated block may be far from other recently used data. Prefer stack allocation wherever lifetime and size constraints allow.

## Heap

The heap is a mechanism that allows programs to dynamically allocate memory as needed and free it for reuse when it is no longer required. It serves as a pool of long-lived memory that is shared across the entire program. Unlike stack memory, which is automatically allocated and deallocated with function calls, heap memory is managed independently, enabling more flexible memory usage. This characteristic is why heap memory is often referred to as "dynamic memory."

```c
int *new_int_array(int size) {
  int *new_arr = (int*)malloc(size * sizeof(int));
  
  if (new_arr == NULL) {
    fprintf(stderr, "Memory allocation failed\n");
    exit(1);
  }
  
  return new_arr;
}
```


![](../../images/comp-sci/heap-memory-dark.png)
<!--
Stack                      Heap
╔════════════════════╗    ╔════════════════════════════════════════════╗
║ ┌────────────────┐ ║    ║ ┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐ ║
║ │ size: 6        │ ║    ║ │     ││     ││     ││     ││     ││     │ ║
║ └────────────────┘ ║    ║ └─────┘└─────┘└─────┘└─────┘└─────┘└─────┘ ║
║ ┌────────────────┐ ║ ┌─▶║ ┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐ ║
║ │ Return Address │ ║ │  ║ │ [0] ││ [1] ││ [2] ││ [3] ││ [4] ││ [5] │ ║
║ └────────────────┘ ║ │  ║ └─────┘└─────┘└─────┘└─────┘└─────┘└─────┘ ║
║ ┌────────────────┐ ║ │  ║ ┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐┌─────┐ ║
║ │ new_arr        │─║─┘  ║ │     ││     ││     ││     ││     ││     │ ║
║ └────────────────┘ ║    ║ └─────┘└─────┘└─────┘└─────┘└─────┘└─────┘ ║
╚════════════════════╝    ╚════════════════════════════════════════════╝
-->

Various techniques have been developed to enhance memory management. For instance, virtual memory systems distinguish between the memory addresses utilized by a process and the actual physical addresses. This separation allows multiple processes to run simultaneously and can extend the size of the virtual address space beyond the available RAM by using methods like paging or swapping with secondary storage.

When a memory allocation request is made, the system's task is to find a block of unused memory large enough to satisfy the request. Memory requests are fulfilled by allocating portions from a large pool known as the heap or free store. At any given time, some sections of the heap are in use, while others remain "free" (unused) and available for future allocations.

### Allocator Strategies and Fragmentation

The allocator must satisfy requests of arbitrary size in arbitrary order. Over time this creates **fragmentation**: free memory exists but is split into non-contiguous pieces too small to satisfy a large request.


![](../../images/comp-sci/mem-fragmentation.png)
<!--
Initial state (64 bytes free, contiguous):
┌────────────────────────────────────────────────────────────────┐
│                         free                                   │
└────────────────────────────────────────────────────────────────┘

After several alloc/free cycles:
┌───────┬────────┬───────┬────────┬───────┬────────┬────────────┐
│ used  │  free  │ used  │  free  │ used  │  free  │    used    │
│  8 B  │  4 B   │ 12 B  │  4 B   │  8 B  │  4 B   │    24 B    │
└───────┴────────┴───────┴────────┴───────┴────────┴────────────┘
         ↑              ↑              ↑
         Only 12 bytes free total, but no single block ≥ 8 B
-->

Three foundational strategies manage the free list:

**Bump (arena) allocation:** maintains a single pointer into a large pre-allocated block and advances it on every allocation. `free` is a no-op; the entire arena is released at once. Allocation is O(1) and produces no fragmentation, but requires all objects to share the same lifetime. Common in compilers, parsers, and per-request web server allocators.

```c
typedef struct { char *base; char *ptr; size_t remaining; } Arena;

void *arena_alloc(Arena *a, size_t size) {
  size = (size + 7) & ~7;            // align to 8 bytes

  if (size > a->remaining) {
    return NULL;
	}

  void *p = a->ptr;
  a->ptr += size;
  a->remaining -= size;
  return p;
}
// arena_free: free(arena.base) — all objects gone at once
```

**Free-list allocation:** `malloc` maintains a linked list of free blocks. On allocation it searches for a block of sufficient size (first-fit, best-fit, or worst-fit policy) and may split it; on `free` it coalesces adjacent free blocks. This is the strategy used by `glibc`'s `ptmalloc` and most general-purpose allocators. Allocation is amortized O(1) but fragmentation accumulates under adversarial access patterns.

**Slab allocation:** a pool of fixed-size slots for a single object type. A slab pre-allocates N slots; allocation pops a slot, free pushes it back. No fragmentation is possible within a slab; the OS kernel uses this extensively (Linux `SLUB` allocator for kernel objects). User-space equivalents include jemalloc's size-class bins and mimalloc.

The tradeoff summary:

|Strategy|Alloc cost|Free cost|Fragmentation|Use case|
|---|---|---|---|---|
|Bump|O(1)|O(1) bulk|None|Short-lived, same-lifetime|
|Free-list|O(1) amort|O(1) amort|Yes|General purpose|
|Slab|O(1)|O(1)|None|Fixed-size objects, high freq|

### Automatic Management: Garbage Collection

Garbage collection is a method that automatically identifies memory allocated to objects that are no longer in use by a program and reclaims that memory to a pool of available memory locations. This approach differs from "manual" memory management, where the programmer explicitly writes code to request and release memory. Although automatic garbage collection reduces the workload for programmers and helps prevent certain types of memory allocation errors, it does require its own memory resources and can compete with the application program for processor time.

Prominent Language Examples: Lisp, C#, Java, Go, and JavaScript

### Automatic Management: Reference Counting

Reference counting is a method used to determine when memory is no longer needed by a program. It does this by keeping track of a counter that indicates how many independent pointers refer to a particular piece of memory. Whenever a new pointer is created to point to this memory, the counter increases. Conversely, when a pointer changes its target, or if it is deleted or no longer points to any area, the counter decreases.

When the counter reaches zero, the memory is considered unused and can be freed. Some reference counting systems require programmer intervention, while others are implemented automatically by the compiler. A drawback of reference counting is that circular references can occur, leading to memory leaks. This issue can be addressed in two ways: by introducing a concept called a "weak reference" (which does not affect the reference count but is notified when the memory it points to is no longer valid) or by combining reference counting with garbage collection.

Prominent Language Examples: Objective-C and Swift

### Automatic Management: Ownership

In an ownership and borrowing model, each value has a single **owner**, a variable that is responsible for managing the memory associated with that value. When the owner goes out of scope, the value is automatically deallocated. When ownership is transferred (or moved) from one variable to another, the previous owner becomes invalid. This mechanism helps prevent issues like double frees and invalid memory access. 

You can create references to a value without taking ownership. Borrowing can be either immutable, allowing multiple references, or mutable, permitting only one reference at a time.

Prominent Language Examples: Rust