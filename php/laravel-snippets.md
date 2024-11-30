---
title: 'Laravel Snippets'
date: '2023-08-30 00:14:00'
---

Some Laravel Snippets.

### simple mail test

run `php artisan tinker`

```php
Mail::raw('Hello World!', function($msg) {$msg->to('myemail@gmail.com')->subject('Test Email'); });
```

### find duplicate records

```php
DB::table('users')
  ->select('id', 'email', DB::raw('COUNT(email)'))
  ->whereNull('deleted_at')
  ->havingRaw('COUNT(email) > 1')
  ->groupBy('email')
  ->get()
  ->toArray();
```

