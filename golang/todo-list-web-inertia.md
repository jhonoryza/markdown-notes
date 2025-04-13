# Building a Todo List Web Application with GORM, Inertia.js, and Vue.js

This tutorial demonstrates how to build a Todo List web application in Go using `GORM` for database operations, cookie-based authentication, `Inertia.js` for server-driven SPA, `Vue.js` for the frontend, and `TailwindCSS` for styling. The application will include a login-protected CRUD page for managing todos and a public page for viewing todos.

---

## Project Structure

```
todo-list-web-inertia/
├── backend/
│   ├── main.go
│   ├── config/
│   │   └── database.go
│   ├── models/
│   │   └── user.go
│   │   └── todo.go
│   ├── controllers/
│   │   └── auth_controller.go
│   │   └── todo_controller.go
│   ├── middlewares/
│   │   └── auth_middleware.go
│   └── routes/
│       └── routes.go
├── frontend/
│   ├── src/
│   │   ├── App.vue
│   │   ├── main.js
│   │   ├── components/
│   │   │   └── TodoList.vue
│   │   │   └── TodoForm.vue
│   │   │   └── Login.vue
│   │   │   └── PublicTodos.vue
│   └── tailwind.config.js
├── Dockerfile
└── docker-compose.yaml
```

---

## Step 1: Backend Setup

1. Initialize the Go module:
   ```bash
   go mod init todo-list-web-inertia
   ```

2. Install dependencies:
   ```bash
   go get -u gorm.io/gorm gorm.io/driver/postgres github.com/gin-gonic/gin github.com/go-playground/validator/v10 github.com/inertiajs/inertia-go
   ```

3. Update the `config/database.go` file to use PostgreSQL:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/config/database.go
   package config

   import (
       "gorm.io/driver/postgres"
       "gorm.io/gorm"
       "log"
       "os"
   )

   var DB *gorm.DB

   func ConnectDatabase() {
       var err error
       dsn := "host=" + os.Getenv("DB_HOST") +
           " user=" + os.Getenv("DB_USER") +
           " password=" + os.Getenv("DB_PASSWORD") +
           " dbname=" + os.Getenv("DB_NAME") +
           " port=" + os.Getenv("DB_PORT") +
           " sslmode=disable TimeZone=Asia/Jakarta"
       DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
       if err != nil {
           log.Fatal("Failed to connect to database:", err)
       }
   }
   ```

4. Define the models with validation:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/models/user.go
   package models

   import "gorm.io/gorm"

   type User struct {
       gorm.Model
       Username string `gorm:"unique" json:"username" validate:"required,min=3,max=32"`
       Password string `json:"password" validate:"required,min=6"`
   }
   ```

   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/models/todo.go
   package models

   import "gorm.io/gorm"

   type Todo struct {
       gorm.Model
       Title     string `json:"title" validate:"required"`
       Completed bool   `json:"completed"`
       UserID    uint   `json:"user_id"`
   }
   ```

5. Create controllers for authentication and todos:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/controllers/auth_controller.go
   package controllers

   import (
       "net/http"
       "todo-list-web-inertia/backend/config"
       "todo-list-web-inertia/backend/models"

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
           c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
           return
       }

       if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(input.Password)); err != nil {
           c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
           return
       }

       c.SetCookie("user_id", string(user.ID), 3600, "/", "", false, true)
       c.JSON(http.StatusOK, gin.H{"message": "Logged in successfully"})
   }

   func Logout(c *gin.Context) {
       c.SetCookie("user_id", "", -1, "/", "", false, true)
       c.JSON(http.StatusOK, gin.H{"message": "Logged out successfully"})
   }
   ```

   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/controllers/todo_controller.go
   package controllers

   import (
       "net/http"
       "todo-list-web-inertia/backend/config"
       "todo-list-web-inertia/backend/models"

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

6. Add middleware for authentication:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/middlewares/auth_middleware.go
   package middlewares

   import (
       "net/http"

       "github.com/gin-gonic/gin"
   )

   func AuthMiddleware() gin.HandlerFunc {
       return func(c *gin.Context) {
           userID, err := c.Cookie("user_id")
           if err != nil || userID == "" {
               c.JSON(http.StatusUnauthorized, gin.H{"error": "Unauthorized"})
               c.Abort()
               return
           }
           c.Next()
       }
   }
   ```

7. Define routes:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/routes/routes.go
   package routes

   import (
       "todo-list-web-inertia/backend/controllers"
       "todo-list-web-inertia/backend/middlewares"

       "github.com/gin-gonic/gin"
   )

   func SetupRoutes(router *gin.Engine) {
       router.GET("/public-todos", controllers.ListTodos) // Public endpoint
       router.POST("/login", controllers.Login)
       router.POST("/logout", controllers.Logout)

       auth := router.Group("/")
       auth.Use(middlewares.AuthMiddleware())
       auth.GET("/todos", controllers.ListTodos)
       auth.POST("/todos", controllers.ManageTodos)
   }
   ```

8. Create the main entry point:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/main.go
   package main

   import (
       "todo-list-web-inertia/backend/config"
       "todo-list-web-inertia/backend/models"
       "todo-list-web-inertia/backend/routes"

       "github.com/gin-gonic/gin"
   )

   func main() {
       config.ConnectDatabase()
       config.DB.AutoMigrate(&models.User{}, &models.Todo{})

       router := gin.Default()
       router.Static("/static", "./static")

       routes.SetupRoutes(router)

       router.Run(":8080")
   }
   ```

---

## Step 2: Frontend Setup

1. Initialize the Vue.js project:
   ```bash
   npm init vue@latest frontend
   cd frontend
   npm install
   ```

2. Install dependencies:
   ```bash
   npm install @inertiajs/inertia @inertiajs/inertia-vue3 tailwindcss
   ```

3. Configure TailwindCSS:
   ```bash
   npx tailwindcss init
   ```

4. Create Vue components for Todo List and Form:
   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/components/TodoList.vue -->
   <template>
       <div>
           <h1 class="text-2xl font-bold mb-4">Todo List</h1>
           <ul>
               <li v-for="todo in todos" :key="todo.id" class="mb-2">
                   <span :class="{ 'line-through': todo.completed }">{{ todo.title }}</span>
                   <button @click="toggleComplete(todo)" class="ml-2 text-blue-500">Toggle</button>
               </li>
           </ul>
       </div>
   </template>

   <script>
   export default {
       props: {
           todos: Array,
       },
       methods: {
           toggleComplete(todo) {
               this.$emit('toggle-complete', todo);
           },
       },
   };
   </script>
   ```

   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/components/TodoForm.vue -->
   <template>
       <div>
           <h1 class="text-2xl font-bold mb-4">Add Todo</h1>
           <form @submit.prevent="submit">
               <input
                   v-model="title"
                   type="text"
                   placeholder="Todo title"
                   class="border p-2 mb-4 w-full"
               />
               <button type="submit" class="bg-blue-500 text-white px-4 py-2">Add</button>
           </form>
       </div>
   </template>

   <script>
   export default {
       data() {
           return {
               title: '',
           };
       },
       methods: {
           submit() {
               this.$emit('add-todo', { title: this.title, completed: false });
               this.title = '';
           },
       },
   };
   </script>
   ```

---

## Step 2.1: Implementing the Login Page

To create a login page, we will use Vue.js and Inertia.js to handle the frontend logic. The login page will send a POST request to the `/login` endpoint and handle the response accordingly.

1. Create a `Login.vue` component:
   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/components/Login.vue -->
   <template>
       <div class="max-w-md mx-auto mt-10">
           <h1 class="text-2xl font-bold mb-4">Login</h1>
           <form @submit.prevent="submit">
               <div class="mb-4">
                   <label for="username" class="block text-sm font-medium">Username</label>
                   <input
                       v-model="username"
                       id="username"
                       type="text"
                       class="border p-2 w-full"
                       required
                   />
               </div>
               <div class="mb-4">
                   <label for="password" class="block text-sm font-medium">Password</label>
                   <input
                       v-model="password"
                       id="password"
                       type="password"
                       class="border p-2 w-full"
                       required
                   />
               </div>
               <button type="submit" class="bg-blue-500 text-white px-4 py-2">Login</button>
           </form>
           <p v-if="error" class="text-red-500 mt-4">{{ error }}</p>
       </div>
   </template>

   <script>
   import { Inertia } from '@inertiajs/inertia';

   export default {
       data() {
           return {
               username: '',
               password: '',
               error: null,
           };
       },
       methods: {
           async submit() {
               try {
                   await Inertia.post('/login', {
                       username: this.username,
                       password: this.password,
                   });
               } catch (err) {
                   this.error = 'Invalid username or password.';
               }
           },
       },
   };
   </script>
   ```

2. Update the `App.vue` file to include a route for the login page:
   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/App.vue -->
   <template>
       <router-view />
   </template>

   <script>
   import { createRouter, createWebHistory } from 'vue-router';
   import Login from './components/Login.vue';
   import TodoList from './components/TodoList.vue';

   const routes = [
       { path: '/login', component: Login },
       { path: '/todos', component: TodoList },
   ];

   const router = createRouter({
       history: createWebHistory(),
       routes,
   });

   export default {
       router,
   };
   </script>
   ```

---

## Step 2.2: Implementing the Public Page for Viewing Todos

To create a public page for viewing todos, we will add a new route and a Vue.js component.

1. Update the backend routes to include a public endpoint for fetching todos:
   ```go
   // filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/backend/routes/routes.go
   package routes

   import (
       "todo-list-web-inertia/backend/controllers"
       "todo-list-web-inertia/backend/middlewares"

       "github.com/gin-gonic/gin"
   )

   func SetupRoutes(router *gin.Engine) {
       router.GET("/public-todos", controllers.ListTodos) // Public endpoint
       router.POST("/login", controllers.Login)
       router.POST("/logout", controllers.Logout)

       auth := router.Group("/")
       auth.Use(middlewares.AuthMiddleware())
       auth.GET("/todos", controllers.ListTodos)
       auth.POST("/todos", controllers.ManageTodos)
   }
   ```

2. Create a `PublicTodos.vue` component:
   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/components/PublicTodos.vue -->
   <template>
       <div>
           <h1 class="text-2xl font-bold mb-4">Public Todos</h1>
           <ul>
               <li v-for="todo in todos" :key="todo.id" class="mb-2">
                   <span :class="{ 'line-through': todo.completed }">{{ todo.title }}</span>
               </li>
           </ul>
       </div>
   </template>

   <script>
   import axios from 'axios';

   export default {
       data() {
           return {
               todos: [],
           };
       },
       async created() {
           const response = await axios.get('/public-todos');
           this.todos = response.data;
       },
   };
   </script>
   ```

3. Update the `App.vue` file to include a route for the public todos page:
   ```vue
   <!-- filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/frontend/src/App.vue -->
   <template>
       <router-view />
   </template>

   <script>
   import { createRouter, createWebHistory } from 'vue-router';
   import Login from './components/Login.vue';
   import TodoList from './components/TodoList.vue';
   import PublicTodos from './components/PublicTodos.vue';

   const routes = [
       { path: '/login', component: Login },
       { path: '/todos', component: TodoList },
       { path: '/public-todos', component: PublicTodos },
   ];

   const router = createRouter({
       history: createWebHistory(),
       routes,
   });

   export default {
       router,
   };
   </script>
   ```

---

## Step 3: Dockerize the Application

1. Create a `Dockerfile`:
   ```dockerfile
   # filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/Dockerfile
   FROM golang:1.20 AS builder
   WORKDIR /app
   COPY backend/ .
   RUN go build -o main .

   FROM node:18 AS frontend
   WORKDIR /app
   COPY frontend/ .
   RUN npm install && npm run build

   FROM alpine:latest
   WORKDIR /app
   COPY --from=builder /app/main .
   COPY --from=frontend /app/dist ./frontend
   CMD ["./main"]
   ```

2. Update the `docker-compose.yaml` file to include a PostgreSQL service:
   ```yaml
   # filepath: /Users/fajar/Documents/github-projects/markdown-notes/golang/todo-list-web-inertia/docker-compose.yaml
   version: "3.9"
   services:
     app:
       build: .
       ports:
         - "8080:8080"
       environment:
         - DB_HOST=db
         - DB_USER=postgres
         - DB_PASSWORD=postgres
         - DB_NAME=todo_list
         - DB_PORT=5432
         - GIN_MODE=release
       depends_on:
         - db
     db:
       image: postgres:15
       environment:
         POSTGRES_USER: postgres
         POSTGRES_PASSWORD: postgres
         POSTGRES_DB: todo_list
       ports:
         - "5432:5432"
       volumes:
         - postgres_data:/var/lib/postgresql/data
   volumes:
     postgres_data:
   ```