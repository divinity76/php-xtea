# php-xtea
XTEA implementation in PHP

# usage
note that the entire library is binary-safe, data-to-encrypt and encryption keys and data-to-decrypt and the resulting decrypted data may be binary data.

the signature for XTEA::encrypt is
```php
public static function encrypt(string $data, array $keys,
    int $padding_scheme = self::PAD_0x00, int $rounds = 32) : string
```
- string $data is the data to be encrypted
- array $keys is an array containing the 4 xtea keys as integers (if you have a binary key, you can convert it to an int array with the XTEA::binary_key_to_int_array() function)
- int $padding_scheme is the padding scheme to use, there are 3 padding schemes: the default `XTEA::PAD_0x00` will right-pad with null-bytes, `XTEA::PAD_RANDOM` will right-pad with cryptographically-secure random bytes, and `XTEA::PAD_NONE` will not do any padding but throw an InvalidArgumentException if padding is required. 
- int $rounds is how many rounds to encrypt, the default is 32, and 32 is recommended by XTEA's designers `Roger Needham` and `David Wheeler`, 0 rounds will effectively result in no encryption whatsoever with only padding being applied.

the return string is the encrypted binary data.

the signature for XTEA::decrypt is
```php
public static function decrypt(string $data, array $keys, int $rounds = 32) : string
```
- string $data is the encrypted binary data
- array $keys is an array containing the 4 xtea keys as integers (if you have a binary key, you can convert it to an int array with the XTEA::binary_key_to_int_array() function)
- int $rounds is how many rounds was used during encryption. the default is 32, and 32 is recommended by XTEA's designers `Roger Needham` and `David Wheeler`, and 0 rounds effectively means that no encryption was applied at all.

the return string is the decrypted (binary?) data.

the signature for XTEA::binary_key_to_int_array is
```php
public static function binary_key_to_int_array(string $key, int $padding_scheme = self::PAD_0x00) : array
```
- string $key is the (binary?) encryption key, it can be anywhere between 0-16 bytes (inclusive), except if the padding scheme is XTEA::PAD_NONE, in which case it needs to be _exactly_ 16 bytes long. if the key is longer than 16 bytes, an InvalidArgumentException is thrown.
- int $padding_scheme is the padding scheme to use when the key is less than 16 bytes long, 2 padding schemes are supported: `XTEA::PAD_0x00` will right-pad it with null-bytes, and `XTEA::PAD_NONE` will not do any padding, but throw an InvalidArgumentException if the key length is not exactly 16 bytes long. (i intentionally did not add support for XTEA::PAD_RANDOM here because i believe that supporting *ANY* non-deterministic padding scheme for the key is a bad idea.)

the return is an array containing the 4 resulting XTEA keys as php integers, which can be used with XTEA::encrypt and XTEA::decrypt

# examples

```php
<?php 
require_once('XTEA.class.php');
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
require_once('XTEA.class.php');
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

running this benchmark script:

```php
<?php 
declare (strict_types = 1);
require_once('XTEA.class.php');
$keys_binary = random_bytes(4 * 4);
$keys_array = XTEA::binary_key_to_int_array($keys_binary);
$data = random_bytes(1 * 1024); // 1KB
$time = microtime(true);
for ($i = 0; $i < 1024; ++$i) {
    XTEA::encrypt($data, $keys_array, XTEA::PAD_NONE, 32);
}
$time = microtime(true) - $time;
echo "encrypted 1 kilobyte 1024 times in " . number_format($time, 9) . " seconds.\n";
```
i got `encrypted 1 kilobyte 1024 times in 23.549000025 seconds.`
which means circa 43.5 kilobytes/sec on my laptop rolling Intel Core i7-6700 and 2133MHz DDR4-ram and PHP 7.1.16-cli
