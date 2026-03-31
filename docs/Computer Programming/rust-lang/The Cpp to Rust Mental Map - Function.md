#  The "C++ to Rust" Mental Map - Functions


# Rust's OOP Alternatives: Structs, Traits, Enums

In Rust, while we don't have classes, we categorize logic based on how it relates to data structures. To understand this, you need to distinguish between code that stands alone and code that is "attached" to a type (like a `struct` or `enum`).

Here is the breakdown of **General Functions**, **Associated Functions**, and **Methods**.

---

## 1. General Function
A general function is a standalone block of code. It is not defined inside an `impl` block and does not belong to a specific type.

* **Definition:** A global or module-level function used for logic that doesn't inherently "belong" to a specific object.
* **Syntax:** Defined using the `fn` keyword outside of any `impl` block.
* **Scenario:** Mathematical utilities, helper functions for a whole module, or the `main()` function itself.

**Example:**
```rust
fn add_numbers(a: i32, b: i32) -> i32 {
    a + b
}
// Called directly: add_numbers(5, 10);
```

---

## 2. Associated Function
Associated functions are defined *inside* an `impl` block, but they do **not** take `self` as a parameter. They are "associated" with the type itself, not a specific instance of it.

* **Definition:** Functions that live within the namespace of a type. In other languages, these are often called **Static Methods**.
* **Syntax:** Defined inside an `impl` block without `self`. They are called using the "path" syntax `::`.
* **Scenario:** Constructors (like `new()`) or utility functions that create a type from raw data.



**Example:**
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // This is an Associated Function (Constructor)
    fn new(w: u32, h: u32) -> Rectangle {
        Rectangle { width: w, height: h }
    }
}
// Called via type: Rectangle::new(10, 20);
```

---

## 3. Method
Methods are also defined inside an `impl` block, but they **must** take `self` (or `&self` or `&mut self`) as their first parameter.

* **Definition:** Functions that act upon a specific instance of a type. They have access to the data stored in that specific object.
* **Syntax:** Defined inside `impl` with `self`. They are called using "dot notation" `.`.
* **Scenario:** Calculating properties of an object (like area), updating an object's state, or closing a connection.

**Example:**
```rust
impl Rectangle {
    // This is a Method
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
// Called via instance: my_rect.area();
```

---

## Comparison Table

| Feature | General Function | Associated Function | Method |
| :--- | :--- | :--- | :--- |
| **Location** | Anywhere (Global/Module) | Inside `impl` block | Inside `impl` block |
| **`self` param?** | No | No | **Yes** (`&self`, `&mut self`, or `self`) |
| **Call Syntax** | `function_name()` | `Type::function_name()` | `instance.method_name()` |
| **Common Use** | Logic independent of data | Constructors (`new`) | Operating on instance data |

### Summary of the "Mental Model"
Think of a **General Function** as a tool sitting on a workbench. An **Associated Function** is like a specialized tool kept inside a specific toolbox (the Type). A **Method** is a feature of the tool itself (like the "on" switch on a drill)—it only makes sense when you are actually holding an instance of that tool.

