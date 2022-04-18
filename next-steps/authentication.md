# Authentication

There are several ways to authenticate with an API, most of the time it happens in headers and/or in request data. Saloon has a couple of common built-in authentication methods to help you with authentication, but you can also create your own authenticators for advanced authentication. You can also provide defaults for authentication if you require it.

If you are using one API key across all your requests, then you can also use the **defaultHeaders** and **defaultConfig** methods. This authentication is recommended if you need to authenticate on a per-request/connector basis.

### Token Authentication

Token Authentication is used if the API expects an _Authorization_ header. For example:

```http
"Authorization": "Bearer my-authentication-token"
```

To use the token authentication, just add the `withTokenAuth` method on your requests before sending.

```php
<?php

$request = new GetForgeServerRequest(serverId: 123456);

$request->withTokenAuth('my-token');

$response = $request->send();
```

You can customize the prefix of the Authorization header with the second argument. By default it is set to _Bearer._

{% hint style="info" %}
If you would like to throw an exception if a token is not provided, add the **RequiresTokenAuth** trait to your connector or your request. If it is added to the connector, the trait's functionality will be applied to all requests.
{% endhint %}

#### Default Authentication

If you would like to provide a default, then add the **defaultAuth** method to your connector or request and return an instance of **TokenAuthenticator**.

```php
<?php

use Sammyjo20\Saloon\Http\Auth\TokenAuthenticator;

class GetForgeServerRequest extends SaloonRequest
{
    // ...
    
    protected function defaultAuth(): ?AuthenticatorInterface
    {
        return new TokenAuthenticator('username', 'password');
    }
}
```

### Basic Authentication

Basic authentication is used when the API requires Username + Password basic authentication.

To use basic authentication, just add the `withBasicAuth` method to your requests before sending.

```php
<?php

$request = new GetForgeServerRequest(serverId: 123456);

$request->withBasicAuth('username', 'password');

$response = $request->send();
```

{% hint style="info" %}
If you would like to throw an exception if basic auth is not provided, add the **RequiresBasicAuth** trait to your connector or your request. If it is added to the connector, the trait's functionality will be applied to all requests.
{% endhint %}

#### Default Authentication

If you would like to provide a default, then add the **defaultAuth** method to your connector or request and return an instance of **BasicAuthenticator**.

```php
<?php

use Sammyjo20\Saloon\Http\Auth\BasicAuthenticator;

class GetForgeServerRequest extends SaloonRequest
{
    // ...
    
    protected function defaultAuth(): ?AuthenticatorInterface
    {
        return new BasicAuthenticator('username', 'password');
    }
}
```

### Digest Authentication

Digest authentication is used when the API requires Username + Password + Digest authentication.

To use digest authentication, just add the `withDigestAuth` method to your requests before sending.

```php
<?php

$request = new GetForgeServerRequest(serverId: 123456);

$request->withDigestAuth('username', 'password', 'digest');

$response = $request->send();
```

{% hint style="info" %}
If you would like to throw an exception if digest auth is not provided, add the **RequiresDigestAuth** trait to your connector or your request. If it is added to the connector, the trait's functionality will be applied to all requests.
{% endhint %}

#### Default Authentication

If you would like to provide a default, then add the **defaultAuth** method to your connector or request and return an instance of **DigestAuthenticator**.

```php
<?php

use Sammyjo20\Saloon\Http\Auth\DigestAuthenticator;

class GetForgeServerRequest extends SaloonRequest
{
    // ...
    
    protected function defaultAuth(): ?AuthenticatorInterface
    {
        return new DigestAuthenticator('username', 'password', 'digest');
    }
}
```

### Creating your own authenticators

Sometimes the API integration you are building requires multiple ways to authenticate, like a token and a certificate or perhaps authenticating an Oauth 2 API. When the built-in authentication is not sufficient, You can build custom "authenticators" that can be transported between your application and Saloon's requests.

#### Authenticators

Authenticators are classes that can be added to connectors or requests and can be programmed to add headers, data or even handlers to requests in order to fully authenticate them before they are sent. Authenticators are really powerful as you can accept as much data as you need, and can even provide a default.

To create an authenticator, make a class that implements the **AuthenticatorInterface**. It will require you to create a `set` method. The set method accepts any properties from the constructor and apply it to the request passed through the `set` method.

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:auth** Artisan command to create an authenticator.
{% endhint %}

#### Example Authenticator

Here is an example of an authenticator class. As you can see, I am accepting a single public property called "apiKey" and in the set method, I am using that property and adding a header in the set method of the authenticator.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use Sammyjo20\Saloon\Interfaces\AuthenticatorInterface;

class CustomAuthenticator implements AuthenticatorInterface
{
    public function __construct(
        public string $apiKey,
    ) {
        //
    }

    public function set(SaloonRequest $request): void
    {
        $request->addHeader('X-API-Key', $this->apiKey);
    }
}
```

#### Using your authenticator

After you have created your authenticator, to apply it to your requests or connector, you can use the `withAuth` method. You can even use it on the underlying connector if you are making multiple requests that should be authenticated with the same connector.

```php
<?php

$user = auth()->user();

$request = new GetForgeServerRequest(serverId: 123456);

$request->withAuth(new CustomAuthenticator($user->forge_api_key));

$response = $request->send();

// You can also apply it to the connector once!

$connector = new ForgeConnector;
$connector->withAuth(new CustomAuthenticator($user->forge_api_key));

$requestA = $connector->request(new GetForgeServerRequest(...));
$requestB = $connector->request(new CreateForgeServerRequest(...));
```

#### Default authenticator

You can also provide a default authenticator to use if one is not provided. On your connector or your request, just add the `defaultAuth` method and return an instance of your custom authenticator.

```php
class ForgeConnector extends SaloonConnector
{
    // ...
    
    protected function defaultAuth(): ?AuthenticatorInterface
    {
        return new CustomAuthenticator('generic-forge-api-key');
    }
}
```

{% hint style="info" %}
If you would like to throw an exception if auth is not provided, add the **RequiresAuth** trait to your connector or your request. If it is added to the connector, the trait's functionality will be applied to all requests.
{% endhint %}
