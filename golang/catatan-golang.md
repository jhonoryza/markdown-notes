# tips

- [using migration with golang](https://dev.to/wiliamvj/using-migrations-with-golang-3449)
- [alexedwards blog](https://www.alexedwards.net/blog)
- [common go mistakes](https://100go.co/)

## article

- [https://victoriametrics.com/blog/go-io-reader-writer/](https://victoriametrics.com/blog/go-io-reader-writer/)

## issue static file mime type

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

# package yang sering digunakan

- [validasi](github.com/go-playground/validator)
- [driver mysql](github.com/go-sql-driver/mysql)
- [driver postgres](github.com/jackc/pgx)
- [migration](github.com/golang-migrate/migrate/)
- [httprouter](github.com/julienschmidt/httprouter)
- [test assertion](github.com/stretchr/testify)
- [orm](gorm.io/gorm)
- [orm driver mysql](gorm.io/driver/mysql)
- [orm driver postgres](gorm.io/driver/postgres)
- [config management](github.com/spf13/viper)
- [redis](github.com/redis/go-redis)
- [logger](github.com/sirupsen/logrus)
- [fiber](github.com/gofiber/fiber)
- [load env file](github.com/joho/godotenv)

# docker configuration

## contoh Dockerfile untuk golang

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

## contoh file `docker-compose.yml`

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

## contoh isi Makefile

```Makefile
build:
	docker build -t app .
run:
	docker compose up -d
update:
	git pull origin main && make build && make run
```
