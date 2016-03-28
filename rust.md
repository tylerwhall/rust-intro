# Rust

High level language features including memory safety without compromising performance
- Systems language
- Generates native code
- Usable in place of C/C++
- Minimal runtime
    - No garbage collector



# Mozilla/Servo



# Features
## Easy interfacing with C
Call C libraries (or C++ with C ABI) from Rust
Build Rust libraries with C interface

```
int main() {}
```


## Type inference


## Functional language features


## Pattern matching

Match expressions

Switch/Case on steroids

Implements a map in a single expression


## Expression-based

```
let x = 5;

let y = if x == 5 {
    10
} else {
    15
};
// y = 10
```
or
```
let x = 5;
let y = if x == 5 { 10 } else { 15 };
```
https://doc.rust-lang.org/book/if.html

Note:
Value of a block is the value of the last expression in the block. Final semicolon is implied.


## Closures


## Safe threading built in
C/C++ threading is an afterthought

Rust: borrow checker checks thread creation

## Memory safety
Immutable variables by default
Ownership

# Unsafe code

# Availability
- Rust 1.0 Launched in May 2015
- Current version 1.7
- Backward compatible
- Available for use in Yocto now!


References:
http://www.steveklabnik.com/nobody_knows_rust/
