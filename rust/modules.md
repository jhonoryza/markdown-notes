# Rust Modules

## Basic Components

- Packages: contain one or more crates that provide a set of functionality.
  Packages allow you to build, test and share crates.
  - cargo.toml: describes the package and define how to build crates
  - rules:
    - must have at least 1 crate
    - at most 1 library crate
    - any number of binary crates
- Crates: A tree of modules that produces executable or library.
- Modules: let you control the organization, scope and privacy.

## Modules

- organize code for readability and reuse
- control scope and privacy
- contain items (function, structs, enums, traits, etc)
- explicitly defined (using mod keyword)
  - not mapped to the filesystem
  - flexibility and straight forward conditional compilation

### Example Module Usage

lets create a library called `auth_service`

```bash
cargo new auth_service --lib
```

install cargo modules

```bash
cargo install cargo-modules
```

```rs
#![allow(dead_code, unused_variables)]

mod database {
    pub enum Status {
        Success,
        Failure,
    }
    pub fn connect_to_database() -> Status {
        Status::Success
    }
    pub fn get_user() {
        //
    }
}

mod auth_utils {

    fn logout() {
        //
    }

    pub fn login() {
        crate::database::get_user();
    }

    pub mod models {
        pub struct Credentials {
            username: String,
            password: String,
        }
    }
}

use auth_utils::login;
use database::connect_to_database;
use auth_utils::models::Credentials;
use database::Status;

pub fn authenticate(creds: Credentials) {
    if let Status::Success = connect_to_database() {
        login();
    }
}
```

check structure

```bash
cargo modules structure
```

### Split to multiple files

create `lib.rs` file

```rs
#![allow(dead_code, unused_variables)]

mod auth_utils;
mod database;

use auth_utils::login;
pub use auth_utils::models::Credentials;
use database::connect_to_database;
use database::Status;

pub fn authenticate(creds: Credentials) {
    if let Status::Success = connect_to_database() {
        login();
    }
}
```

create `auth_utils.rs` file

```rs
fn logout() {
    //
}

pub fn login() {
    crate::database::get_user();
}

pub mod models;
```

create `auth_utils/models.rs` file

```rs
pub struct Credentials {
    pub username: String,
    pub password: String,
}
```

create `database.rs` file

```rs
pub enum Status {
    Success,
    Failure,
}
pub fn connect_to_database() -> Status {
    Status::Success
}
pub fn get_user() {
    //
}
```

now run

```bash
cargo modules structure --lib
```

to check crate tree

example usage auth_service library, create `main.rs` file

```rs
use auth_service::Credentials;
fn main() {
    let creds: Credentials = Credentials {
        username: "admin".to_string(),
        password: "password".to_string(),
    };
    auth_service::authenticate(creds);
}
```

run `cargo run`

## Publishing Package

- create a new account in [`https://crates.io`](https://crates.io)
- generate api token in account settings
- open terminal run this command

```bash
cargo login your_api_token
```

- go to your package directory, example our `auth_service`
- commit your changes with git
- run this command to publish

```bash
cargo publish
```

#### Notes

check inside `cargo.toml`

make sure

- name is unique
- description is filled
- license is filled

```toml
[package]
name = "auth_service"
description = "my package description"
license = "MIT"
```

once published it is permanent, the version cannot be overridden and the code
cannot be deleted. this because crates.io is a permanent archieve.

## Workspace

lets create a directory called `blog`, create a `Cargo.toml` file

```toml
[workspace]

members = [
    "blog_api",
    "blog_web",
    "blog_shared",
]
```

create 3 directory with these command

```bash
cargo new --vcs none blog_api
```

```bash
cargo new --vcs none blog_web
```

```bash
cargo new --vcs none blog_shared
```

edit `Cargo.toml` file in blog_shared folder and add serde package.

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
```

edit `lib.rs` file in blog_shared folder

```rs
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
pub struct Post {
    title: String,
    body: String,
}

impl Post {
    pub fn new(title: String, body: String) -> Post {
        Post { title, body }
    }
}
```

now lets edit `Cargo.toml` file in blog_api and blog_web folder to add
blog_shared package

```
[dependencies]
blog_shared = { path = "../blog_shared" }
```

now lest edit `main.rs` file in blog_api and blog_web folder

```
fn main() {
    let post = blog_shared::Post::new(
        "Hello, world!".to_string(),
        "This is my first post.".to_string(),
    );
    println!("{:?}", post);
}
```

last lets build our apps, from root directory

```bash
cargo build
```

check target folder, there will be 2 binary files blog_api and blog_web binary
and 1 shared library blog_shared
