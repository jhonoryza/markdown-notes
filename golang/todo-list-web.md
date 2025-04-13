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

Update the `config/database.go` file to use PostgreSQL. Ensure you have PostgreSQL installed and running on your system. Replace the connection details (`host`, `user`, `password`, `dbname`, and `port`) with your PostgreSQL configuration.

Install the PostgreSQL driver for GORM:
```bash
go get -u gorm.io/driver/postgres
```

Update the `config/database.go` file:

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/config/database.go
package config

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"log"
)

var DB *gorm.DB

func ConnectDatabase() {
	var err error
	dsn := "host=localhost user=your_user password=your_password dbname=todo_list port=5432 sslmode=disable TimeZone=Asia/Jakarta"
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
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

### Todos Template

The `todos.html` template is used for managing todos. It includes a form to add new todos and a list to display existing todos. Each todo shows its title and status (completed or pending).

```html
<!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/templates/todos.html -->
{{ define "content" }}
<div class="container mx-auto">
    <h1 class="text-2xl font-bold">Manage Todos</h1>
    <form action="/todos" method="POST" class="mt-4">
        <input type="text" name="title" placeholder="Todo Title" required class="border p-2">
        <button type="submit" class="bg-blue-500 text-white px-4 py-2">Add Todo</button>
    </form>
    <ul class="mt-4">
        {{ range . }}
        <li class="border-b py-2">
            <span>{{ .Title }}</span>
            {{ if .Completed }}
            <span class="text-green-500">(Completed)</span>
            {{ else }}
            <span class="text-red-500">(Pending)</span>
            {{ end }}
        </li>
        {{ end }}
    </ul>
</div>
{{ end }}
```

### View Todos Template

The `view_todos.html` template is used for displaying todos to the public. It lists all todos with their titles and statuses (completed or pending).

```html
<!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/templates/view_todos.html -->
{{ define "content" }}
<div class="container mx-auto">
    <h1 class="text-2xl font-bold">View Todos</h1>
    <ul class="mt-4">
        {{ range . }}
        <li class="border-b py-2">
            <span>{{ .Title }}</span>
            {{ if .Completed }}
            <span class="text-green-500">(Completed)</span>
            {{ else }}
            <span class="text-red-500">(Pending)</span>
            {{ end }}
        </li>
        {{ end }}
    </ul>
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

---

## Step 10: Containerize the Application for Production

To make the application production-ready, we will containerize it using Docker. This involves creating a `Dockerfile` to define the application image and a `docker-compose.yaml` file to orchestrate the container.

### Dockerfile

The `Dockerfile` is used to build a Docker image for the application. It uses a multi-stage build process:

1. **Builder Stage**: Uses the official Golang image to compile the application.
2. **Production Stage**: Uses a minimal Alpine image to run the compiled application.

```dockerfile
# filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/Dockerfile
# Use the official Golang image to build the application
FROM golang:1.20 as builder

WORKDIR /app

# Copy go.mod and go.sum files
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download

# Copy the source code
COPY . .

# Build the application
RUN go build -o main .

# Use a minimal image for production
FROM alpine:latest

WORKDIR /root/

# Install necessary dependencies
RUN apk --no-cache add ca-certificates

# Copy the built application from the builder
COPY --from=builder /app/main .
COPY --from=builder /app/templates ./templates
COPY --from=builder /app/static ./static

# Expose the application port
EXPOSE 8080

# Run the application
CMD ["./main"]
```

### Docker Compose

The `docker-compose.yaml` file is used to define and run the application container. It maps port `8080` on the host to the container and ensures the application restarts automatically if it stops unexpectedly.

```yaml
# filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web/docker-compose.yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    volumes:
      - ./templates:/root/templates
      - ./static:/root/static
    environment:
      - GIN_MODE=release
    restart: unless-stopped
```

### Running the Application with Docker

1. Build the Docker image:
   ```bash
   docker-compose build
   ```

2. Start the application:
   ```bash
   docker-compose up -d
   ```

3. Access the application at `http://localhost:8080`.

---