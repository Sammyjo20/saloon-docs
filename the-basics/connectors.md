# Connectors

Connectors are classes that encapsulate the default attributes of an API integration. At the minimum, a connector expects a Base URL to be defined. You can also register default properties that would be shared with all your requests like headers or HTTP client config.

### Getting Started

We firstly recommend creating a directory for your API integrations. Once you have a chosen directory, create a class that extends the `SaloonConnector` abstract class. After that, just extend the `defineBaseUrl` function.

See the example connector for Laravel Forge, an API for server management.

{% hint style="info" %}
If you are using Laravel, Use the artisan command to create a connector for you.

**php artisan saloon:connector \<Integration Name> \<Connector Name>**
{% endhint %}

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class LaravelForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

### Default Headers and Query Parameters

Most API integrations will have common headers that should be shared with every request, like the `Content-Type` or the `Accept` headers. Some API integrations may even have default query parameters to be applied to every request. Saloon allows you to define default properties like these easily.

To add default headers you can use the `defaultHeaders` method to your connector. This method expects a keyed array to be returned. You may use an array in the value of a header for multiple header values.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class LaravelForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
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

You may also add a `defaultQuery` method to your connector to specify default query parameters to be used on every request. This method expects a keyed array to be returned.

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class LaravelForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    public function defaultQuery(): array
    {
        return [
            'per_page' => 500, // ?per_page=500
        ];
    }
}
```

### Default HTTP Client Configuration

When creating a connector, you may also want to define custom options to send to the HTTP Client. For example you may want to register a default timeout of 60 seconds for every request. Saloon uses Guzzle as the HTTP Client so you may use any of Guzzle’s options inside the. `defaultConfig` method. This method expects a keyed array to be returned.

[Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class LaravelForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    public function defaultConfig(): array
    {
        return [
            'timeout' => 60,
        ];
    }
}
```
