# Rust Structs: Creation and Destruction

When an associated function (like `new`) returns a struct in Rust, it behaves quite differently than a C++ constructor or a Java `new` call. In those languages, you are often dealing with pointers or heap-allocated references. In Rust, you are typically moving a **value** directly.

Here is the breakdown of what is happening under the hood.

---

## 1. The Value is Created on the Stack
When you call `Player::new()`, the function allocates space for the struct on its own **stack frame**. Once the function finishes, it returns that block of data to the caller.

In modern Rust, thanks to **Return Value Optimization (RVO)**, the compiler is smart enough to construct the object directly in the memory slot of the calling function. This means there is often zero "copying" overhead; the data just exists where it needs to be.

### C++ vs. Rust Memory Layout
* **C++ `new`:** Returns a pointer to memory on the **Heap**. You must manage its life (or use a smart pointer).
* **Rust `new()`:** Returns the **Value** on the **Stack**. The caller now "owns" that memory.

---

## 2. Moving Ownership (The "Hand-off")
In C++, returning an object by value might trigger a "copy constructor." In Rust, it triggers a **Move**.

Because Rust tracks ownership, when `new` returns the object, it isn't "copied" in the expensive sense. Instead, the responsibility for that memory is handed over to the variable in the main scope.



```rust
fn main() {
    // 1. Space is reserved on main's stack
    // 2. Player::new() fills that space
    let p1 = Player::new("Aris"); 
    
    // 3. If we do this, p1 is "moved" to p2. 
    // p1 is now invalid; no data was duplicated.
    let p2 = p1; 
}
```



---

## Summary 
When an associated function returns a stack object:

1.  **Locality:** The data is stored close to the CPU (Stack), which is extremely fast.
2.  **Deterministic Cleanup:** You know exactly when the memory will be freed (at the end of the `}` brace).
3.  **No Nulls:** Unlike a C++ constructor that might return a null pointer if it fails (or throw an exception), a Rust function returning `Self` **guarantees** a valid, initialized object.
4.  **Ownership:** The function "gives away" the object. Once it returns, the function's scope is gone, and the caller is the new boss of that data.

