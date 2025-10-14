```c++
#include <iostream>

extern "C" {
    char* hello_cpp(const char* name);
    void free_string(char* ptr);
}

int main() {
    char* message = hello_cpp("C++");
    if (message) {
        std::cout << message << std::endl;
        free_string(message);
    }
    return 0;
}
```

```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

#[unsafe(no_mangle)]
pub extern "C" fn hello_cpp(name: *const c_char) -> *mut c_char {
    if name.is_null() {
        return std::ptr::null_mut();
    }
    
    unsafe {
        let c_str = match CStr::from_ptr(name).to_str() {
            Ok(s) => s,
            Err(_) => return std::ptr::null_mut(),
        };
        
        let message = format!("Hello, {}! Greetings from Rust!", c_str);
        
        match CString::new(message) {
            Ok(c_string) => c_string.into_raw(),
            Err(_) => std::ptr::null_mut(),
        }
    }
}

#[unsafe(no_mangle)]
pub extern "C" fn free_string(ptr: *mut c_char) {
    if !ptr.is_null() {
        unsafe {
            let _ = CString::from_raw(ptr);
        }
    }
}
```

## Explanation
```
[package]
name = "cppdll"
version = "0.1.0"
edition = "2024"

[lib]
name = "cppdll"
crate-type = ["cdylib"] 

[dependencies]
libc = "0.2"

[build-dependencies]
cbindgen = "0.26"  
```