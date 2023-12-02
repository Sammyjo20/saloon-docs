# ✉ Requests

API requests in Saloon each have a class. Within a request class, you define the endpoint, the HTTP method, and properties like headers, query parameters and body.

### Getting Started

Create a class that is in a similar place to your connector. The class should extend the `Request` abstract class. Inside the class, define a `$method` property and a `resolveEndpoint` method. The `resolveEndpoint` method will be combined with the connector's base URL.

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

> If you have installed the Laravel plugin, you can use the **php artisan saloon:request** command to  create a request.&#x20;

### Headers

Some API requests require headers to be sent. To define default headers on your request, you can extend the `defaultHeaders` method. These will be merged with the connector's headers.

```php
class GetServersRequest extends Request
{
    // ...

    protected function defaultHeaders(): array
    {
        return [
            'Content-Type' => 'application/json',
            'Accept' => 'application/json',
        ];
    }
}
```

### Query Parameters

Some API requests require query parameters to be sent. You can extend the `defaultQuery` method to provide these query parameters.

```php
class GetServersRequest extends Request
{
    // ...
    
    protected function defaultQuery(): array
    {
        return [
            'sort' => 'name', // ?sort=name
            'filter[active]' => 'true', // ?filter[active]=true
        ];
    }
}
```

### Constructor Arguments

Since requests are just classes, you can define a constructor to populate the request's properties. For example, if you are getting a specific resource you would need to pass its ID into the request class.

```php
class GetServersRequest extends Request
{
    protected Method $method = Method::GET;
    
    public function __construct(protected readonly int $id) {
        //
    }
    
    public function resolveEndpoint(): string
    {
        return '/servers/' . $this->id;
    }
}
```

```php
$request = new GetServerRequest('5664b05b-8b32-4523-a0d5-837f5080417a');
```
