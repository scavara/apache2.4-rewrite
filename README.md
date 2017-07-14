# apache2.4-rewrite
slightly advanced rewrite rules in combo with JWT used for authentication with (lib-)jitsi-meet custom deploy...!!!not properly tested!!!

inside apache virutalhost conf put (assuming that rewrite mod is enabled):
RewriteEngine On
# handle mobile users so they can use app, not browser
RewriteCond %{HTTP_USER_AGENT} "android|blackberry|googlebot-mobile|iemobile|ipad|iphone|ipod|opera mobile|palmos|webos" [NC]
RewriteMap redirect2mobile prg:/var/www/rooms/redirect2mobile
RewriteRule "^/rooms/(.+)" "https://external.mobile.site/${redirect2mobile:$1}"

cat /var/www/rooms/redirect2mobile:
#!/usr/bin/php
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
