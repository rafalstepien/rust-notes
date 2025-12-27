# Rust memory

## What Rust is actually protecting you from
Rust is not trying to stop your program from crashing.
It is trying to prevent undefined behavior (UB).

### Undefined Behavior in Rust
Undefined behavior means the program has entered a state where the language no longer defines what happens. At that point, anything is allowed: crashes, silent data corruption, security vulnerabilities, or seemingly correct behavior that breaks later.

Rust’s definition of undefined behavior is stricter than C and C++.

In C/C++, many invalid programs are treated as “the programmer’s responsibility” and may appear to work depending on the compiler or platform. Rust explicitly forbids these situations. If they occur, the program is considered invalid, even if it seems to run correctly.

Examples of undefined behavior include:
- Accessing memory after it has been deallocated
- Accessing memory outside the bounds of an allocation
- Interpreting memory as a value that does not match its type
- Mutating memory while it is simultaneously accessed in an incompatible way
- Concurrently accessing the same memory from multiple threads without proper synchronization

These are not “bugs that might cause problems.”. They are states the compiler assumes will never happen.


### Why Rust enforces guarantees at compile time
Because once undefined behavior exists, reasoning about the program becomes impossible.

The guarantees exist to:
- Eliminate entire classes of security vulnerabilities
- Allow the compiler to assume the program is valid and optimize aggressively
- Ensure that code behavior is predictable and portable

Performance is a consequence, not the goal. The real goal is soundness: the compiler must be allowed to trust the program.


### Memory safety, data races and memory leaks
These terms are often confused. They are not the same thing.

#### Memory safety
Means the program only:
- Accesses memory that is valid
- Uses memory according to its type
- Respects the lifetime of allocations
- Respects aliasing rules

Violating any of these leads to undefined behavior.

#### Data races
A specific form of UB that occur when:
- Multiple threads access the same memory
- At least one access is a write
- There is no synchronization

Rust treats data races as undefined behavior and prevents them by construction.

#### Memory leaks
A memory leak means allocated memory is never released.
This is allowed in Rust because leaking memory does not create an invalid state. The memory remains allocated and valid; it is simply never reclaimed.
Rust prioritizes correctness over resource cleanup.


### What safety means in Rust
Safety in Rust does not mean: no crashes, no bugs or no logic errors.
Safety means the program cannot enter an invalid state as defined by the language.

If a Rust program crashes, it is still safe.
If it invokes undefined behavior, it is not.

Rust’s entire design is built around making invalid states unrepresentable.

## Rust's memory model
### Stack vs heap
Data is stored either on stack or on the heap.
Both represent just regions of computer's RAM.

**Mental model**
Stack contains plain bytes. Sometimes, those bytes encode an address.
Pointer is integer-sized value, whose numeric value happens to be a memory address.
Stack does not know it points to the heap.
Heap does not know who points to it.
There is no "points to" action, there is just a number.
CPU just follows addresses.

```
Stack frame:
+------------------+
| ptr = 0x7ffd1234 |  <-- just a number
| len = 3          |
| cap = 8          |
+------------------+

Heap:
0x7ffd1234 → [ A | B | C | _ | _ | _ | _ | _ ]
```

**Thin pointer**
Just address, eg. `*mut T`, `&T` (for sized T)
```
+------------------+
| ptr = 0x7ffd1234 |  <-- just a number
+------------------+
```

**Fat pointer**
Address + metadata, eg `&str`, `&[T]`
```
+------------------+
| ptr = 0x7ffd1234 |  <-- just a number
| len = 3          |
+------------------+
```

Idependent of where the data is stored:
- If a value implements copy trait, it is copied on re-assignment to other variable,

**Stack**
- Stack is a pool of bytes in RAM organized into frames
- Frame corresponds to function scope, code blocks (`{}`). 
- When function finishes its execution, the frame is deallocated.
- Deallocation is automatic and implicit.
- Fixed-size values (int, float, bool, char) can be stored directly on the stack.

**Heap**
- Heap is a big pool of bytes in RAM. It stores contiguous chunks of bytes. Heap has no organization like frames in stack.
- `Box` allocates some number of bytes on the heap and stores the resulting pointer on the stack, and knows how to deallocate those bytes later
- Vec, String also allocate on the heap, when the capacity of current memory space is exceeded for Vec/String, the data gets copied, allocated on the new memory space with greater capacity, and freed from current memory space.
- Heap data has no inherent lifetime: it is deallocated when value responsible for freeing it gets dropped
- Heap deallocation is explicit and deliberate (via `drop`)

**Fixed-size data**
- Data that has size known at compile time, eg. `i32` or `[u8; 32]`, or structs with fixed fields

**Dynamically-sized data**
- Data that size is not known at compile time, eg. `str`, `[T]`

### What going out of scope physically means
Variable / frame going out of scope means that the stack frame is popped and memory becomes available for reuse. Popped means that stack pointer is moved back and memory is considered free for reuse.

### What drop actually does
- Runs destructor code (`Drop::drop`)
- Releases owned resources (heap memory, file handles, locks, etc.)

### Why heap memory must have one deallocator
- Issue of double free or mismatched alloc/free happening
- Freeing memory twice -> freeing memory you don't own -> allocator metadata corruption
- Heap memory must be freed exactly once to keep allocator state consistent




# Known unknowns
- Does Rust prevent accessing memory that was deallocated, or even having a pointer to that location?
