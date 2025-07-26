# Lesson 2: How to Create a CRUD API with Rust in Practice

## 📚 Lesson Content

### 🎯 **What is CRUD?**
**CRUD** (Create, Read, Update, Delete) are the essential operations for managing data in any application:

- **C**reate: Add new data
- **R**ead: Query existing data  
- **U**pdate: Modify existing data
- **D**elete: Remove data

### 🏗️ **Client-Server Architecture**
- **Client**: Application that makes requests (frontend, mobile, etc.)
- **Server**: API that processes requests and manages data
- **Communication**: Via HTTP/HTTPS with JSON

---

## 🧠 **Memory Management in Rust**

### **Why Memory Safety Matters**
Rust is known for its memory safety, which means it helps prevent many common programming errors that can cause crashes or vulnerabilities. It does this without needing a "garbage collector," which is a system that automatically manages memory in other languages but can add overhead.

### **The Ownership System**
Rust uses a system called **ownership** to manage memory. This system is based on three simple but powerful rules. Understanding these rules is fundamental for writing Rust code.

#### **The Three Rules of Ownership:**
1. **Each value has an owner** - Every piece of data has exactly one variable that owns it
2. **Only one owner at a time** - When the owner goes out of scope, the value is dropped
3. **Values are moved or borrowed** - You can either move ownership or borrow references

#### **Common Ownership Patterns:**
```rust
// Rule 1: Each value has an owner
let s1 = String::from("hello");  // s1 owns the string

// Rule 2: Only one owner at a time
let s2 = s1;  // s1's value is moved to s2, s1 is no longer valid
// println!("{}", s1);  // This would cause a compile error!

// Rule 3: Borrowing references
let s3 = String::from("world");
let len = calculate_length(&s3);  // We borrow s3, don't take ownership

fn calculate_length(s: &String) -> usize {
    s.len()  // s is a reference, so the String it points to remains valid
}
```

### **References and Lifetimes**
Lifetimes are Rust's way of ensuring that references are always valid. They prevent "dangling references" - references that point to data that no longer exists.

#### **The Problem:**
```rust
fn main() {
    let r;                // Declare reference
    {
        let x = 5;        // Create value
        r = &x;           // Reference to x
    }                     // x goes out of scope here
    println!("r: {}", r); // Error! r references invalid data
}
```

#### **The Solution - Lifetime Annotations:**
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    
    let result = longest(&string1, string2);
    println!("The longest string is {}", result);
}
```

#### **Key Concepts:**
- **`'a`**: Lifetime parameter that tells the compiler how long references should be valid
- **Lifetime elision**: Rust can often infer lifetimes automatically
- **Struct lifetimes**: When structs contain references, they need lifetime annotations

### **Why This Matters for Our API**
In our CRUD API, we'll use `Arc<Mutex<HashMap>>` to safely share data between multiple HTTP requests. The ownership system and lifetimes ensure we don't have data races, memory leaks, or dangling references.

---

## 🛠️ **Tools and Technologies**

### **Asynchronous Runtimes**
- **async-std**: Simple and easy-to-use runtime (our choice)
- **Tokio**: Robust and performant runtime for complex applications

### **Web Frameworks**
- **Hyper**: Low-level HTTP library
- **Tide**: High-level web framework, more productive for APIs

### **Main Dependencies**
```toml
[dependencies]
tide = "0.16"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
async-std = { version = "1.0", features = ["attributes"] }
```

---

## 🏗️ **Project Structure**

### **File Organization**
```
crud/
├── Cargo.toml
└── src/
    ├── main.rs         # Application entry point and route configuration
    ├── models.rs       # Data model definition
    ├── state.rs        # Global state management
    └── handlers/
        ├── mod.rs      # Handlers module configuration
        ├── create.rs   # Logic for creating data (POST)
        ├── read.rs     # Logic for reading data (GET)
        ├── update.rs   # Logic for updating data (PUT)
        └── delete.rs   # Logic for deleting data (DELETE)
```

---

## 📊 **Data Model**

### **DataEntry - Our Main Model**
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Clone, Debug)]
pub struct DataEntry {
    pub data1: Vec<String>,
    pub data2: Vec<u8>,
}
```

### **Used Traits:**
- **`Serialize`**: Allows serializing to JSON in responses
- **`Deserialize`**: Allows deserializing JSON from requests
- **`Clone`**: Allows creating explicit copies of data
- **`Debug`**: Allows printing the struct with `println!("{:?}", data)`

---

## 🔒 **Shared State Management**

### **Arc<Mutex<HashMap>> - Our Solution**
```rust
use std::collections::HashMap;
use std::sync::{Arc, Mutex};
use crate::models::DataEntry;

pub type AppState = Arc<Mutex<HashMap<u32, DataEntry>>>;

pub fn new_state() -> AppState {
    Arc::new(Mutex::new(HashMap::new()))
}
```

### **Why Arc<Mutex<HashMap>>?**

#### **Arc (Atomic Reference Counted)**
- Allows sharing data between multiple parts of the code
- Smart counter that knows how many parts are using it
- Cheap clone (just increases the counter)

#### **Mutex (Mutual Exclusion)**
- Ensures exclusive access to data
- Prevents two requests from modifying data simultaneously
- Prevents race conditions

#### **HashMap**
- Stores our data with key (u32) and value (DataEntry)
- Efficient data structure for CRUD operations

---

## 🚀 **CRUD Routes Implementation**

### **1. CREATE - Create Data (POST /data)**
```rust
// src/handlers/create.rs
use crate::models::DataEntry;
use crate::state::AppState;
use tide::Request;

pub async fn create_data(mut req: Request<AppState>) -> tide::Result {
    // Read request body as JSON
    let entry: DataEntry = req.body_json().await?;

    // Get global state (HashMap protected by Mutex)
    let state = req.state();
    let mut map = state.lock().unwrap();

    // Generate a simple new id
    let new_id = map.len() as u32 + 1;

    // Insert the new record
    map.insert(new_id, entry);

    // Return the created id as JSON
    Ok(tide::Body::from_json(&serde_json::json!({ "id": new_id }))?.into())
}
```

### **2. READ - Read Data**

#### **Read All (GET /data)**
```rust
// src/handlers/read.rs
pub async fn read_all_data(req: Request<AppState>) -> tide::Result {
    let state = req.state();
    let map = state.lock().unwrap();
    
    // Return all records as JSON
    Ok(tide::Body::from_json(&*map)?.into())
}
```

#### **Read by ID (GET /data/:id)**
```rust
pub async fn read_data(req: Request<AppState>) -> tide::Result {
    // Extract id from URL
    let id: u32 = match req.param("id")?.parse() {
        Ok(val) => val,
        Err(_) => return Err(tide::Error::from_str(400, "Invalid id")),
    };

    let state = req.state();
    let map = state.lock().unwrap();

    // Search for record by id
    if let Some(entry) = map.get(&id) {
        Ok(tide::Body::from_json(entry)?.into())
    } else {
        Ok(tide::Response::new(404))
    }
}
```

### **3. UPDATE - Update Data (PUT /data/:id)**
```rust
// src/handlers/update.rs
pub async fn update_data(mut req: Request<AppState>) -> tide::Result {
    // Extract id from URL
    let id: u32 = match req.param("id")?.parse() {
        Ok(val) => val,
        Err(_) => return Err(tide::Error::from_str(400, "Invalid id")),
    };

    // Read request body as JSON
    let entry: DataEntry = req.body_json().await?;

    let state = req.state();
    let mut map = state.lock().unwrap();

    // Update record if it exists
    if let std::collections::hash_map::Entry::Occupied(mut e) = map.entry(id) {
        e.insert(entry);
        Ok(tide::Response::new(200))
    } else {
        Ok(tide::Response::new(404))
    }
}
```

### **4. DELETE - Delete Data (DELETE /data/:id)**
```rust
// src/handlers/delete.rs
pub async fn delete_data(req: Request<AppState>) -> tide::Result {
    // Extract id from URL
    let id: u32 = match req.param("id")?.parse() {
        Ok(val) => val,
        Err(_) => return Err(tide::Error::from_str(400, "Invalid id")),
    };

    let state = req.state();
    let mut map = state.lock().unwrap();

    // Remove record if it exists
    if map.remove(&id).is_some() {
        Ok(tide::Response::new(204))
    } else {
        Ok(tide::Response::new(404))
    }
}
```

---

## 🔧 **Main Configuration**

### **src/main.rs - Connecting Everything**
```rust
mod handlers;
mod models;
mod state;

use handlers::create::create_data;
use handlers::delete::delete_data;
use handlers::read::{read_all_data, read_data};
use handlers::update::update_data;

#[async_std::main]
async fn main() -> tide::Result<()> {
    // Create global application state
    let state = state::new_state();

    // Create Tide app and associate state
    let mut app = tide::with_state(state);

    // Define CRUD routes
    app.at("/data").post(create_data);        // Create
    app.at("/data").get(read_all_data);       // Read all
    app.at("/data/:id").get(read_data);       // Read one
    app.at("/data/:id").put(update_data);     // Update
    app.at("/data/:id").delete(delete_data);  // Delete

    let addr = "127.0.0.1:8080";
    println!("CRUD server running at: http://{addr}");

    // Start the server
    app.listen(addr).await?;
    Ok(())
}
```

---

## 🧪 **Manual Testing with cURL**

### **1. Create a new item**
```bash
curl -s -X POST http://127.0.0.1:8080/data \
  -H 'Content-Type: application/json' \
  -d '{"data1": ["first", "second"], "data2": [1,2,3]}'
```

### **2. Read a specific item**
```bash
curl http://127.0.0.1:8080/data/1
```

### **3. Update an item**
```bash
curl -s -X PUT http://127.0.0.1:8080/data/1 \
  -H 'Content-Type: application/json' \
  -d '{"data1": ["updated"], "data2": [9,8,7]}'
```

### **4. Delete an item**
```bash
curl -s -X DELETE http://127.0.0.1:8080/data/1
```

### **5. Read all data**
```bash
curl http://127.0.0.1:8080/data
```

---

## 🚀 **Production Deployment**

### **Deploy with Railway**

1. **Create an account**: Visit [railway.app](https://railway.app) and create your account
2. **Configure GitHub**: Connect your GitHub account to Railway
3. **Adjust for production**: Change the address in `main.rs`:
   ```rust
   let addr = "0.0.0.0:8080";  // For production
   ```
4. **Connect repository**: Create project → Deploy from GitHub Repo
5. **Automatic deployment**: Railway compiles and generates public URL
6. **Test**: Use the new URL to validate the API in production

---

## 🚀 **Project Repositories**

### **Complete CRUD API Implementation**
**Repository:** [github.com/danielgorgonha/learn-rust-crud](https://github.com/danielgorgonha/learn-rust-crud)

**Features Implemented:**
- ✅ **Full CRUD Operations** - Create, Read, Update, Delete
- ✅ **JWT Authentication** - Secure token-based authentication
- ✅ **Refresh Tokens** - Automatic token renewal system
- ✅ **Authorization** - Owner-only access for sensitive operations
- ✅ **Thread-Safe State** - `Arc<Mutex<HashMap>>` for shared data
- ✅ **Automated Testing** - Complete test suite with shell scripts
- ✅ **Postman Collection** - Ready-to-use API documentation
- ✅ **Production Deployment** - Live on Railway platform

**Getting Started:**
```bash
# Clone the repository
git clone https://github.com/danielgorgonha/learn-rust-crud.git
cd learn-rust-crud

# Run the application
cargo run

# Test the API
./test/run_all_tests.sh
```

**Live Demo:** [https://learn-rust-crud-production.up.railway.app](https://learn-rust-crud-production.up.railway.app)

---

## 🎯 **Lesson 2 Summary & Key Takeaways**

### **What We Accomplished:**
✅ **Complete CRUD API** - Implemented all basic operations (Create, Read, Update, Delete)  
✅ **REST API Design** - Created standardized HTTP endpoints  
✅ **Concurrent State Management** - Safe shared state with Arc<Mutex<HashMap>>  
✅ **JSON Serialization** - Full Serde integration for data handling  
✅ **Modular Architecture** - Organized code into separate handler modules  
✅ **Manual Testing** - Validated API with cURL commands  
✅ **Production Deployment** - Deployed API to Railway cloud platform  
✅ **Async Programming** - Implemented async/await patterns with Tide framework  
✅ **JWT Authentication** - Secure token-based authentication system  
✅ **Refresh Tokens** - Automatic token renewal mechanism  
✅ **Authorization System** - Owner-only access for sensitive operations  
✅ **Automated Testing** - Complete test suite with shell scripts  
✅ **API Documentation** - Postman collection for easy testing  

### **Core Rust Concepts Mastered:**
🦀 **Concurrency Safety** - Arc, Mutex, and thread-safe data access  
🛠️ **Web Development** - Tide framework and HTTP handling  
📦 **State Management** - Shared application state across requests  
🧪 **API Testing** - Manual testing with cURL and HTTP methods  
🔗 **Modular Design** - Code organization with separate handler files  
📡 **HTTP Protocol** - REST endpoints and status codes  
⚡ **Async Runtime** - async-std for non-blocking operations  
🔐 **Authentication** - JWT tokens, refresh mechanisms, and authorization  
🔄 **Token Management** - Access tokens, refresh tokens, and token lifecycle  
🛡️ **Security Patterns** - Bearer token authentication and access control  

### **Practical Skills Developed:**
🎯 **API Architecture** - Designing RESTful APIs from scratch  
🔧 **Web Framework** - Tide configuration and route handling  
📚 **JSON Handling** - Serde serialization and deserialization  
🚀 **Cloud Deployment** - Railway platform and production setup  
🔄 **HTTP Methods** - GET, POST, PUT, DELETE implementation  
💡 **Concurrency Patterns** - Safe multi-threaded data access  
🌐 **Web Services** - Building production-ready APIs  
🔐 **Security Implementation** - JWT authentication, token management, authorization  
🧪 **Test Automation** - Shell scripts and Postman collections for comprehensive testing  
📋 **API Documentation** - Creating user-friendly testing interfaces  

### **Next Steps:**
➡️ **Lesson 3** - WebAssembly and web applications  
➡️ **Meridian Hackathon** - Apply these skills in real-world projects  
➡️ **Advanced APIs** - Authentication, databases, and microservices  

---

*This lesson provides the foundation for building robust web services in Rust and prepares you for complex API development in the Meridian Hackathon! 🚀*
