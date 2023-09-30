# 🔐 Authentication

There are several ways to authenticate with an API; most of the time, you are expected to provide a header or query parameter to authenticate. Saloon has built helpers for the most common authentication methods to help you, but you can also create custom authenticators for advanced authentication. You can also provide defaults for authentication should you require it.

{% hint style="info" %}
While you may opt-in to Saloon's authentication classes, it's not the only way to authenticate with Saloon. Another good method of authentication is using the `defaultHeaders, defaultConfig` or `defaultQuery` on your connector/request.
{% endhint %}

### Available Authentication Methods

These authentication methods are available on both the connector and the request instances.

1.  **withTokenAuth($token, $prefix = 'Bearer')**

    Sends an `Authorization` header e.g `Authorization: Bearer your-api-key`
2. **withBasicAuth($username, $password)**\
   Uses HTTP "basic" authentication
3. **withDigestAuth($username, $password)**\
   Uses HTTP "basic" authentication with a digest
4. **withQueryAuth($parameter, $value)**\
   Uses a query parameter to authenticate. e.g `?api_key=your-api-key`

### Looking for OAuth2 authentication?

Saloon has native support for the client credentials and authorization code grant types for OAuth2. To get setup with this, head on over to the [dedicated OAuth Authentication page](../digging-deeper/oauth2-authentication/oauth2-authentication.md).

### Authenticating all requests

Most of the time, you use one API key for all your API requests to a service, like a personal access token or a username/password combination. Therefore, it's recommended that you use the constructor of your connector and expect an API token to be provided. This is useful if you have a different API key per user that needs to be passed into your connector.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

class ForgeConnector extends Connector
{ 
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    /**
     * Constructor
     *
     * @param string $apiKey
     */
    public function __construct(protected string $apiKey)
    {
        $this->withTokenAuth($this->apiKey);
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$forge = new ForgeConnector('my-api-key');

// All API requests will be authenticated with the API key
```
{% endtab %}
{% endtabs %}

### Default authentication

Sometimes you may use a single API key in your .env file/application config, and you don't want to pass it in every time you instantiate; you may use the `defaultAuth` method on your connector, and every request will be authenticated.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Contracts\Authenticator;
use Saloon\Http\Auth\TokenAuthenticator;

class ForgeConnector extends Connector
{ 
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    protected function defaultAuth(): ?Authenticator
    {
        return new TokenAuthenticator(config('app.services.forge'));
    }
}
```
{% endtab %}

{% tab title="Usage" %}
```php
<?php

$forge = new ForgeConnector;

// All API requests will be authenticated with the default auth.
```
{% endtab %}
{% endtabs %}

#### Authenticator Classes

| Method         | Class                                     |
| -------------- | ----------------------------------------- |
| withTokenAuth  | use Saloon\Http\Auth\TokenAuthenticator;  |
| withBasicAuth  | use Saloon\Http\Auth\BasicAuthenticator;  |
| withDigestAuth | use Saloon\Http\Auth\DigestAuthenticator; |
| withQueryAuth  | use Saloon\Http\Auth\QueryAuthenticator;  |

### Authentication on the fly

You may want to authenticate a request or a connector on the fly on a per-request or per-connector basis. You can use the authentication methods directly.

{% tabs %}
{% tab title="Connector" %}
```php
<?php

$forge = new ForgeConnector;
$forge->withTokenAuth($user->forge_api_key);

// All API requests sent with this connector instance will be authenticated.
```
{% endtab %}

{% tab title="Request" %}
```php
<?php

$request = new GetServersRequest;
$request->withTokenAuth($user->forge_api_key);

// This single request sent will be authenticated.
```
{% endtab %}
{% endtabs %}

### Custom Authenticators

Sometimes the API integration you are building requires multiple ways to authenticate, like a token and a certificate or perhaps authenticating an OAuth 2 API. When the built-in authentication is insufficient, You can build custom authenticators that can be transported between your application and Saloon's requests.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Http\PendingRequest;
use Saloon\Contracts\Authenticator;

class CustomAuthenticator implements Authenticator
{
    public function __construct(
        public string $apiKey,
    ) {
        //
    }

    public function set(PendingRequest $pendingRequest): void
    {
        $pendingRequest->headers()->add('X-API-Key', $this->apiKey);
    }
}
```
{% endtab %}

{% tab title="Usage (Connector)" %}
```php
<?php

$forge = new ForgeConnector;
$forge->authenticate(new CustomAuthenticator('my-api-key'));

// When requests are sent with this connector, the X-API-KEY header is added.

```
{% endtab %}

{% tab title="Usage (Request)" %}
```php
<?php

$request = new GetServersRequest;
$request->authenticate(new CustomAuthenticator('my-api-key'));

// When the request is sent with this connector, the X-API-KEY header is added.
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
You may use custom authenticators in the same way as other authenticators, so you may use the `defaultAuth` method or even authenticate on the fly.
{% endhint %}

### Authenticating APIs that require per-request authentication

Some APIs that you will integrate with require an authentication token, such as a JWT, per request. Usually, this is quite tricky as it requires logic to wrap around your requests. Still, with Saloon, you can create a custom authenticator which makes another request to get the authentication token.

Let's start by creating a custom authenticator. This authenticator will make another request using the same connector and then authenticate the original request with the authentication token.&#x20;

<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeAuthenticator implements Authenticator
{
    public function set(PendingRequest $pendingRequest): void
    {
        // Make sure to ignore the authentication request to prevent loops.

	if ($pendingRequest->getRequest() instanceof AuthRequest) {
	    return;
	}

	// Make a request to the Authentication endpoint using the same connector.

<strong>	$response = $pendingRequest->getConnector()->send(new AuthRequest);
</strong>				
	// Finally, authenticate the previous PendingRequest before it is sent.

<strong>        $pendingRequest->headers()->add('Authorization', 'Bearer ' . $response->json('token'));
</strong>    }
}
</code></pre>

{% hint style="info" %}
Unfortunately, you cannot use the authentication methods like **authenticate** or **withTokenAuth** because you cannot call authenticators inside of each other.&#x20;
{% endhint %}

Next, we will use our authenticator as the default authenticator on the request. If you need to use an API token per user, you should pass a token into the constructor of the connector.

<pre class="language-php"><code class="lang-php">&#x3C;?php

class ForgeConnector extends Connector
{ 
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    protected function defaultAuth(): ?Authenticator
    {
<strong>        return new ForgeAuthenticator;
</strong>    }
}
</code></pre>

Now when we make a request, our authenticator will make an additional request to retrieve the authentication token, and then use that token in the previous request. This way you can send your request like normal.

```php
<?php

$forge = new ForgeConnector;
$response = $forge->send(new GetServersRequest);
```
