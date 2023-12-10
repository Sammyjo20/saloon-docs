# 🔐 Authentication

Saloon has optional authenticator classes to help you with the most common types of authentication. These are classes that can be used on your connector or request.

[You can view all of the built-in authenticators on GitHub.](https://github.com/saloonphp/saloon/tree/v3/src/Http/Auth)

### Authorization "Bearer" Tokens

The `TokenAuthenticator` class can be used to add a `Authorization: Bearer` header to the request. Just extend the `defaultAuth` method and return the authenticator class.

```php
<?php

use Saloon\Http\Auth\TokenAuthenticator;

class ForgeConnector extends Connector
{
    public function __construct(public readonly string $token) {}
    
    protected function defaultAuth(): TokenAuthenticator
    {
        return new TokenAuthenticator($this->token);
    }
}
```

### Basic Auth (Base64 Encoded)

The `BasicAuthenticator` class can be used to add a `Authorization: Basic` header to the request. Just extend the `defaultAuth` method and return the authenticator class.

```php
<?php

use Saloon\Http\Auth\BasicAuthenticator;

class ForgeConnector extends Connector
{
    public function __construct(
        public readonly string $username,
        public readonly string $password
    ){}
    
    protected function defaultAuth(): BasicAuthenticator
    {
        return new BasicAuthenticator($this->username, $this->password);
    }
}
```

### Query Parameter

The `QueryAuthenticator` class can be used to add a query parameter to requests. Just extend the `defaultAuth` method and return the authenticator class.

```php
<?php

use Saloon\Http\Auth\QueryAuthenticator;

class ForgeConnector extends Connector
{
    public function __construct(public readonly string $token) {}
    
    protected function defaultAuth(): QueryAuthenticator
    {
        return new QueryAuthenticator('api-key', $this->token);
    }
}
```

### Certificate Auth

The `CertificateAuthenticator` class can be used to authenticate with a custom client-side certificate. An optional password can be provided.  Just extend the `defaultAuth` method and return the authenticator class.

```php
<?php

use Saloon\Http\Auth\CertificateAuthenticator;

class ForgeConnector extends Connector
{
    public function __construct(
        public readonly string $path,
        public readonly string $password
    ){}
    
    protected function defaultAuth(): CertificateAuthenticator
    {
        return new CertificateAuthenticator($this->path, $this->password);
    }
}
```

### Header Auth

The `HeaderAuthenticator` class can be used to authenticate with a custom header. Just extend the `defaultAuth` method and return the authenticator class.

```php
<?php

use Saloon\Http\Auth\HeaderAuthenticator;

class ForgeConnector extends Connector
{
    public function __construct(public readonly string $token) {}
    
    protected function defaultAuth(): CertificateAuthenticator
    {
        return new HeaderAuthenticator($this->token, 'X-API-KEY');
    }
}
```

### Custom Authentication

If your API integration requires a more complicated authentication process, you can create your own authenticator classes which can be used on your connector. This helps abstract any complicated logic away from the connector keeping it tidy.

```php
<?php

use Saloon\Http\PendingRequest;
use Saloon\Contracts\Authenticator;

class ForgeAuthenticator implements Authenticator
{
    public function __construct(public readonly string $token) {}

    public function set(PendingRequest $pendingRequest): void
    {
        // $pendingRequest->headers()->add(...);
        // $pendingRequest->config()->add(...);
    }
}
```

```php
<?php

class ForgeConnector extends Connector
{
    protected function defaultAuth(): ForgeAuthenticator
    {
        return new ForgeAuthenticator($this->token);
    }
}
```

### The authenticate method

You may also use the `authenticate` method on your connector or request if you would like to use or overwrite an authenticator at runtime.

```php
$forge = new ForgeConnector;

$forge->authenticate(new TokenAuthenticator($user->forge_token));

// $forge->send(...)
```

{% hint style="info" %}
Only one authenticator can be used at the same time.
{% endhint %}
