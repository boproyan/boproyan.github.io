+++
title = 'JsonWebToken(jwt)'
date = 2019-09-26 00:00:00 +0100
categories = json
+++
# JWT on PHP

* https://medium.com/@crmcmullen/simple-example-using-json-web-tokens-with-php-and-jquery-c648a80854c  
* https://github.com/crmcmullen/jwtphpjquery/blob/master/index.html   
* https://medium.com/tag/json-web-token  
* [Authentification d’API via JWT et les Cookies](https://website.simplx.fr/blog/2016/09/27/authentification-api-via-jwt-et-cookies/)  
* [En toute sécurité avec JWT (1re partie)](https://www.ekino.com/articles/securite-avec-jwt-premiere-partie)  
* [En toute sécurité avec JWT (2de partie)](https://www.ekino.fr/articles/securite-avec-jwt-seconde-partie)  
* [Création et gestion des cookies en PHP](https://www.pierre-giraud.com/php-mysql-apprendre-coder-cours/cookie-creation-gestion/)
* [Modern cryptography in PHP 7.2 with Sodium](https://blog.zend.com/2018/11/06/modern-cryptography-in-php-7-2-with-sodium/)
* [Protégez-vous efficacement contre les failles web](https://openclassrooms.com/fr/courses/2091901-protegez-vous-efficacement-contre-les-failles-web/2863569-la-csrf)


    index.html

```
<!DOCTYPE html>
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=utf-8" />
  <title>JWT Example</title>
</head>

<body>
  <div>
    <button id="test">Test if Logged In</button>
    <button id="goodLogin">Good Login</button>
    <button id="badLogin">Bad Login</button>
    <button id="logout">Logout and Clear Token</button>
  </div>
  <script type="text/javascript" src="https://code.jquery.com/jquery-2.1.1.min.js"></script>
  <script type="text/javascript">
  $(document).ready(function() {
    $('#test').click(function() {
      $.ajax({
        type: 'GET',
        url: 'app_client.php',
        dataType: "json",
        data: {
          token: localStorage.token
        },
        success: function(data) {
          if (typeof data['userId'] !== 'undefined') {
            var alertMessage = 'You have a valid token! Here is your user Id: ' + data['userId'];

            if (typeof data['exp'] !== 'undefined') {
              alertMessage = alertMessage + ' and your token expires: ' + data['exp'];
            }

            alert(alertMessage);

          } 
          else if (typeof data['error'] !== 'undefined') {
            alert('Error: ' + data['error']);
          }
          else {
            alert('Error: Your request has failed.');
          }
        }
      });
    });
    $('#goodLogin').click(function() {
      $.ajax({
        type: "POST",
        url: "app_client.php",
        dataType: "json",
        data: {
          username: "john.doe",
          password: "foobar"
        },
        success: function(data) {
          localStorage.token = data['token'];
          alert('Successfully retrieved token from the server! Token: ' + data['token']);
        },
        error: function() {
          alert("Error: Login Failed");
        }
      });
    });
    $('#badLogin').click(function() {
      $.ajax({
        type: "POST",
        url: "app_client.php",
        dataType: "json",
        data: {
          username: "john.doe",
          password: "foobarfoobar"
        },
        success: function(data) {
          if (typeof data['error'] !== 'undefined') {
            alert('Error: ' + data['error']);
            localStorage.clear();
          }
        },
        error: function() {
          alert('Error: Your request has failed.');
        }
      });
    });
    $('#logout').click(function() {
      localStorage.clear();
    });
  });
  </script>
</body>

</html>
```

    app_client.php

```php
<?php
/**
 * This file processes the login request and sends back a token response
 * if successful.
 */
$requestMethod = $_SERVER['REQUEST_METHOD'];

// retrieve the inbound parameters based on request type.
switch($requestMethod) {

    case 'POST':
        $username = '';
        $password = '';
    
        if (isset($_POST['username'])) {$username = $_POST['username'];}
        if (isset($_POST['password'])) {$password = $_POST['password'];}

        if (($username == 'john.doe') && ($password == 'foobar')) {

            require_once('jwt.php');

            /** 
             * Create some payload data with user data we would normally retrieve from a
             * database with users credentials. Then when the client sends back the token,
             * this payload data is available for us to use to retrieve other data 
             * if necessary.
             */
            $userId = 'USER123456';

            /**
             * Uncomment the following line and add an appropriate date to enable the 
             * "not before" feature.
             */
            // $nbf = strtotime('2021-01-01 00:00:01');

            /**
             * Uncomment the following line and add an appropriate date and time to enable the 
             * "expire" feature.
             */
            // $exp = strtotime('2021-01-01 00:00:01');

            // Get our server-side secret key from a secure location.
            $serverKey = '5f2b5cdbe5194f10b3241568fe4e2b24';

            // create a token
            $payloadArray = array();
            $payloadArray['userId'] = $userId;
            if (isset($nbf)) {$payloadArray['nbf'] = $nbf;}
            if (isset($exp)) {$payloadArray['exp'] = $exp;}
            $token = JWT::encode($payloadArray, $serverKey);

            // return to caller
            $returnArray = array('token' => $token);
            $jsonEncodedReturnArray = json_encode($returnArray, JSON_PRETTY_PRINT);
            echo $jsonEncodedReturnArray;

        } 
        else {
            $returnArray = array('error' => 'Invalid user ID or password.');
            $jsonEncodedReturnArray = json_encode($returnArray, JSON_PRETTY_PRINT);
            echo $jsonEncodedReturnArray;
        }

        break;

    case 'GET':

        $token = null;
        
        if (isset($_GET['token'])) {$token = $_GET['token'];}

        if (!is_null($token)) {

            require_once('jwt.php');

            // Get our server-side secret key from a secure location.
            $serverKey = '5f2b5cdbe5194f10b3241568fe4e2b24';

            try {
                $payload = JWT::decode($token, $serverKey, array('HS256'));
                $returnArray = array('userId' => $payload->userId);
                if (isset($payload->exp)) {
                    $returnArray['exp'] = date(DateTime::ISO8601, $payload->exp);;
                }
            }
            catch(Exception $e) {
                $returnArray = array('error' => $e->getMessage());
            }
        } 
        else {
            $returnArray = array('error' => 'You are not logged in with a valid token.');
        }
        
        // return to caller
        $jsonEncodedReturnArray = json_encode($returnArray, JSON_PRETTY_PRINT);
        echo $jsonEncodedReturnArray;

        break;

    default:
        $returnArray = array('error' => 'You have requested an invalid method.');
        $jsonEncodedReturnArray = json_encode($returnArray, JSON_PRETTY_PRINT);
        echo $jsonEncodedReturnArray;
}
```

    jwt-php

```php
<?php

/**
 * JSON Web Token implementation, based on this spec:
 * https://tools.ietf.org/html/rfc7519
 * 
 * This class library is based on original Firebase/JWT source code written by 
 * Neuman Vong and Anant Narayanan found here: https://github.com/firebase/php-jwt
 * 
 * @license  http://opensource.org/licenses/BSD-3-Clause 3-clause BSD
 * 
 */
class JWT {

    public static $leeway = 0;                      // allows for nbf, iat or exp clock skew
    public static $timestamp = null;                // allow timestamp to be specified for testing. Defaults to php (time) if null.
    public static $supported_algs = array(
        'HS256' => array('hash_hmac', 'SHA256'),
        'HS512' => array('hash_hmac', 'SHA512'),
        'HS384' => array('hash_hmac', 'SHA384'),
        'RS256' => array('openssl', 'SHA256'),
        'RS384' => array('openssl', 'SHA384'),
        'RS512' => array('openssl', 'SHA512'),
    );
    /** ----------------------------------------------------------------------------------------------------------
     * Decodes a JWT string into a PHP object.
     *
     * @param string        $token          The JSON web token
     * @param string|array  $key            The secret key                            
     * @param array         $allowed_algs   If the algorithm used is asymmetric, this is the public key list
     *                                      of supported verification algorithms. Supported algorithms are:
     *                                      'HS256', 'HS384', 'HS512' and 'RS256'
     *
     * @return object The JWT's payload as a PHP object
     *
     */
    public static function decode($token, $key, array $allowed_algs = array())
    {
        if ((!isset($timestamp)) || (is_null($timestamp))) {
            $timestamp = time();
        }
        
        if (empty($key)) {
            throw new Exception('Invalid or missing key.');
        }

        $tokenSegments = explode('.', $token);

        if (count($tokenSegments) != 3) {
            throw new Exception('Wrong number of segments');
        }

        list($headb64, $bodyb64, $cryptob64) = $tokenSegments;
        if (null === ($header = static::jsonDecode(static::urlsafeB64Decode($headb64)))) {
            throw new Exception('Invalid header encoding');
        }

        if (null === $payload = static::jsonDecode(static::urlsafeB64Decode($bodyb64))) {
            throw new Exception('Invalid claims encoding');
        }

        if (false === ($sig = static::urlsafeB64Decode($cryptob64))) {
            throw new Exception('Invalid signature encoding');
        }

        if (empty($header->alg)) {
            throw new Exception('Empty algorithm');
        }

        if (empty(static::$supported_algs[$header->alg])) {
            throw new Exception('Algorithm not supported');
        }

        if (!in_array($header->alg, $allowed_algs)) {
            throw new Exception('Algorithm not allowed');
        }

        if (is_array($key) || $key instanceof ArrayAccess) {
            if (isset($header->kid)) {
                if (!isset($key[$header->kid])) {
                    throw new UnexpectedValueException('"kid" invalid, unable to lookup correct key');
                }
                $key = $key[$header->kid];
            } else {
                throw new UnexpectedValueException('"kid" empty, unable to lookup correct key');
            }
        }

        // Check the signature
        if (!static::verify("$headb64.$bodyb64", $sig, $key, $header->alg)) {
            throw new Exception('Signature verification failed');
        }

        // Check if the nbf if it is defined. This is the time that the
        // token can actually be used. If it's not yet that time, abort.
        if (isset($payload->nbf) && $payload->nbf > ($timestamp + static::$leeway)) {
            throw new Exception(
                'Cannot handle token prior to ' . date(DateTime::ISO8601, $payload->nbf)
            );
        }

        // Check that this token has been created before 'now'. This prevents
        // using tokens that have been created for later use (and haven't
        // correctly used the nbf claim).
        if (isset($payload->iat) && $payload->iat > ($timestamp + static::$leeway)) {
            throw new Exception(
                'Cannot handle token prior to ' . date(DateTime::ISO8601, $payload->iat)
            );
        }
        
        // Check if this token has expired.
        if (isset($payload->exp) && ($timestamp - static::$leeway) >= $payload->exp) {
            throw new Exception('Expired token');
        }

        return $payload;
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Converts and signs a PHP object or array into a JWT string.
     *
     * @param object|array  $payload    PHP object or array
     * @param string        $key        The secret key.
     *                                  If the algorithm used is asymmetric, this is the private key
     * @param string        $alg        The signing algorithm.
     *                                  Supported algorithms are 'HS256', 'HS384', 'HS512' and 'RS256'
     * @param mixed         $keyId
     * @param array         $head       An array with header elements to attach
     *
     * @return string A signed JWT
     *
     */
    public static function encode($payload, $key, $alg = 'HS256', $keyId = null, $head = null)
    {
        $header = array('typ' => 'JWT', 'alg' => $alg);

        if ($keyId !== null) {
            $header['kid'] = $keyId;
        }

        if ( isset($head) && is_array($head) ) {
            $header = array_merge($head, $header);
        }

        $segments = array();
        $segments[] = static::urlsafeB64Encode(static::jsonEncode($header));
        $segments[] = static::urlsafeB64Encode(static::jsonEncode($payload));
        $signing_input = implode('.', $segments);
        $signature = static::sign($signing_input, $key, $alg);
        $segments[] = static::urlsafeB64Encode($signature);

        return implode('.', $segments);
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Sign a string with a given key and algorithm.
     *
     * @param string            $msg    The message to sign
     * @param string|resource   $key    The secret key
     * @param string            $alg    The signing algorithm.
     *                                  Supported algorithms are 'HS256', 'HS384', 'HS512' and 'RS256'
     *
     * @return string An encrypted message
     *
     */
    public static function sign($msg, $key, $alg = 'HS256')
    {
        if (empty(static::$supported_algs[$alg])) {
            throw new Exception('Algorithm not supported');
        }
        list($function, $algorithm) = static::$supported_algs[$alg];
        switch($function) {
            case 'hash_hmac':
                return hash_hmac($algorithm, $msg, $key, true);
            case 'openssl':
                $signature = '';
                $success = openssl_sign($msg, $signature, $key, $algorithm);
                if (!$success) {
                    throw new Exception("OpenSSL unable to sign data");
                } else {
                    return $signature;
                }
        }
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Verify a signature with the message, key and method. Not all methods
     * are symmetric, so we must have a separate verify and sign method.
     *
     * @param string            $msg        The original message (header and body)
     * @param string            $signature  The original signature
     * @param string|resource   $key        For HS*, a string key works. for RS*, must be a resource of an openssl public key
     * @param string            $alg        The algorithm
     *
     * @return bool
     */
    private static function verify($msg, $signature, $key, $alg)
    {
        if (empty(static::$supported_algs[$alg])) {
            throw new Exception('Algorithm not supported');
        }
        list($function, $algorithm) = static::$supported_algs[$alg];
        switch($function) {
            case 'openssl':
                $success = openssl_verify($msg, $signature, $key, $algorithm);
                if ($success === 1) {
                    return true;
                } elseif ($success === 0) {
                    return false;
                }
                // returns 1 on success, 0 on failure, -1 on error.
                throw new Exception(
                    'OpenSSL error: ' . openssl_error_string()
                );
            case 'hash_hmac':
            default:
                $hash = hash_hmac($algorithm, $msg, $key, true);
                if (function_exists('hash_equals')) {
                    return hash_equals($signature, $hash);
                }
                $len = min(static::safeStrlen($signature), static::safeStrlen($hash));
                $status = 0;
                for ($i = 0; $i < $len; $i++) {
                    $status |= (ord($signature[$i]) ^ ord($hash[$i]));
                }
                $status |= (static::safeStrlen($signature) ^ static::safeStrlen($hash));
                return ($status === 0);
        }
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Decode a JSON string into a PHP object.
     *
     * @param string $input JSON string
     *
     * @return object Object representation of JSON string
     *
     * @throws Exception Provided string was invalid JSON
     */
    public static function jsonDecode($input)
    {
        if (version_compare(PHP_VERSION, '5.4.0', '>=') && !(defined('JSON_C_VERSION') && PHP_INT_SIZE > 4)) {
            /** In PHP >=5.4.0, json_decode() accepts an options parameter, that allows you
             * to specify that large ints (like Steam Transaction IDs) should be treated as
             * strings, rather than the PHP default behaviour of converting them to floats.
             */
            $obj = json_decode($input, false, 512, JSON_BIGINT_AS_STRING);
        } else {
            /** Not all servers will support that, however, so for older versions we must
             * manually detect large ints in the JSON string and quote them (thus converting
             *them to strings) before decoding, hence the preg_replace() call.
             */
            $max_int_length = strlen((string) PHP_INT_MAX) - 1;
            $json_without_bigints = preg_replace('/:\s*(-?\d{'.$max_int_length.',})/', ': "$1"', $input);
            $obj = json_decode($json_without_bigints);
        }
        if (function_exists('json_last_error') && $errno = json_last_error()) {
            static::handleJsonError($errno);
        } elseif ($obj === null && $input !== 'null') {
            throw new Exception('Null result with non-null input');
        }
        return $obj;
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Encode a PHP object into a JSON string.
     *
     * @param object|array $input A PHP object or array
     *
     * @return string JSON representation of the PHP object or array
     *
     * @throws Exception Provided object could not be encoded to valid JSON
     */
    public static function jsonEncode($input)
    {
        $json = json_encode($input);
        if (function_exists('json_last_error') && $errno = json_last_error()) {
            static::handleJsonError($errno);
        } elseif ($json === 'null' && $input !== null) {
            throw new Exception('Null result with non-null input');
        }
        return $json;
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Decode a string with URL-safe Base64.
     *
     * @param string $input A Base64 encoded string
     *
     * @return string A decoded string
     */
    public static function urlsafeB64Decode($input)
    {
        $remainder = strlen($input) % 4;
        if ($remainder) {
            $padlen = 4 - $remainder;
            $input .= str_repeat('=', $padlen);
        }
        return base64_decode(strtr($input, '-_', '+/'));
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Encode a string with URL-safe Base64.
     *
     * @param string $input The string you want encoded
     *
     * @return string The base64 encode of what you passed in
     */
    public static function urlsafeB64Encode($input)
    {
        return str_replace('=', '', strtr(base64_encode($input), '+/', '-_'));
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Helper method to create a JSON error.
     *
     * @param int $errno An error number from json_last_error()
     *
     * @return void
     */
    private static function handleJsonError($errno)
    {
        $messages = array(
            JSON_ERROR_DEPTH => 'Maximum stack depth exceeded',
            JSON_ERROR_STATE_MISMATCH => 'Invalid or malformed JSON',
            JSON_ERROR_CTRL_CHAR => 'Unexpected control character found',
            JSON_ERROR_SYNTAX => 'Syntax error, malformed JSON',
            JSON_ERROR_UTF8 => 'Malformed UTF-8 characters' //PHP >= 5.3.3
        );
        throw new Exception(
            isset($messages[$errno])
            ? $messages[$errno]
            : 'Unknown JSON error: ' . $errno
        );
    }
    /** ----------------------------------------------------------------------------------------------------------
     * Get the number of bytes in cryptographic strings.
     *
     * @param string
     *
     * @return int
     */
    private static function safeStrlen($str)
    {
        if (function_exists('mb_strlen')) {
            return mb_strlen($str, '8bit');
        }
        return strlen($str);
    }
}
```

---

## Chiffrement par clé secrète

Le cryptage par clé secrète (ou cryptage symétrique comme on l'appelle aussi) utilise une seule clé pour crypter et décrypter les données. Voyons comment nous allons mettre en place un tel mécanisme en utilisant Sodium, qui a été introduit dans PHP 7.2. Si vous utilisez une version plus ancienne de PHP, vous pouvez installer sodium via PECL.

Tout d'abord, nous avons besoin d'une clé de cryptage, qui peut être générée en utilisant la fonction `random_bytes()`. Habituellement, vous ne le ferez qu'une seule fois et le stockerez comme variable d'environnement. Rappelez-vous que cette clé doit être gardée secrète à tout prix. Une fois que la clé est compromise, les données cryptées le sont aussi.

    $key = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES);

Pour crypter la valeur, nous la passons à `sodium_crypto_secretbox()` avec notre `$key` et un `$nonce`. Le nonce est généré en utilisant `random_bytes()`, car le même nonce ne doit jamais être réutilisé.

    $nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
    $ciphertext = sodium_crypto_secretbox('This is a secret!', $nonce, $key);

Cela pose un problème car nous avons besoin du nonce pour décrypter la valeur plus tard. Heureusement, les nonces n'ont pas besoin d'être gardées secrètes pour que nous puissions les préfixer à notre $ciphertext puis base64_encode() la valeur avant de les enregistrer dans la base de données.

```
$encoded = base64_encode($nonce . $ciphertext);
var_dump($encoded);

// string 'v6KhzRACVfUCyJKCGQF4VNoPXYfeFY+/pyRZcixz4x/0jLJOo+RbeGBTiZudMLEO7aRvg44HRecC' (length=76)
```

Lorsqu'il s'agit de décrypter la valeur, nous faisons le contraire.

    $decoded = base64_decode($encoded);

Parce que nous connaissons la longueur de nonce, nous pouvons l'extraire en utilisant `mb_substr()` avant de décrypter la valeur.

```
$nonce = mb_substr($decoded, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES, '8bit');
$ciphertext = mb_substr($decoded, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES, null, '8bit');
$plaintext = sodium_crypto_secretbox_open($ciphertext, $nonce, $key);
var_dump($plaintext);

// string 'This is a secret!' (length=17)
```

Exemples  

```php
<?php

echo "Chiffrage\n";
$key = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES);
$nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES);
$ciphertext = sodium_crypto_secretbox('This is a secret!', $nonce, $key);
$encoded = base64_encode($nonce . $ciphertext);
var_dump($encoded);
echo "Déchiffrage\n";
$decoded = base64_decode($encoded);
$nonce = mb_substr($decoded, 0, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES, '8bit');
$ciphertext = mb_substr($decoded, SODIUM_CRYPTO_SECRETBOX_NONCEBYTES, null, '8bit');
$plaintext = sodium_crypto_secretbox_open($ciphertext, $nonce, $key);
var_dump($plaintext);

$msg = 'This is a super secret message!';

// Generating an encryption key and a nonce
$key   = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES); // 256 bit
$nonce = random_bytes(SODIUM_CRYPTO_SECRETBOX_NONCEBYTES); // 24 bytes

// Encrypt
$ciphertext = sodium_crypto_secretbox($msg, $nonce, $key);
// Decrypt
$plaintext = sodium_crypto_secretbox_open($ciphertext, $nonce, $key);

echo $plaintext === $msg ? 'Success' : 'Error';

$msg = 'This is the message to authenticate!';
$key = random_bytes(SODIUM_CRYPTO_SECRETBOX_KEYBYTES); // 256 bit
 
// Generate the Message Authentication Code
$mac = sodium_crypto_auth($msg, $key);
 
// Altering $mac or $msg, verification will fail
echo sodium_crypto_auth_verify($mac, $msg, $key) ? 'Success' : 'Error';

$password = 'password';
echo "\nArgon2i with Sodium\n";
$hash = sodium_crypto_pwhash_str(
    $password,
    SODIUM_CRYPTO_PWHASH_OPSLIMIT_INTERACTIVE,
    SODIUM_CRYPTO_PWHASH_MEMLIMIT_INTERACTIVE
); // 97 bytes

echo sodium_crypto_pwhash_str_verify($hash, $password) ?
     'OK' : 'Error';

echo "\nArgon2i without Sodium\n";
$hash = password_hash($password, PASSWORD_ARGON2I); // 95 bytes

echo password_verify($password, $hash) ? 'OK' : 'Error';

?>
```



C'est tout ce qu'il y a à faire avec le cryptage par clé secrète en PHP, grâce à Sodium !