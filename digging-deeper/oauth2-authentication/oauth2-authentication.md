# Authorization Code Grant

Some API providers implement the OAuth 2 _Authorization Code Flow_ for authentication. Implementing this grant type every time you create a new API integration can be tedious and time-consuming. Saloon offers a simple, extendable OAuth2 trait to help you get up and running quickly.

### Prerequisites

This section of the documentation assumes that you are familiar with OAuth2 and specifically the _Authorization Code Grant_. If you are not familiar with how this grant type works, [Auth0 has a great explanation on its website.](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)

### Flow Example

Saloon has provided methods for the full Authorization Code grant.

```php
$connector = new SpotifyConnector;

// 1. Redirect the user to the authorization URL...

$authorizationUrl = $connector->getAuthorizationUrl($scopes, $state);

// 2. Handle the callback from the API provider and create an access token...

$authenticator = $connector->getAccessToken($code, $state);

// 3. Authenticate the connector

$connector->authenticate($authenticator);

// 4. Send your requests

$connector->send(new GetTracksRequest);

// 5. Refresh your access tokens...

$newAuthenticator = $connector->refreshAccessTokens($authenticator);
```

### Getting Started

Let’s start with preparing our connector to support the Authorization Code Flow. All we have to do is add the `AuthorizationCodeGrant` trait to our connector.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyConnector extends Connector;
{
    use AuthorizationCodeGrant;
}
```

After you have added the trait, you will need to tell Saloon how to authenticate with your API. First, extend the `defaultOauthConfig` method and use the methods to define your client ID, secret and redirect URI. Saloon also has sensible defaults set for the authorization and token endpoints, but you may customize them if you need to. For example, Spotify has a different base URL than the connector's base URL, so we have overwritten it in this example.

You can also provide default scopes and even provide a callback to modify the OAuth2 requests being sent.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Http\Request;
use Saloon\Helpers\OAuth2\OAuthConfig;
use Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyConnector extends Connector
{
    use AuthorizationCodeGrant;

    public function resolveBaseUrl(): string
    {
        // Spotify's API has a different base URL for OAuth2 auth.
    
        return 'https://api.spotify.com/v1';
    }

    protected function defaultOauthConfig(): OAuthConfig
    {
        return OAuthConfig::make()
            ->setClientId('my-client-id')
            ->setClientSecret('my-client-secret')
	    ->setDefaultScopes(['user-read-currently-playing'])
            ->setRedirectUri('https://my-app.saloon.dev/auth/callback')
	    ->setAuthorizeEndpoint('https://accounts.spotify.com/authorize')
            ->setTokenEndpoint('https://accounts.spotify.com/api/token')
            ->setUserEndpoint('/me')
            ->setRequestModifier(function (Request $request) {
                // Optional: Modify the requests being sent.
            })
    }
}
```

{% hint style="info" %}
Each of the endpoint methods, like `setAuthorizeEndpoint`on the OAuthConfig class support full URLs if you need to overwrite the base URL on the connector however you may just use the endpoint if the base URL is the same.
{% endhint %}

#### Overwriting the OAuth2 config

Sometimes, you may have a different OAuth2 client ID and secret for each user of your application. If your OAuth2 config is dependent on a per-user/tenant basis, it's recommended that you pass in the credentials as constructor arguments of your connector and then set the `oauthConfig` inside the constructor.

In the following example, I will pass in the `$clientId` and the `$clientSecret` as constructor arguments and overwrite the OAuth2 config.

<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Connector;
use Saloon\Traits\OAuth2\AuthorizationCodeGrant;

class SpotifyConnector extends Connector
{
    use AuthorizationCodeGrant;
    
<strong>    public function __construct(string $clientId, string $clientSecret)
</strong>    {
        $this->oauthConfig()->setClientId($clientId);
        $this->oauthConfig()->setClientSecret($clientSecret);
    }
    
    // ...
}
</code></pre>

### Creating an Authorization URL

Now we have setup our connector to support the authorization code grant, we are ready to start the OAuth2 process. Usually, the first stage is to generate a URL to redirect our application users to. To generate an authorization URL, you can use the `getAuthorizationUrl` method on the connector.

```php
<?php

$connector = new SpotifyConnector;
$authorizationUrl = $connector->getAuthorizationUrl();
```

You can also pass in scopes which will be merged with the default scopes if you provided them in the OAuth config. Saloon will separate scopes with spaces but if your API integration requires scopes to be separated any other way, you can specify this with the `scopeSeparator` argument.

```php
$authorizationUrl = $connector->getAuthorizationUrl(
    scopes: ['user-library-read'],
    scopeSeparator: '+',
);
```

You can also provide additional query parameters if you need to with the `additionalQueryParameters` argument. This should be a key-value array where the key is the query parameter name and the value is the value of the query parameter.

```php
$authorizationUrl = $authConnector->getAuthorizationUrl(
    additionalQueryParameters: [
        'username' => 'JohnWayne'
    ],
);
```

#### State

To help prevent CSRF attacks or send additional data during authentication, you can pass an additional unique string in your authorization URL that the API's OAuth2 server will send back to you after your user has approved or denied access to your OAuth2 app.

```php
$authorizationUrl = $authConnector->getAuthorizationUrl(
    state: 'application-user-id',
);
```

If you do not provide your own state, Saloon will automatically generate a unique, 32-character string. Once you have generated the authorization URL, you can then use the `getState` method on your connector to get the state back. You should store this string in your application's session or cache to be verified later.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$connector = new SpotifyConnector;
$authorizationUrl = $connector->getAuthorizationUrl();

<strong>$state = $authConnector->getState(); // '8484b43fdjfdnfdj3llls...'
</strong></code></pre>

### Creating Access Tokens

After the user has approved your application, the API provider will redirect you back to your application with an authorization code and state. This data usually sent in the form of query parameters should be passed into your `getAccessToken` method on your connector. If successful, the method will return an `AccessTokenAuthenticator`. The access token, refresh token and expiry are wrapped up in a [Saloon Authenticator](../../the-basics/authentication.md#custom-authenticators) class that can be used to authenticate your connector/requests. It acts like a DTO that can be easily serialized and transported around your application.

<pre class="language-php"><code class="lang-php">&#x3C;?php

$connector = new SpotifyConnector;
<strong>$authenticator = $authConnector->getAccessToken($code);
</strong>
// Use authenticator to authenticate your connector instance

$connector->authenticate($authenticator);

// Any request sent through this connector will now have authentication applied

$connector->send(new GetTracksRequest);
</code></pre>

{% hint style="info" %}
Once you have received the authenticator instance, you should cache it securely in your application for future use. Read further to see how you can do this.
{% endhint %}

#### Verifying State

If you stored the state that was generated during creating an authorization URL, you should pass this expected state alongside the state sent back by the API provider's OAuth2 server. This will be used to verify the state provided back by the application is valid. If the state does not match the expected state, Saloon will throw an exception.

```php
<?php

$authConnector = new SpotifyConnector;

// It will throw an exception if the state and expected state don't match
// however both must be present

$authenticator = $authConnector->getAccessToken($code, $state, $expectedState);
```

### Storing Authentication For Later

You will likely need to store the authenticator securely so you can use it for future requests. You may serialize and unserialize the authenticator class using the helper methods below, then you can store the string wherever you like, usually encrypted in the database if it's against a user. Then, you can retrieve this authenticator and use it to authenticate your connector.

```php
<?php

$connector = new SpotifyConnector;
$authenticator = $connector->getAccessToken($code);

// Securely store this against your user.

$serialized = $authenticator->serialize(); 

// Unserialize the authenticator when retrieving it

$authenticator = AccessTokenAuthenticator::unserialize($serialized);
```

### Authenticator Methods

The authenticator returned by Saloon when using the `getAccessToken` or `refreshAccessToken` methods will contain the Access Token, Refresh Token and Expiry Date that was returned by the OAuth2 server. You can access these properties with the following methods. You can also check if the authenticator has expired, which will come in handy when refreshing access tokens.

```php
<?php

$authenticator->getAccessToken();
$authenticator->getRefreshToken();
$authenticator->getExpiry();

$authenticator->hasExpired();
$authenticator->hasNotExpired();
```

### Refreshing Access Tokens

When retrieving your authenticator out of storage, you should always check if the access token has expired and if it needs refreshing. If the authenticator's access token has expired, you can call the `refreshAccessToken` method which will create a fresh authenticator.

In this example, `$user` is the user of my application and I have written methods to get and store the authenticators.

```php
<?php

// $user in this example is my application's user

$authenticator = $user->getCachedAuthenticator();
$connector = new SpotifyConnector;

if ($authenticator->hasExpired()) {
    // We'll refresh the access token which will return a new authenticator
    // which we can store against our user in our application.

    $authenticator = $connector->refreshAccessToken($authenticator);    
    $user->updateAuthenticator($authenticator);
}

// Authenticate our connector and send the request

$connector->authenticate($authenticator);

$response = $connector->send(new GetTracksRequest);
```

{% hint style="info" %}
If you are using Laravel and the Saloon Laravel library, you can use built-in `EncryptedOAuthAuthenticatorCast` **/** `OAuthAuthenticatorCast` Eloquent casts to automatically cast the authenticator for storing in your database.
{% endhint %}

### Customising The Authenticator

Sometimes the API provider you are authenticating with may require additional information to be used in the authenticator. You can customise how the authenticator will be created by extending the `createAccessTokenAuthenticator` method on your connector.

```php
<?php

protected function createOAuthAuthenticator(string $accessToken, string $refreshToken, DateTimeImmutable $expiresAt): OAuthAuthenticatorInterface
{
    return new SpotifyAuthenticator($accessToken, $refreshToken, $expiresAt);
}
```

### Customising How The Authenticator Is Created

Sometimes the API provider you are authenticating with may have a different way that they respond with their tokens. If you need to customise the way Saloon creates the authenticator you can extend the `createOAuthAuthenticatorFromResponse` method.

```php
<?php

protected function createOAuthAuthenticatorFromResponse(SaloonResponse $response, string $fallbackRefreshToken = null): OAuthAuthenticatorInterface
{
    $responseData = $response->object();

    $accessToken = $responseData->access_token;
    $refreshToken = $responseData->refresh_token ?? $fallbackRefreshToken;
    $expiresAt = new DateTimeImmutable('+' . $responseData->expires_in . ' seconds');

    return $this->createOAuthAuthenticator($accessToken, $refreshToken, $expiresAt);
}
```

### Customising The Requests

Sometimes you might integrate with an API that requires additional query parameters or headers to be sent with the OAuth2 flow. You may use the `requestModifier` property on the methods or use the `setRequestModifier` method within the `OAuthConfig` to add a callable that is invoked before a request is sent.

{% tabs %}
{% tab title="Per Request" %}
```php
<?php

$connector->getAccessToken($code, requestModifier: function (Request $request) {
    $request->query()->add('access_type', 'offline');
});

$connector->getRefreshToken($code, requestModifier: function (Request $request) {
    $request->headers()->add('X-App-Key', $appKey);
});

$connector->getUser($code, requestModifier: function (Request $request) {
    $request->headers('Accept', 'text/plain');
});
```
{% endtab %}

{% tab title="All Requests" %}
```php
<?php

use Saloon\Http\Request;
use Saloon\Http\OAuth2\GetUserRequest;
use Saloon\Http\OAuth2\GetAccessTokenRequest;
use Saloon\Http\OAuth2\GetRefreshTokenRequest;

protected function defaultOauthConfig(): OAuthConfig
{
    return OAuthConfig::make()
        ->setClientId('my-client-id')
        ->setClientSecret('my-client-secret')
        ->setRedirectUri('https://my-app.saloon.dev/auth/callback')
	->setRequestModifier(function (Request $request) {
	     // This callback is invoked on every request, so you 
	     // may want to use if-statements or a match statement
	     // to apply conditions based on request.
	
             if ($request instanceof GetAccessTokenRequest) {
                 $request->query()->add('access_type', 'offline');
             }
             
             if ($request instanceof GetRefreshTokenRequest) {
                 $request->headers()->add('X-App-Key', $appKey);
             }
             
             if ($request instanceof GetUserRequest) {
                 $request->headers('Accept', 'text/plain');
             }
        }),
}
```
{% endtab %}
{% endtabs %}

### Using your own request classes

There are situations where Saloon's own request classes for getting the access token, refreshing the access token or getting the user might not suit the API you are integrating with. For example, if an API uses JSON encoding instead of form encoding. You may use the following methods on your connector to overwrite the instantiation process of the request classes.

```php
<?php

class SpotifyConnector extends Connector
{
    // ...

    protected function resolveAccessTokenRequest(string $code, OAuthConfig $oauthConfig): Request
    {
        return new CustomGetAccessTokenRequest($code, $oauthConfig);
    }
    
    protected function resolveRefreshTokenRequest(OAuthConfig $oauthConfig, string $refreshToken): Request
    {
        return new CustomGetRefreshTokenRequest($oauthConfig, $refreshToken);
    }
    
    protected function resolveUserRequest(OAuthConfig $oauthConfig): Request
    {
        return new CustomGetUserRequest($oauthConfig);
    }
}
```

### Returning Responses

If you prefer, you may request Saloon to return a `Saloon\Http\Resonse` instance instead of a `AccessTokenAuthenticator` when creating or refreshing access tokens. To use responses, just provide the `returnResponse` argument when creating or refreshing access tokens.

```php
<?php

$connector = new SpotifyConnector;

$response = $connector->getAccessToken(
    returnResponse: true,
);

$response = $connector->refreshAccessToken(
    returnResponse: true,
);
```

### Real-world example

If you would like to see an example integration using the OAuth2 methods mentioned above, you can see the following Laravel application.

[https://github.com/Sammyjo20/saloon-v2-spotify-example](https://github.com/Sammyjo20/saloon-v2-spotify-example)
