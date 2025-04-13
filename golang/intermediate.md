# Golang Intermediate

## Packages and Modules

### Creating a Package

A package is a way to organize code into reusable modules.

1. Create a folder for your package:
   ```bash
   mkdir mathutils
   ```

2. Add a Go file inside the folder:
   ```go
   // filepath: mathutils/mathutils.go
   package mathutils

   func Add(a, b int) int {
       return a + b
   }

   func Multiply(a, b int) int {
       return a * b
   }
   ```

3. Use the package in your main program:
   ```go
   package main

   import (
       "fmt"
       "path/to/your/project/mathutils"
   )

   func main() {
       sum := mathutils.Add(3, 5)
       product := mathutils.Multiply(3, 5)
       fmt.Println("Sum:", sum)
       fmt.Println("Product:", product)
   }
   ```

### Using Third-Party Modules

1. Initialize a module:
   ```bash
   go mod init myproject
   ```

2. Install a third-party module:
   ```bash
   go get github.com/gorilla/mux
   ```

3. Use the module:
   ```go
   package main

   import (
       "fmt"
       "net/http"
       "github.com/gorilla/mux"
   )

   func main() {
       r := mux.NewRouter()
       r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
           fmt.Fprintln(w, "Hello, World!")
       })
       http.ListenAndServe(":8080", r)
   }
   ```

## Error Handling with Custom Errors

```go
package main

import (
    "errors"
    "fmt"
)

type CustomError struct {
    Code    int
    Message string
}

func (e *CustomError) Error() string {
    return fmt.Sprintf("Code: %d, Message: %s", e.Code, e.Message)
}

func riskyOperation(success bool) error {
    if !success {
        return &CustomError{Code: 500, Message: "Operation failed"}
    }
    return nil
}

func main() {
    err := riskyOperation(false)
    if err != nil {
        fmt.Println("Error:", err)
    }
}
```

## Concurrency with WaitGroups

```go
package main

import (
    "fmt"
    "sync"
)

func worker(id int, wg *sync.WaitGroup) {
    defer wg.Done()
    fmt.Printf("Worker %d starting\n", id)
    // Simulate work
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    var wg sync.WaitGroup

    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i, &wg)
    }

    wg.Wait()
    fmt.Println("All workers completed")
}
```

## Context for Cancellation and Timeouts

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func doWork(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            fmt.Println("Work cancelled")
            return
        default:
            fmt.Println("Working...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()

    go doWork(ctx)

    time.Sleep(3 * time.Second)
    fmt.Println("Main function done")
}
```

## Reflection

```go
package main

import (
    "fmt"
    "reflect"
)

type Person struct {
    Name string
    Age  int
}

func main() {
    p := Person{Name: "Alice", Age: 30}
    t := reflect.TypeOf(p)
    v := reflect.ValueOf(p)

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)
        fmt.Printf("Field: %s, Type: %s, Value: %v\n", field.Name, field.Type, value)
    }
}
```

## File Handling

### Reading a File

```go
package main

import (
    "fmt"
    "io/ioutil"
)

func main() {
    data, err := ioutil.ReadFile("example.txt")
    if err != nil {
        fmt.Println("Error reading file:", err)
        return
    }
    fmt.Println(string(data))
}
```

### Writing to a File

```go
package main

import (
    "fmt"
    "os"
)

func main() {
    file, err := os.Create("example.txt")
    if err != nil {
        fmt.Println("Error creating file:", err)
        return
    }
    defer file.Close()

    _, err = file.WriteString("Hello, Go!")
    if err != nil {
        fmt.Println("Error writing to file:", err)
    }
}
```
