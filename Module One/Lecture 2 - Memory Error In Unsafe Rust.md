# Lesson 2: Memory Errors and Unsafe Rust

## Introduction: Rust's Promise of Memory Safety

Rust guarantees memory safety through strict compile-time rules. If your code compiles in safe Rust, it’s free from entire categories of memory bugs — like use-after-free, buffer overflows, and data races.

This makes programs more reliable and less prone to subtle, dangerous errors.

But not all Rust is safe Rust.

The `unsafe` keyword lets you bypass some of these checks. It’s powerful, but it shifts responsibility to you — the programmer. If you get it wrong, you can reintroduce the very bugs Rust is designed to prevent.

In this lesson, we’ll explore memory errors by intentionally implementing them in `unsafe` Rust. This gives us a concrete understanding of what can go wrong when Rust’s safety guarantees are bypassed — and just how serious the consequences of undefined behavior can be.

What you'll learn in this lesson:

- Define and explain three common memory errors
- See how they occur using `unsafe` Rust
- Get practical experience with **undefined behavior (UB)** in action
- Understand why Rust’s safety rules exist — by seeing what happens without them

## 1. Use-After-Free (UAF)

A **use-after-free** (UAF) error occurs when a program accesses memory after it has been deallocated. This is a classic memory bug: freed memory might be reused for something else, so accessing it can lead to corrupted state or even code execution. Like all the bugs in this section, UAF is **undefined behavior (UB)** — meaning anything might happen.

In safe Rust, UAF is impossible: the ownership system ensures that once memory is freed, it can no longer be accessed. But in `unsafe` Rust, we can bypass this protection.

```rust
fn main() {
    let x = Box::new(5); // Allocate an integer on the heap
    let raw_x = Box::into_raw(x); // Convert to a raw pointer

    unsafe {
        // Free the memory manually
        drop(Box::from_raw(raw_x));

        // 🚨 UB: This is a use-after-free
        println!("Value after free: {}", *raw_x);
    }
}
```

**Explanation:**

1. `Box::new(5)` allocates an `i32` on the heap.
2. `Box::into_raw(x)` turns the box into a raw pointer, forgetting the destructor.
3. `drop(Box::from_raw(...))` reclaims ownership and deallocates the memory.

We then dereference `raw_x`, which points to freed memory. That memory may already be reused for something else.

This is classic UB. Sometimes the program prints a garbage value. Sometimes it crashes. Sometimes it seems fine — and that's the danger.

This is a great example to try out locally or in the [Rust Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=698a3a469ca171a14310dcdda544c0c6). Run the program multiple times — you'll likely see the value printed change between runs. Why? Because the heap allocator almost never gives your program the same memory address twice, especially across separate processes.

Now consider this: could the program ever crash instead? What would need to happen for that to occur?

If you think the answer is “no,” what assumptions are you making about your system, the heap allocator, or the OS? Are those assumptions always valid?

## 2. Buffer Overflow

A **buffer overflow** happens when a program writes data past the end of an allocated buffer. Just like use-after-free, this can trigger **undefined behavior (UB)** — meaning anything might happen.

When using safe Rust, you can't overflow a buffer — arrays, tuples, slices, and standard library collections (e.g. Vec) all check that you're within bounds before allowing access. But in `unsafe` Rust, raw pointers and unchecked access let you bypass those safeguards and write past the end of memory.

Here’s what it looks like when you write beyond a buffer in `unsafe` Rust:

```rust
fn main() {
    let mut buffer = [0u8; 5];         // Stack-allocated buffer (5 bytes)
    let not_in_buffer = 56789;         // Stack variable just after the buffer

    unsafe {
        let ptr = buffer.as_mut_ptr(); // Raw pointer to start of buffer

        // 🚨 UB: writing 6 bytes into a 5-byte buffer (1 byte overflow)
        for i in 0..6 {
            *ptr.add(i) = i as u8;
        }
    }

    println!("buffer: {:?}", buffer);             // Expected: [0, 1, 2, 3, 4]
    println!("not_in_buffer: {}", not_in_buffer); // Will this still be 56789?
}

```

**Explanation:**

1. `buffer` is a 5-byte array allocated on the stack.
2. `as_mut_ptr()` gives you a raw pointer to the start of the array.
3. Writing 6 bytes means you're going 1 byte past the end.
4. That final write may corrupt adjacent stack memory — including other variables or runtime bookkeeping.

This overflows the buffer by just one byte. But what happens if you change the loop to `0..7` — or even `0..10`?

Would you still expect the program to print output? Or even finish?

Think about what lies beyond the buffer on the stack — where is the loop variable `i` stored? What data is being overwritten as the loop writes further out of bounds?

When `unsafe` code corrupts its own control flow, things can break silently — or catastrophically. That’s the nature of **undefined behavior**.

Experiment with this code locally or in the [Rust Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=2565be1b4ef7612e871c20e63b87a2c2). Try larger loop bounds, but keep an eye on your terminal...

## 3. Data Race

A **data race** happens when two or more threads access the same memory at the same time, at least one of them writes, and there's no coordination. This leads to **undefined behavior (UB)** — the result is unpredictable.

**Necessary Conditions for Data Race:**

- Two or more threads are running at the same time.
- They access the same memory.
- At least one access is a write.
- There’s no synchronization (like a mutex or atomic operation).

**Example in Unsafe Rust:**

In safe Rust, this can’t happen. The ownership system, borrowing rules, and the `Send` and `Sync` traits ensure threads don’t touch memory at the same time in `unsafe` ways.

But as usual, `unsafe` Rust will not protect us from UB!

```rust
#![allow(static_mut_refs)]

use std::thread;
use std::time::Duration;

static mut COUNTER: usize = 0; // Shared mutable state

fn main() {
    let mut handles = vec![];

    for i in 0..10 {
        handles.push(thread::spawn(move || {
            unsafe {
                // 🚨 UB: multiple threads read and write to COUNTER unsynchronised
                COUNTER += 1;
                println!("Thread {} incremented COUNTER to {}", i, COUNTER);
            }

            thread::sleep(Duration::from_millis(1)); // Simulate work
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }

    unsafe {
        println!("Final COUNTER (unreliable): {}", COUNTER);
    }
}
```

**Explanation:**

1. `COUNTER` is a `static mut` — shared and mutable across all threads.
2. Each thread increments `COUNTER` inside an `unsafe` block, but the operation is not atomic.
3. This creates a race: multiple threads read the same value, increment it, and write it back — but some writes overwrite others.
4. The final value **may be less than 10**. Nothing crashes. No error messages. Just wrong results (quietly).

Experiment with this code locally or in the [Rust Playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=95734cabc90a6296a8c4f0d4f65bbbe6). Try running it multiple times — the final value may vary. For a more convincing demo, try this [modified version](https://play.rust-lang.org/?version=stable&mode=debug&edition=2024&gist=a13bbad2447724862a146e3a1fad5741) that removes most printing and loops until an update to `COUNTER` is missed. It *should* miss an update within 15 seconds… and if not, the Rust Playground will eventually inform you that your program ran too long!

However, we don’t have to rely on inconsistent behavior or long-running loops to detect UB. Rust includes a tool called **Miri**, a specialized interpreter that tracks memory, aliasing, and data races at runtime. If you enable **Miri** in the Playground (under Tools → Miri), it will analyze the program and immediately report the data race on `COUNTER`. Miri would also flag the use-after-free and buffer overflow examples we saw earlier.

Notice these examples are affecting variables *outside* the `unsafe` block. This highlights that the effects of UB aren’t confined to the `unsafe` block. Once triggered, UB can corrupt memory, break assumptions, and influence code far beyond where it started.

How much of the program is at risk when UB occurs? Is there *anything at all* we can consider safe from it?

## Conclusion

In this lesson, we’ve seen just how easy it is to reintroduce serious memory bugs using `unsafe` Rust — even in small, controlled programs.

Use-after-free, buffer overflows, and data races aren’t theoretical risks. They’re real bugs with real consequences. Worse, they often fail silently, making them difficult to detect — and dangerous to ignore. And they’re not alone — there are many other memory bugs out there. All of them share one critical property: they trigger **undefined behaviour**, and with it, a very real threat to reliability and security.

`unsafe` Rust is a powerful tool. It’s essential for systems programming, interfacing with hardware, or performance-critical code. But it puts the burden of memory safety entirely on you. When something goes wrong, the compiler can’t help.

That’s why Rust’s ownership and borrowing system exists: to make sure these classes of bugs **can’t happen** in safe code. In the next lesson, we’ll dig into how safe Rust prevents memory errors entirely — and what rules make that possible.