# Rust Basic

## Installation

check this [`link`](https://www.rust-lang.org/tools/install)

## Get Started

to create the project run this command

```bash
cargo new hello_world
```

to run the project

```bash
cargo run main.rs
```

## Comment

```rs
fn main() {
    // this is single line comment
    /*
        this is a multiple
        line comment
    */
}
```

## Variables

### creation

```rs
let a: i32 = 5;
```

### mutability

by default variable is immutable (value cannot be changed)

```rs
let b: i32 = 5;
b = 10; // cannot assign
```

we need to specify variable as mutable to so it can be changed

```rs
let mut b: i32 = 5;
b = 10;
```

### shadowing

```rs
let mut c: i32 = 5;
let mut c: i32 = 10;

println!("value of c is {c}"); // 10
```

### scope

```rs
let mut d: i32 = 5;

{
    let mut e: i32 = 10;
    println!("e is {e}"); // 10
}

println!("e is {e}"); // compile error cannot find variable e

println!("d is {d}"); // 5
```

## Data Types

```rs
// boolean
let b1: bool = true;

// unsigned integer
let i1: u8 = 1;
let i1: u16 = 1;
let i1: u32 = 1;
let i1: u64 = 1;
let i1: u128 = 1;

// signed integer
let i1: i8 = 1;
let i1: i16 = 1;
let i1: i32 = 1;
let i1: i64 = 1;
let i1: i128 = 1;

// floating point number
let f1: f32 = 1.0;
let f1: f64 = 1.0;

// platform specific integer
let p1: usize = 1;
let p1: isize = 1;

// char, slice of string and string
let c1: char = 'c';
let c1: &str = "hello";
let c1: String = String::from("hello");
let c1: String = "hello".to_string();
let c1: String = "hello".to_owned();

// arrays
let a1: [i32, 5] = [1, 2, 3, 4, 5];
let i1: i32 = a1[0]; // 1

// tupples
let t1: (i32, i32, i32) = (1, 2, 3);
let t1: (i32, f64, &str) = (1, 5.0, "5");

let s1: &str = t1.2; // "5"
let (i1: i32, f1: f64, s1: &str) = t1;

let unit: () = ();

// type aliasing
type age = u8;

let a1: age = 57;
```

## Constant

```rs
const MAX_PLAYERS: u8 = 10; // cannot be change to mutable
static CASINO_NAME: &str = "Rusty Casino"; // can be change to mutable

static mut CASINO_NAME: &str = "Rusty Casino"; // unsafe operation or not recommended
```

- `const` do not occupy in memory
- `static` occupy in memory, good if you want to store large amount of data

## Function

```rs
fn main() {
    let z: i32 = my_function(22); // 10
    println!("z is {}", z);
}

fn my_function(x: i32) -> i32 {
    println!("x is {}", x);
    let y: i32 = 10;
    return y;
}
```

## Control Flow

```rs
let a: i32 = 5;

if a > 5 {
    println!("bigger than 5");
} else if a > 3 {
    println!("bigger than 3");
} else {
    println!("smaller than 3");
}

let b: i32 = if a > 5 { 1 } else { -1 };

// loop
loop {
    println!("loop forever");
}

// break a loop
loop {
    println!("loop forever");
    loop {
        break; // this will break ineer loop not the outer loop
    }
}

`outer: loop {
    println!("loop forever");
    loop {
        break `outer; // this will break outer loop
    }
}

// returned a loop
let x: i32 = loop {
    break 5;
};

// while loop
let mut a: i32 = 0;

while a < 5 {
    println!("a is {a}");
    a = a + 1;
}

// for loop
let a: [i32, 5] = [1, 2, 3, 4, 5];

for element: i32 in a {
    println!("{}", element);
}
```

## Computer Resources

- computation
  - CPU

- memory
  - Persistent
    - Hard Drive or SSD
      - slow
      - abundant
      - used to persist data
  - Volatile
    - RAM
      - fast
      - scarce
      - used during program execution

### Memory Types

| type   | contents                                                                                                                     | size                                 | lifetime                 | cleanup                     |
| ------ | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------ | --------------------------- |
| stack  | - func arg <br>- local var <br>- known size at comp time                                                                     | dynamic <br>/ fixed <br> upper limit | lifetime of func         | auto when func return       |
| heap   | - value live beyond func lifetime <br> value accessed by multiple thread <br>- larger values <br>- unknown size at comp time | dynamic                              | determined by programmer | manual                      |
| static | - programs binary <br>- static variables <br>- string literals                                                               | fixed                                | lifetime of program      | auto when program terminate |

### Ownership Based Resource Management (OBRM)

Ownership is a strategy for managing memory (and other resources) through a set
of rules checked at compile time.

Rules :

- each value in rust has a variable that's called its owner.
- there can only be one owner at a time.
- when the owner goest out of scope. the value will be dropped.
- in rust it's a language feature

```rs
struct Car {}

fn memory_example() {
    let car = Box::new(Car {}); // Box is a smart pointer to make heap allocated in memory
    let my_string = String::from("LGR"); // string heap allocator

    function_that_can_panic();
    if !should_continue() { return; }
}

fn file_example() {
    let path = Path::new("example.txt");
    let file = File::open(&path).unwrap();

    function_that_can_panic();
    if !should_continue() { return; }
}
```

transfer ownership

```rs
struct Car {}

fn memory_example() {
    let car = Box::new(Car {}); // car will be invalidated
    let car2 = car; // car2 will be the new owner

    function_that_can_panic();
    if !should_continue() { return; }
}
```

share ownership

```rs
struct Car {}

fn memory_example() {
    let car = Rc::new(Car {}); // car will point to the same heap allocated memory
    let car2 = car.clone(); // car2 will point to the same heap allocated memory

    function_that_can_panic();
    if !should_continue() { return; }
}
```

### Problems Ownership Solves

- memory / resource leaks
- double free
- use after free

although ownership prevent resource leaks, it doesn't eliminate them completely.

Rules :

- each value in rust has a variable that's called its owner.
- there can only be one owner at a time.
- when the owner goest out of scope. the value will be dropped.
- in rust it's a language feature

```rs
{
    let s1: String = String::from("rust"); // heap allocated string
} // s1 is dropped then heap allocated string is deleted

println!("s1 {s1}"); // error cannot find value s1
```

transfer ownership

```rs
let s1: String = String::from("rust"); // heap allocated string
let s2: String = s1; // the new owner now is s2

println!("s1 {s1}"); // error cannot borrow from s1
```

clone value

```rs
let s1: String = String::from("rust"); // heap allocated string
let s2: String = s1.clone(); // s2 has its own copy of s1 value

println!("s1 {s1}"); // rust
println!("s2 {s2}"); // rust
```

for primitive types (integer, float, boolean and char) are clone by default when
pass to a function argument

```rs
let x1: i32 = 2; // allocated on stack
let x2: i32 = x1; // x1 value is cloned

println!("x1 {x1}"); // 2
println!("x2 {x2}"); // 2
```

#### Ownership in function

```rs
fn main() {
    let s1: String = String::from("rust"); // heap allocated string

    print_string(s1);

    println!("s1 {s1}"); // error borrow a moved value s1
}

fn print_string(p1: String) {
    println!("{p1}");
}
```

as you can see we got error when print s1, because function argument will move
the ownership of s1 to p1, to fix the issue we can use clone

```rs
fn main() {
    let s1: String = String::from("rust"); // heap allocated string

    print_string(s1.clone());

    println!("s1 {s1}"); // rust
}

fn print_string(p1: String) {
    println!("{p1}");
}
```

another example

```rs
fn main() {
    let s1: String = String::from("rust"); // heap allocated string

    let s2: String = prepend_to_string(s1.clone());

    println!("s2 {s2}"); // rust is awesome
}

fn prepend_to_string(mut p1: String) -> String {
    p1.push_str(" is awesome");
    p1
}
```

for primitive types (integer, float, boolean and char) are clone by default when
pass to a function

## Borrowing

- the act of creating a reference
  - reference are pointers with rules / restrictions
  - reference do not take ownership

- why borrow
  - performance
  - when ownership is not needed / desired

### rules

- at any given time you can have either one mutable reference or any number
  immutable reference
- reference must be always valid

problems these rules solve

1. data races
2. dangling reference

example

```rs
fn main() {
    let mut s1: String = String::from("rust"); // heap allocated string
    let r1: &String = &s1;
    print_string(r1);

    let r2: &String = &mut s1;
    prepend_to_string(r2);

    println!("s1 {s1}"); // rust is awesome

    let s2: String = generate_string();
    println!("s2 {s2}");
}

fn generate_string() -> String {
    String::from("rust")
}

fn print_string(p1: &String) {
    println!("{p1}");
}

fn prepend_to_string(mut p1: &String) {
    p1.push_str(" is awesome");
}
```

## Slices

Slices are references to a contiguous (element are next to each other) sequence
of elements in a collection

```rs
fn main() {
    let s: String = String::from("Hello, World!");

    let t: &str = &s[0..5]; // string slice

    println!("{}", t); // Hello

    let a: [i32; 5] = [1, 2, 3, 4, 5];
    let a_slice: &[i32] = &a[1..3];
    println!("{:?}", a_slice); //[2, 3] and :? is for debug formatting
}
```

```rs
fn main() {
    // append to string
    let mut s: String = String::from("foo");
    s.push_str("bar");
    println!("{s}"); // foobar

    // replace string
    s.replace_range(.., "baz");
    println!("{s}"); // baz

    // concat string
    let s1: String = String::from("hello");
    let s2: String = String::from("world");
    let s3 = s1 + &s2;
    println!("{s3}"); // hello world

    let s4: String = String::from("tic");
    let s5: String = String::from("tac");
    let s6 = format!("{}-{}", s4, s5);
    println!("{s6}"); // tic-tac

    let s7 = concat!(s4, s5);
    println!("{s7}"); // tic-tac
}
```

## Struct

```rs
struct Product {
    name: String,
    price: f32,
}

impl Product {
    fn calculate_tax(&self) -> f32 {
        self.price * 0.1
    }
    fn set_price(&mut self, price: f32) {
        self.price = price;
    }
    fn buy(self) {
        println!("You bought {}", self.name);
    }
}

fn main() {
    let book = Product {
        name: String::from("Rust Programming"),
        price: 4200.0,
    };

    let tax = book.calculate_tax();
    println!("The tax for {} is {}", book.name, tax); // 420

    book.set_price(4500.0);
    book.buy(); // You bought Rust Programming
    // book.set_price(5000.0); // error[E0382]: use of moved value: `book`
}
```

### Associated Function / Static Function

```rs
struct Product {
    name: String,
    price: f32,
}

impl Product {
    fn new(name: String, price: f32) -> Product {
        Product {
            name,
            price,
        }
    }
    fn get_discount() -> f32 {
        0.1
    }
}

fn main() {
    let product = Product::new("Book".to_string(), 10.0);
    println!("Product: {}, Price: {}", product.name, product.price);
    println!("Discount: {}", Product::get_discount());
}
```

## Tupple Struct

```rs
fn main() {
    let rgb_color = (255, 0, 0);
    let cmyk_color = (255, 0, 0, 0);

    // Tuple Structs
    struct RGB(i32, i32, i32);
    struct CMYK(i32, i32, i32, i32);

    let rgb_color: RGB = RGB(255, 0, 0);
    let cmyk_color: CMYK = CMYK(255, 0, 0, 0);

    // unit-like structs
    // rarely used
}
```

tupple structs enforce number of element and data type

## Enums

```rs
struct Product {
    name: String,
    price: f64,
    category: Category,
}

enum Category {
    Food,
    Electronics,
    Clothing,
}

impl Category {
    fn tax(&self) -> f64 {
        match self {
            Category::Food => 0.05,
            Category::Electronics => 0.18,
            Category::Clothing => 0.1,
        }
    }
}

fn main() {
    let product = Product {
        name: "Shirt".to_string(),
        price: 20.0,
        category: Category::Clothing,
    };

    let tax = product.category.tax();
    let total = product.price + (product.price * tax);
    println!("Total price: {}", total);
}
```

## Match

```rs
fn main() {
    let age = 5;

    match age {
        0 => println!("I'm not born yet I guess"),
        1..=12 => println!("I'm a child"),
        13..=19 => println!("I'm a teenager"),
        _ => println!("I'm an adult"),
    }
}
```

## Option Enum

```rs
fn main() {
    let username = get_username(1);
    match username {
        Some(name) => println!("Hello, {}", name),
        None => println!("Hello, world"),
    }
    if let Some(name) = username {
        println!("Hello, {}", name);
    } else {
        println!("Hello, world");
    }
}

fn get_username(user_id: i32) -> Option<String> {
    if user_id == 1 {
        Some("Alice".to_string())
    }
    None
}
```

## Result Enum

```rs
fn main() {
    let username = get_username(1);
    match username {
        Some(name) => println!("Hello, {}", name),
        None => println!("Hello, world"),
    }
    if let Some(name) = username {
        println!("Hello, {}", name);
    } else {
        println!("Hello, world");
    }
}

fn get_username(user_id: i32) -> Option<String> {
    let query = format!("SELECT name FROM users WHERE id = {}", user_id);
    let db_result = query_db(query);
    // match db_result {
    //     Ok(name) => Some(name),
    //     Err(_) => None,
    // }
    if let Ok(name) = db_result {
        Some(name)
    } else {
        None
    }
}

fn query_db(query: String) -> Result<String, String> {
    if query.is_empty() {
        return Err("Empty query".to_string());
    }
    Ok("Alice".to_string())
}
```

## Vectors

- like arrays
- hold element with the same sequence type
- scan growable and allocated in heap

```rs
fn main() {
    let mut v: Vec<String> = Vec::new();

    v.push("Hello".to_string());
    v.push("World".to_string());

    let v2 = vec![1, 2, 3, 4, 5];

    let s: &String = &v[0]; // can panic
    let s: String = v.remove(0); // remove and return index element
    let s: Option<&String> = v.get(0); // return Option<&T>

    if let Some(e) = s {
        println!("{}", e);
    }

    // iterate over vector
    for e in &v2 {
        println!("{}", e);
    }
}
```
