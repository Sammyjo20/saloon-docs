# 🔌 Connectors

Connectors are classes where you define the core details of an API, like the Base URL and headers that should be sent with every request. It's a central place to write code that applies to every request. Connectors communicate with the `Sender` (HTTP Client). You can also configure HTTP client options here to be passed directly to the sender.

### Are you building an integration for just one request?

Saloon connectors are great for most API integrations; however, you may not need a connector if you make an API integration with only one request. If this is your scenario, [read through the "Solo Request" section.](../digging-deeper/solo-requests.md) Creating a solo request means you don't need to create a connector.

### Getting Started

First, create a directory for your API integrations. Once you have a chosen directory, create a class that extends the `Connector` abstract class. After that, extend the `resolveBaseUrl` function.

See the example connector for Laravel Forge, an API for server management. We'll name it `ForgeConnector` for the best readability. Let's define our base URL to start.

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

{% hint style="info" %}
Using the Laravel Saloon Plugin? Use the following Artisan command to create a connector!

**php artisan saloon:connector \<Integration Name> \<Connector Name>**
{% endhint %}

### Default Headers and Query Parameters

Most API integrations will have headers that should be shared with every request, like the `Content-Type` or the `Accept` headers. Some API integrations may even have default query parameters to be applied to every request. Saloon allows you to define default properties like these easily.

To add default headers, you can extend the `defaultHeaders` method to your connector. This method expects a keyed array to be returned. You may use an array in the value of a header for multiple header values.

```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
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

You may also add a `defaultQuery` method to your connector to specify default query parameters for every request. This method expects a keyed array to be returned.

```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

    protected function defaultQuery(): array
    {
        return [
            'per_page' => 500, // ?per_page=500
        ];
    }
}
```

### Default HTTP Client Configuration

You may want to define custom options to send to the HTTP client. For example, you may want to register a default timeout of 60 seconds for every request. Saloon uses Guzzle as the default HTTP client, so that you may use any of Guzzle’s options inside the `defaultConfig` method. This method expects a keyed array to be returned.

[Click here to see a list of the available options Guzzle provide.](https://docs.guzzlephp.org/en/stable/request-options.html)

```php
<?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    public function resolveBaseUrl(): string
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

### Using Constructor Arguments

You may add properties to your connector class and use a constructor to provide variables into the connector instance, like an API token. This is great when building SDK-style classes.

{% tabs %}
{% tab title="Definition" %}
<pre class="language-php"><code class="lang-php">&#x3C;?php

use Saloon\Http\Connector;

class ForgeConnector extends Connector
{
    public function resolveBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }

<strong>    public function __construct(
</strong><strong>        protected string $apiToken,
</strong><strong>    ){
</strong><strong>       $this->withTokenAuth($this->apiToken); 
</strong><strong>    }
</strong>}
</code></pre>
{% endtab %}

{% tab title="Usage" %}
```php
$forge = new ForgeConnector('api-token');
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
This example uses a method `withTokenAuth` which is documented on the [authentication ](authentication.md)page.
{% endhint %}
