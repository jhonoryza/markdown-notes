## SSO (Single Sign-On):

- Mekanisme autentikasi yang memungkinkan pengguna mengakses beberapa aplikasi
  atau layanan dengan satu set kredensial.
- Bertujuan meningkatkan pengalaman pengguna dengan mengurangi jumlah login yang
  diperlukan.
- Fokus pada autentikasi.

Berikut beberapa metode SSO yang populer:

1. OpenID Connect (OIDC):

- Lapisan identitas di atas OAuth 2.0
- Lebih ringan dan modern dibandingkan SAML
- Cocok untuk aplikasi web dan mobile
- Menggunakan JSON Web Tokens (JWT)

2. Kerberos:

- Protokol autentikasi jaringan yang kuat
- Sering digunakan dalam lingkungan Windows (Active Directory)
- Berbasis tiket, bukan web

3. Web Services Federation (WS-Federation):

- Dikembangkan oleh Microsoft, IBM, dan lainnya
- Menggunakan token keamanan untuk autentikasi dan otorisasi
- Sering digunakan dengan Active Directory Federation Services (ADFS)

4. Central Authentication Service (CAS):

- Protokol SSO open-source
- Dikembangkan oleh Universitas Yale
- Populer di lingkungan akademik

5. LDAP (Lightweight Directory Access Protocol):

- Bukan protokol SSO, tapi sering digunakan sebagai backend untuk solusi SSO
- Menyediakan direktori terpusat untuk informasi pengguna

6. Social Login / OAuth 2.0 based SSO:

- Menggunakan akun media sosial atau layanan pihak ketiga (Google, Facebook,
  dll.)
- Berbasis OAuth 2.0 atau OpenID Connect

7. Token-based SSO:

- Menggunakan token (seperti JWT) untuk autentikasi
- Bisa diimplementasikan secara custom atau menggunakan framework seperti
  Laravel Passport

8. Form-based SSO:

- Metode sederhana di mana kredensial diteruskan antar aplikasi
- Kurang aman dibandingkan metode lain, tapi mudah diimplementasikan

9. FIDO (Fast IDentity Online):

- Standar autentikasi yang lebih baru
- Mendukung autentikasi tanpa password menggunakan biometrik atau perangkat
  keamanan

10. SAML

Penting untuk dicatat bahwa SAML adalah protokol yang lebih matang dan
komprehensif untuk SSO, Jika Anda bekerja dalam lingkungan enterprise dengan
banyak aplikasi legacy, SAML mungkin lebih sesuai.

- Menggunakan XML untuk pertukaran data autentikasi dan otorisasi
- Menggunakan konsep Identity Provider (IdP) dan Service Provider (SP) dengan
  pertukaran assertion XML.
- Lebih kompleks, tetapi menawarkan fitur keamanan yang lebih kuat dan detail.
- Sering digunakan dalam lingkungan enterprise, terutama untuk aplikasi
  on-premise.
- Sharing status login menggunakan assertion yang berisi informasi tentang
  identitas dan atribut pengguna
- Memiliki protokol Single Logout (SLO) bawaan.
- Menawarkan fitur keamanan bawaan seperti enkripsi XML dan tanda tangan
  digital.
- Standar yang lebih mapan untuk interoperabilitas antar sistem yang berbeda.

Setiap metode memiliki kelebihan dan kekurangannya sendiri. Pilihan metode
tergantung pada berbagai faktor seperti:

- Jenis aplikasi yang Anda miliki (web, mobile, desktop)
- Kebutuhan keamanan
- Infrastruktur yang sudah ada
- Kemudahan implementasi dan pemeliharaan
- Interoperabilitas dengan sistem lain

## OAuth 2.0

- Protokol otorisasi yang memungkinkan aplikasi pihak ketiga mengakses sumber
  daya terbatas dari layanan HTTP tanpa membagikan kredensial pengguna.
- Digunakan untuk memberikan akses terbatas ke data pengguna antar aplikasi.
- Fokus pada otorisasi, bukan autentikasi.
- Referensi
  [https://oauth2.thephpleague.com/authorization-server/which-grant/](https://oauth2.thephpleague.com/authorization-server/which-grant/)

![oauth2.0](https://minio.labkita.my.id/laravelblog/Screenshot%202024-09-10%20at%208.30.05%20AM.png)
![diag1](https://minio.labkita.my.id/laravelblog/Screenshot%202024-09-10%20at%208.57.43%20AM.png)
![diag2](https://minio.labkita.my.id/laravelblog/Screenshot%202024-09-10%20at%208.58.40%20AM.png)

OAuth 2.0 sering digunakan sebagai komponen dalam implementasi SSO, terutama
untuk mengelola otorisasi antar aplikasi setelah pengguna terotentikasi melalui
SSO.

Laravel Passport sebenarnya adalah implementasi server OAuth 2.0 untuk Laravel,
bukan sistem SSO itu sendiri. Namun, Passport bisa digunakan sebagai komponen
dalam membangun sistem SSO. Berikut penjelasan lebih detailnya:

Laravel Passport:

- Merupakan implementasi OAuth 2.0 server untuk Laravel.
- Utamanya digunakan untuk membuat API authentication.
- Menyediakan cara untuk mengelola access tokens.

Hubungannya dengan SSO:

- Passport bisa menjadi bagian dari implementasi SSO, tapi bukan SSO itu
  sendiri.
- Untuk membuat SSO lengkap, Anda perlu mengembangkan komponen tambahan di atas
  Passport.

Untuk menggunakan Laravel Passport dalam konteks SSO, Anda bisa:

1. Membuat satu aplikasi Laravel sebagai "authentication server" menggunakan
   Passport.
2. Mengonfigurasi aplikasi-aplikasi lain untuk menggunakan server ini untuk
   autentikasi.
3. Mengimplementasikan mekanisme untuk berbagi status login antar aplikasi.

Jadi, meskipun Laravel Passport bukan SSO, ia bisa menjadi fondasi yang kuat
untuk membangun sistem SSO custom. Perlu untuk mengembangkan logika tambahan
untuk mengelola sesi pengguna di berbagai aplikasi untuk mencapai fungsionalitas
SSO penuh.

Point 1 & 2 dapat dilihat contoh implementasi nya di
[dokumentasi laravel](https://laravel.com/docs/11.x/passport#main-content)
sementara untuk point 3 untuk berbagi status login antar aplikasi ada beberapa
pendekatan yg dapat dilakukan :

- Shared session storage:

Gunakan penyimpanan session terpusat seperti Redis. Konfigurasikan semua
aplikasi untuk menggunakan penyimpanan session yang sama.

- Token-based authentication:

Simpan access token di browser (misalnya dalam localStorage). Kirim token ini
dengan setiap permintaan ke aplikasi client. Aplikasi client memverifikasi token
dengan server autentikasi.

- Single logout:

Implementasikan endpoint logout di server autentikasi. Saat user logout,
bersihkan token dan session di server autentikasi. Kirim sinyal ke semua
aplikasi client untuk membersihkan session lokal mereka.

- Refresh token:

Gunakan refresh token untuk memperbarui access token tanpa login ulang.
Implementasikan mekanisme untuk memperbarui token secara otomatis di aplikasi
client.

- Middleware autentikasi:

Buat middleware di aplikasi client untuk memeriksa keberadaan dan validitas
token. Jika token tidak ada atau tidak valid, arahkan user kembali ke proses
login.

contoh implementasi pendekatan oidc di sisi authorization server

```php
<?php

// 1. Buat Model dan Migration untuk Client
php artisan make:model OAuthClient -m

// Dalam database/migrations/xxxx_xx_xx_create_oauth_clients_table.php
public function up()
{
    Schema::create('oauth_clients', function (Blueprint $table) {
        $table->id();
        $table->string('client_id')->unique();
        $table->string('client_secret');
        $table->string('redirect_uri');
        $table->timestamps();
    });
}

// 2. Buat Controller untuk endpoint OIDC
php artisan make:controller OIDCController

// Dalam app/Http/Controllers/OIDCController.php
use App\Models\OAuthClient;
use Firebase\JWT\JWT;

class OIDCController extends Controller
{
    public function discovery()
    {
        return response()->json([
            'issuer' => url('/'),
            'authorization_endpoint' => url('/oauth/authorize'),
            'token_endpoint' => url('/oauth/token'),
            'userinfo_endpoint' => url('/oauth/userinfo'),
            'jwks_uri' => url('/.well-known/jwks.json'),
            // Tambahkan endpoint lain sesuai kebutuhan
        ]);
    }

    public function authorize(Request $request)
    {
        // Validasi parameter
        $client = OAuthClient::where('client_id', $request->client_id)->firstOrFail();
        // Implementasikan logika autentikasi dan otorisasi
        // Generate kode otorisasi
        // Redirect ke redirect_uri dengan kode
    }

    public function token(Request $request)
    {
        // Validasi kode otorisasi atau refresh token
        // Generate access token dan ID token
        $payload = [
            'iss' => url('/'),
            'sub' => $user->id,
            'aud' => $client->client_id,
            'exp' => time() + 3600,
            'iat' => time(),
            // Tambahkan klaim lain sesuai kebutuhan
        ];
        $idToken = JWT::encode($payload, config('app.key'), 'HS256');
        
        return response()->json([
            'access_token' => '...',
            'token_type' => 'Bearer',
            'expires_in' => 3600,
            'id_token' => $idToken,
        ]);
    }

    public function userinfo(Request $request)
    {
        // Validasi access token
        // Kembalikan informasi pengguna
        return response()->json([
            'sub' => $user->id,
            'name' => $user->name,
            'email' => $user->email,
            // Tambahkan klaim lain sesuai kebutuhan
        ]);
    }
}

// 3. Tambahkan rute di routes/api.php
Route::get('/.well-known/openid-configuration', [OIDCController::class, 'discovery']);
Route::get('/oauth/authorize', [OIDCController::class, 'authorize']);
Route::post('/oauth/token', [OIDCController::class, 'token']);
Route::get('/oauth/userinfo', [OIDCController::class, 'userinfo'])->middleware('auth:api');

// 4. Implementasikan middleware untuk validasi token
php artisan make:middleware ValidateOIDCToken

// Dalam app/Http/Middleware/ValidateOIDCToken.php
public function handle($request, Closure $next)
{
    $token = $request->bearerToken();
    // Validasi token
    // Jika valid, tambahkan informasi pengguna ke request
    return $next($request);
}

// Daftarkan middleware di app/Http/Kernel.php
protected $routeMiddleware = [
    // ...
    'oidc.auth' => \App\Http\Middleware\ValidateOIDCToken::class,
];
```

contoh oidc di sisi client app

```php
<?php

// 1. Install paket yang diperlukan
// composer require league/oauth2-client

// 2. Buat controller untuk menangani autentikasi
// app/Http/Controllers/AuthController.php

use League\OAuth2\Client\Provider\GenericProvider;

class AuthController extends Controller
{
    private function getProvider()
    {
        return new GenericProvider([
            'clientId'                => config('services.oidc.client_id'),
            'clientSecret'            => config('services.oidc.client_secret'),
            'redirectUri'             => config('services.oidc.redirect_uri'),
            'urlAuthorize'            => config('services.oidc.url_authorize'),
            'urlAccessToken'          => config('services.oidc.url_access_token'),
            'urlResourceOwnerDetails' => config('services.oidc.url_resource_owner_details'),
        ]);
    }

    public function login()
    {
        $provider = $this->getProvider();
        $authorizationUrl = $provider->getAuthorizationUrl([
            'scope' => ['openid', 'profile', 'email']
        ]);
        session(['oauth2state' => $provider->getState()]);
        return redirect($authorizationUrl);
    }

    public function callback(Request $request)
    {
        $provider = $this->getProvider();

        if ($request->get('state') !== session('oauth2state')) {
            session()->forget('oauth2state');
            return redirect('/')->withErrors('Invalid state');
        }

        try {
            $token = $provider->getAccessToken('authorization_code', [
                'code' => $request->get('code')
            ]);

            $user = $provider->getResourceOwner($token);
            $userArray = $user->toArray();

            // Lakukan login atau registrasi user di aplikasi Anda
            $localUser = User::firstOrCreate(
                ['email' => $userArray['email']],
                ['name' => $userArray['name']]
            );

            Auth::login($localUser);

            return redirect('/dashboard');
        } catch (\Exception $e) {
            return redirect('/')->withErrors($e->getMessage());
        }
    }

    public function logout()
    {
        Auth::logout();
        session()->flush();
        return redirect('/');
    }
}

// 3. Tambahkan rute di routes/web.php
Route::get('/login', [AuthController::class, 'login'])->name('login');
Route::get('/callback', [AuthController::class, 'callback']);
Route::post('/logout', [AuthController::class, 'logout'])->name('logout');

// 4. Tambahkan konfigurasi di config/services.php
'oidc' => [
    'client_id' => env('OIDC_CLIENT_ID'),
    'client_secret' => env('OIDC_CLIENT_SECRET'),
    'redirect_uri' => env('OIDC_REDIRECT_URI'),
    'url_authorize' => env('OIDC_URL_AUTHORIZE'),
    'url_access_token' => env('OIDC_URL_ACCESS_TOKEN'),
    'url_resource_owner_details' => env('OIDC_URL_RESOURCE_OWNER_DETAILS'),
],

// 5. Tambahkan variabel lingkungan di .env
OIDC_CLIENT_ID=your_client_id
OIDC_CLIENT_SECRET=your_client_secret
OIDC_REDIRECT_URI=https://your-app.com/callback
OIDC_URL_AUTHORIZE=https://your-oidc-server.com/oauth/authorize
OIDC_URL_ACCESS_TOKEN=https://your-oidc-server.com/oauth/token
OIDC_URL_RESOURCE_OWNER_DETAILS=https://your-oidc-server.com/oauth/userinfo

// 6. Buat middleware untuk memeriksa autentikasi
// app/Http/Middleware/EnsureUserIsAuthenticated.php

public function handle($request, Closure $next)
{
    if (!Auth::check()) {
        return redirect()->route('login');
    }
    return $next($request);
}

// Daftarkan middleware di app/Http/Kernel.php
protected $routeMiddleware = [
    // ...
    'ensure.authenticated' => \App\Http\Middleware\EnsureUserIsAuthenticated::class,
];

// 7. Gunakan middleware pada rute yang memerlukan autentikasi
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware('ensure.authenticated');
```

contoh pendekatan oauth2 dengan authorization code grant with pkce

```php
<?php

// 1. Konfigurasi Laravel Passport (server OAuth)

// config/auth.php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],

// app/Providers/AuthServiceProvider.php
use Laravel\Passport\Passport;

public function boot()
{
    $this->registerPolicies();

    Passport::routes();
    
    // Aktifkan dukungan PKCE
    Passport::enableImplicitGrant();
}

// 2. Implementasi di sisi client (contoh menggunakan JavaScript)

// Fungsi untuk generate code verifier dan challenge
function generateCodeVerifier() {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-._~';
    let verifier = '';
    for (let i = 0; i < 128; i++) {
        verifier += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return verifier;
}

async function generateCodeChallenge(verifier) {
    const encoder = new TextEncoder();
    const data = encoder.encode(verifier);
    const digest = await window.crypto.subtle.digest('SHA-256', data);
    return btoa(String.fromCharCode(...new Uint8Array(digest)))
        .replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');
}

// Fungsi untuk memulai proses autentikasi
async function startAuth() {
    const codeVerifier = generateCodeVerifier();
    const codeChallenge = await generateCodeChallenge(codeVerifier);
    
    // Simpan code verifier untuk digunakan nanti
    localStorage.setItem('code_verifier', codeVerifier);
    
    const authUrl = `https://your-oauth-server.com/oauth/authorize?` +
        `client_id=your-client-id` +
        `&redirect_uri=${encodeURIComponent('https://your-app.com/callback')}` +
        `&response_type=code` +
        `&scope=` +
        `&code_challenge=${codeChallenge}` +
        `&code_challenge_method=S256`;
    
    window.location.href = authUrl;
}

// Fungsi untuk menangani callback dan menukar kode dengan token
async function handleCallback(code) {
    const codeVerifier = localStorage.getItem('code_verifier');
    
    const response = await fetch('https://your-oauth-server.com/oauth/token', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: new URLSearchParams({
            grant_type: 'authorization_code',
            client_id: 'your-client-id',
            redirect_uri: 'https://your-app.com/callback',
            code_verifier: codeVerifier,
            code: code,
        }),
    });
    
    const data = await response.json();
    
    // Simpan token
    localStorage.setItem('access_token', data.access_token);
    localStorage.setItem('refresh_token', data.refresh_token);
}

// 3. Route untuk menangani callback di sisi server (opsional, jika diperlukan)

// routes/web.php
Route::get('/callback', function (Request $request) {
    $code = $request->get('code');
    // Proses kode dan kirim ke frontend
    return view('auth.callback', ['code' => $code]);
});

// 4. Menggunakan token untuk mengakses API

async function fetchUserData() {
    const token = localStorage.getItem('access_token');
    const response = await fetch('https://your-oauth-server.com/api/user', {
        headers: {
            'Authorization': `Bearer ${token}`,
        },
    });
    return await response.json();
}
```
