# Requests

Saloon's requests are classes that store all the information required to make a request. Within a request, you can define the connector, the HTTP Method (GET, POST, etc.) and the endpoint that you would like to make a request. You can also define headers and query parameters. Traditionally, you would write your HTTP requests each time you need to, but this way, you can write a request class once and use it multiple times in your application.

### Getting Started

Create a class that is in a similar place to your connector. The class should extend the `SaloonRequest` abstract class. After that, create two properties. The first property should be the class name for your connector, the second should be the HTTP method.

* `protected ?string $connector = LaravelForgeConnector::class`
* `protected ?string $method = 'GET'`

You should also extend the `defineEndpoint` public method. This method should contain the endpoint of the request you are making. You may wish to leave this string blank if you do not have a specific endpoint, like when consuming GraphQL APIs. The endpoint will be concatenated with the base URL that you have defined within your connector.

See the example request. This request will GET all of the servers from the Laravel Forge API.

{% hint style="info" %}
If you are using Laravel, Use the artisan command to create a connector for you.

**php artisan saloon:request \<Integration Name> \<Request Name>**
{% endhint %}

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use App\Http\Integrations\LaravelForge\LaravelForgeConnector;

class GetServersRequest extends SaloonRequest
{
    protected ?string $connector = LaravelForgeConnector::class;

    protected ?string $method = 'GET';

    public function defineEndpoint(): string
    {
        return '/servers';
    }
}
```

### Default Headers and Query Parameters

Some requests require specific headers or query parameters to be sent. To define default headers on your request, you can extend the `defaultHeaders` method. This method expects a keyed array to be returned. You may use an array in the value of a header for multiple header values. These headers will be merged with the connector’s headers.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use App\Http\Integrations\LaravelForge\LaravelForgeConnector;

class GetServersRequest extends SaloonRequest
{
    protected ?string $connector = LaravelForgeConnector::class;

    protected ?string $method = 'GET';

    public function defineEndpoint(): string
    {
        return '/servers';
    }

    public function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/json',
            'Accept' => 'application/json',
            'Multiple-Values-Header' => ['Value1', 'Value2'], // Value1;Value2
        ];
    }
}
```

When you want to add query parameters to your request you can extend the `defaultQuery` method. This method expects a keyed array to be returned. These query parameters will be merged with the connector’s query parameters.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use App\Http\Integrations\LaravelForge\LaravelForgeConnector;

class GetServersRequest extends SaloonRequest
{
    protected ?string $connector = LaravelForgeConnector::class;
    
    protected ?string $method = 'GET';
    
    public function defineEndpoint(): string
    {
        return '/servers';
    }
    
    public function defaultQuery(): array
    {
        return [
            'per_page' => 500, // ?per_page=500,
            'page' => 1, // &page=1
        ];
    }
}
```

### Default HTTP Client Config

When creating a request, you may also want to define custom options to send to the HTTP Client. For example you may want to register a default timeout of 120 seconds on the request. Saloon uses Guzzle as the HTTP Client so you may use any of Guzzle’s options inside the. `defaultConfig` method. This method expects a keyed array to be returned. The configuration options will be merged with the connector’s config.

[Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use App\Http\Integrations\LaravelForge\LaravelForgeConnector;

class GetServersRequest extends SaloonRequest
{
    protected ?string $connector = LaravelForgeConnector::class;
    
    protected ?string $method = 'GET';
    
    public function defineEndpoint(): string
    {
        return '/servers';
    }
    
    public function defaultConfig(): array
    {
        return [
            'timeout' => 120,
        ];
    }
}
```

### Using Constructor Arguments

You will often have variables that you want to pass into the request. You may add your own properties to your request class or use a constructor to provide variables into the request instance. Since the request is still a regular class you may customise it how you like.

For example, I want to create a request to retrieve an individual server by an ID. I will add a constructor to accept the server ID and I will concatenate the variable with the endpoint. This way I  can pass the ID into every instance of the request.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonRequest;
use App\Http\Integrations\LaravelForge\LaravelForgeConnector;

class GetServerRequest extends SaloonRequest
{
    protected ?string $connector = LaravelForgeConnector::class;

    protected ?string $method = 'GET';

    public function __construct(protected int $id)
    {
        //
    }
    
    public function defineEndpoint(): string
    {
        return '/servers/' . $this->id;
    }
} 

// 

$request = new GetServerRequest(id: 12345);
```
