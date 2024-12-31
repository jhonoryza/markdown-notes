# Tips

- [Using Migrations with Golang](https://dev.to/wiliamvj/using-migrations-with-golang-3449)
- [Alex Edwards Blog](https://www.alexedwards.net/blog)
- [Common Go Mistakes](https://100go.co/)

## Articles

- [Go io.Reader and io.Writer](https://victoriametrics.com/blog/go-io-reader-writer/)

## Issue with Static File MIME Type

To set the correct MIME type for static files in a Fiber application, use the following code:

```go
router := fiber.New()

router.All("/build/*.js", func(ctx *fiber.Ctx) error {
	ctx.Set(fiber.HeaderContentType, "text/javascript")
	return ctx.Next()
})
router.All("/build/*.css", func(ctx *fiber.Ctx) error {
	ctx.Set(fiber.HeaderContentType, "text/css")
	return ctx.Next()
})
router.Static("/build", "./public/build")
```

# Frequently Used Packages

- [Validation](github.com/go-playground/validator)
- [MySQL Driver](github.com/go-sql-driver/mysql)
- [Postgres Driver](github.com/jackc/pgx)
- [Migration](github.com/golang-migrate/migrate/)
- [HTTP Router](github.com/julienschmidt/httprouter)
- [Test Assertion](github.com/stretchr/testify)
- [ORM](gorm.io/gorm)
- [ORM Driver MySQL](gorm.io/driver/mysql)
- [ORM Driver Postgres](gorm.io/driver/postgres)
- [Config Management](github.com/spf13/viper)
- [Redis](github.com/redis/go-redis)
- [Logger](github.com/sirupsen/logrus)
- [Fiber](github.com/gofiber/fiber)
- [Load Env File](github.com/joho/godotenv)

# Docker Configuration

## Example Dockerfile for Golang

```Dockerfile
# Stage 1: Build stage
FROM golang:1.22-alpine AS build

# Set the working directory
WORKDIR /app

# Copy and download dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the source code
COPY . .

# Build the Go application
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

# Stage 2: Final stage
FROM alpine:edge

# Set the working directory
WORKDIR /app

# Copy the binary from the build stage
COPY --from=build /app/myapp .

# Set the timezone and install CA certificates
RUN apk --no-cache add ca-certificates tzdata

# Set the entrypoint command
ENTRYPOINT ["/app/myapp"]

EXPOSE 8080
```

## Example `docker-compose.yml` File

```yaml
version: "3"

services:
    app:
        image: "app"
        container_name: "app"
        restart: unless-stopped
        ports:
            - 10200:8080
        env_file:
            - ./.env
```

## Example Makefile

```Makefile
build:
	docker build -t app .
run:
	docker compose up -d
update:
	git pull origin main && make build && make run
```
