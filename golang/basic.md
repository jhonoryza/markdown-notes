# Golang Basic

## Installation

Follow the official installation guide: [https://go.dev/doc/install](https://go.dev/doc/install)

## Get Started

To create a new project, initialize a module:

```bash
go mod init hello_world
```

To run a Go program:

```bash
go run main.go
```

## Comment

```go
package main

import "fmt"

func main() {
    // This is a single-line comment

    /*
       This is a multi-line comment
    */

    fmt.Println("Hello, World!")
}
```

## Variables

### Declaration

```go
var a int = 5
```

### Short Declaration

```go
a := 5
```

### Mutability

Variables in Go are mutable by default:

```go
a := 5
a = 10 // Allowed
```

### Shadowing

```go
package main

import "fmt"

func main() {
    a := 5
    {
        a := 10 // Shadows the outer 'a'
        fmt.Println(a) // 10
    }
    fmt.Println(a) // 5
}
```

## Data Types

```go
// Boolean
var b1 bool = true

// Integer
var i1 int = 42
var i2 int8 = 127
var i3 uint = 42

// Floating-point
var f1 float32 = 3.14
var f2 float64 = 2.718

// String
var s1 string = "Hello, Go"

// Array
var arr [5]int = [5]int{1, 2, 3, 4, 5}

// Slice
var slice []int = []int{1, 2, 3, 4, 5}

// Map
var m map[string]int = map[string]int{"one": 1, "two": 2}

// Struct
type Person struct {
    Name string
    Age  int
}
```

## Constants

```go
const Pi = 3.14
const Greeting = "Hello, Go"
```

## Functions

```go
package main

import "fmt"

func main() {
    result := add(5, 3)
    fmt.Println("Result:", result)
}

func add(a int, b int) int {
    return a + b
}
```

## Control Flow

### If-Else

```go
if a > 5 {
    fmt.Println("Greater than 5")
} else if a > 3 {
    fmt.Println("Greater than 3")
} else {
    fmt.Println("Less than or equal to 3")
}
```

### Switch

```go
switch day := "Monday"; day {
case "Monday":
    fmt.Println("Start of the week")
case "Friday":
    fmt.Println("End of the week")
default:
    fmt.Println("Midweek")
}
```

### Loops

#### For Loop

```go
for i := 0; i < 5; i++ {
    fmt.Println(i)
}
```

#### While-Like Loop

```go
i := 0
for i < 5 {
    fmt.Println(i)
    i++
}
```

#### Infinite Loop

```go
for {
    fmt.Println("Infinite loop")
    break
}
```

## Structs

```go
package main

import "fmt"

type Product struct {
    Name  string
    Price float64
}

func (p Product) CalculateTax() float64 {
    return p.Price * 0.1
}

func main() {
    book := Product{Name: "Go Programming", Price: 50.0}
    fmt.Println("Tax:", book.CalculateTax())
}
```

## Pointers

```go
package main

import "fmt"

func main() {
    a := 42
    p := &a // Pointer to 'a'
    fmt.Println(*p) // Dereference pointer
    *p = 21         // Modify value through pointer
    fmt.Println(a)  // 21
}
```

## Arrays and Slices

### Arrays

```go
arr := [5]int{1, 2, 3, 4, 5}
fmt.Println(arr[0]) // Access element
```

### Slices

```go
slice := []int{1, 2, 3, 4, 5}
slice = append(slice, 6) // Add element
fmt.Println(slice)
```

## Maps

```go
m := map[string]int{"one": 1, "two": 2}
fmt.Println(m["one"]) // Access value
m["three"] = 3        // Add key-value pair
delete(m, "two")      // Delete key
```

## Goroutines

```go
package main

import (
    "fmt"
    "time"
)

func sayHello() {
    fmt.Println("Hello")
}

func main() {
    go sayHello() // Run in a separate goroutine
    time.Sleep(1 * time.Second) // Wait for goroutine to finish
}
```

## Channels

```go
package main

import "fmt"

func main() {
    ch := make(chan int)

    go func() {
        ch <- 42 // Send value to channel
    }()

    value := <-ch // Receive value from channel
    fmt.Println(value)
}
```

## Error Handling

```go
package main

import (
    "errors"
    "fmt"
)

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("Result:", result)
    }
}
```

## Interfaces

```go
package main

import "fmt"

type Shape interface {
    Area() float64
}

type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return 3.14 * c.Radius * c.Radius
}

func main() {
    var s Shape = Circle{Radius: 5}
    fmt.Println("Area:", s.Area())
}
```
