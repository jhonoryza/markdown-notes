# CSRF

CSRF (Cross-Site Request Forgery) adalah jenis serangan keamanan pada aplikasi
web di mana penyerang mengeksploitasi kepercayaan pengguna terhadap sebuah situs
web untuk mengirimkan permintaan yang tidak diinginkan dan berbahaya atas nama
pengguna tersebut.

## Contoh Serangan CSRF

Misalkan Anda login ke bank online di situs bank.com. Penyerang membuat halaman
HTML seperti ini:

```html
<form action="https://bank.com/transfer" method="POST">
    <input type="hidden" name="amount" value="1000000" />
    <input type="hidden" name="to_account" value="12345678" />
    <input type="submit" />
</form>
```

Saat Anda mengunjungi halaman ini, browser Anda secara otomatis mengirimkan
permintaan ke bank.com menggunakan cookie sesi Anda, dan transfer dana terjadi
tanpa persetujuan Anda

Untuk mengatasi hal tersebut salah satu nya menggunakan csrf token

1. Setiap permintaan yang melibatkan perubahan data (POST, PUT, DELETE) harus
   menyertakan token unik yang hanya diketahui oleh server dan pengguna.
2. Jika token tidak cocok, server menolak permintaan.

```js
'X-CSRF-TOKEN': csrfToken, // Kirim token CSRF di header
```

contoh implementasi di php framework laravel:

```php
Route::get('/csrf-token', function () {
    return response()->json(['csrf_token' => csrf_token()]);
});
```

add middleware

```php
\App\Http\Middleware\VerifyCsrfToken::class,
```
