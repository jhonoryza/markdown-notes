# Captcha

## Google Recaptcha v3

Google reCAPTCHA v3 bersifat invisible karena tidak memunculkan tantangan
(challenge) seperti pada versi sebelumnya. reCAPTCHA v3 bekerja dengan
memberikan skor (score-based system) berdasarkan perilaku pengguna di halaman
web. Bagaimana Google Menentukan Apakah User Itu Bot atau Bukan?

Google menggunakan machine learning dan berbagai sinyal perilaku pengguna untuk
memberikan skor antara 0.0 hingga 1.0, di mana:

    - Skor 1.0 → User sangat mungkin manusia.
    - Skor 0.0 → User sangat mungkin bot.

Faktor-faktor yang digunakan untuk menentukan skor meliputi:

    - Interaksi dengan halaman
        - Pergerakan mouse atau scrolling
        - Kecepatan mengetik
        - Klik pada elemen halaman
        - Pola navigasi yang mencurigakan

    - IP Address & Browser Fingerprinting
        - Google memeriksa apakah IP user sering digunakan oleh bot atau VPN/proxy mencurigakan.
        - Analisis data dari User-Agent, resolusi layar, cookies, dan data browser lainnya.

    - Histori Aktivitas Google
        - Jika pengguna sudah sering login ke akun Google dan memiliki aktivitas normal, mereka lebih mungkin dianggap sebagai manusia.
        - Sebaliknya, akun yang baru dibuat atau tidak pernah login bisa dianggap mencurigakan.

    - Analisis JavaScript & Keystroke Timing
        - Google menjalankan JavaScript di halaman dan mengukur kecepatan serta pola input pengguna.
        - Bot sering kali memiliki pola input yang terlalu sempurna atau terlalu cepat.

    - Interaksi Sebelumnya dengan reCAPTCHA
        - Jika user sebelumnya lolos verifikasi reCAPTCHA dengan skor tinggi, mereka lebih dipercaya di sesi berikutnya.

### Bagaimana Implementasinya?

Ketika reCAPTCHA v3 aktif di halaman, Google akan mengembalikan skor dalam
response API. Anda bisa menggunakan skor ini untuk:

    - Membiarkan user lanjut tanpa hambatan jika skornya tinggi.
    - Meminta user melakukan verifikasi tambahan (misalnya login atau OTP) jika skornya rendah.
    - Memblokir user jika sangat mencurigakan.

### Contoh Response dari reCAPTCHA v3 API:a.

```json
{
    "success": true,
    "score": 0.9,
    "action": "login",
    "challenge_ts": "2025-02-18T12:34:56Z",
    "hostname": "example.com"
}
```

Keuntungan reCAPTCHA v3:

    - Tanpa Interaksi: Pengguna tidak perlu menyelesaikan tantangan apa pun.
    - Skor Kepercayaan: Anda dapat menyesuaikan ambang batas skor sesuai kebutuhan.
    - Mudah Diintegrasikan: Hanya memerlukan sedikit modifikasi pada kode Anda.

Langkah Integrasi reCAPTCHA v3:

    1. Daftar reCAPTCHA v3:

        - Kunjungi Google reCAPTCHA.
        - Daftarkan situs Anda dan pilih reCAPTCHA v3.
        - Anda akan mendapatkan Site Key dan Secret Key.
    2. Tambahkan Skrip reCAPTCHA ke Halaman Web:

```html
<script
    src="https://www.google.com/recaptcha/api.js?render=your_site_key"
></script>
<script>
    function onSubmit(token) {
        document.getElementById("demo-form").submit();
    }
</script>
```

    3. Tambahkan reCAPTCHA ke Formulir:

```html
<form id="demo-form" action="/submit" method="POST">
    <input type="text" name="name" required />
    <input type="email" name="email" required />
    <button
        class="g-recaptcha"
        data-sitekey="your_site_key"
        data-callback="onSubmit"
        data-action="submit"
    >
        Submit
    </button>
</form>
```

    4. Validasi Token di Sisi Server:

    - Setelah formulir dikirim, Anda perlu memvalidasi token reCAPTCHA di sisi server menggunakan Secret Key.

    - Contoh validasi menggunakan PHP:

```php
<?php
$secretKey = "your_secret_key";
$response = $_POST['g-recaptcha-response'];
$remoteIp = $_SERVER['REMOTE_ADDR'];

$url = "https://www.google.com/recaptcha/api/siteverify?secret=$secretKey&response=$response&remoteip=$remoteIp";
$response = file_get_contents($url);
$responseKeys = json_decode($response, true);

if ($responseKeys["success"] && $responseKeys["score"] >= 0.5) { // Sesuaikan ambang batas skor
    // Formulir valid, proses data
} else {
    // Formulir tidak valid, tampilkan pesan error
}
?>
```

## Google Recaptcha v2

Google reCAPTCHA v2 memiliki dua versi:

    - reCAPTCHA v2 Checkbox: Pengguna harus mencentang kotak "I'm not a robot".

    - reCAPTCHA v2 Invisible: Tidak ada interaksi langsung dari pengguna. Tantangan muncul hanya jika sistem mendeteksi perilaku mencurigakan.

Kedua versi ini menggunakan teknik yang sama untuk membedakan antara manusia dan
bot, tetapi reCAPTCHA v2 Invisible lebih halus karena tidak memerlukan interaksi
pengguna kecuali sistem mendeteksi sesuatu yang mencurigakan. Bagaimana Google
reCAPTCHA v2 Menghitung/Membedakan User dan Bot?

Google reCAPTCHA v2 menggunakan kombinasi teknik canggih untuk menganalisis
perilaku pengguna dan menentukan apakah mereka manusia atau bot. Berikut adalah
beberapa faktor yang digunakan:

1. Analisis Perilaku Pengguna (Behavioral Analysis):

   Gerakan Mouse: Google menganalisis cara pengguna menggerakkan mouse. Manusia
   cenderung memiliki gerakan yang tidak teratur, sementara bot sering memiliki
   gerakan yang lurus dan mekanis.

   Klik dan Interaksi: Cara pengguna mengklik atau berinteraksi dengan elemen
   halaman web juga dianalisis. Bot mungkin mengklik dengan presisi yang tidak
   wajar atau dalam pola yang konsisten.

   Waktu yang Dihabiskan: Google memperhatikan berapa lama pengguna menghabiskan
   waktu di halaman sebelum menyelesaikan reCAPTCHA. Bot sering bekerja sangat
   cepat.

2. Cookies dan Aktivitas Browser:

   Google menggunakan cookies untuk melacak aktivitas pengguna di situs web yang
   menggunakan reCAPTCHA. Jika pengguna telah berinteraksi dengan reCAPTCHA
   sebelumnya, sistem dapat mengingatnya dan mengurangi tantangan.

   Browser fingerprinting juga digunakan untuk mengidentifikasi perilaku
   mencurigakan.

3. Alamat IP dan Reputasi Jaringan:

   Google memeriksa reputasi alamat IP pengguna. Jika alamat IP tersebut dikenal
   sebagai sumber bot atau aktivitas mencurigakan, tantangan akan lebih ketat.

   Jaringan yang sering digunakan untuk serangan bot (seperti VPN atau proxy)
   mungkin lebih sering mendapatkan tantangan.

4. Analisis Perangkat dan Lingkungan:

   Google menganalisis informasi perangkat (seperti sistem operasi, browser, dan
   resolusi layar) untuk mendeteksi ketidakwajaran.

   Jika perangkat atau lingkungan tampak tidak biasa (misalnya, menggunakan
   browser headless seperti Puppeteer), sistem akan menandainya sebagai bot.

5. Machine Learning:

   Google menggunakan model machine learning yang dilatih dengan miliaran contoh
   perilaku manusia dan bot. Model ini terus diperbarui untuk mengidentifikasi
   pola baru yang digunakan oleh bot.

   Bagaimana reCAPTCHA v2 Invisible Bekerja?

reCAPTCHA v2 Invisible bekerja dengan cara yang sama seperti reCAPTCHA v2
Checkbox, tetapi tantangan hanya muncul jika sistem mendeteksi perilaku
mencurigakan. Jika tidak, reCAPTCHA akan lolos secara otomatis tanpa interaksi
pengguna. Alur Kerja reCAPTCHA v2 Invisible:

    - Pemantauan Perilaku:

        - Saat pengguna mengakses halaman, reCAPTCHA mulai memantau perilaku mereka (gerakan mouse, klik, waktu, dll.).

    - Analisis:

        - Jika perilaku pengguna tampak normal (seperti manusia), reCAPTCHA akan memberikan token tanpa tantangan.

        - Jika perilaku mencurigakan terdeteksi, reCAPTCHA akan menampilkan tantangan (bisa berupa CAPTCHA gambar atau tantangan lainnya).

    - Token:

        - Jika tantangan berhasil diselesaikan (atau tidak diperlukan), reCAPTCHA akan menghasilkan token yang dikirim ke server Anda untuk validasi.

Perbedaan reCAPTCHA v2 Checkbox dan invisible

| Fitur              | reCAPTCHA v2 Checkbox                                      | reCAPTCHA v2 Invisible                                        |
| ------------------ | ---------------------------------------------------------- | ------------------------------------------------------------- |
| Interaksi Pengguna | Pengguna harus mencentang kotak.                           | Tidak ada interaksi kecuali terdeteksi mencurigakan.          |
| Tantangan          | Tantangan muncul setelah kotak dicentang.                  | Tantangan muncul hanya jika perilaku mencurigakan terdeteksi. |
| Pengalaman         | Pengguna	Lebih terlihat, mungkin mengganggu.               | Lebih halus dan tidak mengganggu.                             |
| Penggunaan Ideal   | Cocok untuk formulir yang memerlukan verifikasi eksplisit. | Cocok untuk formulir yang ingin minim interaksi pengguna.     |

Google reCAPTCHA v2 (baik Checkbox maupun Invisible) menggunakan kombinasi
analisis perilaku, cookies, reputasi IP, dan machine learning untuk membedakan
antara manusia dan bot. reCAPTCHA v2 Invisible memberikan pengalaman pengguna
yang lebih baik karena tantangan hanya muncul jika diperlukan.

Jika Anda ingin mengurangi gangguan bagi pengguna sambil tetap menjaga keamanan,
reCAPTCHA v2 Invisible adalah pilihan yang sangat baik. Namun, pastikan untuk
menguji implementasinya untuk memastikan bahwa tantangan tidak terlalu sering
muncul untuk pengguna yang sah.
