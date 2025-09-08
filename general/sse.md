# SSE

```php
// routes/web.php
Route::get('/sse', [SseController::class, 'stream']);

// app/Http/Controllers/SseController.php
use Symfony\Component\HttpFoundation\StreamedResponse;

public function stream()
{
    return response()->stream(function () {
        while (true) {
            echo "data: " . json_encode(['time' => now()]) . "\n\n";
            ob_flush();
            flush();
            sleep(1);
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'Connection' => 'keep-alive',
    ]);
}

```

```js
const evtSource = new EventSource("/sse");
evtSource.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log("Data dari server:", data);
};
```

pastikan nginx `proxy_buffering off`


## Simpan data ke redis

```php
Cache::put('notifications', $notifications, 60); // expires in 60s

return response()->stream(function () {
        while (true) {
            $notifications = Cache::pull('notifications', []);
            foreach ($notifications as $note) {
                echo "data: " . json_encode($note) . "\n\n";
            }
            ob_flush();
            flush();
            sleep(1);
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'Connection' => 'keep-alive',
    ]);
```


## Simpan data ke database

```php
class UserRegisteredNotification extends Notification
{
    protected $user;

    public function __construct($user)
    {
        $this->user = $user;
    }

    public function via($notifiable)
    {
        return ['database'];
    }

    public function toDatabase($notifiable)
    {
        return [
            'title' => 'User Baru',
            'message' => $this->user->name . ' telah mendaftar.',
        ];
    }
}

return response()->stream(function () {
        $userId = 1; // Sesuaikan dengan user (misalnya admin)

        while (true) {
            $notifications = DB::table('notifications')
                ->where('notifiable_id', $userId)
                ->whereNull('read_at')
                ->orderBy('created_at', 'desc')
                ->get();

            foreach ($notifications as $note) {
                echo "data: " . json_encode([
                    'id' => $note->id,
                    'title' => $note->data['title'] ?? '',
                    'message' => $note->data['message'] ?? '',
                    'time' => $note->created_at
                ]) . "\n\n";

                // Tandai sudah dibaca
                DB::table('notifications')
                    ->where('id', $note->id)
                    ->update(['read_at' => now()]);
            }

            ob_flush();
            flush();
            sleep(1);
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'Connection' => 'keep-alive',
    ]);
```

## Redis get/set

1. Saat Laravel membuat notifikasi, selain menyimpan ke DB: Simpan juga tanda di Redis (misal: `user:1:has_new_notification = true`).

```php
Redis::set("user:{$admin->id}:has_new_notification", true);
```

2. SSE loop cukup cek Redis:
```php
while (true) {
    if (Redis::get("user:$userId:has_new_notification") === 'true') {
        Redis::del("user:$userId:has_new_notification");

        $notifications = ...; // baru query DB
        // kirim data ke client
    }

    sleep(1);
}
```

## Redis Pub/Sub

Publish ke Redis saat Notifikasi Dibuat

```php
Redis::publish("notifications:{$admin->id}", json_encode([
        'title' => 'User Baru',
        'message' => $event->user->name . ' telah mendaftar.',
        'time' => now()->toDateTimeString(),
    ]));
```

Karena Redis Pub/Sub bersifat blocking, kita tidak bisa pakai Laravel Redis facade dalam loop biasa. Gunakan Redis extension low-level langsung:

```php
public function stream()
{
    $userId = 1; // ganti sesuai user login jika ada sistem auth
    $channel = "notifications:$userId";

    return response()->stream(function () use ($channel) {
        $redis = new \Redis();
        $redis->connect('127.0.0.1', 6379);

        $redis->subscribe([$channel], function ($redis, $chan, $msg) {
            echo "data: {$msg}\n\n";
            ob_flush();
            flush();
        });
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'Connection' => 'keep-alive',
    ]);
}
```

Kelebihan Redis Pub/Sub:

✅ Benar-benar event-driven
✅ Tidak ada polling atau pengecekan manual
✅ Skalabel untuk banyak user jika Redis cukup kuat

- Redis Pub/Sub tidak menyimpan pesan, hanya realtime. Jadi client harus sedang tersambung untuk menerima data
- Untuk Laravel backend production: gunakan Laravel Octane atau worker process agar Redis subscription tidak diblokir kernel request.

## Redis pub/sub data tetap di database

```php
$admin->notify(new UserRegisteredNotification($event->user));

Redis::publish("notifications:{$admin->id}", 'new');
```

listen Redis + query DB

```php
public function stream()
{
    $userId = 1; // atau dari session/auth
    $channel = "notifications:$userId";

    return response()->stream(function () use ($userId, $channel) {
        $redis = new \Redis();
        $redis->connect('127.0.0.1', 6379);

        $redis->subscribe([$channel], function ($redis, $chan, $msg) use ($userId) {
            // Saat pesan Redis diterima → ambil notifikasi terbaru dari DB
            $notification = DB::table('notifications')
                ->where('notifiable_id', $userId)
                ->whereNull('read_at')
                ->latest()
                ->first();

            if ($notification) {
                echo "data: " . json_encode([
                    'id' => $notification->id,
                    'title' => $notification->data['title'] ?? '',
                    'message' => $notification->data['message'] ?? '',
                    'time' => $notification->created_at
                ]) . "\n\n";

                // Tandai sudah dikirim agar tidak dikirim ulang
                DB::table('notifications')->where('id', $notification->id)->update([
                    'read_at' => now()
                ]);

                ob_flush();
                flush();
            }
        });
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'Connection' => 'keep-alive',
    ]);
}
```

Kelebihan Pendekatan Ini:

- Redis tidak digunakan untuk menyimpan payload → tetap ringan
- Notifikasi tetap disimpan dan di-query dari DB Laravel
- Real-time, tanpa polling terus-menerus
- Cocok untuk Laravel + SSE tanpa third-party service

Catatan:

- Redis Pub/Sub bersifat blocking (proses akan menunggu pesan).
- Laravel Redis facade tidak didesain untuk operasi seperti itu (akan menggantung worker / request).
- implementasi diatas menggunakan \Redis PHP extension langsung:
- Pastikan extension phpredis sudah aktif (php -m | grep redis)
- phpredis lebih cepat dan lebih stabil untuk Pub/Sub

Kalau kamu pakai Predis (bukan ekstensi phpredis), kamu bisa juga pakai cara ini:

```php
$client = new \Predis\Client();

$client->pubSubLoop()->subscribe("notifications:1", function ($message) {
    // Tidak semua versi predis support ini dengan baik di Laravel
});

```


### Kasus: 1000 User SSE dengan Redis Pub/Sub

Setiap user membuka koneksi SSE → 1 koneksi HTTP yang panjang (long-lived), dan jika kamu pakai Redis->subscribe() langsung, maka:
- 1 koneksi Redis per user (karena tiap SSE listener buka Redis sendiri)
- 1 koneksi HTTP (SSE) dari browser ke server

#### Berapa banyak koneksi yang bisa ditangani?

1. Koneksi Redis
    Redis sendiri sangat cepat dan ringan, tapi...
    Redis default bisa menangani ~10.000 koneksi secara simultan.
    Tapi jika kamu pakai 1000 Redis client aktif (1 per user), itu dianggap berat karena Redis Pub/Sub bersifat blocking (1 thread/process per koneksi di server kamu).

    Masalahnya bukan Redis-nya, tapi proses PHP kamu.
    Laravel (tanpa Octane/worker) tidak efisien untuk long-running process seperti subscribe()

2. Koneksi HTTP
    - Nginx default bisa handle ~1024 koneksi aktif (tergantung konfigurasi worker_connections)
    - HTTP SSE tidak berat per koneksi, tapi perlu server yang bisa handle concurrent connection tinggi
    - Realita: Laravel biasa (tanpa Octane) bukan ideal untuk >100 user SSE Karena:
        - Redis->subscribe() blocking = 1 PHP process hang di tiap koneksi
        - PHP-FPM bukan untuk long-lived request

Solusi:
1. Laravel Octane (Swoole/RoadRunner)
2. Pisahkan SSE handler dari Laravel misal, Laravel → Publish to Redis → NodeJS/GO SSE Worker akan lebih scallable
3. Gunakan Redis Flag (get/set) + Polling ringan (Lebih ringan dan tidak blocking. Bisa tahan ratusan user tanpa worker khusus.)

Kondisi	Rekomendasi
< 100 user aktif	Laravel + SSE + Redis Flag (polling tiap 1–2 detik)
100–1000 user aktif	Laravel Octane + Redis Pub/Sub
>1000 user aktif	Pisahkan SSE server (NodeJS/Go) untuk streaming Redis Pub/Sub
Mau cepat & murah	Redis flag + polling ringan sudah cukup

## Arsitektur Redis Pub/Sub + SSE dengan Go/NodeJS

1. Laravel App

    - Menyimpan notifikasi ke DB (notifications)
    - Mempublish sinyal ke Redis channel notifications:<user_id>

2. SSE Server (Node.js atau Go)
    - Subscribe ke Redis Pub/Sub
    - Saat ada pesan baru, tarik data dari database (opsional — bisa pakai Laravel API juga)
    - Kirim ke client via SSE

```go
package main

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"log"
	"net/http"

	"github.com/go-redis/redis/v8"
	_ "github.com/lib/pq"
	"golang.org/x/net/context"
)

var ctx = context.Background()
var clients = make(map[string][]chan string) // map[user_id] to list of channels

var rdb = redis.NewClient(&redis.Options{
	Addr: "localhost:6379",
})

var db *sql.DB

func initDB() {
	var err error
	db, err = sql.Open("postgres", "postgres://user:pass@localhost:5432/dbname?sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}
}

func redisSubscribe(userID string) {
	pubsub := rdb.Subscribe(ctx, "notifications:"+userID)
	ch := pubsub.Channel()

	go func() {
		for msg := range ch {
			// Ambil notifikasi dari DB (atau via Laravel API)
			row := db.QueryRow("SELECT id, data, created_at FROM notifications WHERE notifiable_id=$1 ORDER BY created_at DESC LIMIT 1", userID)

			var id string
			var data string
			var createdAt string

			err := row.Scan(&id, &data, &createdAt)
			if err != nil {
				log.Println("DB error:", err)
				continue
			}

			payload, _ := json.Marshal(map[string]interface{}{
				"id":    id,
				"data":  json.RawMessage(data),
				"time":  createdAt,
			})

			for _, ch := range clients[userID] {
				ch <- string(payload)
			}
		}
	}()
}

func sseHandler(w http.ResponseWriter, r *http.Request) {
	userID := r.URL.Query().Get("user_id")
	if userID == "" {
		http.Error(w, "user_id is required", http.StatusBadRequest)
		return
	}

	flusher, ok := w.(http.Flusher)
	if !ok {
		http.Error(w, "Streaming unsupported", http.StatusInternalServerError)
		return
	}

	messageChan := make(chan string)
	clients[userID] = append(clients[userID], messageChan)

	// Subscribe Redis channel (once per user)
	if len(clients[userID]) == 1 {
		redisSubscribe(userID)
	}

	w.Header().Set("Content-Type", "text/event-stream")
	w.Header().Set("Cache-Control", "no-cache")
	w.Header().Set("Connection", "keep-alive")

	for {
		select {
		case msg := <-messageChan:
			fmt.Fprintf(w, "data: %s\n\n", msg)
			flusher.Flush()
		case <-r.Context().Done():
			log.Printf("Client %s disconnected", userID)
			return
		}
	}
}
func main() {
	initDB()
	http.HandleFunc("/sse", sseHandler)
	log.Println("SSE server running on :8080")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```

```html
<script>
const userId = 1;
const es = new EventSource("http://localhost:8080/sse?user_id=" + userId);

es.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log("New notification:", data);
};
</script>
```

## Issue2 laravel stream

- Pastikan Laravel tidak menjalankan middleware yang bisa mem-buffer response seperti ThrottleRequests atau TrimStrings
- check php.ini `output_buffering=4096 → artinya PHP akan buffer output hingga 4KB sebelum dikirim`
- Tambahkan `ob_implicit_flush(true)` di awal `stream()` untuk memastikan semua echo langsung flush
- tambahkan header

```
'Cache-Control' => 'no-cache, no-transform',
'X-Accel-Buffering' => 'no', //untuk Nginx agar jangan buffer output
'Content-Type' => 'text/event-stream',
'Connection' => 'keep-alive',
```

```
location /sse {
    proxy_pass http://your-php-backend;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_set_header Cache-Control 'no-cache';
    proxy_set_header X-Accel-Buffering no;
    chunked_transfer_encoding on;
    proxy_buffering off; # ← WAJIB MATIKAN
    proxy_cache off;
    keepalive_requests 1000;
}
```

- proxy_http_version 1.1 → biar keep-alive jalan.
- proxy_set_header Connection '' → jangan override ke close.
- proxy_read_timeout 3600 → koneksi SSE bisa bertahan 1 jam.
- X-Accel-Buffering no → cegah Nginx buffering, supaya data langsung dikirim ke browser.

Catatan Penting:

- Gunakan header SSE yang tepat: Content-Type: text/event-stream
- Gunakan echo, bukan print_r, dd(), atau var_dump() — karena itu bisa memanggil formatter Laravel
- Gunakan set_time_limit(0); jika kamu ingin stream berjalan terus-menerus

## Security SSE

EventSource tidak bisa mengirim header kustom seperti Authorization

solusi

### Gunakan token di URL query string

```js
const token = 'your_jwt_token_here';
const evtSource = new EventSource(`/sse/notifications?token=${token}`);
```

```php
public function stream(Request $request)
{
    $token = $request->query('token');

    if (!$token || !auth()->once(['api_token' => $token])) {
        abort(403);
    }

    return response()->stream(...);
}

````

- Token di URL terlihat di access logs dan browser history, jadi:
- Batasi masa aktif token
- Gunakan hanya untuk SSE (bukan token utama login)
- Pastikan server mengirim header CORS jika lintas domain:

```
'Access-Control-Allow-Origin' => '*',
'Access-Control-Allow-Headers' => 'Authorization',
```

### Polyfill (Experimental)

Kalau kamu benar-benar butuh header Authorization, kamu bisa pakai polyfill seperti event-source-polyfill yang menggunakan fetch + ReadableStream.

Tapi ini:

- Tidak native
- Lebih kompleks
- Tidak seandal EventSource asli

### Via COOKIE

asalkan sebelum halaman sse tsb, user sudah login, nanti bisa di verifikasi di backend

## Tips menggunakan redis subscribe di OCTANE

Redis client (phpredis) tidak support digunakan ulang antar process/worker dengan aman, terutama dalam mode subscribe() yang long-lived

Karena Octane menjalankan Laravel dalam long-lived workers

Artinya:

- Laravel tidak me-reboot app di setiap request (berbeda dari tradisional PHP-FPM)
- Setiap request akan diserahkan ke worker yang tetap hidup
- Jika 1 worker terjebak di proses blocking (misalnya subscribe), maka worker itu tidak bisa melayani request lain

solusi: gunakan worker terpisah untuk subscribe Redis.

buat command baru

```bash
php artisan make:command RedisSubscriber
```

```php
// app/Console/Commands/RedisSubscriber.php
public function handle()
{
    Redis::subscribe(['notif-channel'], function ($message) {
        // Simpan ke database, broadcast via SSE, dsb
    });
}
```

Gunakan Laravel Octane Hooks jika perlu clean Redis per worker

```php
Octane::tick(function () {
    Redis::disconnect();
});

Octane::booting(fn () => Redis::disconnect());
```

### Kalau kamu pakai Predis (REDIS_CLIENT=predis)

maka kamu tidak akan kena error read error on connection seperti di PhpRedis, karena:

Predis adalah client murni PHP (pure PHP)
✔ Tidak persistent
✔ Tidak share koneksi antarraya process
✔ Aman digunakan bahkan dalam Octane (Swoole/RoadRunner)

Walaupun Predis aman digunakan di Octane, subscribe() tetap blocking, jd tetap buat command terpisah

## Arsitektur: Redis Pub/Sub + SSE

![sse](./sse.png)

worker

```php
// app/Console/Commands/RedisNotifier.php
public function handle()
{
    Redis::subscribe(['notif-channel'], function ($message) {
        $payload = json_decode($message, true);
        $userId = $payload['user_id'];

        // Kirim ke Redis stream khusus user
        Redis::rpush("sse:notifications:user:{$userId}", $message);
    });
}
```

sse endpoint

```php
Route::get('/sse/notifications', function (Request $request) {
    $user = auth()->user(); // Asumsi pakai session

    return response()->stream(function () use ($user) {
        $redisKey = "sse:notifications:user:{$user->id}";

        while (true) {
            // Tunggu data via Redis List (blocking 10 detik)
            $message = Redis::blpop($redisKey, 10);

            if ($message) {
                echo "data: {$message[1]}\n\n";
                ob_flush(); flush();
            }
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'X-Accel-Buffering' => 'no',
        'Connection' => 'keep-alive',
    ]);
});

```

`Redis::blpop($key, 10)` di SSE route:

- Ini blocking tapi ada timeout → worker tidak hang selamanya
- Aman untuk dipakai dalam Octane

frontend

```js
const evtSource = new EventSource("/sse/notifications");

evtSource.onmessage = function (event) {
    const data = JSON.parse(event.data);
    console.log("Notifikasi:", data);
};

```


## Kemungkinan SSE di close connection

SSE butuh data dikirim secara berkala, atau koneksi dianggap "idle" dan bisa:

- Ditutup oleh browser
- Ditutup oleh proxy (misalnya Nginx, Cloudflare, dsb)
- Dianggap timeout oleh Laravel atau server

Solusi
- Kirim “heartbeat” setiap beberapa detik

```
Route::get('/sse', function () {
    return response()->stream(function () {
        while (true) {
            // kirim heartbeat setiap 25 detik
            echo "event: ping\ndata: {}\n\n";
            ob_flush(); flush();
            sleep(25);
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'X-Accel-Buffering' => 'no',
        'Connection' => 'keep-alive',
    ]);
});
```

## Redis RPUSH dan LRANGE

Saat kirim notifikasi:

```
Redis::rpush("notifications:{$userId}", json_encode([
    'id' => Str::uuid(),
    'title' => 'Pesan baru',
    'message' => 'Kamu punya pesan baru!',
    'timestamp' => now()->timestamp,
]));
```

Di SSE (polling per detik misalnya):

```
$notifs = Redis::lrange("notifications:{$userId}", 0, -1);
Redis::del("notifications:{$userId}");
```

Kenapa pakai RPUSH langsung lebih baik?

Karena:

- Lebih sederhana: Tidak perlu proses subscriber untuk dengarkan channel.
- Tidak blocking: SSE endpoint cukup polling LRANGE dari Redis list → cepat dan ringan.
- Reliable: Data disimpan di Redis (misal list per user), jadi bisa diambil kapan saja (tidak seperti pub/sub yang ephemeral).
- Scalable: Bisa menyimpan notifikasi user dalam key list notifications:{user_id} → read dan delete fleksibel.

Kekurangan:
- ketika 1 user buka byk browser atau tab halaman yg sama, yg terkirim notifikasi itu hanya 1 client ?

solusi :
- Pakai Redis list notifications:{userId}.
- Jangan hapus langsung (DEL) di client.
- Tambahkan TTL (pakai EXPIRE) agar Redis gak penuh.
- Client jaga sendiri last_seen_id → hanya tampilkan yang baru.

```
// Ambil semua notifikasi dari Redis
$all = Redis::lrange("notifications:{$userId}", 0, -1);

// Filter berdasarkan last seen
$lastSeen = request()->get('last_seen'); // dikirim dari client misalnya
$filtered = collect($all)->filter(function ($json) use ($lastSeen) {
    $notif = json_decode($json, true);
    return $notif['id'] > $lastSeen;
});
```

full code
```
return response()->stream(function () {
            $userId = auth()->id(); // user login
            $cacheKey = "notifications:user:{$userId}";
            $lastSeenId = null;

            // pastikan output langsung dikirim
            ini_set('output_buffering', 'off');
            ini_set('zlib.output_compression', 'off');
            ob_implicit_flush(true);

            while (true) {
                // ambil data notifikasi
                $all = Redis::lrange($cacheKey, 0, -1);

                if ($all) {
                    foreach ($all as $raw) {
                        $notif = json_decode($raw, true);

                        // skip notifikasi lama
                        if ($lastSeenId && $notif['id'] <= $lastSeenId) {
                            continue;
                        }

                        echo "data: " . json_encode($notif) . "\n\n";
                        $lastSeenId = $notif['id'];
                    }

                    // flush output ke client
                    ob_flush();
                    flush();
                }

                // sleep sebentar biar tidak makan CPU
                sleep(2);
            }
        }, 200, [
            'Content-Type'      => 'text/event-stream',
            'Cache-Control'     => 'no-cache',
            'X-Accel-Buffering' => 'no',
            'Connection'        => 'keep-alive',
        ]);
    }
```

```js
document.addEventListener('DOMContentLoaded', () => {
    const evtSource = new EventSource("/sse/notifications");

    evtSource.onmessage = function (event) {
        const data = JSON.parse(event.data);
        console.log("Notif:", data);
    };

    evtSource.onerror = function () {
        console.error("SSE connection error");
    };
});
```
