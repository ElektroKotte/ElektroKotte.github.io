---
layout: post
title:  "Link Rust with C libraries"
date:   2017-09-09
author: Emil Ohlsson
categories: rust c
---
C program:
```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

#include "foo.h"

foo_t *foo_new(const char *msg)
{   
    static int id = 0;
    foo_t *f = malloc(sizeof(foo_t));
        
    id += 1;

    f->id = id;
    f->data = strdup(msg);
    
    printf("msg: %s\n", msg);

    return f;
}

void foo_destroy(foo_t *foo)
{
    free(foo->data);
    free(foo);
}
```
C interface / header
```c
#pragma once

typedef struct {
    int id;
    void *data;
} foo_t;

foo_t *foo_new(const char *msg);
    
void foo_destroy(foo_t *foo);
```

`build.rs`
```rust
use std::env;

fn main() {
    let dir = "/home/emo/prog/rust-bindgen";

    println!("cargo:rustc-link-search={}", dir);
    println!("cargo:rustc-flags=-l foo");
}
```

`Cargo.toml`
```toml
[package]
name = "rust-bindgen"
version = "0.1.0"
authors = ["Emil Ohlsson <emo@svep.se>"]
links = "foo"

[dependencies]
libc = "^0.2"
```

Makefile
```makefile
.PHONY: lib bindgen

all: lib bindgen

lib: libfoo.so
libfoo.so: foo.c
        gcc -shared -fPIC -o $@ $^

bindgen: src/foo.rs
src/foo.rs: foo.h
        bindgen -o $@ $^
```

Calling program:
```rust
mod foo;

use std::ffi::CString;
use std::os::raw::c_char;

fn test_with_string(my_string: String) {
    let to_use = CString::new(my_string).unwrap();
    unsafe {
        let my_foo = foo::foo_new(to_use.as_ptr());

        foo::foo_destroy(my_foo);
    }
}

fn main() {
    test_with_string(String::from("foo-bar"));
}
```

TODO: mention `drop` trait
