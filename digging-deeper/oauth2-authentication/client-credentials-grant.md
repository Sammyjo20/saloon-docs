# Client Credentials Grant

Some API providers implement the OAuth2 _Client Credentials Grant_ for authentication. Implementing this grant type every time you create a new API integration can be tedious and time-consuming. Saloon offers a simple, extendable OAuth2 trait to help you get up and running quickly.

### Prerequisites

This section of the documentation assumes that you are familiar with OAuth2 and specifically the _Client Credentials Grant_. If you are not familiar with how this grant type works, [Auth0 has a great explanation on its website.](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow)

### Flow Example

Saloon has provided methods for the full client credentials grant.

```php
$connector = new WarehouseConnector;

// 1. Create an access token authenticator

$authenticator = $connector->getAccessToken($scopes);

// 2. Authenticate the connector

$connector->authenticate($authenticator);

// 3. Send your requests

$connector->send(new GetInventoryRequest);
```

### Getting Started

Let's start with preparing our connector to support the client credentials grant. All we have to do is add the `ClientCredentialsGrant` trait to our connector.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Traits\OAuth2\ClientCredentialsGrant;

class WarehouseConnector extends Connector
{
    use ClientCredentialsGrant;
}
```

After you have added the trait, you will need to tell Saloon how to authenticate with your API. First, extend the `defaultOauthConfig` method and use the methods to define your client ID and secret. Saloon also has sensible defaults set for the token endpoint, but you may customize it if you need to. For example, some APIs have a different base URL than the connector's base URL. You can also provide default scopes and even provide a callback to modify the OAuth2 requests being sent.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Traits\OAuth2\ClientCredentialsGrant;

class WarehouseConnector extends Connector
{
    use ClientCredentialsGrant;
    
    public function resolveBaseUrl(): string
    {
        return 'https://local-warehouse.app';
    }
    
    protected function defaultOauthConfig(): OAuthConfig
    {
        return OAuthConfig::make()
            ->setClientId('my-client-id')
            ->setClientSecret('my-client-secret')
	    ->setDefaultScopes(['inventory.read'])
            ->setTokenEndpoint('/oauth/token')
            ->setRequestModifier(function (Request $request) {
                // Optional: Modify the requests being sent.
            })
    }
}
```

{% hint style="info" %}
The endpoint method, like `setTokenEndpoint` on the OAuthConfig class support full URLs if you need to overwrite the base URL on the connector however you may just use the endpoint if the base URL is the same.
{% endhint %}

#### Overwriting the OAuth2 config

Sometimes, you may have a different OAuth2 client ID and secret for each user of your application. If your OAuth2 config is dependent on a per-user/tenant basis, it's recommended that you pass in the credentials as constructor arguments of your connector and then set the `oauthConfig` inside the constructor.&#x20;

In the following example, I will pass in the `$clientId` and the `$clientSecret` as constructor arguments and overwrite the OAuth2 config.

```php
<?php

use Saloon\Http\Connector;
use Saloon\Traits\OAuth2\ClientCredentialsGrant;

class WarehouseConnector extends Connector
{
    use ClientCredentialsGrant;
    
    public function __construct(string $clientId, string $clientSecret)
    {
        $this->oauthConfig()->setClientId($clientId);
        $this->oauthConfig()->setClientSecret($clientSecret);
    }
    
    // ...
}
```

### Creating Access Tokens

You are now ready to create access tokens. You should use the `getAccessToken` method on your connector. If successful, the method will return an `AccessTokenAuthenticator`.  The access token and expiry (if provided) are wrapped up in a [Saloon Authenticator](../../the-basics/authentication.md#custom-authenticators) class that can be used to authenticate your connector/requests. It acts like a DTO that can be easily serialized and transported around your application.&#x20;

```php
<?php

$connector = new WarehouseConnector;
$authenticator = $authConnector->getAccessToken();

// Use authenticator to authenticate your connector instance

$connector->authenticate($authenticator);

// Any request sent through this connector will now have authentication applied

$connector->send(new GetInventoryRequest);
```

{% hint style="info" %}
Once you have received the authenticator instance, you should cache it securely in your application for future use. Read further to see how you can do this.
{% endhint %}

#### Custom Scopes

Sometimes you may need to provide an additional "scope" to declare the level of access that your token needs. You may provide default scopes in the OauthConfig class on your connector, but you can also provide additional scopes when creating access tokens. Saloon will separate scopes with spaces but if your API integration requires scopes to be separated any other way, you can specify this with the `scopeSeparator` argument.

```php
<?php

$connector = new WarehouseConnector;

$authenticator = $authConnector->getAccessToken(
    scopes: ['inventory.update', 'inventory.delete'],
    scopeSeparator: '+',
);
```

#### Returning Responses

If you prefer, you may request Saloon to return a `Saloon\Http\Resonse` instance instead of a `AccessTokenAuthenticator` when creating access tokens. To use responses, just provide the `returnResponse` argument when creating access tokens.

```php
<?php

$connector = new WarehouseConnector;

$response = $authConnector->getAccessToken(
    returnResponse: true,
);
```

### Authenticator Methods

The authenticator returned by Saloon when using the `getAccessToken`  method will contain the Access Token, and optionally an expiry date that was returned by the OAuth2 server. You can access these properties with the following methods. You can also check if the authenticator has expired, which will come in handy when refreshing access tokens.

```php
<?php

$authenticator->getAccessToken();
$authenticator->getExpiry();

$authenticator->hasExpired();
$authenticator->hasNotExpired();
```

### Customising The Authenticator

Sometimes the API provider you are authenticating with may require additional information to be used in the authenticator. You can customise how the authenticator will be created by extending the `createAccessTokenAuthenticator` method on your connector.

```php
<?php

protected function createOAuthAuthenticator(string $accessToken, ?DateTimeImmutable $expiresAt = null): OAuthAuthenticatorInterface
{
    return new WarehouseAuthenticator($accessToken, $expiresAt);
}
```

### Customising How The Authenticator Is Created

Sometimes the API provider you are authenticating with may have a different way that they respond with their tokens. If you need to customise the way Saloon creates the authenticator you can extend the `createOAuthAuthenticatorFromResponse` method.

```php
<?php

protected function createOAuthAuthenticatorFromResponse(SaloonResponse $response): OAuthAuthenticatorInterface
{
    $responseData = $response->object();

    $accessToken = $responseData->access_token;
    $expiresAt = new DateTimeImmutable('+' . $responseData->expires_in . ' seconds');

    return $this->createOAuthAuthenticator($accessToken, $expiresAt);
}
```

### Customising The Requests

Sometimes you might integrate with an API that requires additional query parameters or headers to be sent with the OAuth2 flow. You may use the `requestModifier` method on the `getAccessToken` method or use the `setRequestModifier` method within the `OAuthConfig` to add a callable that is invoked before a request is sent.

{% tabs %}
{% tab title="Per Request" %}
```php
<?php

$connector->getAccessToken(requestModifier: function (Request $request) {
    $request->query()->add('access_type', 'offline');
});
```
{% endtab %}

{% tab title="Untitled" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Request;
use Saloon\Http\OAuth2\GetClientCredentialsTokenRequest;

protected function defaultOauthConfig(): OAuthConfig
{
    return OAuthConfig::make()
        ->setClientId('my-client-id')
        ->setClientSecret('my-client-secret')
<strong>	->setRequestModifier(function (GetClientCredentialsTokenRequest $request) {
</strong><strong>	     //
</strong><strong>        )},
</strong>}
</code></pre>
{% endtab %}
{% endtabs %}

### Using your own request classes

There are situations where Saloon's own request classes for getting the access token might not suit the API you are integrating with. For example, if an API uses JSON encoding instead of form encoding. You may use the following method on your connector to overwrite the instantiation process of the request class.

```php
<?php

class SpotifyConnector extends Connector
{
    // ...

    protected function resolveAccessTokenRequest(OAuthConfig $oauthConfig, array $scopes = [], string $scopeSeparator = ' '): Request
    {
        return new GetClientCredentialsTokenRequest($oauthConfig, $scopes, $scopeSeparator);
    }
}
```
