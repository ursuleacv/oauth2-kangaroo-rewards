# Kangaroo Rewards Provider for OAuth 2.0 Client

[![Build Status](https://travis-ci.org/ursuleacv/oauth2-kangaroo-rewards.png?branch=master)](https://travis-ci.org/ursuleacv/oauth2-kangaroo-rewards)
[![Latest Version](https://img.shields.io/github/release/ursuleacv/oauth2-kangaroo-rewards.svg?style=flat-square)](https://github.com/ursuleacv/oauth2-kangaroo-rewards/releases)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

This package provides Kangaroo Rewards OAuth 2.0 support for the PHP League's [OAuth 2.0 Client](https://github.com/thephpleague/oauth2-client).

This package is compliant with [PSR-1][], [PSR-2][], [PSR-4][], and [PSR-7][]. If you notice compliance oversights, please send a patch via pull request.

[PSR-1]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md
[PSR-2]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md
[PSR-4]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md
[PSR-7]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-7-http-message.md


## Requirements

The following versions of PHP are supported.

* PHP 5.5
* PHP 5.6
* PHP 7.0
* HHVM

## Installation

Add the following to your `composer.json` file.

```json
{
    "require": {
        "ursuleacv/oauth2-kangaroo-rewards": "~1.0"
    }
}
```

## Usage

### Authorization Code Flow

```php
session_start();

$provider = new KangarooRewards\OAuth2\Client\Provider\Kangaroo([
    'clientId' => CLIENT_ID,
    'clientSecret' => CLIENT_SECRET,
    'redirectUri' => REDIRECT_URI,
]);

if (!isset($_GET['code'])) {

    // If we don't have an authorization code then get one
    $authUrl = $provider->getAuthorizationUrl([]);
    $_SESSION['oauth2state'] = $provider->getState();
    
    echo '<a href="'.$authUrl.'">Log in with Kangaroo Rewards!</a>';
    exit;

// Check given state against previously stored one to mitigate CSRF attack
} elseif (empty($_GET['state']) || ($_GET['state'] !== $_SESSION['oauth2state'])) {

    unset($_SESSION['oauth2state']);
    echo 'Invalid state.';
    exit;

}

// Try to get an access token (using the authorization code grant)
$token = $provider->getAccessToken('authorization_code', [
    'code' => $_GET['code']
]);

try {

    // We got an access token, let's make some requests
    $resourceOwner = $kangaroo->getResourceOwner($token);
    
    $http = new GuzzleHttp\Client;
    $response = $http->get('https://api.kangaroorewards.com/customers', [
        'headers' => [
            'User-Agent' => '{Client Name}',
            'Content-Type' => 'application/json',
            'Accept' => 'application/vnd.kangaroorewards.api.v1+json;',
            'Authorization' => 'Bearer ' . $token->getToken(),
        ]
    ]);
    $r = json_decode((string) $response->getBody(), true);

    echo '<pre>';
    var_export($resourceOwner->toArray());
    var_export($r);
    echo '</pre>';

} catch (Exception $e) {
    exit($e->getMessage());
}

echo '<pre>';
// Use this to interact with the API on the client behalf
var_dump($token->getToken());

echo '</pre>';
```

## Testing

``` bash
$ ./vendor/bin/phpunit
```

## Contributing

Please see [CONTRIBUTING](https://github.com/ursuleacv/oauth2-kangaroo-rewards/blob/master/CONTRIBUTING.md) for details.


## Credits

- [Valentin Ursuleac](https://github.com/ursuleacv)

## License

The MIT License (MIT). Please see [License File](https://github.com/ursuleacv/oauth2-kangaroo-rewards/blob/master/LICENSE) for more information.
