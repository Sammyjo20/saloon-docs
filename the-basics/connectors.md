# 🔌 Connectors

Connectors are classes which define an API integration's properties like its URL and headers. Any behaviour that should be used on every request like authentication should be defined in a connector. You should have a separate connector for each API integration.

### Getting Started

You should establish a standard place to keep your API connectors. For example in Laravel, a sensible place would be to place them inside the `App/Http/Integrations`folder.&#x20;

Create a new class and extend the abstract `Connector` class. You will then need to define a method `resolveBaseUrl`. This is the URL that points to the API.

```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
}
```

> If you have installed the Laravel plugin, you can use the **php artisan saloon:connector** command to  create a connector.

### Headers

Most API integrations will have common headers used by all of its requests. You can extend the `defaultHeaders` method to define the headers.

<pre class="language-php"><code class="lang-php">class ForgeConnector extends Connector
{
    // ...

<strong>    protected function defaultHeaders(): array
</strong>    {
        return [
            'Content-Type' => 'application/json',
            'Accept' => 'application/json',
        ];
    }
}
</code></pre>

You can also use the `headers` method on a connector instance.

```php
$forge = new ForgeConnector;

$forge->headers()->add('Content-Type', 'text/plain');
```

### HTTP Client Config

The connector uses a HTTP client to send the request. By default, this client is [Guzzle](https://github.com/guzzle/guzzle). If you would like to define config options like timeout then you can extend the `defaultConfig`  method.

[Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

```php
class ForgeConnector extends Connector
{
    // ...

    public function defaultConfig(): array
    {
        return [
            'timeout' => 60,
        ];
    }
}
```

You can also use the `config` method on a connector instance.

```php
$forge = new ForgeConnector;

$forge->config()->add('timeout', 120);
```

### Constructor Arguments

Since connectors are just classes, you can define a constructor to populate its default properties. For example, if the URL changes per user of your application you can define this as a constructor argument.

```php
class ForgeConnector extends Connector
{
    public function __construct(protected readonly int $baseUrl) {
        //
    }

    public function resolveBaseUrl(): string
    {
        return $this->baseUrl;
    }
}
```

```php
$connector = new ForgeConnector('https://forge.laravel.com/api/v1');
```
