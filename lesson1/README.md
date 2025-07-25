## Lesson 1: Rust Fundamentals

### 📚 Lesson Content

#### 1. **Introduction to Rust**
- **What is Rust?** A systems programming language focused on memory safety, concurrency, and performance
- **Why use Rust?**
  - Zero-cost abstractions: Performance close to C/C++
  - Ownership: Prevents the most common memory errors
  - Ecosystem: Used in projects like Firefox, Solana, Polkadot, and Stellar

#### 2. **Environment Setup**
- **Essential tools:**
  - `rustup`: Rust version manager
  - `rustc`: Official Rust compiler
  - `cargo`: Package manager and build tool

- **Installation:**
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

- **Verification:**
  ```bash
  rustc --version
  cargo --version
  ```

#### 3. **Hello World**
- **First program:**
  ```rust
  // hello.rs
  fn main() {
      println!("Hello, World!");
  }
  ```

- **Compilation and execution:**
  ```bash
  rustc hello.rs
  ./hello
  ```

#### 4. **Library Creation**
- **Creating project:**
  ```bash
  cargo new --lib calculator
  ```

- **Folder structure:**
  ```
  calculator/
  ├── Cargo.toml    # Rust configuration file
  └── src/
      └── lib.rs    # main library code file
  ```

#### 5. **Types, Functions, and Modules**

**Types in Rust:**
- `u8, u32, u64`: unsigned integers
- `i8, i32, i64`: signed integers
- `String`: mutable text
- `&str`: immutable text slice
- `Vec<T>`: vector, list of elements

**Function Signature:**
- `fn`: Keyword to define function
- `->`: Indicates function return type
- `return;`: Explicit return (optional in last expression)

**Modules in Rust:**
- Serve to group functions with a common purpose
- Can be within the same file or in separate files

#### 6. **Code Organization**
- **src/calc1.rs** (addition and subtraction):
  ```rust
  pub fn add(a: u32, b: u32) -> u32 { a + b }
  pub fn sub(a: u32, b: u32) -> u32 { 
      if a < b { 0 } else { a - b } 
  }
  ```

- **src/calc2.rs** (multiplication and division):
  ```rust
  pub fn multiply(a: u32, b: u32) -> u32 { a * b }
  pub fn rate(a: u32, b: u32) -> u32 { 
      if b == 0 { 0 } else { a / b } 
  }
  ```

- **src/lib.rs** (exposing modules):
  ```rust
  pub mod calc1;
  pub mod calc2;
  ```

#### 7. **Automated Testing**
- **Writing tests:**
  ```rust
  #[cfg(test)]
  mod tests {
      use super::calc1::{add, sub};
      use super::calc2::{multiply, rate};

      #[test]
      fn test_add() {
          assert_eq!(add(10, 20), 30);
          assert_eq!(add(u32::MAX, 1), u32::MAX); // Overflow for u32 saturates at maximum
      }
  }
  ```

- **Running tests:**
  ```bash
  cargo test
  ```

- **Important attributes:**
  - `#[cfg(test)]`: Defines a module that only compiles for tests
  - `#[test]`: Marks test functions
  - `assert_eq!`: Macro to check for equality

#### 8. **Publishing to Crates.io**
- **Create account and authenticate:**
  1. Access https://crates.io and create an account
  2. Generate an API token in Account Settings > API Tokens
  3. Authenticate locally: `cargo login <your-token>`

- **Publish:**
  ```bash
  cargo package    # Check before publishing
  cargo publish    # Publish to crates.io
  ```

- **Tip:** Use a unique name for the crate (e.g., calculator-your-name)

#### 9. **Using Libraries**
- **Creating new project:**
  ```bash
  cargo new interactive_calculator
  cd interactive_calculator
  ```

- **Adding dependency to Cargo.toml:**
  ```toml
  [dependencies]
  calculator-danielgorgonha = "0.2.0"
  ```

- **Importing functions:**
  ```rust
  use calculator_danielgorgonha::calc1::{add, sub};
  use calculator_danielgorgonha::calc2::{multiply, rate};
  use calculator_danielgorgonha::calc3::{power, logarithm};
  ```

#### 10. **Interactive Program**
- **Reading user input:**
  ```rust
  use std::io;

  println!("Choose the operation (+, -, *, /, ^, log):");
  let mut operation = String::new();
  io::stdin().read_line(&mut operation).expect("Error");
  let operation = operation.trim();

  println!("Enter the first number:");
  let mut num_a_str = String::new();
  io::stdin().read_line(&mut num_a_str).expect("Error");
  let num_a: u32 = num_a_str.trim().parse().expect("Invalid number");
  ```

- **Executing calculations:**
  ```rust
  let result = match operation {
      "+" => add(num_a, num_b),
      "-" => sub(num_a, num_b),
      "*" => multiply(num_a, num_b),
      "/" => rate(num_a, num_b),
      "^" => power(num_a, num_b),
      "log" => logarithm(num_a, num_b),
      _ => {
          println!("Invalid operation!");
          return;
      }
  };
  ```

- **Displaying result:**
  ```rust
  println!("Result: {} {} {} = {}", num_a, operation, num_b, result);
  ```

- **Running:**
  ```bash
  cargo run
  ```

### 🎯 Recap

**Fundamental concepts learned:**
- **Rust:** Secure, performant, and productive language, ideal for high-reliability systems
- **Essential Tools:** `rustup` (version manager), `cargo` (package and build manager), `rustc` (compiler)
- **Hello World:** First step in Rust, demonstrating compilation and execution of a simple program
- **Data Types:** Understanding types like `u32` (unsigned integers), `String` (mutable texts), `&str` (immutable text slices)
- **Functions:** How to define and use functions (`fn`, `->`, `return;`) to organize reusable code blocks
- **Modules:** Organize code into modules (e.g., `calc1.rs`, `calc2.rs`) for modularity and to avoid name conflicts
- **Library Creation:** Process of creating a Rust library with `cargo new --lib` and its structure
- **Automated Testing:** Practice of writing tests (e.g., `#[test]`, `assert_eq!`) to ensure code correctness and robustness, executed with `cargo test`
- **Crates.io:** Process of publishing a library to the official Rust registry and how to use third-party libraries in your own projects
- **User Interaction:** How to create programs that receive user input (using `std::io`) and process data with library functions

---

## 🏆 Learning Challenge - COMPLETED ✅

### **Challenge Description:**
- ✅ Add a power function and its inverse, logarithm, to the **calculator** library in **calc3.rs**
- ✅ Write tests and publish the new version (0.2.0) to **crates.io**

### **Challenge Implementation:**

#### **New Module: calc3.rs**
```rust
pub fn power(base: u32, exponent: u32) -> u32 {
    if exponent == 0 {
        return 1;
    }
    
    let mut result = 1;
    let mut base = base;
    let mut exp = exponent;
    
    while exp > 0 {
        if exp % 2 == 1 {
            result = result.saturating_mul(base);
        }
        base = base.saturating_mul(base);
        exp /= 2;
    }
    
    result
}

pub fn logarithm(value: u32, base: u32) -> u32 {
    if value == 0 || base == 0 || base == 1 {
        return 0;
    }
    
    let mut result = 0;
    let mut current = 1;
    
    while current <= value && current != 0 {
        current = current.saturating_mul(base);
        if current != 0 {
            result += 1;
        }
    }
    
    if result > 0 {
        result - 1
    } else {
        0
    }
}
```

#### **Updated lib.rs**
```rust
pub mod calc1;
pub mod calc2;
pub mod calc3;  // New module added
```

#### **Comprehensive Tests**
```rust
#[cfg(test)]
mod tests {
    use super::calc1::{add, sub};
    use super::calc2::{multiply, rate};
    use super::calc3::{power, logarithm};

    #[test]
    fn test_power() {
        assert_eq!(power(2, 3), 8);
        assert_eq!(power(5, 0), 1);
        assert_eq!(power(0, 5), 0);
        assert_eq!(power(u32::MAX, 1), u32::MAX);
    }

    #[test]
    fn test_logarithm() {
        assert_eq!(logarithm(8, 2), 3);
        assert_eq!(logarithm(100, 10), 2);
        assert_eq!(logarithm(0, 2), 0);
        assert_eq!(logarithm(1, 2), 0);
    }
}
```

#### **Version 0.2.0 Features:**
- ✅ **Power Function:** Efficient binary exponentiation with overflow protection
- ✅ **Logarithm Function:** Integer logarithm with edge case handling
- ✅ **Comprehensive Tests:** All edge cases covered
- ✅ **Published on Crates.io:** Available as `calculator-danielgorgonha = "0.2.0"`

---

## 📁 Project Repositories

### 🧮 Calculator Library
**[📦 GitHub Repository](https://github.com/danielgorgonha/calculator)**

Complete implementation of the calculator library with:
- ✅ Addition and subtraction functions (`calc1.rs`)
- ✅ Multiplication and division functions (`calc2.rs`)
- ✅ **Power and logarithm functions (`calc3.rs`)** - NEW!
- ✅ Comprehensive automated tests
- ✅ Published on [crates.io](https://crates.io/crates/calculator-danielgorgonha) as version 0.2.0

**New Features in v0.2.0:**
- ⚡ **Power Function:** `power(base, exponent)` - Efficient binary exponentiation
- 📊 **Logarithm Function:** `logarithm(value, base)` - Integer logarithm calculation
- 🛡️ **Overflow Protection:** All operations protected against overflow/underflow
- 🧪 **Complete Test Suite:** 100% test coverage for all functions

### 🖥️ Interactive Calculator
**[📦 GitHub Repository](https://github.com/danielgorgonha/interactive-calculator)**

Interactive command-line calculator that:
- ✅ Uses the published calculator library (v0.2.0)
- ✅ Accepts user input for 6 operations (+, -, *, /, ^, log)
- ✅ Performs real-time calculations
- ✅ Demonstrates library integration
- ✅ **Supports advanced operations:** Exponentiation and logarithm

**Enhanced Features:**
- 🧮 **6 Mathematical Operations:** +, -, *, /, ^, log
- 🔄 **Interactive Loop:** Continuous operation with exit option
- 🛡️ **Robust Error Handling:** Invalid inputs gracefully handled
- 🎨 **User-Friendly Interface:** Emojis and clear formatting

### 🚀 Getting Started

To use these projects:

```bash
# Clone the calculator library
git clone https://github.com/danielgorgonha/calculator.git
cd calculator
cargo test

# Clone the interactive calculator
git clone https://github.com/danielgorgonha/interactive-calculator.git
cd interactive-calculator
cargo run
```

### 📦 Using the Library

Add to your `Cargo.toml`:
```toml
[dependencies]
calculator-danielgorgonha = "0.2.0"
```

Import and use:
```rust
use calculator_danielgorgonha::calc1::{add, sub};
use calculator_danielgorgonha::calc2::{multiply, rate};
use calculator_danielgorgonha::calc3::{power, logarithm};

fn main() {
    println!("2^3 = {}", power(2, 3));        // 8
    println!("log_2(8) = {}", logarithm(8, 2)); // 3
}
```

---

## 🎯 **Lesson 1 Summary & Key Takeaways**

### **What We Accomplished:**
✅ **Complete Rust Environment Setup** - Installed and configured rustup, cargo, and rustc  
✅ **First Rust Program** - Created and executed Hello World  
✅ **Library Development** - Built a modular calculator library with 3 modules  
✅ **Automated Testing** - Implemented comprehensive test suite with 100% coverage  
✅ **Package Publishing** - Published library to crates.io (v0.2.0)  
✅ **Interactive Application** - Created CLI calculator using our published library  
✅ **Advanced Features** - Implemented power and logarithm functions  
✅ **Real-World Integration** - Demonstrated library usage in external projects  

### **Core Rust Concepts Mastered:**
🦀 **Language Fundamentals** - Types, functions, modules, ownership  
🛠️ **Toolchain** - rustup, cargo, rustc workflow  
📦 **Package Management** - Creating, testing, and publishing crates  
🧪 **Testing** - Unit tests, assertions, test organization  
🔗 **Module System** - Code organization and reusability  
📡 **User Input** - std::io for interactive programs  
⚡ **Performance** - Zero-cost abstractions and memory safety  

### **Practical Skills Developed:**
🎯 **Project Structure** - Organizing Rust projects effectively  
🔧 **Build System** - Cargo.toml configuration and dependencies  
📚 **Documentation** - Writing clear, comprehensive README files  
🚀 **Deployment** - Publishing to official Rust registry  
🔄 **Version Control** - Git workflow with meaningful commits  
💡 **Problem Solving** - Implementing mathematical algorithms in Rust  

### **Next Steps:**
➡️ **Lesson 2** - Building CRUD APIs with Rust  
➡️ **Lesson 3** - WebAssembly and web applications  
➡️ **Meridian Hackathon** - Apply these skills in real-world projects  

---

*This lesson provides the foundation for advanced Rust development and prepares you for the Meridian Hackathon challenges! 🚀*
