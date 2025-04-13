# GORM Tutorial: From Basics to Intermediate

This tutorial provides a comprehensive guide to using `GORM`, covering basic to intermediate features commonly used in Go applications.

---

## Getting Started with GORM

### Installation

To install `GORM` and the SQLite driver, run:

```bash
go get -u gorm.io/gorm gorm.io/driver/sqlite
```

### Basic Setup

```go
package main

import (
    "gorm.io/driver/sqlite"
    "gorm.io/gorm"
)

func main() {
    db, err := gorm.Open(sqlite.Open("test.db"), &gorm.Config{})
    if err != nil {
        panic("failed to connect database")
    }
    println("Database connected!")
}
```

---

## Defining Models

Models in `GORM` are Go structs that map to database tables.

```go
type User struct {
    ID        uint   `gorm:"primaryKey"`
    Name      string
    Email     string `gorm:"unique"`
    CreatedAt time.Time
}
```

### Auto-Migration

`GORM` can automatically create or update tables based on your models.

```go
db.AutoMigrate(&User{})
```

---

## Basic CRUD Operations

### Create

```go
user := User{Name: "Alice", Email: "alice@example.com"}
db.Create(&user)
```

### Read

```go
var user User
db.First(&user, 1) // Find user with ID 1
db.First(&user, "email = ?", "alice@example.com") // Find user by email
```

### Update

```go
db.Model(&user).Update("Name", "Alice Updated")
db.Model(&user).Updates(User{Name: "Alice Updated", Email: "new@example.com"})
```

### Delete

```go
db.Delete(&user)
```

---

## Associations

### One-to-Many

```go
type Post struct {
    ID     uint
    Title  string
    UserID uint
}

type User struct {
    ID    uint
    Name  string
    Posts []Post
}

db.AutoMigrate(&User{}, &Post{})
```

### Example Usage

```go
user := User{Name: "Bob", Posts: []Post{{Title: "First Post"}, {Title: "Second Post"}}}
db.Create(&user)

var posts []Post
db.Model(&user).Association("Posts").Find(&posts)
```

---

## Query Builder

`GORM` provides a flexible query builder for advanced queries.

```go
var users []User
db.Where("name LIKE ?", "%Alice%").Find(&users)
db.Order("created_at desc").Limit(10).Find(&users)
```

---

## Transactions

Use transactions for atomic operations.

```go
err := db.Transaction(func(tx *gorm.DB) error {
    if err := tx.Create(&User{Name: "Charlie"}).Error; err != nil {
        return err
    }
    if err := tx.Create(&Post{Title: "Charlie's Post"}).Error; err != nil {
        return err
    }
    return nil
})
if err != nil {
    println("Transaction failed:", err)
}
```

---

## Hooks

`GORM` supports lifecycle hooks for models.

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
    println("Before creating user:", u.Name)
    return
}
```

---

## Conclusion

This tutorial covered the basics and intermediate features of `GORM`. For more advanced topics, refer to the [official documentation](https://gorm.io/docs/).
