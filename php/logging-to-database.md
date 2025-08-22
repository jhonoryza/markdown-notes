# Logging ke Database di Laravel

Untuk mencatat log ke database di Laravel, ikuti langkah-langkah berikut:

## 1. Konfigurasi Logging

Tambahkan konfigurasi berikut pada file `config/logging.php`:

```php
'channels' => [
    // ...existing code...
    'database' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
        'connection' => env('DB_CONNECTION_LOGGER', 'mysql-logger'),
        'level' => 'error',
    ],
    // ...existing code...
],
```

## 2. Membuat Class `CreateCustomLogger`

Buat class berikut di `app/Logging/CreateCustomLogger.php`:

```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * Membuat instance Monolog kustom.
     */
    public function __invoke(array $config): Logger
    {
        $connection = $config['connection'] ?? config('database.default');
        $level = Logger::toMonologLevel($config['level'] ?? 'debug');
        $logger = new Logger('laravel_logs');
        $logger->pushHandler(new LoggingHandler($connection, $level));
        return $logger;
    }
}
```

## 3. Membuat Class `LoggingHandler`

Buat class berikut di `app/Logging/LoggingHandler.php`:

```php
<?php

namespace App\Logging;

use App\Models\LaravelLog;
use Monolog\Handler\AbstractProcessingHandler;
use Monolog\Logger;

class LoggingHandler extends AbstractProcessingHandler
{
    public function __construct($connection = null, $level = Logger::DEBUG, $bubble = true)
    {
        $this->table = 'laravel_logs';
        $this->connection = $connection ?: config('database.default');
        parent::__construct($level, $bubble);
    }

    protected function write(array $record): void
    {
        if ($record) {
            $log = new LaravelLog();
            $log->setConnection($this->connection);
            $log->create([
                'message' => $record['message'],
                'context' => $record['context'] ? json_encode($record['context'], JSON_FORCE_OBJECT) : null,
                'level' => $record['level'],
                'level_name' => $record['level_name'],
                'channel' => $record['channel'],
                'record_datetime' => $record['datetime'] ? json_encode($record['datetime']) : null,
                'extra' => $record['extra'] ? json_encode($record['extra']) : null,
                'formatted' => $record['formatted'],
            ]);
        }
    }
}
```

## 4. Membuat Model `LaravelLog`

Buat model berikut di `app/Models/LaravelLog.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class LaravelLog extends Model
{
    use SoftDeletes;

    protected $connection = 'mysql-logger';

    /**
     * Atribut yang dapat diisi secara massal.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'message',
        'context',
        'level',
        'level_name',
        'channel',
        'record_datetime',
        'extra',
        'formatted',
        'remote_addr',
        'user_agent'
    ];

    /**
     * Atribut yang dapat dicari.
     *
     * @var array
     */
    protected $searchable = [
        'message',
    ];

    /**
     * Atribut yang dapat diurutkan.
     *
     * @var array
     */
    protected $sortable = [
        'message',
        'level',
        'level_name',
        'channel'
    ];

    /**
     * Default atribut untuk pengurutan data.
     *
     * @var string
     */
    protected string $defaultSortable = 'updated_at';

    protected string $defaultSort = 'desc';
}
```

## 5. Membuat File Migrasi

Buat file migrasi berikut di `database/migrations/xxxx_xx_xx_create_laravel_logs_table.php`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateLaravelLogsTable extends Migration
{
    /**
     * Jalankan migrasi.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('laravel_logs', function (Blueprint $table) {
            $table->id();
            $table->longText('message');
            $table->json('context')->nullable();
            $table->string('level')->index();
            $table->string('level_name');
            $table->string('channel')->index();
            $table->string('record_datetime')->nullable();
            $table->longText('extra')->nullable();
            $table->longText('formatted');
            $table->string('remote_addr')->nullable();
            $table->string('user_agent')->nullable();
            $table->timestamps();
            $table->softDeletes();
        });
    }

    /**
     * Batalkan migrasi.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('laravel_logs');
    }
}
```

## 6. Konfigurasi Koneksi Database

Pada file `config/database.php`, duplikat konfigurasi koneksi database `mysql` dan buat koneksi baru misal `mysql-logger`. Kemudian, pada file `.env` tambahkan:

```
DB_CONNECTION_LOGGER=mysql-logger
```

Tujuannya agar koneksi database untuk logging terpisah sehingga proses pencatatan log menjadi lebih andal dan robust.