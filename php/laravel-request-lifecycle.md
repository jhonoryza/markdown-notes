## Diagram sederhana laravel request lifecycle

<p float="left">
	<img src="https://minio.labkita.my.id/laravelblog/Screenshot%202024-08-02%20at%204.28.39%20PM.png" width="500" />
</p>

## Service Container

apa itu service container ?

1. tools untuk memanage `class depedencies`
2. tools untuk melakukan `dependency injection`

class dependencies are "injected" into the class via the constructor

proses yang dilakukan intinya hanya 2 yaitu: binding & resolving

pada dasarnya binding dan resolving akan jarang dilakukan kecuali pada kasus
tertentu

### tidak memerlukan konfigurasi (zero configuration resolution)

<p float="left">
	<img src="https://minio.labkita.my.id/laravelblog/Screenshot%202024-08-02%20at%204.37.52%20PM.png" width="600" />
</p>

type-hint dependencies on routes, controllers, event listeners and elsewhere
without ever manually interacting with the container

### kapan diperlukan interaksi manual dengan service container ?

- ketika membuat laravel package
- ketika type-hint dilakukan menggunakan interface

contoh: (binding interface to implementation)

<p float="left">
	<img src="https://minio.labkita.my.id/laravelblog/Screenshot%202024-08-02%20at%204.39.52%20PM.png" width="800" />
</p>

### binding

- bind

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

- bindIf

<p float="left">
	<img src="https://minio.labkita.my.id/laravelblog/Screenshot%202024-08-02%20at%204.42.24%20PM.png" width="300" />
</p>

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

- singleton (Once a singleton binding is resolved, the same object instance will
  be returned on subsequent calls into the container)

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

- scoped (singleton scoped)(will be flushed whenever the Laravel application
  starts a new "lifecycle")

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;
 
$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

- instance (bind an existing object instance into the container)

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
 
$service = new Transistor(new PodcastParser);
 
$this->app->instance(Transistor::class, $service);
```

- binding interface to implementation

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;
 
$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

```php
use App\Contracts\EventPusher;
 
/**
 * Create a new class instance.
 */
public function __construct(
    protected EventPusher $pusher
) {}
```

- contextual binding

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;
 
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });
 
$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

- binding primitives

```php
use App\Http\Controllers\UserController;
 
$this->app->when(UserController::class)
          ->needs('$variableName')
          ->give($value);
```

dst...

### resolving

- make

```php
use App\Services\Transistor;
 
$transistor = $this->app->make(Transistor::class);
```

- makeWith (jika depedency tidak di resolve lewat container)

```php
use App\Services\Transistor;
 
$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

or if we are outside of a service provider in a location of your code that does
not have access to the $app variable, you may use the App facade or the app
helper to resolve a class instance from the container

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;
 
$transistor = App::make(Transistor::class);
 
$transistor = app(Transistor::class);
```

#### Method Invocation & Injection

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;
 
$stats = App::call([new PodcastStats, 'generate']);
```

```php
<?php
 
namespace App;
 
use App\Services\AppleMusic;
 
class PodcastStats
{
    /**
     * Generate a new podcast stats report.
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}

use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;
 
$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

## Service Provider

apa itu service provider ?

- tempat untuk melakukan bootstraping aplikasi laravel
- bootstraping memiliki arti `meregistrasikan` :
  - service container bindings
  - event listener
  - middleware
  - routes

### create custom service provider

cara membuat : `php artisan make:provider MyCustomServiceProvider`

### registering custom service provider

custom service provider mesti di registrasikan di `config/app.php` pada variable
$providers.

### register method

register() function di service provider hanya digunakan untuk melakukan binding
ke dalam service container

### boot method

boot() function akan di panggil setelah semua service provider di register

### deferred provider

jika service provider hanya digunakan untuk binding ke dalam container maka
deferred provider bisa jadi salah satu opsi untuk meningkatkan performa aplikasi

registrasi hanya akan dilakukan jika binding tsb benar2 di perlukan

caranya hanya cukup `implement DeferrableProvider` pada class service provider

### cara alternative tanpa menggunakan register function

jika hanya melakukan binding di service provider

```php
<?php
 
namespace App\Providers;
 
use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;
 
class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];
 
    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```
