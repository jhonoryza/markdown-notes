translate carbon

```php
$c = Carbon::createFromLocaleFormat('l d/m/Y', 'id', 'Jumat 12/01/2024')->startOfDay();
$c->settings(['formatFunction' => 'translatedFormat'])
    ->locale('id')
    ->format('l d F Y');
```
