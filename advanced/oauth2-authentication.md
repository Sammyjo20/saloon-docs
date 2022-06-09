# OAuth2 Authentication

Some API providers implement the OAuth 2 _Authorization Code Flow_ for authentication. Implementing this grant type every time you create a new API integration can be tedious and time-consuming. Saloon offers a simple, extendable OAuth2 framework to help you get up and running quickly.

{% hint style="info" %}
Saloon has implemented the Authorization Code grant type since it is the most common OAuth2 flow. If you require any of the other grant types, please open a discussion on the GitHub repository.
{% endhint %}

### Overview

Saloon has provided methods for the full Authorization Code grant.

```php
$authConnector = new SpotifyAuthConnector;

// 1. Redirect the user to the authorization URL...

$authorizationUrl = $authConnector->getAuthorizationUrl($scopes, $state);

// 2. Handle the callback from the API provider and create an access token...

$authenticator = $authConnector->getAccessTokens($code, $state);

// 3. Authenticate your requests!

$request = new GetTracksRequest;
$request->withAuth($authenticator);
$request->send(); // 🚀

// 4. Refresh your access tokens...

$newAuthenticator = $authConnector->refreshAccessTokens($authenticator);
```

### Prerequisites

This section of the documentation assumes that you are familiar with OAuth2 and specifically the _Authorization Code Grant_.

This feature has been heavily inspired by the [“OAuth2 Client” package by “The PHP League”](https://github.com/thephpleague/oauth2-client) installed millions of times.

### Getting Setup

Let’s get started by preparing our Saloon connector ready to support the Authorization Code Flow. We recommend that you create a new connector in your integration just for authentication with the third-party provider. This can help keep the authentication and API code separate. Some providers may even have a separate OAuth2 server on a different subdomain to the API.

#### 1. Add the AuthorizationCodeGrant trait to your connector

{% hint style="warning" %}
We strongly recommend that you create a new connector just for the OAuth2 flow.
{% endhint %}

```php
<?php

use Sammyjo20\Saloon\Helpers\OAuth2\OAuthConfig;
use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyAuthConnector extends SaloonConnector
{
    use AuthorizationCodeGrant;
}
```

#### 2. Configure the base endpoint

After you have created the connector and added the trait, make sure to set the base endpoint to the URL of the OAuth2 server. For example:

```php
<?php

use Sammyjo20\Saloon\Helpers\OAuth2\OAuthConfig;
use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyAuthConnector extends SaloonConnector
{
    use AuthorizationCodeGrant;

    public function defineBaseUrl(): string
    {
        return 'https://accounts.spotify.com';
    }
}
```

#### 3. Configure the default OAuth config

Extend the **defaultOauthConfig()** method into your connector and start with setting your client ID, secret and redirect URI. You can also customise the various endpoints if the third party requires it.

```php
<?php

use Sammyjo20\Saloon\Helpers\OAuth2\OAuthConfig;
use Sammyjo20\Saloon\Http\SaloonConnector;
use Sammyjo20\Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyAuthConnector extends SaloonConnector
{
    use AuthorizationCodeGrant;

    public function defineBaseUrl(): string
    {
        return 'https://accounts.spotify.com';
    }

    protected function defaultOauthConfig(): OAuthConfig
    {
        return OAuthConfig::make()
            ->setClientId('my-client-id')
            ->setClientSecret('my-client-secret')
	    ->setDefaultScopes(['user-read-currently-playing'])
            ->setRedirectUri('https://my-app.saloon.dev/auth/callback')
	    ->setAuthorizeEndpoint('authorize')
            ->setTokenEndpoint('token')
            ->setUserEndpoint('user')
    }
}
```

{% hint style="info" %}
If your OAuth2 config is dependant on a per-user/tenant basis, you can modify the config after you have instantiated the connector.
{% endhint %}

### Generating An Authorization URL

To generate an authorization URL, you can use the **getAuthorizationUrl()** method on the connector. Firstly instantiate the connector and then run the method. You can also provide scopes as well. It will also generate state for you if it has been provided.

```php
<?php

$authConnector = new SpotifyAuthConnector;

$scopes = ['user-library-read'];

$authorizationUrl = $authConnector->getAuthorizationUrl($scopes, $state);

// Redirect the user to the URL...
```

#### Optional State

To help prevent CSRF attacks, It’s highly recommended that you create a token in your authorization URL that you can confirm when the user redirects back to your application. This is known as state. If you do not provide state, Saloon will generate a 32 character alpha-numeric string for the state. You should generate the authorization URL first, then store the state securely. The application will return the state in your callback url which you can confirm. If you are using a framework like Laravel, you could store this state token in the user’s session.

```php
<?php

$authConnector = new SpotifyAuthConnector;

$state = 'secret';

$authorizationUrl = $authConnector->getAuthorizationUrl($scopes, $state);

// Get the state, secure it somewhere like the session.

$state = $authConnector->getState(); // 'secret'
```

### Creating Access Tokens

After the user has approved your application, the API provider will redirect you back to your application with an authorization code and state. This should be passed into your **getAccessToken()** method on your connector. If successful, the method will return an **AccessTokenAuthenticator**. This is a Saloon Authenticator that can be used to authenticate the rest of your requests.

```php
<?php

$authConnector = new SpotifyAuthConnector;

$authenticator = $authConnector->getAccessTokens($code);

// ... Use authenticator in your other requests.
```

{% hint style="info" %}
The method will return an **AccessTokenAuthenticator**. This is a Saloon Authenticator that can be used to authenticate the rest of your requests. [Click here to read more about using authenticators.](../next-steps/authentication.md)
{% endhint %}

### Storing Authentication On Users

You will likely need to store the authenticator securely against a user, like in an encrypted field in the database. You may serialise and unserialise the authenticator using the helper methods below.

```php
<?php

$authConnector = new SpotifyAuthConnector;

$authenticator = $authConnector->getAccessTokens($code);

$serialized = $authenticator->serialize(); // Securely store this against your user.

$authenticator = AccessTokenAuthenticator::unserialize($serialized); // You can unserialize it too.
```

{% hint style="info" %}
If you are using Laravel Eloquent, you can use the **OAuthAuthenticatorCast** to automatically cast the authenticator for storing into your database.
{% endhint %}

### Authenticate Your Requests

After you have created the access tokens above, you should have an **AccessTokenAuthenticator.** This class can be used to authenticate your other connectors/requests with the access token you have just received. Just use the **withAuth()** method on either your request or your connector.

```php
<?php

$authConnector = new SpotifyAuthConnector;

$authenticator = $authConnector->getAccessTokens($code);

$request = new CurrentSongRequest;
$request->withAuth($authenticator);
$response = $request->send();

// Or you can authorize the whole connector

$connector = new SpotifyApiConnector;
$connector->withAuth($authenticator);

// Make requests...
```

### Refreshing Access Tokens

Before using the authenticator, you should always check if the access token has expired and if it needs refreshing. When you need to refresh access tokens, you can call the **refreshAccessToken()** method which will create a fresh authenticator.

```php
<?php

$authenticator = $user->auth; // Your authenticator class.

// Check if the authenticator has expired, if it has - we can refresh
// the access token.

if ($authenticator->hasExpired() === true) {
    $authConnector = new SpotifyAuthConnector;
    $authenticator = $authConnector->refreshAccessToken($authenticator);
    
    $user->auth = $authenticator;
    $user->save();
}

// Continue to make your request...

$request = new CurrentSongRequest;
$request->withAuth($authenticator);
$response = $request->send();
```

### Customising The Authenticator

Sometimes the API provider you are authenticating with may require additional information to be used in the authenticator. You can customise how the authenticator will be created by extending the **createAccessTokenAuthenticator()** protected method. Make sure that you return a class that implements the **AccessTokenAuthenticatorInterface**.

```php
<?php

protected function createOAuthAuthenticator(string $accessToken, string $refreshToken, CarbonInterface $expiresAt): OAuthAuthenticatorInterface
{
    return new SpotifyAuthenticator($accessToken, $refreshToken, $expiresAt);
}
```

### Customising How The Authenticator Is Created

Sometimes the API provider you are authenticating with may have a different way that they respond with their tokens. If you need to customise the way Saloon creates the authenticator you can extend the **createOAuthAuthenticatorFromResponse()** method.

```php
protected function createOAuthAuthenticatorFromResponse(SaloonResponse $response, string $fallbackRefreshToken = null): OAuthAuthenticatorInterface
{
    $responseData = $response->object();

    $accessToken = $responseData->access_token;
    $refreshToken = $responseData->refresh_token ?? $fallbackRefreshToken;
    $expiresAt = CarbonImmutable::now()->addSeconds($responseData->expires_in);

    return $this->createOAuthAuthenticator($accessToken, $refreshToken, $expiresAt);
}
```

### Per User/Tenant OAuth Config

Sometimes you may have separate client credentials for every tenant or user that you need to authenticate the third party with. You can instantiate the connector and then change the configuration depending on your user.

```php
<?php

$user = auth()->user(); // Your tenant/user.

$authConnector = new SpotifyAuthConnector;

// Overwrite the config just for this connector.

$authConnector->oauthConfig()->setClientId($user->spotify_client_id);
$authConnector->oauthConfig()->setClientSecret($user->spotify_client_secret);

// Continue like normal...

$authorizationUrl = $authConnector->**getAuthorizationUrl(...);**
```

### Real-world example

If you would like to see an example integration using the OAuth2 methods mentioned above, see the following Laravel app.

[https://github.com/Sammyjo20/saloon-spotify-example](https://github.com/Sammyjo20/saloon-spotify-example)

#### Controller

[https://github.com/Sammyjo20/saloon-spotify-example/blob/main/app/Http/Controllers/SpotifyController.php](https://github.com/Sammyjo20/saloon-spotify-example/blob/main/app/Http/Controllers/SpotifyController.php)

#### Saloon Connector/Requests

[https://github.com/Sammyjo20/saloon-spotify-example/tree/main/app/Http/Integrations/Spotify](https://github.com/Sammyjo20/saloon-spotify-example/tree/main/app/Http/Integrations/Spotify)

### Available Methods / API

#### Connector

*   **defaultOauthConfig()**

    Allows you to define the default configuration for the OAuth2 server that you are attempting to authenticate with.
*   **oauthConfig()**

    Method that allows you to access and modify the OAuth2 configuration in the connector and after the connector has been created. This is useful if you need to set the credentials on a per-tenant basis.
*   **getAuthorizationUrl($scopes, $state, $scopeSeparator)**

    This method will return the authorization URL that the user should be redirected to in order to authorize your application with the third-party provider.
*   **getAccessToken($code, $state)**

    This method will make a request to create an access token from the authorization code that your application receives in the call-back URL.
*   **refreshAccessToken($accessTokenAuthenticator|$refreshToken)**

    This method will make a request to refresh an access token. It will return an instance of AccessTokenAuthenticator which can be used to authenticate your connectors.
*   **getState()**

    If state has not been provided in the **getAuthorizationUrl()** method, this method will return the randomly generated state that was sent to the provider. You should store this in a secure location and use it to verify the flow was not tampered with.
*   **getUser($accessTokenAuthenticator)**

    This method can be used to retrieve the resource owner / user once you have successfully authenticated. **This may not work, depending on if the OAuth2 server/API provides it.**

#### OAuth2 Config Methods

* **getClientId / setClientId**
* **getClientSecret / setClientSecret**
* **getRedirectUri / setRedirectUri**
* **getAuthorizeEndpoint / setAuthorizeEndpoint**
* **getTokenEndpoint / setTokenEndpoint**
* **getUserEndpoint / setUserEndpoint**
* **getDefaultScopes / setDefaultScopes**
* **validate**
