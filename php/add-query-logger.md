# Add Query Logger

```bash
php artisan make:middleware LogQueryExecutionTime
```

edit the middleware

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Log;
use Symfony\Component\HttpFoundation\Response;

class LogQueryExecutionTime
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (!app()->hasDebugModeEnabled()) {
            return $next($request);
        }
        $queries = [];
        $totalQueryTime = 0;
        $startRequest = microtime(true);

        DB::listen(function (QueryExecuted $query) use (&$queries, &$totalQueryTime) {
            $queries[] = [
                'sql' => $query->sql,
                'bindings' => $query->bindings,
                'time' => $query->time, // in ms
            ];
            $totalQueryTime += $query->time;
        });

        $response = $next($request);

        $endRequest = microtime(true);
        $durationRequest = round(($endRequest - $startRequest) * 1000, 2); // ms

        if (!empty($queries)) {
            Log::debug("=== Start of Queries for: " . $request->path() . " ===");

            foreach ($queries as $i => $query) {
                $sql = $this->interpolateQuery($query['sql'], $query['bindings']);
                Log::debug(sprintf("#%d | %s ms | %s", $i + 1, $query['time'], $sql));
            }

            Log::debug("Total Query Time: {$totalQueryTime} ms");
            Log::debug("Total Request Time: {$durationRequest} ms");
            Log::debug("=== End of Queries for: " . $request->path() . " ===");
        }

        return $response;
    }

    protected function interpolateQuery($query, $bindings)
    {
        foreach ($bindings as $binding) {
            $value = is_numeric($binding) ? $binding : "'".addslashes($binding)."'";
            $query = preg_replace('/\?/', $value, $query, 1);
        }

        return $query;
    }
}
```

register the middleware in `bootstrap/app.php`

```php
<?php

use App\Http\Middleware\HandleAppearance;
use App\Http\Middleware\HandleInertiaRequests;
use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;
use Illuminate\Http\Middleware\AddLinkHeadersForPreloadedAssets;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->encryptCookies(except: ['appearance', 'sidebar_state']);

        $middleware->web(append: [
            HandleAppearance::class,
            HandleInertiaRequests::class,
            AddLinkHeadersForPreloadedAssets::class,
            \App\Http\Middleware\LogQueryExecutionTime::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```