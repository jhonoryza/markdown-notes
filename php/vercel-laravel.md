# Deploy Laravel to Vercel

## Easy, Free, Serverless Laravel With Vercel

First in your existing project create this file `api/index.php`

```php
<?php

// Forward Vercel requests to normal index.php
require __DIR__ . '/../public/index.php';
```

Vercel only allows an app’s entry-point to live inside the api directory, then
we have to set up a simple script to forward to Laravel’s normal
`public/index.php` entry-point

Create file `.vercelignore` to ignore vendor dir when deployed

```
/vendor
```

Create file `vercel.json`, the explanation is
[here](https://vercel.com/docs/projects/project-configuration)

```json
{
    "version": 2,
    "framework": null,
    "builds": [
        {
            "src": "/api/index.php",
            "use": "vercel-php@0.6.2"
        },
        {
            "src": "/public/build/assets/**",
            "use": "@vercel/static"
        },
        {
            "src": "/public/**",
            "use": "@vercel/static"
        }
    ],
    "routes": [
        {
            "src": "/build/assets/(.*)",
            "dest": "/public/build/assets/$1"
        },
        {
            "src": "/favicon.ico",
            "headers": {
                "Content-Type": "image/x-icon"
            },
            "dest": "/public/favicon.ico"
        },
        {
            "src": "/(.*)",
            "dest": "/api/index.php"
        }
    ],
    "outputDirectory": "public",
    "env": {
        "APP_NAME": "Your App Name",
        "APP_ENV": "production",
        "APP_DEBUG": "false",
        "APP_URL": "https://laravel-app.vercel.app",

        "LOG_CHANNEL": "stderr",
        "CACHE_DRIVER": "array",
        "SESSION_DRIVER": "array",

        "APP_CONFIG_CACHE": "/tmp/config.php",
        "APP_EVENTS_CACHE": "/tmp/events.php",
        "APP_PACKAGES_CACHE": "/tmp/packages.php",
        "APP_ROUTES_CACHE": "/tmp/routes.php",
        "APP_SERVICES_CACHE": "/tmp/services.php",
        "VIEW_COMPILED_PATH": "/tmp"
    }
}
```

`vercel-php` is a community-built PHP runtime for Vercel functions. It does all
the hard work for us like installing the proper dependencies and running
composer install. Please change this `vercel-php@0.6.2` version refer to this
link
[https://github.com/vercel-community/php](https://github.com/vercel-community/php)
adjust it according with your php version

Change `APP_URL` with your prefered domain name, this will be setted as your
production domain.

You can change `CACHE_DRIVER` or `SESSION_DRIVER` using redis.

For more sensitive environment variables like APP_KEY or anything that you dont
want people to know it, you can visit the vercel `Environment Variables` tab in
your project’s `Settings`:

## Install Vercel CLI

Install in your local machine, check this [link](https://vercel.com/docs/cli)

run `vercel login` and follow the instruction

to deploy in preview mode, run

```bash
vercel deploy
```

when finished you can click the preview link provided from vercel

to deploy in preview production mode, run

```bash
vercel --prod
```

when finished you can click the production link provided from vercel

## Summary

This makes it incredibly easy to deploy apps to vercel even if we use php and
laravel

Because Vercel is serverless, your database has to be hosted on a separate cloud
platform.

To link your database with Vercel is actually easy you just need to update the
`Environment Variables` in `vercel.json` or in vercel project’s `Settings`.
