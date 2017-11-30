# apache2.4-rewrite
slightly advanced rewrite rules in combo with JWT used for authentication with (lib-)jitsi-meet custom deploy...!!!not properly tested!!!

inside apache virutalhost conf put (assuming that rewrite mod is enabled):
```RewriteEngine On
# handle mobile users so they can use app, not browser
RewriteCond %{HTTP_USER_AGENT} "android|blackberry|googlebot-mobile|iemobile|ipad|iphone|ipod|opera mobile|palmos|webos" [NC]
RewriteMap redirect2mobile prg:/var/www/rooms/redirect2mobile
RewriteRule "^/rooms/(.+)" "https://external.mobile.site/${redirect2mobile:$1}"
#otherwise...
#match in url a token (alphanum + few .-_) 
RewriteRule ^/rooms/([\w._-]+)/?$ /rooms/room.php?token=$1 [NC,L]
```
cat /var/www/rooms/redirect2mobile:
```#!/usr/bin/php
<?php
/** wont't do any token verification since recommendation for using rewritemap
 says "Keep your rewrite map program as simple as possible".
 Btw, don't forget (if changing the script below) to include "\n" since
 same it also says: "should return one new-line terminated response string on STDOUT."
 scavara-10072017 **/

require '/path/to/jwt/vendor/autoload.php';
use Lcobucci\JWT\Parser;
use Lcobucci\JWT\Signer\Hmac\Sha256;
$signer = new Sha256();
set_time_limit(0);
$input = fopen("php://stdin","r");
while (1) {
        $line = trim(fgets($input));
        $url_token = str_replace("/rooms/","", $line);
        $decoded = (new Parser())->parse((string) $url_token);
        $room = $decoded->getClaim('room');
        print $room . "?jwt=" . $decoded . "\n";
}
?>
```
Another example where short url (just REQUEST_URI, not HOST_NAME) is generated in db:
```RewriteEngine On
 DBDriver mysql  
 DBDParams "host=localhost,user=homestead,pass=0za8ATeUNvmZfgDx,dbname=homestead"
 RewriteCond %{REQUEST_URI} ^/(.+)$
 RewriteMap long_url "dbd:SELECT CONCAT('https://jitsi-prod01.carnet.hr/', name, '?jwt=', jwt) FROM reservations where short_url = %s"
 RewriteCond ${long_url:%1} ^(.+)$
 RewriteRule (.*) %1 
```
where 5char long short_url is generated as php function using digits from jwt:
```/**
* Create new short URL.
*
* @return string
*/
/*code copied from:
 * ShortURL (https://github.com/delight-im/ShortURL)
 * Copyright (c) delight.im (https://www.delight.im/)
 * Licensed under the MIT License (https://opensource.org/licenses/MIT)
 *
 * Slightly ajusted by scavara-29112017
 */
/**
 * ShortURL: Bijective conversion between natural numbers (IDs) and short strings
 *
 * ShortURL::encode() takes an ID and turns it into a short string
 * ShortURL::decode() takes a short string and turns it into an ID
 *
 * Features:
 * + large alphabet (51 chars) and thus very short resulting strings
 * + proof against offensive words (removed 'a', 'e', 'i', 'o' and 'u')
 * + unambiguous (removed 'I', 'l', '1', 'O' and '0')
 *
 * Example output:
 * 123456789 <=> pgK8p
 */

    private function getShortUrl($jwt)

    {
        define('ALPHABET', '23456789bcdfghjkmnpqrstvwxyzBCDFGHJKLMNPQRSTVWXYZ-_');
        define('BASE', 51); // strlen(self::ALPHABET)
        $num = substr(str_shuffle(preg_replace('/[^0-9]/', '', $jwt)),0,9);
        $str = '';
                while ($num > 0) {
                        $str = ALPHABET[($num % BASE)] . $str;
                        $num = (int) ($num / BASE);
                        if (strlen($str) == 5) {
                                break; 
                        }
                }
        return $str;
    }
}
```
