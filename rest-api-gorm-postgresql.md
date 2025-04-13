# Building a REST API with GORM and PostgreSQL

This tutorial demonstrates how to build a REST API using GORM and PostgreSQL with a complex relational database schema. It also covers best practices for implementing pagination, filtering, sorting, and searching.

---

## Prerequisites

1. Go installed on your machine.
2. PostgreSQL installed and running.
3. Basic knowledge of Go, REST APIs, and SQL.

---

## Step 1: Initialize the Project

1. Create a new Go project:
   ```bash
   mkdir gorm-postgresql-api
   cd gorm-postgresql-api
   go mod init gorm-postgresql-api
   ```

2. Install required dependencies:
   ```bash
   go get -u gorm.io/gorm gorm.io/driver/postgres github.com/gin-gonic/gin
   ```

---

## Step 2: Define the Database Schema

We'll create a relational schema with the following entities:
- `User`: Represents a user in the system.
- `Post`: Represents a blog post authored by a user.
- `Comment`: Represents comments on posts.

### Example Schema

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/models.go
package models

import "gorm.io/gorm"

type User struct {
    ID       uint   `gorm:"primaryKey"`
    Name     string `gorm:"size:100;not null"`
    Email    string `gorm:"uniqueIndex;not null"`
    Posts    []Post
}

type Post struct {
    ID       uint   `gorm:"primaryKey"`
    Title    string `gorm:"size:200;not null"`
    Content  string `gorm:"type:text;not null"`
    UserID   uint   `gorm:"not null"`
    Comments []Comment
}

type Comment struct {
    ID      uint   `gorm:"primaryKey"`
    Content string `gorm:"type:text;not null"`
    PostID  uint   `gorm:"not null"`
}
```

---

## Step 3: Connect to PostgreSQL

Create a database connection using GORM.

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/database.go
package main

import (
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
    "log"
)

var DB *gorm.DB

func ConnectDatabase() {
    dsn := "host=localhost user=postgres password=yourpassword dbname=yourdb port=5432 sslmode=disable"
    var err error
    DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("Failed to connect to database:", err)
    }

    // Migrate the schema
    DB.AutoMigrate(&models.User{}, &models.Post{}, &models.Comment{})
}
```

---

## Step 4: Implement Pagination, Filtering, Sorting, and Searching

### Pagination

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/utils.go
package utils

import (
    "gorm.io/gorm"
)

func Paginate(page, pageSize int) func(db *gorm.DB) *gorm.DB {
    return func(db *gorm.DB) *gorm.DB {
        if page < 1 {
            page = 1
        }
        offset := (page - 1) * pageSize
        return db.Offset(offset).Limit(pageSize)
    }
}
```

### Filtering, Sorting, and Searching

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/handlers.go
package handlers

import (
    "gorm.io/gorm"
    "net/http"
    "github.com/gin-gonic/gin"
    "gorm-postgresql-api/models"
    "gorm-postgresql-api/utils"
)

func GetPosts(c *gin.Context) {
    var posts []models.Post
    db := utils.DB

    // Filtering
    userID := c.Query("user_id")
    if userID != "" {
        db = db.Where("user_id = ?", userID)
    }

    // Searching
    search := c.Query("search")
    if search != "" {
        db = db.Where("title ILIKE ? OR content ILIKE ?", "%"+search+"%", "%"+search+"%")
    }

    // Sorting
    sort := c.DefaultQuery("sort", "created_at desc")
    db = db.Order(sort)

    // Pagination
    page := c.DefaultQuery("page", "1")
    pageSize := c.DefaultQuery("page_size", "10")
    db.Scopes(utils.Paginate(page, pageSize)).Find(&posts)

    c.JSON(http.StatusOK, posts)
}
```

---

## Step 5: Set Up Routes

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/main.go
package main

import (
    "github.com/gin-gonic/gin"
    "gorm-postgresql-api/handlers"
)

func main() {
    ConnectDatabase()

    r := gin.Default()

    r.GET("/posts", handlers.GetPosts)

    r.Run(":8080")
}
```

---

## Step 6: Run the Application

1. Start the application:
   ```bash
   go run main.go
   ```

2. Test the API using tools like Postman or curl.

---

## Conclusion

This tutorial demonstrated how to build a REST API with GORM and PostgreSQL, including a complex relational schema and best practices for pagination, filtering, sorting, and searching.
