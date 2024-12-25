# Redis Queue and Cron in Go

In this tutorial we will interacts with queue and put it to a redis server using
`github.com/hibiken/asynq` package and create a scheduler for a scheduled task
using `github.com/robfig/cron` package.

First we need to init a module

```bash
go mod init learn_queue_and_cron
```

create `cron.go` file

```go
package main

import (
	"fmt"
	"log"
	"time"

	"github.com/robfig/cron/v3"
)

func runCron(c *cron.Cron) {

	// Schedule a task to run every minute
	_, err := c.AddFunc("@every 1m", func() {
		fmt.Printf("Task executed every minute at: %v \n", time.Now().Local())
	})
	if err != nil {
		log.Fatal(err)
	}

	// Start the cron scheduler
	c.Start()
	log.Println("Cron scheduler started")

	// Keep the main goroutine running
	select {}
}
```

create `queue.go` file

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"

	"github.com/hibiken/asynq"
)

func runQueue(server *asynq.Server) {
	mux := asynq.NewServeMux()
	mux.HandleFunc("send_email", emailHandler)
	mux.HandleFunc("generate_report", reportHandler)

	if err := server.Run(mux); err != nil {
		log.Fatalf("Failed to run Asynq server: %v", err)
	}
}

func emailHandler(ctx context.Context, task *asynq.Task) error {
	var payload struct {
		To string `json:"to"`
	}
	if err := json.Unmarshal(task.Payload(), &payload); err != nil {
		return fmt.Errorf("failed to unmarshal payload: %w", err)
	}
	fmt.Printf("Sending email to: %s\n", payload.To)
	return nil
}

func reportHandler(ctx context.Context, task *asynq.Task) error {
	var payload struct {
		ReportID int `json:"report_id"`
	}
	if err := json.Unmarshal(task.Payload(), &payload); err != nil {
		return fmt.Errorf("failed to unmarshal payload: %w", err)
	}
	fmt.Printf("Generating report for ID: %d\n", payload.ReportID)
	return nil
}
```

create `router.go` file

```go
package main

import (
	"encoding/json"
	"net/http"

	"github.com/gin-gonic/gin"
	"github.com/hibiken/asynq"
)

func setupRouter(client *asynq.Client) *gin.Engine {
	r := gin.Default()

	r.POST("/enqueue/email", func(c *gin.Context) {
		var payload struct {
			To string `json:"to"`
		}
		if err := c.ShouldBindJSON(&payload); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request body"})
			return
		}

		jsonPayload, err := json.Marshal(payload)
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to marshal payload"})
			return
		}

		task := asynq.NewTask("send_email", jsonPayload)
		_, err = client.Enqueue(task)
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to enqueue task"})
			return
		}

		c.JSON(http.StatusOK, gin.H{"message": "Email job enqueued"})
	})

	r.POST("/enqueue/report", func(c *gin.Context) {
		var payload struct {
			ReportID int `json:"report_id"`
		}
		if err := c.ShouldBindJSON(&payload); err != nil {
			c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request body"})
			return
		}

		jsonPayload, err := json.Marshal(payload)
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to marshal payload"})
			return
		}

		task := asynq.NewTask("generate_report", jsonPayload)
		_, err = client.Enqueue(task)
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{"error": "Failed to enqueue task"})
			return
		}

		c.JSON(http.StatusOK, gin.H{"message": "Report job enqueued"})
	})

	return r
}
```

create `main.go` file

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/hibiken/asynq"
	"github.com/robfig/cron/v3"
)

func main() {
	c := cron.New()

	server := asynq.NewServer(
		asynq.RedisClientOpt{Addr: "localhost:6379"},
		asynq.Config{
			Concurrency: 10,
		},
	)

	client := asynq.NewClient(asynq.RedisClientOpt{Addr: "localhost:6379"})
	defer client.Close()

	router := setupRouter(client)

	httpServer := &http.Server{
		Addr:    ":8080",
		Handler: router,
	}

	// prepare shutdown context
	ctx, stop := context.WithCancel(context.Background())
	defer stop()
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, os.Interrupt, syscall.SIGTERM)

	go runQueue(server)
	go runCron(c)
	go func() {
		if err := httpServer.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Fatalf("Failed to run HTTP server: %v", err)
		}
	}()

	appShutdown(ctx, httpServer, c, server, quit)
}

func appShutdown(ctx context.Context, httpServer *http.Server, c *cron.Cron, server *asynq.Server, quit chan os.Signal) {
	// Wait for termination signal
	<-quit
	log.Println("Shutting down gracefully...")

	httpCtx, httpCancel := context.WithTimeout(ctx, 5*time.Second)
	defer httpCancel()
	if err := httpServer.Shutdown(httpCtx); err != nil {
		log.Printf("HTTP server shutdown error: %v", err)
	}

	server.Shutdown()
	c.Stop()

	log.Println("Application stopped")
}
```

install all depedency

```bash
go mod tidy
```

build it using this command

```bash
go build -o run *.go && ./run
```

try to open the browser

- [http://localhost:8080/enqueue/email](http://localhost:8080/enqueue/email)
- [http://localhost:8080/enqueue/report](http://localhost:8080/enqueue/report)

then watch the terminal output
