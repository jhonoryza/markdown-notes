# Comparison of Go Router Packages

This guide compares popular Go router packages: `net/http`, `mux`, `httprouter`, `gin`, `fiber`, and `chi`. Each router has its own strengths and weaknesses, and the choice depends on your application's requirements.

---

## 1. `net/http`

`net/http` is the standard library for HTTP handling in Go. It is lightweight and does not require any external dependencies.

### Features
- Built into the Go standard library.
- Minimalistic and lightweight.
- No additional features like middleware or parameter parsing.

### Example

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

### Official Documentation
- [net/http Documentation](https://pkg.go.dev/net/http)

### Pros
- No external dependencies.
- Simple and lightweight.

### Cons
- No built-in support for route parameters or middleware.
- Requires more boilerplate for advanced use cases.

---

## 2. `mux` (Gorilla Mux)

`mux` is a powerful router with support for route parameters, middleware, and more.

### Features
- Route parameters and query matching.
- Middleware support.
- Regular expression-based routing.

### Example

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/gorilla/mux"
)

func handler(w http.ResponseWriter, r *http.Request) {
    vars := mux.Vars(r)
    name := vars["name"]
    fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
    r := mux.NewRouter()
    r.HandleFunc("/hello/{name}", handler)
    http.ListenAndServe(":8080", r)
}
```

### Official Documentation
- [Gorilla Mux Documentation](https://www.gorillatoolkit.org/pkg/mux)

### Pros
- Flexible and feature-rich.
- Widely used and well-documented.

### Cons
- Slightly slower than other routers due to its flexibility.
- Larger memory footprint.

---

## 3. `httprouter`

`httprouter` is a lightweight router optimized for performance.

### Features
- Extremely fast.
- Minimalistic API.
- Supports route parameters.

### Example

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/julienschmidt/httprouter"
)

func handler(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
    name := ps.ByName("name")
    fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
    router := httprouter.New()
    router.GET("/hello/:name", handler)
    http.ListenAndServe(":8080", router)
}
```

### Official Documentation
- [httprouter Documentation](https://pkg.go.dev/github.com/julienschmidt/httprouter)

### Pros
- High performance.
- Lightweight and simple.

### Cons
- Limited feature set compared to `mux`.
- No built-in middleware support.

---

## 4. `gin`

`gin` is a high-performance web framework with a built-in router.

### Features
- Middleware support.
- JSON validation and rendering.
- Built-in error handling.

### Example

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    r.GET("/hello/:name", func(c *gin.Context) {
        name := c.Param("name")
        c.JSON(200, gin.H{"message": "Hello, " + name})
    })
    r.Run(":8080")
}
```

### Official Documentation
- [Gin Documentation](https://gin-gonic.com/docs/)

### Pros
- High performance.
- Rich feature set.
- Easy to use.

### Cons
- Larger binary size.
- Opinionated design.

---

## 5. `fiber`

`fiber` is an Express-inspired web framework built on top of `fasthttp`.

### Features
- Extremely fast.
- Middleware support.
- Built-in utilities for JSON, cookies, etc.

### Example

```go
package main

import (
    "github.com/gofiber/fiber/v2"
)

func main() {
    app := fiber.New()
    app.Get("/hello/:name", func(c *fiber.Ctx) error {
        name := c.Params("name")
        return c.JSON(fiber.Map{"message": "Hello, " + name})
    })
    app.Listen(":8080")
}
```

### Official Documentation
- [Fiber Documentation](https://docs.gofiber.io/)

### Pros
- Very fast due to `fasthttp`.
- Easy to use with a familiar API.

### Cons
- Not compatible with `net/http`.
- Smaller community compared to `gin`.

---

## 6. `chi`

`chi` is a lightweight router with a focus on composability.

### Features
- Middleware support.
- Nested routing.
- Context-based request handling.

### Example

```go
package main

import (
    "fmt"
    "net/http"
    "github.com/go-chi/chi/v5"
)

func handler(w http.ResponseWriter, r *http.Request) {
    name := chi.URLParam(r, "name")
    fmt.Fprintf(w, "Hello, %s!", name)
}

func main() {
    r := chi.NewRouter()
    r.Get("/hello/{name}", handler)
    http.ListenAndServe(":8080", r)
}
```

### Official Documentation
- [Chi Documentation](https://pkg.go.dev/github.com/go-chi/chi/v5)

### Pros
- Lightweight and composable.
- Good balance between features and simplicity.

### Cons
- Slightly less performant than `httprouter`.

---

## Performance Comparison

| Router         | Performance (Requests/sec) | Middleware Support | Features       | Community Support |
|----------------|----------------------------|--------------------|----------------|-------------------|
| `net/http`     | Very High                 | No                 | Minimal        | Built-in          |
| `mux`          | Medium                    | Yes                | Feature-rich   | High              |
| `httprouter`   | Very High                 | No                 | Minimal        | Medium            |
| `gin`          | High                      | Yes                | Feature-rich   | High              |
| `fiber`        | Very High                 | Yes                | Feature-rich   | Medium            |
| `chi`          | High                      | Yes                | Composable     | Medium            |

---

## Recommendations

- **Use `net/http`** if you want a lightweight solution and don't need advanced features.
- **Use `mux`** if you need flexibility and advanced routing features.
- **Use `httprouter`** if performance is your top priority.
- **Use `gin`** if you want a full-featured framework with high performance.
- **Use `fiber`** if you prefer an Express-like API and need extreme performance.
- **Use `chi`** if you want a lightweight and composable router with middleware support.
