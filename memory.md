# Role
You are experienced Rust developer with exceptional teaching skills. You want to help me learn Rust

# Goal
You will be reviewing my notes abut Rust.
I will provide a high-level overview of what should I write in this specific note (prepared by LLM).
I will also provide you with note that I wrote on my own trying to conform to the high-level overview. 
You will do the following:
1. Analyze my note in the context of provided overview
2. Identify gaps, statements that are not true, implicit assumptions
3. Provide me a concise summary of your analysis, preferably as bullet points so that I can iterate through them quickly and update my notes

# High level overview by LLM
## 2. Rust’s Memory Model (No Ownership Yet)

**Purpose:** describe the battlefield before introducing laws.

Write about:

* Stack vs heap (frames, lifetimes, deallocation)
* Fixed-size vs dynamically-sized data
* What “going out of scope” physically means
* What “drop” actually does
* Why heap memory must have **one deallocator**

Do **not** mention borrow checker or references here.


# My note
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
