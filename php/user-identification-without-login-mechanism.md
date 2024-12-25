# User Identification without Login Mechanism

## Pendekatan Cookie

Cara Kerja dengan Cookie di Frontend dan Backend

Backend akan mengatur cookie dengan informasi tertentu, seperti token atau flag,
saat user melakukan claim hadiah. Cookie ini disimpan di browser user.

Frontend tidak perlu secara eksplisit mengelola cookie, karena browser secara
otomatis menyertakan cookie pada setiap request ke domain yang sama (jika cookie
diatur dengan benar).

Kelebihan dan Kekurangan Pendekatan Cookie

Kelebihan:

- Sederhana
- Browser mengelola cookie secara otomatis.

Kekurangan:

- Cookie bisa dihapus oleh user.
- Kurang efektif untuk mode incognito atau perangkat yang berbeda.

contoh fetch

```js
const response = await fetch("https://backend-domain.com/api/claim-reward", {
    method: "POST",
    credentials: "include", // Mengaktifkan pengiriman kredensial (cookie)
    headers: {
        "Content-Type": "application/json",
    },
});
```

contoh axios

```js
const response = await axios.post(
    "https://backend-domain.com/api/claim-reward",
    {},
    { withCredentials: true }, // Mengaktifkan pengiriman kredensial
);
```

Perbedaan fetch vs Axios dalam Mengelola Kredensial

| Fitur                  | Fetch                             | Axios                                 |
| ---------------------- | --------------------------------- | ------------------------------------- |
| Konfigurasi Kredensial | Menggunakan credentials (include) | Menggunakan withCredentials: true     |
| Default Behavior       | same-origin                       | Tidak mengirim kredensial             |
| Global Configuration   | Harus membungkus fetch            | Dapat diatur secara global (defaults) |
| Browser Support        | Native di browser modern          | Memerlukan library eksternal          |

Hal yang Perlu Diperhatikan

1. HTTPS dan SameSite:

Jika menggunakan SameSite=None untuk cookie, backend harus berjalan di HTTPS.
Browser modern memerlukan ini untuk alasan keamanan.

2. CORS:

Backend harus dikonfigurasi untuk mendukung CORS dengan kredensial
(supports_credentials: true).

3. Mode Incognito:

Beberapa browser mungkin membatasi atau menghapus cookie dalam mode incognito,
yang dapat memengaruhi mekanisme berbasis cookie.

4. Alternatif:

Jika Anda ingin menghindari pengelolaan kredensial manual, library seperti Axios
dapat lebih nyaman dibandingkan dengan fetch.

```php
    $cookieName = 'reward_claimed';

    // Periksa apakah cookie sudah ada
    if ($request->cookie($cookieName)) {
        return response()->json([
            'message' => 'Anda sudah mengklaim hadiah!'
        ], 403);
    }

    $response = response()->json([
        'message' => 'Hadiah berhasil diklaim!'
    ]);

    return $response->cookie(
        $cookieName, 
        true, 
        60 * 24 * 30, // 30 hari
        null, 
        null, 
        true, // Secure
        true, // HttpOnly
        false, // Raw
        'None' // SameSite
    );
```

## Konfigurasi Cors

```php
return [
    'paths' => ['api/*', '/csrf-token'],

    'allowed_methods' => ['*'],

    'allowed_origins' => ['http://localhost:3000'], // Domain frontend 

    'allowed_origins_patterns' => [],

    'allowed_headers' => ['*'],

    'exposed_headers' => [],

    'max_age' => 0,

    'supports_credentials' => true, // Penting untuk mengizinkan kredensial
];
```

## Kombinasi Cookie dan Backend Session Id

1. Frontend Mengakses /csrf-token (Pertama Kali):

- Saat frontend pertama kali dibuka, frontend mengirimkan permintaan GET ke
  endpoint /csrf-token untuk mendapatkan token CSRF.
- Backend mengatur sesi jika belum dimulai, dan menyertakan cookie
  backend_session pada response.
- Backend mengembalikan token CSRF melalui respons dalam bentuk JSON.

2. Frontend Menyimpan CSRF Token:

- Setelah menerima respons, frontend menyimpan token CSRF yang diterima dari
  backend.

3. Frontend Mengirim Permintaan Klaim Hadiah:

- Saat pengguna submit form klaim hadiah, frontend mengirimkan permintaan POST
  ke API /claim-reward.
- CSRF Token yang diterima sebelumnya disertakan di dalam header X-CSRF-TOKEN.
- Cookie backend_session yang berisi ID sesi pengguna dikirim secara otomatis
  bersama permintaan karena sudah diatur oleh backend sebelumnya (jika
  menggunakan withCredentials: true di frontend)

`config/session.php`

```php
return [
    'driver' => 'cookie',
    'lifetime' => 10080, // Durasi sesi dalam menit (7 hari)
    'expire_on_close' => false, // Hapus cookie sesi saat browser ditutup
    'cookie' => 'backend_session',
    'secure' => env('SESSION_SECURE_COOKIE', false), // Gunakan `true` jika aplikasi berjalan di HTTPS
    'same_site' => 'none', // Cegah CSRF lintas situs
    'http_only' => true // cookie will only be accessible through HTTP protocol
];
```

`web/routes.php`

```php
Route::post('/claim-reward', function () {
    $sessionId = session()->getId();

    $period = Period::where('name', 'current')->first(); 
    if (!$period) {
        return response()->json(['message' => 'Periode tidak ditemukan.'], 404);
    }
    $endAt = Carbon::parse($period->end_at);
    $duration = $endAt->diffInMinutes(Carbon::now());

    if (session()->has('claimed')) {
        return response()->json(['message' => 'Anda sudah melakukan klaim.'], 403);
    }

    session(['claimed' => true]);

    $cookieLifetime = max($duration, 1); // Jangan sampai durasi jadi negatif, set minimal 1 menit
    $cookie = cookie('backend_session', session()->getId(), $cookieLifetime);

    return response()->json(['message' => 'Klaim berhasil!'])->withCookie($cookie);
});
```
