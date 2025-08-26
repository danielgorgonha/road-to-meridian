# Lesson 3: How to Transform Rust Code into WebAssembly

**Content:** WebAssembly application development with Rust, integrating WASM modules into CRUD APIs

**Main topics:**
- ⚡ Introduction to WebAssembly (WASM, WASI, WAT)
- 🔧 Rust to WASM compilation and optimization
- 🌐 WebAssembly runtimes (Wasmi, Wasmtime, Wasmer)
- 🚀 Integration with CRUD APIs (CRUDE - Create, Read, Update, Delete, Execute)
- 🎯 Practical applications and blockchain use cases

---

## 🎯 **What is WebAssembly?**

### **WebAssembly Overview**
WebAssembly (WASM) is a portable, secure runtime environment designed to be host-agnostic and secure by default. It's not just assembly, nor is it only for the web.

### **Key Components:**
- **WASM**: Binary format and abstract virtual machine
- **WASI**: System interface for accessing host resources
- **WAT**: Text format for human-readable WASM
- **Runtimes**: Wasmi, Wasmtime, Wasmer for execution

### **Why Blockchains Adopt WASM:**
- **Performance**: 10-100x faster than EVM (Ethereum Virtual Machine)
- **Multiple Languages**: Support for Rust, C++, Go (not just Solidity)
- **Determinism**: Same code produces same results in any environment
- **Enhanced Security**: Sandboxed environment reduces attack surface
- **Interoperability**: Facilitates communication between blockchains

---

## 🛠️ **Creating WASM Functions in Rust**

### **Project Setup**
```bash
cargo new --lib wasm-math
cd wasm-math
```

### **Cargo.toml Configuration**
```toml
[package]
name = "math"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[profile.release]
lto = true
codegen-units = 1
opt-level = "z"

[dependencies]
```

### **Rust Functions (`src/lib.rs`)**
```rust
#[no_mangle]
pub extern "C" fn add(x: i32, y: i32) -> i32 {
    x + y
}

#[no_mangle]
pub extern "C" fn mul(x: i32, y: i32) -> i32 {
    x * y
}

#[no_mangle]
pub extern "C" fn sub(x: i32, y: i32) -> i32 {
    if x < y {
        return 0;
    }
    x - y
}

#[no_mangle]
pub extern "C" fn div(x: i32, y: i32) -> i32 {
    if y == 0 {
        return 0;
    }
    x / y
}
```

### **Key Concepts:**
- **`extern "C"`**: Defines C ABI calling convention for compatibility
- **`#[no_mangle]`**: Prevents function name mangling in the binary
- **Function signature**: `(u32, u32) -> u32` for mathematical operations

---

## ⚡ **Compiling to WebAssembly**

### **Install WASM Target**
```bash
rustup target add wasm32-unknown-unknown
```

### **Compile the Project**
```bash
cargo build --target wasm32-unknown-unknown --release
```

### **Convert WASM to Bytes**
```bash
od -An -v -t uC *.wasm \
| tr -s ' ' \
| tr ' ' ',' \
| tr -d '\n' \
| sed 's/^,//;s/,$//g' > BYTES_RESULT.txt
```

**Output:** `target/wasm32-unknown-unknown/release/math.wasm`

---

## 🌐 **WebAssembly Runtimes**

### **Wasmi**
- **Description**: Pure Rust WASM interpreter, embeddable and lightweight
- **Integration**: Perfect for embedding WASM execution within Rust applications
- **Ideal Use**: Smart contracts, blockchain runtimes, APIs with third-party plugins

### **Wasmtime**
- **Description**: Performance-focused WASM runtime with WASI compliance
- **Integration**: Exposes bindings for multiple languages and powerful CLI
- **Ideal Use**: Testing, command line, isolated WASM execution with system access

### **Wasmer**
- **Description**: WASM runtime focused on portability and virtualization
- **Integration**: Multiple backends (LLVM, Cranelift, Singlepass)
- **Ideal Use**: Universal binaries, edge servers, cross-platform plugins

---

## 🔧 **Integrating WASM with CRUD API**

### **Dependencies (`Cargo.toml`)**
```toml
[dependencies]
tide = "0.16.0"
async-std = { version = "1.12.0", features = ["attributes"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
wasmi = "0.47.0"
```

### **Updated Data Model (`src/models.rs`)**
```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct DataEntry {
    pub func_names: Vec<String>,
    pub bytecode: Vec<u8>,
}
```

### **Execute Route (`src/handlers/execute.rs`)**
```rust
use tide::{Request, Response, StatusCode};
use crate::state::AppState;
use serde::Deserialize;
use serde_json::json;
use wasmi::{Engine, Module, Store, Instance, TypedFunc};

#[derive(Deserialize)]
struct ExecRequest {
    #[serde(rename = "fn")]
    func: String,
    arg: [i32; 2],
}

pub async fn execute_fn(mut req: Request<AppState>) -> tide::Result {
    // Parse request body
    let exec_req: ExecRequest = req.body_json().await.map_err(|_| {
        tide::Error::from_str(400, "Invalid JSON: expected { fn: string, arg: [i32; 2] }")
    })?;

    // Extract ID and find WASM module
    let id: u32 = match req.param("id") {
        Ok(s) => s.parse().map_err(|_| tide::Error::from_str(400, "Invalid id"))?,
        Err(_) => return Err(tide::Error::from_str(400, "Missing id")),
    };
    
    let map = req.state().lock().unwrap();
    let entry = match map.get(&id) {
        Some(e) => e,
        None => return Err(tide::Error::from_str(404, "Not found")),
    };
    let wasm_bytes = &entry.bytecode;

    // Load and instantiate WASM module
    let engine = Engine::default();
    let module = Module::new(&engine, wasm_bytes)
        .map_err(|e| tide::Error::from_str(StatusCode::BadRequest, format!("Invalid wasm: {e}")))?;
    let mut store = Store::new(&engine, ());
    let instance = Instance::new(&mut store, &module, &[]).map_err(|e| {
        tide::Error::from_str(StatusCode::InternalServerError, format!("Wasm instantiation error: {e}"))
    })?;

    // Resolve and execute function
    let func = instance.get_func(&mut store, &exec_req.func).ok_or_else(|| {
        tide::Error::from_str(StatusCode::BadRequest, format!("Function not found: {}", exec_req.func))
    })?;

    let typed: TypedFunc<(i32, i32), i32> = func.typed(&store).map_err(|e| {
        tide::Error::from_str(StatusCode::BadRequest, format!("Signature error: {e}"))
    })?;

    let result = typed.call(&mut store, (exec_req.arg[0], exec_req.arg[1])).map_err(|e| {
        tide::Error::from_str(StatusCode::InternalServerError, format!("Call error: {e}"))
    })?;

    // Return result
    Ok(Response::builder(StatusCode::Ok)
        .body(serde_json::to_string(&json!({ "result": result }))?)
        .content_type(tide::http::mime::JSON)
        .build())
}
```

---

## 🧪 **Testing the CRUDE API**

### **Step 1: Save WASM Module**
```bash
curl -s -X POST http://127.0.0.1:8080/data \
  -H 'Content-Type: application/json' \
  -d '{"func_names": ["sum", "add", "mul", "div"], "bytecode": [BYTE_CODE]}'
```

### **Step 2: Execute Functions**
```bash
# Test addition
export ID=1
curl -s -X POST http://127.0.0.1:8080/execute/$ID \
  -H "Content-Type: application/json" \
  -d '{"fn": "add", "arg": [1, 2]}'

# Test multiplication
curl -s -X POST http://127.0.0.1:8080/execute/$ID \
  -H "Content-Type: application/json" \
  -d '{"fn": "mul", "arg": [1, 2]}'
```

---

## 📦 **Project Repositories**

### **🚀 Advanced CRUD + WASM Implementation**
**[github.com/danielgorgonha/learn-rust-crud](https://github.com/danielgorgonha/learn-rust-crud)**

A **production-ready, enterprise-level** implementation that expands the basic Aula 3 concepts with senior developer practices.

#### **🔥 Key Features:**
- ✅ **Complete Authentication System** - JWT + Refresh tokens + Owner-only access
- ✅ **Enhanced WASM Functions** - 9 mathematical functions with validation
- ✅ **Performance Optimizations** - Module caching + Rate limiting + Metrics
- ✅ **Production Deployment** - Live API on Railway platform
- ✅ **Comprehensive Testing** - Automated shell test suite
- ✅ **API Documentation** - Complete Postman collection
- ✅ **Security Features** - Input validation + Error handling + Logging

#### **🚀 Getting Started:**
```bash
# Clone the repository
git clone https://github.com/danielgorgonha/learn-rust-crud.git
cd learn-rust-crud

# Set up environment
cp env.example .env
# Configure your environment variables

# Run the application
cargo run

# Test the API
./test/0_login.sh      # Authentication
./test/1_create.sh     # Create operations
./test/2_read_all.sh   # Read operations
./test/3_read_one.sh   # Read specific
./test/4_update.sh     # Update operations
./test/5_delete.sh     # Delete operations
```

#### **🌐 Live Demo:**
- **Production API**: Deployed and monitored
- **Postman Collection**: Complete API documentation
- **Automated Tests**: Shell scripts for CI/CD
- **WASM Functions**: 9 mathematical operations ready to execute

#### **📊 WASM Functions Available:**
```rust
add(x: i32, y: i32) -> i32      // Addition
mul(x: i32, y: i32) -> i32      // Multiplication
sub(x: i32, y: i32) -> i32      // Subtraction
div(x: i32, y: i32) -> i32      // Division
rem(x: i32, y: i32) -> i32      // Remainder
abs(x: i32) -> i32              // Absolute value
max(x: i32, y: i32) -> i32      // Maximum
min(x: i32, y: i32) -> i32      // Minimum
pow(x: i32, y: i32) -> i32      // Power
```

#### **🏗️ Architecture Highlights:**
- **Modular Design**: Clear separation of concerns
- **Security First**: Authentication and authorization
- **Performance Focus**: Caching and monitoring
- **Testing Strategy**: Comprehensive automated tests
- **Documentation**: Complete and professional
- **Production Ready**: Deployed and monitored

---

## 🧠 **Memory Management in Rust for WASM**

### **WASM Memory Model**
WebAssembly uses a linear memory model that's separate from the host system's memory. This provides:

- **Isolation**: WASM modules can't directly access host memory
- **Security**: Sandboxed execution environment
- **Portability**: Same memory layout across different platforms

### **Rust Ownership in WASM Context**
```rust
// Functions are exported with C ABI
#[no_mangle]
pub extern "C" fn add(x: i32, y: i32) -> i32 {
    // No ownership transfer - values are copied
    x + y  // Return value is copied back to caller
}

// No dynamic memory allocation in simple functions
// WASM modules can have their own memory, but we're using simple types
```

### **Why This Matters for Our API**
In our CRUDE API, we load WASM bytecode into memory and execute functions dynamically. The ownership system ensures:
- **Safe execution**: Functions can't access unauthorized memory
- **Deterministic results**: Same inputs always produce same outputs
- **Resource management**: Memory is properly managed during WASM execution

---

## 🎯 **Lesson 3 Summary & Key Takeaways**

### **What We Accomplished:**
✅ **WebAssembly Fundamentals** - Understood WASM, WASI, WAT, and runtimes  
✅ **Rust to WASM Compilation** - Created and compiled Rust functions to WASM  
✅ **WASM Integration** - Integrated WASM modules into CRUD API  
✅ **Dynamic Execution** - Built CRUDE API with execute functionality  
✅ **Runtime Management** - Used Wasmi for embedded WASM execution  
✅ **Testing and Validation** - Verified WASM execution in production environment  

### **Core Concepts Mastered:**
- **WebAssembly Ecosystem**: WASM, WASI, WAT, and various runtimes
- **Cross-Platform Compilation**: Rust to WASM with optimization
- **Dynamic Code Execution**: Loading and running WASM modules at runtime
- **API Extension**: CRUD to CRUDE with execute functionality
- **Memory Safety**: WASM's isolated memory model and security benefits
- **Blockchain Applications**: Understanding why blockchains adopt WASM

### **Practical Skills Developed:**
- Setting up Rust projects for WASM compilation
- Optimizing WASM binaries for size and performance
- Integrating WASM runtimes into web applications
- Building dynamic execution APIs
- Testing WASM modules in production environments
- Understanding blockchain and edge computing use cases

---

## 🚀 **Advanced Implementation: Senior Developer Expansion**

### **🎯 Project Repository: [learn-rust-crud](https://github.com/danielgorgonha/learn-rust-crud)**

Based on senior backend development experience, this project demonstrates how to expand the basic Aula 3 concepts into a **production-ready, enterprise-level application**.

### **🔥 Advanced Features Implemented:**

#### **🔐 Complete Authentication System**
```rust
// JWT Authentication with Refresh Tokens
- Access tokens (1h expiration)
- Refresh tokens (30d expiration)  
- Owner-only operations
- Token invalidation on logout
- Bearer token authentication
- Comprehensive error handling
```

#### **⚡ Performance Optimizations**
```rust
// WASM Module Caching & Performance
- Module caching to reduce compilation time
- Rate limiting protection
- Comprehensive logging with metrics
- Input validation with bounds checking
- Execution time tracking
- Function usage statistics
```

#### **🧪 Production-Ready Testing**
```bash
# Automated Test Suite
./test/0_login.sh      # Authentication tests
./test/1_create.sh     # Create operation tests
./test/2_read_all.sh   # Read all tests
./test/3_read_one.sh   # Read one tests
./test/4_update.sh     # Update tests
./test/5_delete.sh     # Delete tests
```

#### **📊 Enhanced WASM Functions (9 Functions)**
```rust
// Extended Math Library
add(x: i32, y: i32) -> i32      // Addition
mul(x: i32, y: i32) -> i32      // Multiplication
sub(x: i32, y: i32) -> i32      // Subtraction (with validation)
div(x: i32, y: i32) -> i32      // Division (with validation)
rem(x: i32, y: i32) -> i32      // Remainder (with validation)
abs(x: i32) -> i32              // Absolute value
max(x: i32, y: i32) -> i32      // Maximum
min(x: i32, y: i32) -> i32      // Minimum
pow(x: i32, y: i32) -> i32      // Power (with validation)
```

### **🏗️ Enterprise Architecture:**

#### **📁 Modular Project Structure**
```
learn-rust-crud/
├── src/           # Core application logic
├── math/          # WASM library with build scripts
├── test/          # Automated shell test suite
├── postman/       # API documentation & collections
├── docs/          # Comprehensive documentation
├── examples/      # Usage examples and guides
└── tests/         # Integration tests
```

#### **🔧 Technology Stack**
- **Tide Framework** - Async web framework
- **async-std** - Async runtime
- **Serde** - Serialization/deserialization
- **wasmi** - WASM runtime
- **jsonwebtoken** - JWT implementation
- **Chrono** - Date/time handling
- **Railway** - Production deployment

### **🌐 Production Deployment:**

#### **🚀 Live Demo**
- **Production API**: Deployed on Railway platform
- **Environment Variables**: Secure configuration with Dotenv
- **Postman Collection**: Complete API documentation
- **Automated Testing**: Shell scripts for CI/CD

#### **📈 Monitoring & Logging**
```rust
// Structured Logging with Metrics
- Execution time tracking
- Function usage statistics  
- Error tracking and debugging
- Performance metrics
- Comprehensive audit trail
```

### **🔮 Future Roadmap (Senior Vision):**

#### **🔄 Framework Migration Strategy**
```rust
// Planned Migration to Tokio-based Stack
- Axum/Warp/Actix-web for better ecosystem
- Native timeout support for WASM execution
- Enhanced error handling capabilities
- Improved testing with Tokio runtime
```

#### **📈 Advanced Features Planned**
```rust
// Enterprise-Level Enhancements
- LRU cache for frequently used modules
- WebSocket endpoint for real-time metrics
- Function composition (chaining multiple WASM functions)
- Memory usage monitoring and limits
- Dynamic function signature detection
- Custom validation rules per function
```

### **🏆 Key Achievements:**

#### **✅ Production Excellence**
- **Security**: JWT + Refresh tokens + Owner-only access
- **Performance**: WASM caching + Rate limiting + Metrics
- **Reliability**: Comprehensive testing + Error handling
- **Documentation**: Complete API docs + Postman collections
- **Deployment**: Railway production environment

#### **🎯 Senior Developer Practices**
- **Modular Architecture**: Clear separation of concerns
- **Security First**: Authentication and authorization
- **Performance Optimization**: Caching and monitoring
- **Testing Strategy**: Automated and comprehensive
- **Documentation**: Complete and professional
- **Production Ready**: Deployed and monitored

### **💡 Learning Outcomes:**

This advanced implementation demonstrates how to:
- **Scale basic concepts** into production applications
- **Apply senior-level practices** in Rust development
- **Integrate multiple technologies** seamlessly
- **Build enterprise-ready** WebAssembly applications
- **Prepare for real-world** blockchain development

---

### **Next Steps:**
Ready for the **Meridian Hackathon** - Apply all learned skills (Rust libraries, CRUD APIs, WebAssembly) in real-world blockchain projects with **enterprise-level quality**!

---

*This lesson provides the foundation for building WebAssembly applications in Rust and prepares you for advanced blockchain development in the Meridian Hackathon! 🚀*

