# Redis Queue in Nestjs

## Overview

Redis Queue is a powerful way to manage background tasks and messaging in a
distributed system. By using BullMQ with NestJS, we can create robust and
scalable queues for processing tasks asynchronously. In this guide, you'll learn
how to set up a Redis queue in a NestJS application.

## 1. Install Required Packages

To begin, install the required dependencies:

```bash
npm i @nestjs/bullmq bullmq
```

These packages include:

- **@nestjs/bullmq**: NestJS integration for BullMQ.
- **bullmq**: A high-performance Node.js library for managing Redis-based
  queues.

## 2. Configure `app.module.ts`

Update the main application module to include the BullMQ configuration:

```js
import { Module } from "@nestjs/common";
import { AppsExampleModule } from "./example/apps.example.module";

@Module({
    imports: [
        BullModule.forRoot({
            connection: {
                host: "localhost",
                port: 6379,
            },
        }),
        AppsExampleModule,
    ],
})
export class AppsModule {}
```

### Explanation

- **BullModule.forRoot**: Configures the global BullMQ connection to the Redis
  server.
- **host** and **port**: Define the Redis server's connection details.
- **AppsExampleModule**: A feature module that contains the queue-related logic.

## 3. Create the Feature Module

Create the `example/apps.example.module.ts` file to organize queue
functionality:

```js
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bullmq';
import { AppsExampleConsumer } from './consumers/apps.example.consumer';
import { AppsExampleController } from './controllers/apps.example.controller';

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'default',
      prefix: 'example',
    }),
  ],
  providers: [AppsExampleConsumer],
  controllers: [AppsExampleController],
})
export class AppsExampleModule {}
```

### Explanation

- **BullModule.registerQueue**: Registers a queue named `default` with the
  prefix `example`.
- **AppsExampleConsumer**: A class responsible for processing jobs in the queue.
- **AppsExampleController**: A controller to handle API requests and add jobs to
  the queue.

## 4. Define the Controller

Create the `example/controllers/apps.example.controller.ts` file:

```js
import {
  Controller,
  Get,
} from '@nestjs/common';
import { InjectQueue } from '@nestjs/bullmq';
import { Queue } from 'bullmq';

@Controller({ path: 'example' })
export class AppsExampleController {
  constructor(@InjectQueue('default') private queue: Queue) {}

  @Get('/')
  async test() {
    const job = await this.queue.add('data', {
      foo: 'bar',
    }, {
      attempts: 3,
      delay: 3000,
    });
    return job;
  }
}
```

### Explanation

- **@InjectQueue('default')**: Injects the queue named `default`.
- **queue.add**: Adds a job named `data` to the queue with retry attempts and a
  delay.
- **@Get('/')**: Defines an API endpoint at `http://localhost:3000/example` to
  trigger job creation.

## 5. Create the Consumer

Create the `example/consumers/app.example.consumer.ts` file to process jobs:

```js
import { Processor, WorkerHost } from '@nestjs/bullmq';
import { Job } from 'bullmq';
import { Logger } from '@nestjs/common';

@Processor('default')
export class AppsExampleConsumer extends WorkerHost {
  private readonly logger = new Logger(AppsExampleConsumer.name);

  async process(job: Job<any, any, string>): Promise<any> {
    switch (job.name) {
      case 'data': {
        this.logger.log(`Processing job data: ${JSON.stringify(job.data)}`);
        this.logger.log(`Progress: ${job.progress}`);
        this.logger.log(`Job ID: ${job.id}`);
        break;
      }
      default: {
        this.logger.warn(`Unhandled job: ${job.name}`);
        break;
      }
    }
    return {};
  }
}
```

### Explanation

- **@Processor('default')**: Registers the consumer for the `default` queue.
- **WorkerHost**: A base class that provides functionality to process jobs.
- **process(job)**: Handles job processing logic based on the job's name.

## 6. Run the Application

Start the application:

```bash
npm run start
```

This will connect the application to the Redis server and initialize the queue.

## 7. Test the Setup

Access the API endpoint to add a job:

[http://localhost:3000/example](http://localhost:3000/example)

### Expected Behavior

1. The job is added to the queue.
2. The `AppsExampleConsumer` logs the job details and processes it.

### Example Output in Logs

```plaintext
Processing job data: {"foo":"bar"}
Progress: 0
Job ID: abc123
```
