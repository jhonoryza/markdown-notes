# Dynamic Cron Jobs in NestJS

NestJS memungkinkan untuk membuat cron job yang dinamis dengan interval yang dapat diubah sesuai kebutuhan. Guide ini akan menjelaskan cara membuat cron job dengan interval yang dapat diubah melalui database dan API.

## Prerequisites

Install paket yang diperlukan:

```bash
npm install --save @nestjs/schedule
npm install --save-dev @types/cron
```

## Implementasi

### 1. Setup Module

```typescript
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [
    ScheduleModule.forRoot(),
    // ...other modules
  ],
})
export class AppModule {}
```

### 2. Implementasi Cron Service

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { CronJob } from 'cron';
import { SchedulerRegistry } from '@nestjs/schedule';

@Injectable()
export class CronService {
  private readonly logger = new Logger(CronService.name);

  constructor(private schedulerRegistry: SchedulerRegistry) {}

  addOrUpdateCronJob(name: string, seconds: number) {
    const existingJob = this.schedulerRegistry.getCronJob(name);
    if (existingJob) {
      existingJob.stop();
      this.schedulerRegistry.deleteCronJob(name);
      this.logger.warn(`Cron job ${name} dihapus`);
    }

    const job = new CronJob(`*/${seconds} * * * * *`, () => {
      this.logger.debug(`Cron job ${name} dieksekusi setiap ${seconds} detik`);
      // Tambahkan logika task disini
    });

    this.schedulerRegistry.addCronJob(name, job);
    job.start();

    this.logger.warn(`Cron job ${name} ditambahkan. Interval: ${seconds} detik`);
  }
}
```

### 3. Implementasi Controller

```typescript
import { Controller, Post, Body } from '@nestjs/common';
import { CronService } from './cron.service';

@Controller('cron')
export class CronController {
  constructor(private readonly cronService: CronService) {}

  @Post('update')
  updateCronJob(@Body('name') name: string, @Body('duration') duration: number) {
    this.cronService.addOrUpdateCronJob(name, duration);
    return {
      message: `Cron job ${name} diperbarui. Interval: ${duration} detik`,
    };
  }
}
```

### 4. Integrasi dengan Database

```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { CronService } from './cron.service';
import { AppConfigService } from './app-config.service';

@Injectable()
export class AppService implements OnModuleInit {
  constructor(
    private readonly cronService: CronService,
    private readonly appConfigService: AppConfigService,
  ) {}

  async onModuleInit() {
    const cronConfig = await this.appConfigService.getConfig('cronTask');
    if (cronConfig) {
      this.cronService.addOrUpdateCronJob(cronConfig.name, cronConfig.duration);
    }
  }
}
```

## Schema Database

Contoh schema untuk menyimpan konfigurasi cron:

```sql
CREATE TABLE app_config (
  id SERIAL PRIMARY KEY,
  name VARCHAR(50) NOT NULL UNIQUE,
  duration INTEGER NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Penggunaan

1. Update interval cron melalui API:
```bash
curl -X POST http://localhost:3000/cron/update \
  -H "Content-Type: application/json" \
  -d '{"name":"taskName","duration":3600}'
```

2. Interval akan tersimpan di database dan cron job akan langsung menyesuaikan dengan interval baru.

## Tips
- Selalu validasi input duration untuk mencegah interval yang terlalu pendek
- Pertimbangkan untuk menambahkan mekanisme retry jika task gagal
- Gunakan transaction saat update config di database
