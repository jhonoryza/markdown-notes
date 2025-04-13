# Building a Todo List Web Application with GORM and Cookie Authentication

This tutorial demonstrates how to build a Todo List web application in Go using `GORM` for database operations, cookie-based authentication, and `TailwindCSS` for styling. The application will include a login-protected CRUD page for managing todos and a public page for viewing todos.

---

## Project Structure

```
todo-list-web/
├── main.go
├── config/
│   └── database.go
├── models/
│   └── user.go
│   └── todo.go
├── controllers/
│   └── auth_controller.go
│   └── todo_controller.go
├── middlewares/
│   └── auth_middleware.go
├── templates/
│   └── base.html
│   └── login.html
│   └── todos.html
│   └── view_todos.html
├── static/
│   └── css/
│       └── tailwind.css
└── routes/
    └── routes.go
```

---

## Step 1: Initialize the Project

1. Create a new Go module:
   ```bash
   go mod init todo-list-web
   ```

2. Install dependencies:
   ```bash
   go get -u gorm.io/gorm gorm.io/driver/sqlite github.com/gin-gonic/gin github.com/go-playground/validator/v10
   ```

3. Install TailwindCSS:
   ```bash
   npm install -D tailwindcss
   npx tailwindcss init
   ```

---

## Step 2: Configure the Database

Update the `config/database.go` file to use SQLite.

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/config/database.go
package config

import (
	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
	"log"
)

var DB *gorm.DB

func ConnectDatabase() {
	var err error
	DB, err = gorm.Open(sqlite.Open("todo_list.db"), &gorm.Config{})
	if (err != nil) {
		log.Fatal("Failed to connect to database:", err)
	}
}
```

---

## Step 3: Define Models

Add validation tags to the models.

### User Model

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/models/user.go
package models

import "gorm.io/gorm"

type User struct {
	gorm.Model
	Username string `gorm:"unique" json:"username" validate:"required,min=3,max=32"`
	Password string `json:"password" validate:"required,min=6"`
}
```

### Todo Model

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/models/todo.go
package models

import "gorm.io/gorm"

type Todo struct {
	gorm.Model
	Title     string `json:"title" validate:"required"`
	Completed bool   `json:"completed"`
	UserID    uint   `json:"user_id"`
}
```

---

## Step 4: Create Controllers

### Authentication Controller

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/controllers/auth_controller.go
package controllers

import (
	"net/http"
	"todo-list-web/config"
	"todo-list-web/models"

	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
)

func Login(c *gin.Context) {
	var input struct {
		Username string `json:"username" binding:"required"`
		Password string `json:"password" binding:"required"`
	}

	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	var user models.User
	if err := config.DB.Where("username = ?", input.Username).First(&user).Error; err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid username or password"})
		return
	}

	if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(input.Password)); err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid username or password"})
		return
	}

	c.SetCookie("user_id", string(user.ID), 3600, "/", "", false, true)
	c.JSON(http.StatusOK, gin.H{"message": "Login successful"})
}

func Logout(c *gin.Context) {
	c.SetCookie("user_id", "", -1, "/", "", false, true)
	c.Redirect(http.StatusFound, "/login")
}
```

### Todo Controller

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/controllers/todo_controller.go
package controllers

import (
	"net/http"
	"todo-list-web/config"
	"todo-list-web/models"

	"github.com/gin-gonic/gin"
)

func ListTodos(c *gin.Context) {
	var todos []models.Todo
	config.DB.Find(&todos)
	c.JSON(http.StatusOK, todos)
}

func ManageTodos(c *gin.Context) {
	var input models.Todo
	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	config.DB.Create(&input)
	c.JSON(http.StatusOK, input)
}
```

---

## Step 5: Add Middleware

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/middlewares/auth_middleware.go
package middlewares

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		userID, err := c.Cookie("user_id")
		if err != nil || userID == "" {
			c.Redirect(http.StatusFound, "/login")
			c.Abort()
			return
		}
		c.Next()
	}
}
```

---

## Step 6: Define Routes

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/routes/routes.go
package routes

import (
	"todo-list-web/controllers"
	"todo-list-web/middlewares"

	"github.com/gin-gonic/gin"
)

func SetupRoutes(router *gin.Engine) {
	router.GET("/login", controllers.Login)
	router.POST("/login", controllers.Login)
	router.GET("/logout", controllers.Logout)

	protected := router.Group("/")
	protected.Use(middlewares.AuthMiddleware())
	protected.GET("/todos", controllers.ListTodos)
	protected.POST("/todos", controllers.ManageTodos)
}
```

---

## Step 7: Create Templates

### Base Template

```html
<!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <link href="/static/css/tailwind.css" rel="stylesheet">
</head>
<body>
    {{ template "content" . }}
</body>
</html>
```

### Login Template

```html
<!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/templates/login.html -->
{{ define "content" }}
<div class="container mx-auto">
    <form action="/login" method="POST">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>
</div>
{{ end }}
```

---

## Step 8: Main Entry Point

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/main.go
package main

import (
	"todo-list-web/config"
	"todo-list-web/models"
	"todo-list-web/routes"

	"github.com/gin-gonic/gin"
)

func main() {
	config.ConnectDatabase()
	config.DB.AutoMigrate(&models.User{}, &models.Todo{})

	router := gin.Default()
	router.Static("/static", "./static")
	router.LoadHTMLGlob("templates/*")

	routes.SetupRoutes(router)

	router.Run(":8080")
}
```

---

## Step 9: Testing the Application

1. Start the application:
   ```bash
   go run main.go
   ```
2. Access the application at `http://localhost:8080`.