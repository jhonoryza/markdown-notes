# Golang Advanced

## Generics

```go
package main

import "fmt"

func Sum[T int | float64](a, b T) T {
    return a + b
}

func main() {
    fmt.Println(Sum(3, 5))       // Integers
    fmt.Println(Sum(3.5, 5.5))   // Floats
}
```

## Goroutines and Select

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Message from channel 1"
    }()

    go func() {
        time.Sleep(2 * time.Second)
        ch2 <- "Message from channel 2"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println(msg1)
        case msg2 := <-ch2:
            fmt.Println(msg2)
        }
    }
}
```

## Mutex and Race Conditions

```go
package main

import (
    "fmt"
    "sync"
)

var counter int
var mu sync.Mutex

func increment(wg *sync.WaitGroup) {
    defer wg.Done()
    mu.Lock()
    counter++
    mu.Unlock()
}

func main() {
    var wg sync.WaitGroup

    for i := 0; i < 1000; i++ {
        wg.Add(1)
        go increment(&wg)
    }

    wg.Wait()
    fmt.Println("Final Counter:", counter)
}
```

## Building REST APIs

```go
package main

import (
    "encoding/json"
    "net/http"
)

type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

var users = []User{
    {ID: 1, Name: "Alice"},
    {ID: 2, Name: "Bob"},
}

func getUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

func main() {
    http.HandleFunc("/users", getUsers)
    http.ListenAndServe(":8080", nil)
}
```

## Testing

### Writing Unit Tests

```go
package main

import "testing"

func Add(a, b int) int {
    return a + b
}

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("Expected 5, got %d", result)
    }
}
```

### Benchmarking

```go
package main

import "testing"

func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}
```

## Advanced Interfaces

```go
package main

import "fmt"

type Stringer interface {
    String() string
}

type Person struct {
    Name string
    Age  int
}

func (p Person) String() string {
    return fmt.Sprintf("%s (%d years old)", p.Name, p.Age)
}

func main() {
    p := Person{Name: "Alice", Age: 30}
    fmt.Println(p)
}
```

## Dependency Injection

```go
package main

import "fmt"

type Service interface {
    PerformTask()
}

type RealService struct{}

func (s RealService) PerformTask() {
    fmt.Println("Performing real task")
}

type App struct {
    Service Service
}

func main() {
    app := App{Service: RealService{}}
    app.Service.PerformTask()
}
```
