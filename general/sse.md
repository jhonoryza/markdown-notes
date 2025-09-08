# SSE (Server-Sent Events) dengan Laravel & Redis

Panduan lengkap implementasi SSE (Server-Sent Events) di Laravel, termasuk integrasi Redis untuk notifikasi real-time.

---

## 1. Dasar SSE di Laravel

**Route & Controller:**
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

**Frontend:**
```js
const evtSource = new EventSource("/sse");
evtSource.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log("Data dari server:", data);
};
```

> **Catatan:** Pastikan Nginx `proxy_buffering off` agar data langsung dikirim ke client.

---

## 2. Simpan Data ke Redis

**Menyimpan notifikasi ke Redis:**
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

---

## 3. Simpan Data ke Database

**Contoh Notifikasi ke Database:**
```php
class UserRegisteredNotification extends Notification
{
    protected $user;
    public function __construct($user) { $this->user = $user; }
    public function via($notifiable) { return ['database']; }
    public function toDatabase($notifiable) {
        return [
            'title' => 'User Baru',
            'message' => $this->user->name . ' telah mendaftar.',
        ];
    }
}
```

**Streaming dari Database:**
```php
return response()->stream(function () {
    $userId = 1; // Sesuaikan dengan user (misal admin)
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

---

## 4. Redis Get/Set Flag

- Simpan flag di Redis saat notifikasi dibuat:
```php
Redis::set("user:{$admin->id}:has_new_notification", true);
```
- SSE loop hanya cek flag:
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

---

## 5. Redis Pub/Sub

- Publish ke Redis saat notifikasi dibuat:
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

**Kelebihan:**
- Event-driven, tanpa polling
- Skalabel jika Redis cukup kuat

**Catatan:**
- Redis Pub/Sub tidak menyimpan pesan (hanya realtime)
- Gunakan Laravel Octane/worker agar tidak blocking

---

## 6. Redis Pub/Sub + Database

- Publish ke Redis, data tetap di DB:
```php
$admin->notify(new UserRegisteredNotification($event->user));
Redis::publish("notifications:{$admin->id}", 'new');
```
- Subscribe Redis, lalu query DB:
```php
public function stream()
{
    $userId = 1; // atau dari session/auth
    $channel = "notifications:$userId";
    return response()->stream(function () use ($userId, $channel) {
        $redis = new \Redis();
        $redis->connect('127.0.0.1', 6379);
        $redis->subscribe([$channel], function ($redis, $chan, $msg) use ($userId) {
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

**Kelebihan:**
- Redis tidak digunakan untuk menyimpan payload
- Notifikasi tetap disimpan dan di-query dari DB Laravel
- Real-time, tanpa polling terus-menerus
- Cocok untuk Laravel + SSE tanpa third-party service

**Catatan:**
- Redis Pub/Sub bersifat blocking (proses akan menunggu pesan)
- Laravel Redis facade tidak didesain untuk operasi seperti itu (akan menggantung worker / request)
- Implementasi di atas menggunakan \Redis PHP extension langsung
- Pastikan extension phpredis sudah aktif (`php -m | grep redis`)
- phpredis lebih cepat dan lebih stabil untuk Pub/Sub

Jika menggunakan Predis (bukan ekstensi phpredis):
```php
$client = new \Predis\Client();

$client->pubSubLoop()->subscribe("notifications:1", function ($message) {
    // Tidak semua versi predis support ini dengan baik di Laravel
});
```

---

### Kasus: 1000 User SSE dengan Redis Pub/Sub

Setiap user membuka koneksi SSE → 1 koneksi HTTP panjang (long-lived), dan jika pakai Redis->subscribe() langsung:
- 1 koneksi Redis per user (karena tiap SSE listener buka Redis sendiri)
- 1 koneksi HTTP (SSE) dari browser ke server

#### Berapa banyak koneksi yang bisa ditangani?

1. **Koneksi Redis**
    - Redis sangat cepat dan ringan, default bisa menangani ~10.000 koneksi simultan
    - 1000 Redis client aktif (1 per user) berat karena Pub/Sub blocking (1 thread/process per koneksi di server)
    - Masalah utama: proses PHP, bukan Redis-nya
    - Laravel (tanpa Octane/worker) tidak efisien untuk long-running process seperti subscribe()

2. **Koneksi HTTP**
    - Nginx default bisa handle ~1024 koneksi aktif (tergantung konfigurasi worker_connections)
    - HTTP SSE tidak berat per koneksi, tapi server harus bisa handle concurrent connection tinggi
    - Laravel biasa (tanpa Octane) bukan ideal untuk >100 user SSE
        - Redis->subscribe() blocking = 1 PHP process hang di tiap koneksi
        - PHP-FPM bukan untuk long-lived request

**Solusi:**
1. Laravel Octane (Swoole/RoadRunner)
2. Pisahkan SSE handler dari Laravel (Laravel → Publish to Redis → NodeJS/GO SSE Worker)
3. Gunakan Redis Flag (get/set) + Polling ringan (bisa tahan ratusan user tanpa worker khusus)

| Kondisi              | Rekomendasi                                      |
|----------------------|--------------------------------------------------|
| < 100 user aktif     | Laravel + SSE + Redis Flag (polling 1–2 detik)   |
| 100–1000 user aktif  | Laravel Octane + Redis Pub/Sub                   |
| >1000 user aktif     | SSE server terpisah (NodeJS/Go) + Redis Pub/Sub  |
| Mau cepat & murah    | Redis flag + polling ringan                      |

---

## 7. Arsitektur Redis Pub/Sub + SSE dengan Go/NodeJS

1. **Laravel App**
    - Menyimpan notifikasi ke DB (notifications)
    - Mempublish sinyal ke Redis channel `notifications:<user_id>`
2. **SSE Server (Node.js/Go)**
    - Subscribe ke Redis Pub/Sub
    - Saat ada pesan baru, tarik data dari database (atau via Laravel API)
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

**Frontend:**
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

---

## 8. Tips Laravel SSE

- Nonaktifkan middleware yang mem-buffer response (ThrottleRequests, TrimStrings)
- Cek `php.ini` → `output_buffering=4096` (ubah ke off jika perlu)
- Tambahkan `ob_implicit_flush(true)` di awal stream()
- Header penting:
    - `'Cache-Control' => 'no-cache, no-transform'`
    - `'X-Accel-Buffering' => 'no'` (untuk Nginx)
    - `'Content-Type' => 'text/event-stream'`
    - `'Connection' => 'keep-alive'`

**Nginx config:**
```
location /sse {
    proxy_pass http://your-php-backend;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_set_header Cache-Control 'no-cache';
    proxy_set_header X-Accel-Buffering no;
    chunked_transfer_encoding on;
    proxy_buffering off;
    proxy_cache off;
    keepalive_requests 1000;
}
```

- proxy_http_version 1.1 → biar keep-alive jalan
- proxy_set_header Connection '' → jangan override ke close
- proxy_read_timeout 3600 → koneksi SSE bisa bertahan 1 jam
- X-Accel-Buffering no → cegah Nginx buffering, supaya data langsung dikirim ke browser

**Catatan Penting:**
- Gunakan header SSE yang tepat: Content-Type: text/event-stream
- Gunakan echo, bukan print_r, dd(), atau var_dump() — karena itu bisa memanggil formatter Laravel
- Gunakan set_time_limit(0); jika ingin stream berjalan terus-menerus

---

## 9. Security SSE

EventSource tidak bisa mengirim header kustom seperti Authorization.

**Solusi:**

### a. Gunakan token di URL query string
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
```
- Token di URL terlihat di access logs dan browser history
- Batasi masa aktif token
- Gunakan hanya untuk SSE (bukan token utama login)
- Pastikan server mengirim header CORS jika lintas domain

### b. Polyfill (Experimental)
- Jika butuh header Authorization, bisa pakai polyfill seperti event-source-polyfill (fetch + ReadableStream)
- Tidak native, lebih kompleks, tidak seandal EventSource asli

### c. Via COOKIE
- Asalkan sebelum halaman SSE, user sudah login, bisa diverifikasi di backend

---

## 10. Redis Subscribe di Octane

- Redis client (phpredis) tidak aman dipakai antar worker dalam mode subscribe()
- Solusi: buat command worker terpisah

Karena Octane menjalankan Laravel dalam long-lived workers

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

- Untuk Predis (REDIS_CLIENT=predis): aman di Octane, tapi subscribe tetap blocking

---

## 11. Arsitektur Redis Pub/Sub + SSE (Octane/Worker)

![sse](./sse.png)

**Worker:**
```php
// app/Console/Commands/RedisNotifier.php
public function handle()
{
    Redis::subscribe(['notif-channel'], function ($message) {
        $payload = json_decode($message, true);
        $userId = $payload['user_id'];
        Redis::rpush("sse:notifications:user:{$userId}", $message);
    });
}
```

**SSE Endpoint:**
```php
Route::get('/sse/notifications', function (Request $request) {
    $user = auth()->user();
    return response()->stream(function () use ($user) {
        $redisKey = "sse:notifications:user:{$user->id}";
        while (true) {
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
- Blocking tapi ada timeout → worker tidak hang selamanya
- Aman untuk dipakai dalam Octane

**Frontend:**
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

---

## 12. Heartbeat (Ping) agar Koneksi Tidak Ditutup

- Kirim event ping setiap beberapa detik agar koneksi tidak dianggap idle
```php
Route::get('/sse', function () {
    return response()->stream(function () {
        while (true) {
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

---

## 13. Redis RPUSH & LRANGE untuk Notifikasi

- Kirim notifikasi:
```php
Redis::rpush("notifications:{$userId}", json_encode([
    'id' => Str::uuid(),
    'title' => 'Pesan baru',
    'message' => 'Kamu punya pesan baru!',
    'timestamp' => now()->timestamp,
]));
```
- Ambil notifikasi di SSE:
```php
$notifs = Redis::lrange("notifications:{$userId}", 0, -1);
Redis::del("notifications:{$userId}");
```

**Kelebihan:**
- Sederhana, tidak blocking
- Data bisa diambil kapan saja
- Scalable (list per user)

**Solusi multi-tab:**
- Jangan hapus langsung (DEL) di client
- Tambahkan TTL (EXPIRE) agar Redis tidak penuh
- Client simpan last_seen_id, hanya tampilkan yang baru

**Contoh streaming:**
```php
return response()->stream(function () {
    $userId = auth()->id();
    $cacheKey = "notifications:user:{$userId}";
    $lastSeenId = null;
    ini_set('output_buffering', 'off');
    ini_set('zlib.output_compression', 'off');
    ob_implicit_flush(true);
    while (true) {
        $all = Redis::lrange($cacheKey, 0, -1);
        if ($all) {
            foreach ($all as $raw) {
                $notif = json_decode($raw, true);
                if ($lastSeenId && $notif['id'] <= $lastSeenId) continue;
                echo "data: " . json_encode($notif) . "\n\n";
                $lastSeenId = $notif['id'];
            }
            ob_flush();
            flush();
        }
        sleep(2);
    }
}, 200, [
    'Content-Type'      => 'text/event-stream',
    'Cache-Control'     => 'no-cache',
    'X-Accel-Buffering' => 'no',
    'Connection'        => 'keep-alive',
]);
```

**Frontend:**
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

---

## Referensi
- [Laravel Docs: Broadcasting](https://laravel.com/docs/broadcasting)
- [Redis Pub/Sub](https://redis.io/docs/manual/pubsub/)
- [MDN: Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
