- https://gist.github.com/syahzul/2632262df21974ecd02be3ccce66fbef#install-extension-with-pecl
- https://patriqueouimet.ca/tip/installing-php-and-pecl-extensions-on-macos
- https://github.com/yajra/laravel-oci8

`pecl install oci8`

snippet code to check oci8 installed

```php
<?php

if (function_exists('oci_connect')) {
    echo 'OCI8 is working!';
}
else {
    echo 'Whoopss...not working!';
}
```
