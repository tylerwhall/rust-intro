# Rust

Modern, high-level language features and memory safety without compromising
performance


## About
*Rust is a systems programming language that runs blazingly fast, prevents nearly all segfaults, and guarantees thread safety*
- Not object-oriented, but has some OO features
- Some functional aspects
- Strongly typed (lots of inference for convenience)
- Ready to use now!


## Systems Language
- Aims to compete with C/C++
- Compiles to native code
    - As a native Linux application
    - As a [kernel module](https://github.com/tsgates/rust.ko)
    - Even on bare metal


## Speed
- No runtime
- No garbage collector required or by default
- Uses LLVM compiler framework
- Zero-cost abstractions



# History
- 2006: Graydon Hoare's part-time side project
- 2009: Supported by Mozilla
- 2015: Version 1.0 released (backward compatibility)



# Servo

Mozilla's new browser engine designed to be appropriate for applications including embedded use.

Aims to achieve better parallelism, security, modularity, and performance.

Developed alongside the language.

https://servo.org/



# Why Rust?

- Safety and expressiveness in systems programming


# Safety vs. Control

Control |      | Safety
---        | ---   | ---
C   | Go        | Python
C++ | Java      | JavaScript
    | C#        | Haskell
    | Swift     |
Rust|           | Rust

Note:
Swift uses reference counts transparently (less control) but doesn't prevent data races between threads (less safety), though it is easy to use. Both are compiled with LLVM.


## What is memory safety?

Data race freedom

What causes a data race?
- Sharability + mutability, or
- Aliasing + mutation

Aliasing: more than one pointer to the same memory

https://doc.rust-lang.org/book/ownership.html
https://youtu.be/d1uraoHM8Gg?t=462


## Data Race (C++ example)

```c
vector<int> v = {1, 2, 3};

auto &elem = v[0];

v.push_back(4);

cout << "v[0]: " << v[0] << endl;
cout << "elem: " << elem << endl;
```

Output:

```
v[0]: 1
elem: 425319272
```

This code compiles with no warnings!

GCC 5.3.0 -Wall -Wextra

Note:
C++ compiler can warn about specific cases, but Rust prevents on a more fundamental level


## Data Race (Rust version)

```rust
let mut v = vec![1, 2, 3];

let elem = &v[0];

v.push(4);

println!("v[0] is: {}", v[0]);
println!("elem is: {}", elem);
```

Output:
```
<anon>:7:1: 7:2 error: cannot borrow `v` as mutable because it is also borrowed as immutable [E0502]
<anon>:7 v.push(4);
         ^
<anon>:5:13: 5:14 note: previous borrow of `v` occurs here; the immutable borrow prevents subsequent moves or mutable borrows of `v` until the borrow ends
<anon>:5 let elem = &v[0];
```

https://doc.rust-lang.org/book/ownership.html


## Memory safety
- Automatic memory management through "ownership"
    - Values can be moved, borrowed or copied
    - One writer **or** multiple readers


## Memory safety
- No direct pointer manipulation
    - No double-free, invalid free
    - Bounds checking


- No direct pointer manipulation
    - No null pointers
        - Option type compiles down to a nullable pointer (zero-cost abstraction)

```rust
fn print_string(s: Option<&str>) {
    match s {
        Some(s) => println!("{}", s),
        None => println!("Null string"),
    }
}
let s = "Valid string";
let mut nullablestring = Some(s);
print_string(nullablestring);
nullablestring = None;
print_string(nullablestring);
```


## Memory safety
- Retains fine grained control over allocation
    - stack, heap
    - Single owner, RC, GC, custom
- RAII enforced
    - No uninitialized memory allowed
    - Destructors
    - Helps prevent memory leaks



## Error handling
How to handle allocation failure?

1. Check return, handle gracefully
2. Explicitly abort program
3. Ignore errors (Undefined behavior)

**Undefined behavior not allowed in Rust**


## All errors must be handled

Used commonly for I/O operations that can fail.
```rust
use std::fs::File;
use std::io::prelude::*;

let mut file = File::create("valuable_data.txt").unwrap();
file.write_all(b"important message").expect("failed to write message");
```

https://doc.rust-lang.org/std/result/

https://doc.rust-lang.org/std/macro.try!.html

Note:
Remove expect;
Remove unwap;
Write to root /;
File automatically closed



## Safe threading built in
- C/C++
    - Concurrency is an afterthought
    - Varying support in standard libraries (platform-specific or C++11)
    - No way to ensure threading primitives are used correctly
- Rust
    - Concurrency support in standard library
    - Language ensures no data races
    - Borrow checker checks values passed at thread creation and over channels between threads
https://doc.rust-lang.org/nomicon/send-and-sync.html


## Thread safety

```rust
use std::thread;
use std::time::Duration;

let mut data = vec![1, 2, 3];

for i in 0..3 {
    thread::spawn(move || {
        data[i] += 1;
        //error: capture of moved value: `data`
    });
}

thread::sleep(Duration::from_millis(50));
```


## Thread safety

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

let data = Arc::new(Mutex::new(vec![1, 2, 3]));

for i in 0..3 {
    let data = data.clone();
    thread::spawn(move || {
        let mut data = data.lock().unwrap();
        data[i] += 1;
    });
}

thread::sleep(Duration::from_millis(50));
```


Immutable variables by default
```rust
let x = 1;
x = 2; // error: re-assignment of immutable variable `x`
```
```rust
let mut x = 1;
x = 2; // Ok
```



## Unsafe code
- Interfacing with C
- Bypass the borrow checker
- Work with pointers



# Modern Features


## Type inference

```rust
// Because of the annotation, the compiler knows that `elem` has type u8.
let elem = 5u8;

// Create an empty vector (a growable array).
let mut vec = Vec::new();
// At this point the compiler doesn't know the exact type of `vec`, it
// just knows that it's a vector of something (`Vec<_>`).

// Insert `elem` in the vector.
vec.push(elem);
// Aha! Now the compiler knows that `vec` is a vector of `u8`s (`Vec<u8>`)
// TODO ^ Try commenting out the `vec.push(elem)` line

println!("{:?}", vec);
```

http://rustbyexample.com/cast/inference.html


## Expression-based

```rust 
let x = 5;

let y = if x == 5 {
    10
} else {
    15
};
// y = 10
```
or
```rust
let x = 5;
let y = if x == 5 { 10 } else { 15 };
```
https://doc.rust-lang.org/book/if.html

Note:
Value of a block is the value of the last expression in the block. Final semicolon is implied.


## Tuples

```rust
//tuples can be destructured
let tuple = (1, "hello", 4.5, true);
println!("{:?}, {:?}, {:?}, {:?}", tuple.0, tuple.1, tuple.2, tuple.3);

let (a, b, c, d) = tuple;
println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);
```

http://rustbyexample.com/primitives/tuples.html

Note:
Tuples behave like in other languages like Python.
Effectively anonymous structs.
Allow functions to easily return multiple values.



## Anonymous Functions / Closures

```rust
let plus_one = |x: i32| x + 1;

assert_eq!(2, plus_one(1));
```


## Closures
- Capture environment

```rust
let x = 1;
let add_to_x = |y| x + y;

assert_eq!(2, add_to_x(1));
```


## Closures
- Subject to the borrow checker

```rust
let mut x = 1;
let add_to_x = |y| x + y;

assert_eq!(2, add_to_x(1));

x = x + 1;
//error: cannot assign to `x` because it is borrowed
```



## Enums

- Represents one of several variants
- Can have associated data

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}
```
https://doc.rust-lang.org/book/enums.html


Enums are like tagged unions in C

```c
struct Message {
    enum {
        QUIT,
        CHANGE_COLOR,
        MOVE,
        WRITE,
    } type;
    union {
        struct { int color[3]; } change_color;
        struct { int x, y; } move ;
        struct {
            char *string; // Who owns this memory?
        } write ;
    } val;
};
```

Note:
No one writes code like this in C.
Too tedious.
Can't always be optimized.
Nothing binds the enum value to the union element.


## Pattern matching

```rust
let x = 5;

let number = match x {
    1 => "one",
    2 => "two",
    3 | 4 => "three or four",
    5...10 => "five to ten",
    _ => "something else",
};
```
https://doc.rust-lang.org/book/patterns.html

Note:
Switch/Case on steroids.
Implements a map in a single expression


## Pattern matching with enums

```rust
enum Character {
    Number(i32),
    Letter(char),
    Symbol(char),
}
let x = Character::Number(5);

match x {
    Character::Number(i) =>
        println!("Do something with number {}", i),
    _ => (),
}

if let Character::Number(i) = x {
    println!("Do something with number {}", i)
}

```
https://doc.rust-lang.org/book/patterns.html

Note:
If-let can be used if you only care about one case.



## Iterator Adapters
- Chain multiple operations on a collection
- Lazy evaluation
```rust
let v = [1, 2, 3, 4, 5];
v.iter().map(|x| println!("{}", x));
// No output...
```
```rust
let v = 1..6;
println!("{:?} = {:?}", v, v.clone().collect::<Vec<_>>());
let w: Vec<_> = v.map(|x| 2 * x)
                    .take(3)
                    .collect();
println!("{:?}", w);
```

https://doc.rust-lang.org/std/iter/

Note:
Functional language feature.
Exist in other languages but Rust prevents iterator invalidation.
Range expression is an iterator.
Because collect() is generic, type can be specified at the call site or inferred from assignment and later usage.



## Traits
- Both static and dynamic dispatch supported

```rust
trait HasArea {
    fn area(&self) -> f64;
}

struct Square {
    width: f64,
}

impl HasArea for Square {
    fn area(&self) -> f64 {
        self.width * self.width
    }
}

struct Rectangle {
    width: f64,
    length: f64,
}

impl HasArea for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.length
    }
}

fn print_area<T: HasArea + ?Sized>(shape: &T) {
    println!("Shape area: {}", shape.area());
}

let square = Square { width: 10f64 };
let rect = Rectangle { width: 10f64, length: 5f64 };
print_area(&square);
print_area(&rect);
print_area(&square as &HasArea);
print_area(&rect as &HasArea);
```
https://doc.rust-lang.org/book/traits.html

Note:
Traits are somewhat like interfaces in Java.
Used with generics like in C++ or Java.
In the assembly, you can see 3 instances of print_area are created. Two static and one dynamic.



## Built-in unit testing

```
fn hello() -> &'static str {
    "Hello"
}

#[test]
fn positive_test() {
    assert_eq!(hello(), "Hello");
}

#[test]
#[should_panic(expected = "assertion failed")]
fn negative_test() {
    assert_eq!("Hello", "world");
}
```

https://doc.rust-lang.org/book/testing.html


## Built-in documentation

Source code examples in documentation run as unit tests!
```
    /// Creates a new `BufReader` with a default buffer capacity.
    ///  
    /// # Examples
    /// 
    /// ` ``
    /// use std::io::BufReader;
    /// use std::fs::File;
    ///  
    /// # fn foo() -> std::io::Result<()> {
    /// let mut f = try!(File::open("log.txt"));
    /// let mut reader = BufReader::new(f);
    /// # Ok(())
    /// # }
    /// ` ``
    pub fn new(inner: R) -> BufReader<R> {
        BufReader::with_capacity(DEFAULT_BUF_SIZE, inner)
    }    
```

https://doc.rust-lang.org/std/io/struct.BufReader.html#method.new


## Easy interfacing with C
Call C libraries (or C++ with C ABI) from Rust

```
extern crate libc;
use libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```
https://doc.rust-lang.org/book/ffi.html


Build Rust libraries with C interface

Rust

```
#[no_mangle]
pub extern fn hello_rust() -> *const u8 {
    "Hello, world!\0".as_ptr()
}
```

C

```c
#include <stdio.h>
extern const char *hello_rust(void);
int main(void)
{
    puts(hello_rust());
}

```

Note:
rustc -C prefer-dynamic --crate-type=dylib lib.rs



# Availability
- Rust 1.0 Launched in May 2015
- Current stable version 1.7
- Backward compatible since 1.0
- Available for use in Yocto now!



## How to use in Yocto

```
inherit rust-bin

do_compile () {
	oe_compile_rust_lib
}

do_install () {
	oe_install_rust_lib
}
```
https://github.com/jmesmon/meta-rust/blob/master/recipes-core/udev/libudev-rs_0.1.2.bb

https://github.com/jmesmon/meta-rust



References:

https://doc.rust-lang.org

https://www.rust-lang.org/faq.html

http://www.steveklabnik.com/nobody_knows_rust/



# Backup


# Speed

## Stack allocations preferred

- The stack is fast!
- Borrow checker allows maximal usage of stack, safely
- All structures have a known size at compile time
- Containers available for dynamic allocations
https://doc.rust-lang.org/std/boxed/


## Hygenic macros

https://doc.rust-lang.org/book/macros.html#hygiene
