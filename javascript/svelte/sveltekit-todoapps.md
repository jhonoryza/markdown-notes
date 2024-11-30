---
title: 'Sveltekit Todoapps API'
date: '2023-08-17 10:50:00'
---

Kali ini kita akan membuat todoapps api menggunakan svelkit, drizzle-orm dan playwright untuk test rest api.

## Requirement

- svelkit (js framework)
- playwright (test package)
- drizzle-orm (orm)
- drizzle-kit (orm cli helper)
- mysql2 (mysql driver)
- joi (validation package)
- bcrypt (hash package)
- winston (logger package)
- uuid (for generate token)

## Setup

### create project

pertama kita buat project nya dahulu

```bash
pnpm create svelte@latest sveltekit-todoapps
```

pilih skeleton project, typescript, checklist eslint, prettier dan playwright

```bash
cd sveltekit-todoapps
pnpm install
npx playwright install chromium
```

### install packages

```bash
pnpm install drizzle-orm mysql2 bcrypt joi mysql2 uuid winston vite-node
pnpm install -D drizzle-kit @types/bcrypt @types/uuid
```

### setup playwright config

sesuaikan isi playwright.config.ts seperti ini:

```typescript
import type { PlaywrightTestConfig } from '@playwright/test';

const config: PlaywrightTestConfig = {
 workers: 1,
 webServer: {
   command: 'npm run dev:test',
   port: 8000
 },
 testDir: 'tests',
 testMatch: /(.+\.)?(test|spec)\.[jt]s/
};

export default config;
```

### adjust scripts

tambahkan scripts di package.json seperti ini:

```json
 "scripts": {
  "dev:test": "vite dev -m test",
  "migration:generate": "drizzle-kit generate:mysql",
  "migration:push": "vite-node --options.transformMode.ssr='/.*/' src/lib/migrate.ts"
 }
```

### adjust vite config

adjust vite.config.ts seperti ini:

```typescript
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
 const env = loadEnv(mode, process.cwd(), '');
 return {
  plugins: [sveltekit()],
  server: {
   port: Number(env.APP_PORT) || 5173,
   host: env.APP_HOST || `localhost`,
   https: false
  }
 };
});
```

### adjust drizzle config

buat file drizzle.config.ts dengan isi seperti ini:

```typescript
import type { Config } from 'drizzle-kit';

export default {
 schema: 'src/lib/schema.ts',
 out: 'drizzle'
} satisfies Config;
```

### setup env files

buat file .env dan .env.example dengan isi seperti ini:

```env
APP_URL=http://localhost
APP_HOST=localhost
APP_PORT=4173

DB_HOST=localhost
DB_DATABASE=todo_api
DB_USER=root
DB_PASS=''
DB_PORT=3306
```

buat file .env.test dengan isi seperti ini:

```env
APP_URL=http://localhost
APP_HOST=localhost
APP_PORT=8000

DB_HOST=localhost
DB_DATABASE=todo_api_test
DB_USER=root
DB_PASS=''
DB_PORT=3306
```

## Mulai

### prepare database

buat file src/lib/database.ts

```typescript
import { drizzle } from "drizzle-orm/mysql2";
import mysql from "mysql2/promise";
import { loadEnv } from "vite";

const env = loadEnv(import.meta.env?.MODE || 'test', process.cwd(), '')

const poolConnection = mysql.createPool({
  host: env.DB_HOST,
  user: env.DB_USER,
  password: env.DB_PASSWORD,
  database: env.DB_DATABASE,
  port: Number(env.DB_PORT)
});
 
export const db = drizzle(poolConnection);
```

buat file src/lib/schema.ts

```typescript
import { timestamp, customType, mysqlTable, varchar } from 'drizzle-orm/mysql-core';

// custom type unsigned big integer
const unsignedBigint = customType<{ data: number }>({
 dataType() {
  return 'bigint UNSIGNED';
 }
});

// start
// custom type unsigned multiple of integer autoincrement primary key
type IdType = "tinyint" | "smallint" | "mediumint" | "int" | "bigint";
interface UIntConfig {
    type?: IdType;
}

export const unsignedIntAutoIncrement = customType<{ data: number; config: UIntConfig; primaryKey: true; default: true }>({
    dataType: (config) => {
        return `${config?.type || "int"} UNSIGNED AUTO_INCREMENT`;
    }
});

export function primary(dbName: string, config?: UIntConfig) {
    return unsignedIntAutoIncrement(dbName, config).primaryKey()
};
// end

// Table users
export const users = mysqlTable('users', {
 id: primary('id', { type: 'bigint' }),
 name: varchar('username', { length: 255 }).notNull(),
 email: varchar('email', { length: 255 }).notNull().unique(),
 password: varchar('password', { length: 255 }).notNull(),
 token: varchar('token', { length: 255 }),
 created_at: timestamp('created_at').defaultNow().notNull(),
 updated_at: timestamp('updated_at').defaultNow().notNull().onUpdateNow()
});

// Table todos
export const todos = mysqlTable('todos', {
 id: primary('id', { type: 'bigint' }),
 user_id: unsignedBigint('user_id')
  .notNull()
  .references(() => users.id, { onDelete: 'cascade', onUpdate: 'cascade' }),
 title: varchar('title', { length: 255 }).notNull(),
 description: varchar('description', { length: 255 }),
 order: unsignedBigint('order').notNull(),
 created_at: timestamp('created_at').defaultNow().notNull(),
 updated_at: timestamp('updated_at').defaultNow().notNull().onUpdateNow()
});
```

buat file src/lib/migrate.ts

```typescript
import { drizzle } from 'drizzle-orm/mysql2';
import mysql from 'mysql2/promise';
import { loadEnv } from 'vite';
import { migrate } from 'drizzle-orm/mysql2/migrator';
import { askQuestion } from './util';

const ans = await askQuestion('input your environment: (default: development)') || 'development';
if (ans == '') process.exit(0)

const env = loadEnv(String(ans), process.cwd(), '');
const poolConnection = mysql.createPool({
 host: env.DB_HOST,
 user: env.DB_USER,
 password: env.DB_PASSWORD,
 database: env.DB_DATABASE,
 port: Number(env.DB_PORT)
});
const db = drizzle(poolConnection);

async function main() {
 console.log('migration start ..!');
    await migrate(db, { migrationsFolder: "drizzle" });
 console.log('migration success ..!');
 process.exit(0);
}

await main().catch(console.error);
process.exit(0)

```

lalu jalankan

```bash
pnpm run migration:generate
pnpm run migration:push
```

### prepare model

model user dan todo berikut hanyalah type untuk keperluan typescript

buat file src/lib/model/user.ts

```typescript
export type user = {
 id: number;
 name: string;
 email: string;
 password?: string;
 updated_at: Date;
 created_at: Date;
};
```

dan file src/lib/model/todo.ts

```typescript
export type todo = {
  id: number;
  user_id: number;
  title: string;
  description: string | null;
  order: number;
  created_at: Date;
  updated_at: Date;
};
```

### prepare factory

kita akan membuat userFactory dan todoFactory sebagai template factory ketika akan membuat fake user dan todo

buat file src/lib/factory/user.ts

```typescript
import { db } from "../database";
import { users } from "../schema";
import bcrypt from 'bcrypt'

export async function userFactory(number: number, token?: string) {
    for (let index = 1; index <= number; index++) {
        const password = await bcrypt.hash("Password" + index, 10);
        await db.insert(users).values({
            name: "name" + index,
            password: password,
            token: token,
            email: "email" + index + "@email.com",
        }).execute();
    }
}
```

buat file src/lib/factory/todo.ts

```typescript
import { db } from "../database";
import { todos } from "../schema";

export async function todoFactory(number: number, userId: number) {
    for (let index = 1; index <= number; index++) {
        await db.insert(todos).values({
            title: "title" + index,
            description: "description" + index,
            order: index,
            user_id: userId,
        }).execute();
    }
}
```

### prepare util.ts

buat file src/lib/util.ts

```typescript
export function mysqlDatetimeUtc(date: Date = new Date()) {
 return date.toISOString().slice(0, 19).replace('T', ' ');
}

// Use this function instead of new Date() when converting a MySQL datetime to a
// Date object so that the date is interpreted as UTC instead of local time (default behavior)
export function mysqlDatetimeUtcToDate(mysqlDatetimeUtc: string) {
 return new Date(mysqlDatetimeUtc.replace(' ', 'T') + 'Z');
}

import readline from 'readline';

export function askQuestion(query: string) {
 const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
 });

 return new Promise((resolve) =>
  rl.question(query, (ans) => {
   rl.close();
   resolve(ans);
  })
 );
}
```

### prepare logger

buat file src/lib/logger.ts

```typescript
import winston from 'winston';

export const logger = winston.createLogger({
 level: 'info',
 format: winston.format.json(),
 transports: [
  new winston.transports.File({ filename: 'app.log' })
 ]
});
```

buat file src/hooks.server.ts

```typescript
import { db } from '$lib/database';
import { logger } from '$lib/logger';
import { users } from '$lib/schema';
import type { Handle } from '@sveltejs/kit';
import { error as responseError } from '@sveltejs/kit';
import { sequence } from '@sveltejs/kit/hooks';
import { eq } from 'drizzle-orm';

const authMiddleware: Handle = async ({ event, resolve }) => {
 if (event.url.pathname.startsWith('/api/authenticated')) {
  const token = event.request.clone().headers.get('authorization');
  if (!token) {
   throw responseError(401, {
    message: 'unauthenticated'
   });
  }
  const user = (await db.select().from(users).where(eq(users.token, token)).limit(1)).at(0);
  if (!user) {
   throw responseError(401, {
    message: 'invalid token'
   });
  }
  event.locals.user = user;
 }
 return await resolve(event)
}

const loggingMiddleware: Handle = async ({ event, resolve }) => {
 const resp: Response = await resolve(event);
 if (resp.status >= 400) {
  logger.info(await resp.clone().json());
 }
 return resp;
}

export const handle: Handle = sequence(authMiddleware, loggingMiddleware);
```

### membuat api register

buat file src/routes/register/+server.ts

```typescript
import { db } from '$lib/database.js';
import { registerRequest } from '$lib/request/register.js';
import { users } from '$lib/schema.js';
import { json, error as responseError, type RequestHandler } from '@sveltejs/kit';
import { eq } from 'drizzle-orm';
import bcrypt from 'bcrypt';

export const POST: RequestHandler = async ({ request }) => {
 const { name, email, password } = await request.json();
 const { error } = registerRequest.validate({ name, email, password });
 if (error) {
  throw responseError(400, {
   message: error.message
  });
 }
 const isUserExist = await db.select().from(users).where(eq(users.email, email));
 if (isUserExist.length > 0) {
  throw responseError(400, {
   message: 'user already exist'
  });
 }
 const hashPassword = await bcrypt.hash(password, 10);
 const user = await db.insert(users).values({
  name: name,
  email: email,
  password: hashPassword
 });

 return json(
  {
   data: user,
   message: 'register successfully',
   code: 201
  },
  { status: 201 }
 );
}
```

buat registerRequest untuk validasi

buat file src/lib/request/register.ts

```typescript
import Joi from 'joi';

export const registerRequest = Joi.object({
 name: Joi.string().min(3).max(255).required(),

 password: Joi.string().min(6).max(100).required(),

 email: Joi.string().max(255).email().required()
});
```

buat userResource untuk mapping data yg akan di return di response api ini

buat file src/lib/resource/user.ts

```typescript
import { mysqlDatetimeUtc } from "$lib/util";
import type { user } from "$lib/model/user";

type result = {
 id: number;
 name: string;
 email: string;
 updated_at: string;
 created_at: string;
};
export const userResource = ( users: user[] ): result[] => {
    
    const result: result[] = [];
 users.forEach((user: user) => {
  result.push({
   id: user.id,
   name: user.name,
   email: user.email,
   updated_at: mysqlDatetimeUtc(user.updated_at),
   created_at: mysqlDatetimeUtc(user.created_at)
  })
 })
    return result
};
```

### buat unit test untuk api register

buat file tests/register.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { users } from '../src/lib/schema';

test.describe('test POST register api', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
 });

 test('it can register user', async ({ request }) => {
  const res = await request.post('/api/register', {
   data : {
    name: 'fajar sp',
    email: 'email@gmail.com',
    password: 'Password123456'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(201);
  expect(json.message).toEqual('register successfully');
 });

 test('it can validate exists user', async ({ request }) => {
        await db.insert(users).values({
            name: 'fajar sp',
            email: 'email@gmail.com',
            password: 'Password123456'
        })
        const res = await request.post('/api/register', {
   data: {
    name: 'agung sp',
    email: 'email@gmail.com',
    password: 'Password123456'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(400);
  expect(json.message).toEqual('user already exist');
 });

 test('it can validate required field', async ({ request }) => {
  const res = await request.post('/api/register', {
   data: {
    name: 'fajar sp',
    email: '',
    password: 'Password123456'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(400);
  expect(json.message).toEqual('"email" is not allowed to be empty');
 });
});
```

untuk menjalankan migration pada database testing run `pnpm run migration:push`

untuk melakukan test jalankan `pnpm run test`

### membuat api login

buat file src/routes/login/+server.ts

```typescript
import { db } from '$lib/database.js';
import { loginRequest } from '$lib/request/login.js';
import { users } from '$lib/schema.js';
import { json, error as responseError, type RequestHandler } from '@sveltejs/kit';
import { eq } from 'drizzle-orm';
import bcrypt from 'bcrypt';
import { v4 as uuidv4 } from 'uuid';

export const POST: RequestHandler = async ({ request }) => {
 const { email, password } = await request.json();
 const { error } = loginRequest.validate({ email, password });
 if (error) {
  throw responseError(400, {
   message: error.message
  });
 }

 const user = (await db.select().from(users).where(eq(users.email, email)).limit(1)).at(0);

 if (!user || (await bcrypt.compare(password, user.password)) === false) {
  throw responseError(400, {
   message: 'email or password is invalid'
  });
 }

    const token = uuidv4()
 await db
  .update(users)
  .set({
   token: token
  })
  .where(eq(users.id, user.id));

    return json(
        {
            token: token,
            message: 'login successfully',
            code: 200
        }
    )
}
```

buat loginRequest untuk validasi

buat file src/lib/request/login.ts

```typescript
import Joi from 'joi';

export const loginRequest = Joi.object({
 email: Joi.string().max(255).email().required(),
 password: Joi.string().min(6).max(100).required(),
});
```

### buat unit test untuk api login

buat file tests/login.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { userFactory } from '../src/lib/factory/user';

test.describe('test POST login api', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
  await userFactory(1)
 });

 test('it can login', async ({ request }) => {
  const res = await request.post('/api/login', {
   data : {
    email: 'email1@email.com',
    password: 'Password1'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.token).not.toBeNull();
  expect(json.message).toBe('login successfully');
 });

 test('it can validate wrong email', async ({ request }) => {
        const res = await request.post('/api/login', {
   data: {
    email: 'email@gmail.com',
    password: 'Password1'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(400);
  expect(json.message).toEqual('email or password is invalid');
 });

 test('it can validate wrong pass', async ({ request }) => {
  const res = await request.post('/api/login', {
   data: {
    email: 'email1@email.com',
    password: 'Password123456'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(400);
  expect(json.message).toEqual('email or password is invalid');
 });
});
```

### buat api logout

buat file src/routes/authenticated/logout/+server.ts

```typescript
import { db } from '$lib/database.js';
import { users } from '$lib/schema.js';
import { json, error as responsError, RequestHandler } from '@sveltejs/kit';
import { eq } from 'drizzle-orm';

export const DELETE: RequestHandler = async ({ locals }) => {
 await db
  .update(users)
  .set({
   token: null
  })
  .where(eq(users.id, locals.user.id))
        .catch(err => {
            throw responsError(400, {
                message: err
            })
        });
 return json({
  data: locals.user,
        message: 'Logged out successfully',
  code: 200
 });
}
```

### buat test api logout

buat file tests/logout.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { userFactory } from '../src/lib/factory/user';

test.describe('test DELETE logout api', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
  await userFactory(1, 'mytoken');
 });

 test('it can logout', async ({ request }) => {
  const res = await request.delete('/api/authenticated/logout', {
   headers: {
    authorization: 'mytoken'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.message).toBe('Logged out successfully');
 });
});
```

### buat test auth middleware

buat file tests/auth-middleware.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { userFactory } from '../src/lib/factory/user';

test.describe('test auth middleware', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
  await userFactory(1, 'mytoken');
 });

 test('it can validate with invalid token', async ({ request }) => {
  const res = await request.post('/api/authenticated/myprofile', {
   data: {
    email: 'email@gmail.com',
    password: 'Password1'
   },
   headers: {
    authorization: 'invalidtoken'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(401);
  expect(json.message).toEqual('invalid token');
 });

 test('it can validate with no token', async ({ request }) => {
  const res = await request.post('/api/authenticated/myprofile', {
   data: {
    email: 'email@gmail.com',
    password: 'Password1'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(401);
  expect(json.message).toEqual('unauthenticated');
 });
});
```

### buat api myprofile

buat file src/routes/authenticated/myprofile/+server.ts

```typescript
import { json, RequestHandler } from '@sveltejs/kit';

export const GET: RequestHandler = async ({ locals }) => {
 return json({
  data: locals.user,
  message: 'get user profile successfully',
  code: 200
 });
}
```

### buat test api myprofile

buat file tests/myprofile.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { userFactory } from '../src/lib/factory/user';

test.describe('test GET myprofile api', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
  await userFactory(1, 'mytoken');
 });

 test('it can get myprofile with valid token', async ({ request }) => {
  const res = await request.get('/api/authenticated/myprofile', {
   headers: {
    authorization: 'mytoken'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.message).toBe('get user profile successfully');
 });
});
```

### buat api GET dan POST todos

buat file src/routes/authenticated/todos/+server.ts

```typescript
import { db } from '$lib/database';
import { todoResource } from '$lib/resource/todo';
import { todos } from '$lib/schema';
import { json, type RequestHandler } from '@sveltejs/kit';
import { eq } from 'drizzle-orm';
import { error as responseError } from '@sveltejs/kit';
import { todoSaveRequest } from '$lib/request/todo';

export const GET = (async ({ locals, url }) => {
 const size: number = url.searchParams.get('page[size]')
  ? Number(url.searchParams.get('page[size]'))
  : 10;
 const page: number = url.searchParams.get('page[number]')
  ? Number(url.searchParams.get('page[number]'))
  : 1;
 const offset = page > 0 ? (page - 1) * size : 0;

 const data = await db
  .select()
  .from(todos)
  .where(eq(todos.user_id, locals.user.id))
  .limit(size)
  .offset(offset);

 return json(
  {
            message: 'get mytodo successfully',
   data: todoResource(data),
   code: 200,
   meta: {
    page: page,
    size: size
   }
  },
  { status: 200 }
 );
}) satisfies RequestHandler;

export const POST = (async ({ locals, request }) => {
 const { title, description, order } = await request.json();
 const { error } = todoSaveRequest.validate({ title, description, order });
 if (error) {
  throw responseError(400, {
   message: error.message
  });
 }
 const todo = await db.insert(todos).values({
  title: title,
  description: description,
  order: order,
  user_id: locals.user.id
 });
 return json(
  {
   data: todo,
   message: 'Todo added successfully',
   code: 201
  },
  { status: 201 }
 );
}) satisfies RequestHandler;
```

buat todoSaveRequest untuk validasi

buat file src/lib/request/todo.ts

```typescript
import Joi from 'joi';

export const todoSaveRequest = Joi.object({
 title: Joi.string().min(3).max(255).required(),

 description: Joi.string().max(255).not().required(),

 order: Joi.number().required()
});
```

buat todoResource untuk mapping data yg akan di return di response api ini

buat file src/lib/resource/todo.ts

```typescript
import { mysqlDatetimeUtc } from "$lib/util";
import type { todo } from "$lib/model/todo";

type result = {
 id: number;
 title: string;
 description: string;
 order: number;
 updated_at: string;
 created_at: string;
};
export const todoResource = ( todos: todo[] ): result[] => {
    
    const result: result[] = [];
 todos.forEach((todo: todo) => {
  result.push({
   id: todo.id,
   title: todo.title,
   description: todo.description || '',
   order: todo.order,
   updated_at: mysqlDatetimeUtc(todo.updated_at),
   created_at: mysqlDatetimeUtc(todo.created_at)
  });
 })
    return result
};
```

### buat api PUT dan DELETE todos

buat file src/routes/authenticated/todos/[id]/+server.ts

```typescript
import { db } from '$lib/database';
import { todos } from '$lib/schema';
import { json, type RequestHandler } from '@sveltejs/kit';
import { and, eq } from 'drizzle-orm';
import { error as responseError } from '@sveltejs/kit';
import { todoSaveRequest } from '$lib/request/todo';

export const PUT = (async ({ params, locals, request }) => {
 const { title, description, order } = await request.json();
 const { error } = todoSaveRequest.validate({ title, description, order });
 if (error) {
  throw responseError(400, {
   message: error.message
  });
 }
 const mytodo = (
  await db
   .select()
   .from(todos)
   .where(eq(todos.user_id, locals.user.id))
   .where(eq(todos.id, Number(params.id)))
   .limit(1)
 ).at(0);
 if (!mytodo) {
  throw responseError(404, {
   message: 'Todo not found'
  });
 }
 await db
  .update(todos)
  .set({
   title: title,
   description: description,
   order: order
  })
  .where(eq(todos.user_id, locals.user.id))
  .where(eq(todos.id, Number(mytodo.id)))
  .catch((err) => {
   throw responseError(400, {
    message: err
   });
  });

 return json({
  message: `Todo ${mytodo.id} updated successfully`,
  code: 200
 });
}) satisfies RequestHandler;

export const DELETE = (async ({ locals, params }) => {
 const mytodo = (
  await db
   .select()
   .from(todos)
   .where(and(eq(todos.user_id, locals.user.id), eq(todos.id, Number(params.id))))
   .limit(1)
 ).at(0);
 if (!mytodo) {
  throw responseError(404, {
   message: 'Todo not found'
  });
 }
 await db
  .delete(todos)
  .where(and(eq(todos.user_id, locals.user.id), eq(todos.id, Number(params.id))))
  .catch((err) => {
   throw responseError(400, {
    message: err
   });
  });

 return json({
  data: mytodo,
  message: `Todo ${mytodo.id} deleted successfully`,
  code: 200
 });
}) satisfies RequestHandler;
```

### buat test api todos

buat file tests/todos.test.ts

```typescript
import { sql } from 'drizzle-orm';
import { db } from '../src/lib/database';
import { test, expect } from '@playwright/test';
import { userFactory } from '../src/lib/factory/user';
import { todoFactory } from '../src/lib/factory/todo';

test.describe('test todo api', () => {
 test.beforeEach(async () => {
  await db.execute(sql`set FOREIGN_KEY_CHECKS=0`);
  await db.execute(sql`truncate users`);
  await db.execute(sql`truncate todos`);
  await db.execute(sql`set FOREIGN_KEY_CHECKS=1`);
  await userFactory(1, 'mytoken');
 });

 test('it can add new todo', async ({ request }) => {
  const res = await request.post('/api/authenticated/todos', {
   headers: {
    authorization: 'mytoken'
   },
   data: {
    title: 'title',
    description: 'description',
    order: 1
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(201);
  expect(json.message).toBe('Todo added successfully');
 });

 test('it can get my todo', async ({ request }) => {
  await todoFactory(2, 1);
  const res = await request.get('/api/authenticated/todos', {
   headers: {
    authorization: 'mytoken'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.message).toBe('get mytodo successfully');
 });

 test('it can update my todo', async ({ request }) => {
  await todoFactory(2, 1);
  const res = await request.put('/api/authenticated/todos/1', {
   headers: {
    authorization: 'mytoken'
   },
   data: {
    title: 'updated title',
    description: 'description',
    order: 1
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.message).toBe(`Todo 1 updated successfully`);
 });

 test('it can delete my todo', async ({ request }) => {
  await todoFactory(2, 1);
  const res = await request.delete('/api/authenticated/todos/1', {
   headers: {
    authorization: 'mytoken'
   }
  });
  const json = await res.json();
  expect(res.status()).toBe(200);
  expect(json.message).toBe(`Todo 1 deleted successfully`);
 });
});
```

## source code

untuk full source code bisa cek disini [https://github.com/jhonoryza/svelkit-todoapps-api](https://github.com/jhonoryza/svelkit-todoapps-api)
