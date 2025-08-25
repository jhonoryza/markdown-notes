# Cara Membandingkan Kolom Datetime Berdasarkan Tanggal Saja di Laravel

Misal kamu punya kolom `published_at` bertipe `datetime`, tapi saat filtering ingin membandingkan tanggalnya saja (tanpa jam/menit/detik).

## 1. Bandingkan Tanggal Saja

Laravel Eloquent menyediakan method bawaan `whereDate()` untuk membandingkan hanya tanggal.

**Contoh:**

```php
// Ambil semua data yang published_at tanggal 2025-08-25
$posts = Post::whereDate('published_at', '2025-08-25')->get();
```

**Dengan operator:**

```php
// published_at >= 2025-08-01
$posts = Post::whereDate('published_at', '>=', '2025-08-01')->get();
```

## 2. Bandingkan dengan Kolom Lain

Jika ingin membandingkan dengan kolom lain (misal `expired_at`):

```php
$posts = Post::whereRaw('DATE(published_at) = DATE(expired_at)')->get();
```

## 3. Pakai Query Builder

```php
use Illuminate\Support\Facades\DB;

$posts = Post::where(DB::raw('DATE(published_at)'), '2025-08-25')->get();
```

> ⚡ Untuk kasus paling umum, cukup pakai `whereDate()` karena itu built-in Eloquent helper untuk membandingkan tanggal dari kolom datetime.

## 4. Bandingkan Full Datetime (Tanggal + Jam)

Jika ingin membandingkan tanggal beserta jam/menit/detik, jangan pakai `whereDate()`, tapi langsung pakai kolom datetime-nya.

**Contoh:**

```php
// Persis sama (tanggal + jam)
$posts = Post::where('published_at', '2025-08-25 14:30:00')->get();

// Dengan operator
$posts = Post::where('published_at', '>=', '2025-08-25 00:00:00')
             ->where('published_at', '<=', '2025-08-25 23:59:59')
             ->get();
```

## 5. Bandingkan Jam Saja

Gunakan `whereTime()`:

```php
// Ambil semua record dengan jam lebih besar dari 14:00
$posts = Post::whereTime('published_at', '>', '14:00:00')->get();
```

## 6. Kombinasi Tanggal & Jam

```php
$posts = Post::whereDate('published_at', '2025-08-25')
             ->whereTime('published_at', '>=', '08:00:00')
             ->whereTime('published_at', '<=', '17:00:00')
             ->get();
```

---

## Ringkasan

- **whereDate()** → hanya tanggal (abaikan jam).
- **whereTime()** → hanya jam/menit/detik (abaikan tanggal).
- **where() biasa** → full datetime (tanggal + jam).

---

## 7. whereBetween & whereIn untuk Date Only

Secara default, `whereBetween` dan `whereIn` bekerja langsung ke value datetime (ikut jam juga). Kalau mau hanya tanggal, bungkus kolom datetime pakai `DATE()`.

**whereBetween untuk date only:**

```php
use Illuminate\Support\Facades\DB;

$posts = Post::whereBetween(
    DB::raw('DATE(published_at)'),
    ['2025-08-01', '2025-08-25']
)->get();
```

**whereIn untuk date only:**

```php
$dates = ['2025-08-01', '2025-08-05', '2025-08-10'];

$posts = Post::whereIn(DB::raw('DATE(published_at)'), $dates)->get();
```

**Alternatif (lebih “eloquent way”):**

Biasanya untuk range waktu, pakai `whereBetween` langsung di kolom datetime tapi kasih batas jam manual:

```php
$posts = Post::whereBetween('published_at', [
    '2025-08-01 00:00:00',
    '2025-08-25 23:59:59',
])->get();
```

---

### Perbedaan

- `DB::raw('DATE(...)')` → benar-benar mengabaikan jam, jadi semua `2025-08-01 xx:yy:zz` dianggap sama.
- `whereBetween('published_at', [...])` → tetap ikut jam, tapi kamu bisa set jam `00:00:00 – 23:59:59` untuk mencakup full hari.