# php-xtea
XTEA implementation in PHP

# examples

```php
<?php 
require_once('xtea.class.php');
$key_binary = random_bytes(4 * 4);
$keys_array = XTEA::binary_key_to_int_array($key_binary);
$data = "Hello, World!";
$encrypted = XTEA::encrypt($data, $keys_array);
$decrypted = XTEA::decrypt($encrypted, $keys_array);
var_dump($data, $encrypted, $decrypted);
```
should output something like:
```
string(13) "Hello, World!"
string(16) "□□Jx□□□□□□□ܴ9"
string(16) "Hello, World!   "
```
the length is different because of xtea length padding, which is possible to disable by instead doing

```php
XTEA::encrypt($data, $keys_array, XTEA::PAD_NONE);
```
which will give you:

```
PHP Fatal error:  Uncaught InvalidArgumentException: with PAD_NONE the data MUST be a multiple of 8 bytes! in /cygdrive/c/projects/tibia_login_php/xtea.class.php:73
Stack trace:
#0 /cygdrive/c/projects/tibia_login_php/xtea_tests.php(8): XTEA::encrypt('Hello, World!', Array, 0)
#1 {main}
  thrown in /cygdrive/c/projects/tibia_login_php/xtea.class.php on line 73
```
because the XTEA algorithm requires data-to-encrypt to be in multiples of 8 bytes. but if we change it to
```php
<?php 
require_once('xtea.class.php');
$keys_binary = random_bytes(4 * 4);
$keys_array = XTEA::binary_key_to_int_array($keys_binary);
$data = "Hello, World!123";
$encrypted = XTEA::encrypt($data, $keys_array, XTEA::PAD_NONE);
$decrypted = XTEA::decrypt($encrypted, $keys_array);
var_dump($data, $encrypted, $decrypted);

```
you will get 
```
string(16) "Hello, World!123"
string(16) "%t□□□n□□aʓ'□□H"
string(16) "Hello, World!123"
```
- no padding, no length change, no exception.

# Warning

the implementation will *PROBABLY* not work on 32bit systems, and it has only been tested on 64bit systems. (something about 32bit integer overflow)

also, i barely managed to make a *WORKING* implementation, but it wasn't written to be fast, a much faster implementation is probably possible to make.
