# Level Logging di Laravel / Monolog

| Level        | Constant              | Angka | Keterangan                                                                 |
|--------------|----------------------|-------|----------------------------------------------------------------------------|
| **debug**    | `Logger::DEBUG`      | 100   | Informasi detail untuk debugging (misal isi variabel, flow kecil). Biasanya hanya dipakai saat dev. |
| **info**     | `Logger::INFO`       | 200   | Event umum, status normal (user login, job berhasil jalan).                |
| **notice**   | `Logger::NOTICE`     | 250   | Event penting tapi bukan error (misal service mulai, cache rebuild).       |
| **warning**  | `Logger::WARNING`    | 300   | Potensi masalah (misal disk hampir penuh, query lambat).                   |
| **error**    | `Logger::ERROR`      | 400   | Error yang butuh perhatian (exception, gagal kirim email).                 |
| **critical** | `Logger::CRITICAL`   | 500   | Kondisi serius, sistem sebagian gagal (database down).                     |
| **alert**    | `Logger::ALERT`      | 550   | Harus segera ditangani, sistem penting tidak jalan (payment service mati). |
| **emergency**| `Logger::EMERGENCY`  | 600   | Sistem tidak bisa dipakai sama sekali (total down).                        |

---

## Cara Kerja

Jika channel di config menggunakan:

```php
'level' => 'error',
```

Maka log dengan level **error**, **critical**, **alert**, dan **emergency** akan dicatat.  
Tetapi **debug**, **info**, **notice**, dan **warning** tidak akan masuk.

Jika level = `debug`, semua log akan masuk.

---

### Contoh Penggunaan

```php
Log::debug('Test debug');
Log::info('User login');
Log::warning('Low disk space');
Log::error('DB down');
```

Hasilnya bergantung pada level channel.

---

## Channel Stack

Jika menggunakan channel stack:

```php
'stack' => [
    'driver' => 'stack',
    'channels' => ['daily', 'custom-database'],
    'ignore_exceptions' => false,
],
```

Setiap kali memanggil:

```php
Log::error('Something went wrong');
```

Laravel akan mencoba menulis log ke semua channel di dalam array (`daily` dan `custom-database`).

---

### Kasus: Tabel logs (custom-database) tidak ada

- **Channel daily** → berhasil tulis ke `storage/logs/laravel-YYYY-MM-DD.log` ✅
- **Channel custom-database** → ketika `LoggingHandler` mencoba insert ke tabel logs, akan kena exception "Base table or view not found" ❌

Karena di config `ignore_exceptions = false`, maka exception itu akan diteruskan ke aplikasi.  
👉 Akibatnya aplikasi bisa error / crash hanya gara-gara log gagal ditulis ke DB.

---

### Jika `ignore_exceptions = true`

```php
'stack' => [
    'driver' => 'stack',
    'channels' => ['daily', 'custom-database'],
    'ignore_exceptions' => true,
],
```

- Laravel akan mengabaikan error dari salah satu channel.
- Log tetap masuk ke daily (file).
- Gagal insert ke DB akan diabaikan.
- Aplikasi tetap jalan tanpa crash 🚀

---

## Kesimpulan

- `ignore_exceptions = false` (default) → lebih strict, tapi bisa bikin app error kalau channel rusak.
- `ignore_exceptions = true` → lebih aman di production, karena log channel yang gagal tidak mengganggu jalannya aplikasi.