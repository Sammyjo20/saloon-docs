# 🔐 Authentication

Saloon has built-in authentication methods for the most common authentication types. You should use these on your connector if you would like the authentication to be used on all requests.

### Authorization "Bearer" Tokens

The `withTokenAuth` method can be used when a `Authorization: Bearer` header is required.

{% tabs %}
{% tab title="Definition" %}
```php
class ForgeConnector extends Connector
{
    // ...
    
    public function __construct(string $token)
    {
        $this->withTokenAuth($token, 'Bearer');
    }
}
```
{% endtab %}

{% tab title="Result" %}
```php
$forge = new ForgeConnector('my-token');

// Sends Header: "Authorization: Bearer my-token"
```
{% endtab %}
{% endtabs %}

### Basic Auth (Base64 Encoded)

The `withBasicAuth` method can be used when a `Authorization: Basic` header is required.

{% tabs %}
{% tab title="Definition" %}
```php
class ForgeConnector extends Connector
{
    // ...
    
    public function __construct(string $username, string $password)
    {
        $this->withBasicAuth($username, $password);
    }
}
```
{% endtab %}

{% tab title="Result" %}
```php
$forge = new ForgeConnector('username', 'password');

// Sends Header: "Authorization: Basic ..."
```
{% endtab %}
{% endtabs %}

### Query Parameter

The `withQueryAuth` method can be used to add a query parameter for authentication.&#x20;

{% tabs %}
{% tab title="Definition" %}
```php
class ForgeConnector extends Connector
{
    // ...
    
    public function __construct(string $token)
    {
        $this->withQueryAuth('api-token', $token);
    }
}
```
{% endtab %}

{% tab title="Result" %}
```php
$forge = new ForgeConnector('my-token');

// Sends URL: "https://forge.laravel.com/api/v1?api-token=my-token"
```
{% endtab %}
{% endtabs %}

### Certificate Auth

The `withCertificateAuth` method can be used to authenticate with a custom client-side certificate. An optional password can be provided.

{% tabs %}
{% tab title="Definition" %}
```php
class ForgeConnector extends Connector
{
    // ...
    
    public function __construct()
    {
        $this->withCertificateAuth('/path/certificate.pem', 'password');
    }
}
```
{% endtab %}

{% tab title="Result" %}
```php
$forge = new ForgeConnector();

// Sends Certificate: "/path/certificate.pem"
```
{% endtab %}
{% endtabs %}

### Custom Authentication

If your API uses a different way to authenticate, the recommended way is to use the methods like`defaultHeaders` on your connector to define how the authentication is applied.

```php
class ForgeConnector extends Connector
{
    // ...

    protected function defaultHeaders(): array
    {
        return [
            'X-API-Key' => $this->token,
        ];
    }
}
```

If your authentication requires further logic, you can also use the constructor of the connector to define these options.

```php
class ForgeConnector extends Connector
{
    public function __construct(
        protected readonly string  $certificatePath,
        protected readonly string  $certificatePassword,
        protected readonly ?string $region = null,
    ) {
        $this->withCertificateAuth($certificatePath, $certificatePassword);

        if (isset($region)) {
            $this->headers()->add('X-Region', $region);
        }
    }
}
```

### APIs that require per-request authentication

Some APIs require an authentication token, such as a JWT, to be generated before each request. Usually, this is quite tricky to implement but with Saloon, you can use the `boot` method on your connector to make another request before the original was sent. [Click here to read more.](../conclusion/cookbook.md#authenticating-before-every-request)
