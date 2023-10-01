# ✉ Requests

The Saloon request class stores the information of a single API request. Within a request, you can set the HTTP Method (GET, POST, etc.) and define the endpoint of that request. You don't have to include the base URL because your connector will provide it for you. You can also define headers, query parameters and HTTP client config. Saloon request classes are reusable, so you can write a request class once and use it multiple times in your application.

### Getting Started

Create a class that is in a similar place to your connector. The class should extend the `Request` abstract class. After that, overwrite the `method` property and set the HTTP method your request needs. You should import the `Saloon\Enums\Method` enum class.

* `protected Method $method = Method::GET;`

After that, extend the `resolveEndpoint` public method. This method should contain the endpoint of the request. You may wish to leave this string blank if you do not have a specific endpoint, like when consuming GraphQL APIs. The endpoint will be combined with the base URL defined within your connector.

See the example request. This request will GET all of the servers from a Laravel Forge account.

{% hint style="info" %}
Using the Laravel Saloon Plugin? Use the following Artisan command to create a request!

**php artisan saloon:request \<Integration Name> \<Request Name>**
{% endhint %}

```php
<?php

use Saloon\Enums\Method;
use Saloon\Http\Request;

class GetServersRequest extends Request
{
    protected Method $method = Method::GET;

    public function resolveEndpoint(): string
    {
        return '/servers';
    }
}
```

{% hint style="info" %}
Typically, you should let Saloon combine the Base URL in your connector with the endpoint in your request, but you can also provide fully qualified URLs in requests which overwrite the base URL if you need to.
{% endhint %}

```php
public function resolveEndpoint(): string
{
    return 'https://some-other-domain.com/endpoint';
}
```

### Using Constructor Arguments

You may add properties to your request class and use a constructor to provide variables into the request instance. Since the request is still a regular class, you may customise it how you like.

For example, If I want to create a request to retrieve an individual server by an ID. I could add a constructor to accept the server ID and use the variable within the endpoint method. This way, I can pass the ID into every request instance.

{% tabs %}
{% tab title="Definition" %}
```php
<?php

use Saloon\Enums\Method;
use Saloon\Http\Request;

class GetServersRequest extends Request
{
    protected Method $method = Method::GET;
    
    public function resolveEndpoint(): string
    {
        return '/servers/' . $this->id;
    }

    public function __construct(
        protected int $id,
    ) { }
} 
```
{% endtab %}

{% tab title="Usage" %}
```php
$request = new GetServerRequest(id: 12345);
```
{% endtab %}
{% endtabs %}

### Default Headers and Query Parameters

Some requests require specific headers or query parameters to be sent. To define default headers on your request, you can extend the `defaultHeaders` method. This method expects a keyed array to be returned. You may use an array in the value of a header for multiple header values. These headers will be merged with the connector’s headers.

```php
<?php

use Saloon\Enums\Method;
use Saloon\Http\Request;

class GetServersRequest extends Request
{
    protected Method $method = Method::GET;

    public function resolveEndpoint(): string
    {
        return '/servers';
    }

    protected function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/json',
            'Accept' => 'application/json',
        ];
    }
}
```

You may also add a `defaultQuery` method to your request to specify default query parameters. This method expects a keyed array to be returned.

```php
<?php

use Saloon\Enums\Method;
use Saloon\Http\Request;

class GetServersRequest extends Request
{
    protected Method $method = Method::GET;

    public function resolveEndpoint(): string
    {
        return '/servers';
    }
    
    protected function defaultQuery(): array
    {
        return [
            'per_page' => 500, // ?per_page=500,
            'page' => 1, // &page=1
        ];
    }
}
```

### Default HTTP Client Config

You may want to define custom options to send to the HTTP Client when creating a request. For example, you may want to register a default timeout of 120 seconds on the request. Saloon uses Guzzle as the default HTTP client, so you may use any of Guzzle’s options inside the`defaultConfig` method. This method expects a keyed array to be returned. The configuration options will be merged with the connector’s config.

[Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

```php
<?php

use Saloon\Enums\Method;
use Saloon\Http\Request;

class GetServersRequest extends Request
{
    protected Method $method = Method::GET;

    public function resolveEndpoint(): string
    {
        return '/servers';
    }
    
    protected function defaultConfig(): array
    {
        return [
            'timeout' => 120,
        ];
    }
}
```
