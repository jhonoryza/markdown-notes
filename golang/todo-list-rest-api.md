# Building a Todo List REST API with GORM and JWT Authentication

This tutorial demonstrates how to build a Todo List REST API in Go using `GORM` for database operations and `JWT` for authentication. We'll follow best practices for structuring the project, including validation using `go-playground/validator`.

---

## Project Structure

```
todo-list-rest-api/
├── main.go
├── config/
│   └── database.go
├── models/
│   └── user.go
│   └── todo.go
├── controllers/
│   └── auth_controller.go
│   └── todo_controller.go
├── routes/
│   └── routes.go
└── middlewares/
    └── jwt_middleware.go
```

---

## Step 1: Initialize the Project

1. Create a new Go module:
   ```bash
   go mod init todo-list-rest-api
   ```

2. Install dependencies:
   ```bash
   go get -u gorm.io/gorm gorm.io/driver/postgres github.com/golang-jwt/jwt/v4 github.com/gin-gonic/gin github.com/go-playground/validator/v10
   ```

---

## Step 2: Configure the Database

Update the `config/database.go` file to use PostgreSQL.

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/config/database.go
package config

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"log"
	"os"
)

var DB *gorm.DB

func ConnectDatabase() {
	dsn := "host=localhost user=postgres password=yourpassword dbname=todo_list port=5432 sslmode=disable TimeZone=Asia/Jakarta"
	var err error
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database:", err)
	}
}
```

---

## Step 3: Define Models

Add validation tags to the models using `go-playground/validator`.

### User Model

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/models/user.go
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
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/models/todo.go
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

Update the controllers to include validation using `go-playground/validator`.

### Authentication Controller

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/controllers/auth_controller.go
package controllers

import (
	"net/http"
	"time"
	"todo-list-rest-api/config"
	"todo-list-rest-api/models"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v4"
	"golang.org/x/crypto/bcrypt"
	"github.com/go-playground/validator/v10"
)

var jwtKey = []byte("secret_key")
var validate = validator.New()

func Register(c *gin.Context) {
	var user models.User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	if err := validate.Struct(user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"validation_error": err.Error()})
		return
	}

	hashedPassword, _ := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	user.Password = string(hashedPassword)

	config.DB.Create(&user)
	c.JSON(http.StatusOK, gin.H{"message": "User registered successfully"})
}

func Login(c *gin.Context) {
	var user models.User
	var input models.User

	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	config.DB.Where("username = ?", input.Username).First(&user)
	if user.ID == 0 || bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(input.Password)) != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
		"user_id": user.ID,
		"exp":     time.Now().Add(time.Hour * 24).Unix(),
	})

	tokenString, _ := token.SignedString(jwtKey)
	c.JSON(http.StatusOK, gin.H{"token": tokenString})
}
```

### Todo Controller

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/controllers/todo_controller.go
package controllers

import (
	"net/http"
	"todo-list-rest-api/config"
	"todo-list-rest-api/models"

	"github.com/gin-gonic/gin"
	"github.com/go-playground/validator/v10"
)

var validate = validator.New()

func GetTodos(c *gin.Context) {
	var todos []models.Todo
	userID := c.MustGet("user_id").(uint)

	config.DB.Where("user_id = ?", userID).Find(&todos)
	c.JSON(http.StatusOK, todos)
}

func CreateTodo(c *gin.Context) {
	var todo models.Todo
	userID := c.MustGet("user_id").(uint)

	if err := c.ShouldBindJSON(&todo); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	if err := validate.Struct(todo); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"validation_error": err.Error()})
		return
	}

	todo.UserID = userID
	config.DB.Create(&todo)
	c.JSON(http.StatusOK, todo)
}
```

---

## Step 5: Add Middleware

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/middlewares/jwt_middleware.go
package middlewares

import (
	"net/http"
	"strings"
	"todo-list-rest-api/config"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v4"
)

var jwtKey = []byte("secret_key")

func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		authHeader := c.GetHeader("Authorization")
		if !strings.HasPrefix(authHeader, "Bearer ") {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Unauthorized"})
			c.Abort()
			return
		}

		tokenString := strings.TrimPrefix(authHeader, "Bearer ")
		token, _ := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			return jwtKey, nil
		})

		if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
			c.Set("user_id", uint(claims["user_id"].(float64)))
		} else {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Unauthorized"})
			c.Abort()
		}
	}
}
```

---

## Step 6: Define Routes

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/routes/routes.go
package routes

import (
	"todo-list-rest-api/controllers"
	"todo-list-rest-api/middlewares"

	"github.com/gin-gonic/gin"
)

func SetupRoutes(router *gin.Engine) {
	router.POST("/register", controllers.Register)
	router.POST("/login", controllers.Login)

	protected := router.Group("/todos")
	protected.Use(middlewares.AuthMiddleware())
	{
		protected.GET("/", controllers.GetTodos)
		protected.POST("/", controllers.CreateTodo)
	}
}
```

---

## Step 7: Main Entry Point

Update the database connection string in the `main.go` file.

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/main.go
package main

import (
	"todo-list-rest-api/config"
	"todo-list-rest-api/models"
	"todo-list-rest-api/routes"

	"github.com/gin-gonic/gin"
)

func main() {
	config.ConnectDatabase()
	config.DB.AutoMigrate(&models.User{}, &models.Todo{})

	router := gin.Default()
	routes.SetupRoutes(router)

	router.Run(":8080")
}
```

---

## Step 8: Containerize the Application

To deploy the application in a production-ready environment, we will create a `Dockerfile` and a `docker-compose.yaml` file.

### Dockerfile

Create a `Dockerfile` in the root directory of the project:

```dockerfile
# Use the official Golang image as the base image
FROM golang:1.20 as builder

# Set the working directory
WORKDIR /app

# Copy the Go modules manifests
COPY go.mod go.sum ./

# Download Go modules
RUN go mod download

# Copy the source code
COPY . .

# Build the application
RUN go build -o main .

# Use a minimal image for the final build
FROM alpine:latest

# Set the working directory
WORKDIR /root/

# Copy the binary from the builder stage
COPY --from=builder /app/main .

# Expose the application port
EXPOSE 8080

# Command to run the application
CMD ["./main"]
```

### docker-compose.yaml

Create a `docker-compose.yaml` file to define the services, including the PostgreSQL database:

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=yourpassword
      - DB_NAME=todo_list
      - DB_PORT=5432
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD=yourpassword
      POSTGRES_DB: todo_list
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Update the Database Configuration

Modify the `dsn` in `config/database.go` to use environment variables:

```go
// filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-rest-api/config/database.go
// ...existing code...
import "github.com/joho/godotenv"

func ConnectDatabase() {
	godotenv.Load()
	dsn := os.Getenv("DB_HOST") + " user=" + os.Getenv("DB_USER") +
		" password=" + os.Getenv("DB_PASSWORD") + " dbname=" + os.Getenv("DB_NAME") +
		" port=" + os.Getenv("DB_PORT") + " sslmode=disable TimeZone=Asia/Jakarta"
	// ...existing code...
}
```

### Build and Run the Application

1. Build the Docker images:
   ```bash
   docker-compose build
   ```

2. Start the containers:
   ```bash
   docker-compose up
   ```

3. Access the API at `http://localhost:8080`.

---

## Testing the API

1. Start the PostgreSQL server and create a database named `todo_list`.
2. Update the `dsn` in `config/database.go` with your PostgreSQL credentials.
3. Start the API server:
   ```bash
   go run main.go
   ```
4. Use tools like `Postman` or `curl` to test the endpoints.
