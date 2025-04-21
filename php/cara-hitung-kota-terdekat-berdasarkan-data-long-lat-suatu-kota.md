# Calculating Distance Between Coordinates Using Haversine and Vincenty Formulas

## Overview

The Haversine formula and Vincenty formula are two common methods for
calculating the distance between two points on the Earth based on their latitude
and longitude coordinates. These methods can be used to determine the distance
between a user-specified location and cities stored in a database. This article
explains how to implement these calculations in a Laravel application and
highlights the differences between the two methods.

## Using the Haversine Formula in Laravel

### Step 1: Create a Route

Define a route to handle requests with latitude and longitude parameters.

```php
<?php
Route::get('/nearest-city', 'CityController@findNearestCity');
```

### Step 2: Create a Controller Method

Create a `CityController` with a `findNearestCity` method to calculate
distances.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\City; // Adjust to match your city model

class CityController extends Controller
{
    public function findNearestCity(Request $request)
    {
        // Validate parameters
        $request->validate([
            'latitude' => 'required|numeric',
            'longitude' => 'required|numeric',
        ]);

        $latitude = $request->latitude;
        $longitude = $request->longitude;

        // Retrieve all cities from the database
        $cities = City::all();

        // Calculate distances to each city
        foreach ($cities as $city) {
            $distance = $this->haversine($latitude, $longitude, $city->latitude, $city->longitude);
            $city->distance = $distance;
        }

        // Sort cities by distance and return the nearest city
        $nearestCity = $cities->sortBy('distance')->first();

        return response()->json($nearestCity);
    }

    private function haversine($lat1, $lon1, $lat2, $lon2)
    {
        $R = 6371; // Radius of Earth in kilometers

        $dLat = deg2rad($lat2 - $lat1);
        $dLon = deg2rad($lon2 - $lon1);

        $a = sin($dLat / 2) * sin($dLat / 2) +
             cos(deg2rad($lat1)) * cos(deg2rad($lat2)) *
             sin($dLon / 2) * sin($dLon / 2);
        $c = 2 * atan2(sqrt($a), sqrt(1 - $a));
        $distance = $R * $c;

        return $distance;
    }
}
```

### Explanation

When the user provides specific coordinates, the Haversine formula calculates
the distance from the user's coordinates to each city in the database. The
cities are then sorted by distance, and the nearest city is returned.

## Using the Vincenty Formula for More Accurate Calculations

The Vincenty formula is more accurate for long distances as it considers the
Earth's ellipsoidal shape.

### Step 1: Update the Controller

Modify the `CityController` to use the Vincenty formula.

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\City;
use App\Services\Vincenty; // Adjust the namespace as needed

class CityController extends Controller
{
    public function findNearestCity(Request $request)
    {
        // Validate parameters
        $request->validate([
            'latitude' => 'required|numeric',
            'longitude' => 'required|numeric',
        ]);

        $latitude = $request->latitude;
        $longitude = $request->longitude;

        // Retrieve all cities from the database
        $cities = City::all();

        // Initialize variables for the nearest city
        $nearestCity = null;
        $minDistance = PHP_INT_MAX;

        // Calculate distances using the Vincenty formula
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

### Step 2: Create the Vincenty Service

Create a service class for the Vincenty formula calculations.

```php
<?php

namespace App\Services;

class Vincenty
{
    public static function calculateDistance($lat1, $lon1, $lat2, $lon2)
    {
        $a = 6378137; // Semi-major axis in meters
        $f = 1 / 298.257223563; // Flattening
        $b = (1 - $f) * $a; // Semi-minor axis

        $U1 = atan((1 - $f) * tan(deg2rad($lat1)));
        $U2 = atan((1 - $f) * tan(deg2rad($lat2)));
        $L = deg2rad($lon2 - $lon1);
        $lambda = $L;

        $iterLimit = 100;
        $lambdaP = 2 * M_PI;
        while (abs($lambda - $lambdaP) > 1e-12 && --$iterLimit > 0) {
            $sinSigma = sqrt(pow(cos($U2) * sin($lambda), 2) +
                             pow(cos($U1) * sin($U2) - sin($U1) * cos($U2) * cos($lambda), 2));
            $cosSigma = sin($U1) * sin($U2) + cos($U1) * cos($U2) * cos($lambda);
            $sigma = atan2($sinSigma, $cosSigma);
            $sinAlpha = cos($U1) * cos($U2) * sin($lambda) / $sinSigma;
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
                      ($cos2SigmaM + $B / 4 *
                       ($cosSigma * (-1 + 2 * pow($cos2SigmaM, 2)) -
                        $B / 6 * $cos2SigmaM * (-3 + 4 * pow($sinSigma, 2)) * (-3 + 4 * pow($cos2SigmaM, 2))));
        $s = $b * $A * ($sigma - $deltaSigma);

        return $s;
    }
}
```

## Comparison of Haversine and Vincenty Formulas

| Feature     | Haversine                | Vincenty                               |
| ----------- | ------------------------ | -------------------------------------- |
| Accuracy    | Good for short distances | High accuracy, even for long distances |
| Complexity  | Simple                   | Complex                                |
| Performance | Fast                     | Slower                                 |
| Earth Model | Spherical                | Ellipsoidal                            |

Choose the method based on the accuracy and performance requirements of your
application.

## Using PostgreSQL for Nearest Location Queries

PostgreSQL, with the PostGIS extension, provides powerful tools for working with geospatial data. Depending on your use case, you can use either the `geometry` or `geography` data types:

- **Geometry**: Faster for calculations within a flat coordinate system (e.g., meters), but less accurate for Earth's curvature.
- **Geography**: More accurate for real-world distances (e.g., kilometers/meters) as it accounts for Earth's spheroidal shape.

### Setting Up the Database

1. **Create a GIST Index**  
    To optimize queries, create a GIST index on the location column:

    ```sql
    CREATE INDEX idx_yourtable_location ON yourtable USING GIST(location);
    ```

2. **Define the Table Structure**  
    Example table structure for storing places:

    ```sql
    CREATE TABLE places (
      id SERIAL PRIMARY KEY,
      name TEXT,
      location GEOGRAPHY(Point, 4326) -- latitude, longitude
    );
    ```

    > **Note**: `4326` is the EPSG code for Earth's coordinate system (WGS 84, used by Google Maps).

3. **Insert Data**  
    Insert a place with latitude and longitude:

    ```sql
    INSERT INTO places (name, location)
    VALUES ('Place A', ST_SetSRID(ST_MakePoint(106.8272, -6.1751), 4326)::geography);
    ```

    > **Important**: The order is `longitude, latitude` (not `latitude, longitude`).

### Querying for Nearest Locations

1. **Find the Nearest Locations**  
    To find the 5 nearest places to a user's location:

    ```sql
    SELECT id, name,
      ST_Distance(location, ST_SetSRID(ST_MakePoint(106.8, -6.2), 4326)::geography) AS distance
    FROM places
    ORDER BY distance ASC
    LIMIT 5;
    ```

2. **Filter by Radius**  
    To find places within a specific radius (e.g., 10 km):

    ```sql
    SELECT id, name
    FROM places
    WHERE ST_DWithin(
      location,
      ST_SetSRID(ST_MakePoint(106.8, -6.2), 4326)::geography,
      10000 -- radius in meters
    )
    ORDER BY ST_Distance(
      location,
      ST_SetSRID(ST_MakePoint(106.8, -6.2), 4326)::geography
    ) ASC;
    ```

### Summary

- Use **PostGIS** for geospatial queries.
- Use the `geography(Point, 4326)` type for real-world distances.
- Create a **GIST index** for faster queries.
- Use `ST_Distance` for calculating distances and `ST_DWithin` for filtering by radius.
