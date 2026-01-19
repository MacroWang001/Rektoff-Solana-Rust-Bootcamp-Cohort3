# Week 1 Working Questions - Rust Memory Model Quiz

Welcome to the Week 1 Working Questions!
These questions are designed to test your knowledge of how a Rust process allocates stack, heap, and static memory.

## Important Notes

In this quiz we read small snippets of Rust and reason about a simplified memory model. Real programs, debuggers, and binaries may lay out memory differently — compilers optimize, ABIs vary, and platforms use different conventions. We intentionally ignore those details here to build intuition. I wouldn't expect dumping the memory of these processes on your machine to look exactly like this.

The rules below are there to be explicit, but really I just want things to be simple - values go in slots. Each slot can take 1 char, or 1 int, or 1 bool, etc. Don't get caught up in the details about padding or word size. The same rules are used for the Memory Visualiser, so you might get more intuition by looking through those examples.

## Rules for the Simplified Memory Model

- **Address space & diagrams**: diagrams are vertical. The top shows higher addresses; the bottom shows lower addresses.
- **Stack frames placement vs. growth**: each new stack frame is allocated below the previous frame in the diagram (frames grow downward), but inside a frame the contents grow upward (data in frame grows upward).
- **Frame layout** (what's in the frame):
  - Arguments sit at the bottom of the frame.
  - Then local variables in the reverse of their source order (the last declared local in the source is placed higher in the frame).
  - The return slot is at the top of the frame.
- **Contiguous data**: when multiple adjacent values appear (e.g. arrays/tuples or multiple slots for a value), the left/earlier source value goes nearer the bottom, then the next value above it, and so on.
- **Slot size**: each cell in the diagram is exactly 64 bits (8 bytes).
- **Type sizes**: any type < 64 bits is padded to a full slot; types > 64 bits do not exist in this model. For our purposes: treat each value as occupying one slot.
- **Addresses**: all addresses are shown as truncated 64-bit hexadecimal e.g. 0xFACE (more bits / leading zeros aren't needed).

## Mini Reference Diagrams

### 1) Example frame skeleton

```
      +------------------+
      | Function: main   |
      +------------------+
      | ret = ()         | // Use ret for the return slot
      +------------------+
      | baz = 42         | // baz could be i8, u64, etc. just put the value in the slot
      +------------------+
      | foo = 142        | // same here, type will be obvious from the question
      +------------------+
      | bar = 'g'        | // bar must of been a char in the question
      +------------------+
```

### 2) Contiguous values order (earlier/leftmost placed lower)

```
      +------------------+
      | '0'              |
      +------------------+
      | 'o'              |
      +------------------+
      | 'f'              |
      +------------------+
```

### 3) Allocated and uninitialised

```
      +------------------+
      | foo = [???]      | // slot corresponds to foo but it is not initialised
      +------------------+
      | 0x9000 = [???]   | // slot corresponds to allocated but uninitialised memory
      +------------------+
```

---

**Good luck!**

---

## Question 1: Stack Memory - Arrays

Fill out each lettered slot ([A] etc.) in the diagram with the var and current value for where the program is up to in the source code, some questions might have the var or address already so just add the value. Make sure you use correct rust notation for referring to the variables.

```rust
fn main() {
    let a: i32 = 10;
    let b: [i32; 3] = [1, 2, 3]; // <-- HERE
}
```

**Stack (grows downwards):**
```
+------------------+
| Function: main   |
+------------------+
| ret = [???]      |
+------------------+
| [A]              |
+------------------+
| [B]              |
+------------------+
| [C]              |
+------------------+
| [D]              |
+------------------+
```

**Answers:**
- **A**: ________________
- **B**: ________________
- **C**: ________________
- **D**: ________________

---

## Question 2: Stack Memory - Tuples

Fill out each lettered slot ([A] etc.) in the diagram with the var and current value for where the program is up to in the source code, some questions might have the var or address already so just add the value. Make sure you use correct rust notation for referring to the variables.

```rust
fn main() {
    let tup: (i32, usize) = (1, 2);
    let arr = [3, 4] // <-- HERE
}
```

**Stack (grows downwards):**
```
+------------------+
| Function: main   |
+------------------+
| ret = [???]      |
+------------------+
| [A]              |
+------------------+
| [B]              |
+------------------+
| [C]              |
+------------------+
| [D]              |
+------------------+
```

**Answers:**
- **A**: ________________
- **B**: ________________
- **C**: ________________
- **D**: ________________

---

## Question 3: Stack, Heap, and Static Memory

Fill out each lettered slot ([A] etc.) in the diagram with the var and current value for where the program is up to in the source code, some questions might have the var or address already so just add the value. Make sure you use correct rust notation for referring to the variables. Layout the vector the same as in the visualiser example.

```rust
fn main() {
    let v = vec![11, 22];
    let l: &str = "ab";
    let s = String::from("cde"); // <-- HERE
}
```

**Memory Layout:**
```
Stack (grows downwards):
+------------------+
| Function: main   |
+------------------+
| ret = [???]      |
+------------------+
| s.ptr = 0x1232   |
+------------------+
| [A]              |
+------------------+
| [B]              |
+------------------+
| l.ptr = 0x0013   |
+------------------+
| [C]              |
+------------------+
| v.ptr = 0x1230   |
+------------------+
| [D]              |
+------------------+
| [E]              |
+------------------+ <-- End of stack, start of heap
| 0x1234 = [F]     |
+------------------+
| 0x1233 = [G]     |
+------------------+
| 0x1232 = [H]     |
+------------------+
| 0x1231 = [I]     |
+------------------+
| 0x1230 = [J]     |
+------------------+ <-- Static Memory start
| 0x0014 = 'b'     |
+------------------+
| 0x0013 = 'a'     |
+------------------+
| 0x0012 = 'e'     |
+------------------+
| 0x0011 = 'd'     |
+------------------+
| 0x0010 = 'c'     |
+------------------+
```

**Answers:**
- **A**: ________________
- **B**: ________________
- **C**: ________________
- **D**: ________________
- **E**: ________________
- **F**: ________________
- **G**: ________________
- **H**: ________________
- **I**: ________________
- **J**: ________________

---

## Question 4: Dynamic Vec Growth

Fill out each lettered slot ([A] etc.) in the diagram with the var and current value for where the program is up to in the source code, some questions might have the var or address already so just add the value. Make sure you use correct rust notation for referring to the variables. Layout the vector the same as in the visualiser example.

```rust
fn main() {
    let mut v = vec![(44, "a"), (55, "b")];
    v.push((66, "c")); // <-- HERE
}
```

**Memory Layout:**
```
Stack (grows downwards):
+------------------+
| Function: main   |
+------------------+
| ret = [???]      |
+------------------+
| v.ptr = 0x1230   |
+------------------+
| [A]              |
+------------------+
| [B]              |
+------------------+ <-- End of stack, start of heap
| 0x123b = [C]     |
+------------------+
| 0x123a = [D]     |
+------------------+
| 0x1239 = [E]     |
+------------------+
| 0x1238 = [F]     |
+------------------+
| 0x1237 = [G]     |
+------------------+
| 0x1236 = [H]     |
+------------------+
| 0x1235 = [I]     |
+------------------+
| 0x1234 = [J]     |
+------------------+
| 0x1233 = [K]     |
+------------------+
| 0x1232 = [L]     |
+------------------+
| 0x1231 = [M]     |
+------------------+
| 0x1230 = [N]     |
+------------------+ <-- Static Memory start
| 0x0012 = 'c'     |
+------------------+
| 0x0011 = 'b'     |
+------------------+
| 0x0010 = 'a'     |
+------------------+
```

**Answers:**
- **A**: ________________
- **B**: ________________
- **C**: ________________
- **D**: ________________
- **E**: ________________
- **F**: ________________
- **G**: ________________
- **H**: ________________
- **I**: ________________
- **J**: ________________
- **K**: ________________
- **L**: ________________
- **M**: ________________
- **N**: ________________

---

## Question 5: Call Stack - Function Calls

Which program corresponds to the diagram? (just put a 1 or 2 in the answer field)

### Program 1
```rust
fn main() {
    let a = f();
}
fn f() {
    g()
}
fn g() {
    // Do nothing
}
```

### Program 2
```rust
fn main() {
    let a = f();
    g();
}
fn f() {
    // Do nothing
}
fn g() {
    // Do nothing
}
```

**Stack Diagram:**
```
Stack (grows downwards):
+------------------+
| Function: main   |
+------------------+
| a = [???]        |
+------------------+
| Function: f      |
+------------------+
| ret = ()         |
+------------------+
| Function: g      |
+------------------+
| ret = ()         |
+------------------+
```

**Answer:** ________________

---

## Question 6: Static Memory Scope Error

What line will cause the error? (just put the line number in the answer field)

```rust
1  static GREETING_OUTER: &str = "Hello, universe!";
2  
3  fn main() {
4      greet();
5      println!("{}", GREETING);
6      static GREETING: &str = "Hello, world!";
7  }
8  
9  fn greet() {
10     println!("{}", GREETING);
11     println!("{}", GREETING_OUTER)
12 }
```

**Answer:** ________________

---

## Question 7: Research / Investigate Possible Memory Layouts

Answer 'Y' or 'N' in each box below. Blank answers are marked incorrect.

First, read the 3 rules for Rust's default layout (#[repr(Rust)]) in the Rust Reference and consider how they apply to structs.

Below is a diagram of a Vec's memory layout from the Rust source code. Based on the layout rules, which orderings of ptr, len, and capacity are possible on the stack?

- **Y** = ordering is possible
- **N** = ordering is not possible

> **Note:** Assume PhantomData and Allocator are zero-sized and can be ignored.

```
///             ptr      len      cap
///        +--------+--------+--------+
///        | 0x0123 |      2 |      4 |
///        +--------+--------+--------+
///             |
///             v
/// Heap   +--------+--------+--------+--------+
///        |    'a' |    'b' | uninit | uninit |
///        +--------+--------+--------+--------+
```

**Answers:**
- **(ptr, len, cap)**: ______
- **(ptr, cap, len)**: ______
- **(len, ptr, cap)**: ______
- **(len, cap, ptr)**: ______
- **(cap, ptr, len)**: ______
- **(cap, len, ptr)**: ______
