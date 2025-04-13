# Comparison of Go ORM Libraries

This guide compares popular Go ORM libraries: `GORM`, `Ent`, `SQLBoiler`, and `XORM`. Each ORM has its own strengths and weaknesses, and the choice depends on your application's requirements.

---

## 1. `GORM`

`GORM` is a widely used ORM in the Go ecosystem, known for its simplicity and feature-rich API.

### Features
- Auto-migrations.
- Associations (e.g., one-to-one, one-to-many).
- Hooks (before/after create, update, delete).
- Query builder.

### Example

```go
package main

import (
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

type User struct {
    ID   uint   `gorm:"primaryKey"`
    Name string
}

func main() {
    db, _ := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    db.AutoMigrate(&User{})

    db.Create(&User{Name: "John"})
    var user User
    db.First(&user, "name = ?", "John")
    println(user.Name)
}
```

### Official Documentation
- [GORM Documentation](https://gorm.io/docs/)

### Pros
- Feature-rich and easy to use.
- Large community and extensive documentation.

### Cons
- Performance overhead for complex queries.
- Can be verbose for advanced use cases.

---

## 2. `Ent`

`Ent` is a powerful ORM focused on schema generation and type safety.

### Features
- Code generation for schema and queries.
- Type-safe API.
- Supports GraphQL integration.

### Example

```go
package main

import (
    "context"
    "log"
    "entgo.io/ent/dialect/sqlite"
    "github.com/your_project/ent"
)

func main() {
    client, _ := ent.Open("sqlite3", "file:test.db?cache=shared&_fk=1")
    defer client.Close()

    ctx := context.Background()
    client.Schema.Create(ctx)

    user := client.User.Create().SetName("John").SaveX(ctx)
    log.Println(user.Name)
}
```

### Official Documentation
- [Ent Documentation](https://entgo.io/docs/)

### Pros
- Type-safe and modern API.
- Great for large, complex applications.

### Cons
- Requires code generation.
- Smaller community compared to `GORM`.

---

## 3. `SQLBoiler`

`SQLBoiler` is a strict ORM that focuses on generating Go code from your database schema.

### Features
- Code generation from database schema.
- Type-safe queries.
- Minimal runtime overhead.

### Example

```go
package main

import (
    "context"
    "log"
    "github.com/your_project/models"
    _ "github.com/mattn/go-sqlite3"
)

func main() {
    db, _ := sql.Open("sqlite3", "test.db")
    defer db.Close()

    ctx := context.Background()
    user := models.User{Name: "John"}
    user.Insert(ctx, db, boil.Infer())
    log.Println(user.Name)
}
```

### Official Documentation
- [SQLBoiler Documentation](https://github.com/volatiletech/sqlboiler)

### Pros
- High performance.
- Strictly enforces database schema.

### Cons
- Requires database-first approach.
- Limited flexibility for dynamic queries.

---

## 4. `XORM`

`XORM` is a simple and lightweight ORM for Go.

### Features
- Simple API.
- Auto-migrations.
- Supports multiple databases.

### Example

```go
package main

import (
    "xorm.io/xorm"
)

type User struct {
    ID   int64
    Name string
}

func main() {
    engine, _ := xorm.NewEngine("sqlite3", "test.db")
    engine.Sync2(new(User))

    user := User{Name: "John"}
    engine.Insert(&user)

    var users []User
    engine.Find(&users)
    println(users[0].Name)
}
```

### Official Documentation
- [XORM Documentation](https://xorm.io/docs/)

### Pros
- Lightweight and easy to use.
- Good for small to medium projects.

### Cons
- Limited feature set compared to `GORM`.
- Smaller community.

---

## Recommendations

- **Use `GORM`** if you need a feature-rich and widely supported ORM.
- **Use `Ent`** if you want type safety and schema generation.
- **Use `SQLBoiler`** if you prefer a database-first approach with strict schema enforcement.
- **Use `XORM`** if you need a lightweight and simple ORM for smaller projects.
