Kita dapat menggunakan formula Haversine untuk menghitung jarak antara dua titik
berdasarkan koordinat lintang dan bujur.

Kita dapat menggunakannya untuk menghitung jarak antara titik yang dimasukkan
pengguna (latitude dan longitude) dan setiap kota dalam database. Kemudian,
lakukan sortir kota-kota berdasarkan jarak dan mengambil kota terdekat.

Berikut adalah contoh implementasi dalam Laravel :

Buat sebuah route di Laravel untuk menerima permintaan dengan parameter latitude
dan longitude

```php
<?php
Route::get('/nearest-city', 'CityController@findNearestCity');
```

Buatlah sebuah controller (CityController) dengan method findNearestCity:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\City; // Sesuaikan dengan model kota Anda

class CityController extends Controller
{
    public function findNearestCity(Request $request)
    {
        // Validasi parameter
        $request->validate([
            'latitude' => 'required|numeric',
            'longitude' => 'required|numeric',
        ]);

        $latitude = $request->latitude;
        $longitude = $request->longitude;

        // Ambil semua kota dari database
        $cities = City::all();

        // Hitung jarak dari setiap kota ke koordinat yang diberikan
        foreach ($cities as $city) {
            $distance = $this->haversine($latitude, $longitude, $city->latitude, $city->longitude);
            $city->distance = $distance; // Menambahkan properti jarak ke setiap kota
        }

        // Mengurutkan kota berdasarkan jarak terdekat
        $nearestCity = $cities->sortBy('distance')->first();

        return response()->json($nearestCity);
    }

    private function haversine($lat1, $lon1, $lat2, $lon2)
    {
        $R = 6371; // Radius bumi dalam kilometer

        $dLat = deg2rad($lat2 - $lat1);
        $dLon = deg2rad($lon2 - $lon1);

        $a = sin($dLat/2) * sin($dLat/2) + cos(deg2rad($lat1)) * cos(deg2rad($lat2)) * sin($dLon/2) * sin($dLon/2);
        $c = 2 * atan2(sqrt($a), sqrt(1-$a));
        $distance = $R * $c;

        return $distance;
    }
}
```

Ketika user memberikan koordinat tertentu, formula Haversine dijalankan untuk
menghitung jarak dari koordinat pengguna ke koordinat setiap kota dalam
database. Kemudian, kita menyimpan jarak tersebut dalam properti kota. Dari
situ, kita dapat menyortir kota-kota berdasarkan jaraknya dari koordinat
pengguna dan memilih kota dengan jarak terdekat.

Jadi, dengan menggunakan formula Haversine, kita bukan secara langsung
"menentukan" apakah suatu titik masuk atau dekat dengan suatu kota. Sebaliknya,
kita menghitung jaraknya dan memutuskan kota mana yang paling dekat berdasarkan
jarak tersebut.

cara lain kita dapat menggunakan metode Vincenty

Kelebihan dan Kekurangan 2 metode tsb

Haversine Formula:

Kelebihan:

- Mudah diimplementasikan.
- Kinerja yang baik untuk jarak yang relatif dekat.

Kekurangan:

- Tidak memberikan hasil yang akurat untuk jarak yang panjang karena menggunakan
  model bola sederhana.
- Tidak memperhitungkan bentuk bumi yang sebenarnya.

Vincenty Formula:

Kelebihan:

- Memberikan hasil yang lebih akurat daripada Haversine untuk jarak yang
  panjang.
- Memperhitungkan bentuk elipsoid bumi yang lebih realistis.

Kekurangan:

- Lebih kompleks dalam implementasinya.
- Lebih lambat dalam kinerja dibandingkan Haversine.

buat file `CityController`

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\City;
use App\Services\Vincenty; // Sesuaikan dengan namespace Anda

class CityController extends Controller
{
    public function findNearestCity(Request $request)
    {
        // Validasi parameter
        $request->validate([
            'latitude' => 'required|numeric',
            'longitude' => 'required|numeric',
        ]);

        $latitude = $request->latitude;
        $longitude = $request->longitude;

        // Ambil semua kota dari database
        $cities = City::all();

        // Inisialisasi variabel untuk menyimpan kota terdekat
        $nearestCity = null;
        $minDistance = PHP_INT_MAX;

        // Hitung jarak dari setiap kota ke koordinat yang diberikan
        foreach ($cities as $city) {
            $distance = Vincenty::calculateDistance($latitude, $longitude, $city->latitude, $city->longitude);
            if ($distance < $minDistance) {
                $minDistance = $distance;
                $nearestCity = $city;
            }
        }

        return response()->json($nearestCity);
    }
}
```

buat class service `Vincenty`

```php
<?php

namespace App\Services;

class Vincenty
{
    public static function calculateDistance($lat1, $lon1, $lat2, $lon2)
    {
        $a = 6378137; // ellipsoidal semi-major axis (radius at equator), in meters
        $f = 1 / 298.257223563; // ellipsoidal flattening
        $b = (1 - $f) * $a; // ellipsoidal semi-minor axis (radius at the poles), in meters
        $U1 = atan((1 - $f) * tan(deg2rad($lat1)));
        $U2 = atan((1 - $f) * tan(deg2rad($lat2)));
        $L = deg2rad($lon2 - $lon1);
        $lambda = $L;

        $iterLimit = 100;
        $lambdaP = 2 * M_PI;
        while (abs($lambda - $lambdaP) > 1e-12 && --$iterLimit > 0) {
            $sinSigma = sqrt(pow(cos($U2) * sin($lambda), 2) + pow(cos($U1) * sin($U2) - sin($U1) * cos($U2) * cos($lambda), 2));
            $cosSigma = sin($U1) * sin($U2) + cos($U1) * cos($U2) * cos($lambda);
            $sigma = atan2($sinSigma, $cosSigma);
            $sinAlpha = cos($U1) * cos($U2) * sin($lambda) / sin($sigma);
            $cosSqAlpha = 1 - pow($sinAlpha, 2);

            $cos2SigmaM = $cosSigma - 2 * sin($U1) * sin($U2) / $cosSqAlpha;
            $C = $f / 16 * $cosSqAlpha * (4 + $f * (4 - 3 * $cosSqAlpha));
            $lambdaP = $lambda;
            $lambda = $L + (1 - $C) * $f * $sinAlpha *
                ($sigma + $C * $sinSigma *
                    ($cos2SigmaM + $C * $cosSigma *
                        (-1 + 2 * pow($cos2SigmaM, 2))));
        }

        if ($iterLimit == 0) {
            throw new \Exception("Vincenty formula failed to converge");
        }

        $uSq = $cosSqAlpha * ($a * $a - $b * $b) / ($b * $b);
        $A = 1 + $uSq / 16384 * (4096 + $uSq * (-768 + $uSq * (320 - 175 * $uSq)));
        $B = $uSq / 1024 * (256 + $uSq * (-128 + $uSq * (74 - 47 * $uSq)));
        $deltaSigma = $B * $sinSigma *
            ($cos2SigmaM + $B / 4 * ($cosSigma * (-1 + 2 * pow($cos2SigmaM, 2)) -
                $B / 6 * $cos2SigmaM * (-3 + 4 * pow($sinSigma, 2)) * (-3 + 4 * pow($cos2SigmaM, 2))));
        $s = $b * $A * ($sigma - $deltaSigma);

        return $s;
    }
}
```
