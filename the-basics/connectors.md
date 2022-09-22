# Connectors

Connectors are classes that register and store the requirements of a third-party API integration. Within a connector, you can define the base URL, default headers and default configuration. Connectors are only defined once and then requests use a connector to know the base requirements of an API without you having to repeat yourself in each request. Saloon is built on top of Guzzle, so every config option that Guzzle provides is supported by Saloon.

### Getting Started

{% hint style="info" %}
If you are using Laravel, you can use the **php artisan saloon:connector** Artisan command to create a connector.
{% endhint %}

Firstly create a class in your application and extend the base **SaloonConnector** abstract class. Then extend the **defineBaseUrl** public function and define the base URL of the API. You do not have to worry about trailing slashes as Saloon will clean these up for you.

This is an example of a connector for the Laravel Forge service. As you can see, I have defined the base URL. I have also registered some default headers and configuration options.&#x20;

```php
<?php

use Sammyjo20\Saloon\Http\SaloonConnector;

class ForgeConnector extends SaloonConnector
{
    public function defineBaseUrl(): string
    {
        return 'https://forge.laravel.com/api/v1';
    }
    
    public function defaultHeaders(): array
    {
        return [
            'Accept' => 'application/json',
            'X-Custom-Header' => 'Hello',
        ];
    }
    
    public function defaultConfig(): array
    {
       // Guzzle Config Options...
    
        return [
            'timeout' => 30,
        ];
    }
}
```

{% hint style="info" %}
Saloon provides many pre-written plugins to save you from manually defining headers and config variables for common use cases. [Read more about plugins here](../next-steps/plugins.md). Alternatively, to see the full list of Guzzle config options, [click here](https://docs.guzzlephp.org/en/stable/request-options.html).
{% endhint %}

### Available Methods

#### defineBaseUrl()

_Required_

This method should return the base URL of the API. This is typically the hostname of the API and any suffix like `/api/v1`.

#### defaultHeaders()

If defined, this method should return an array of the headers that should be used on every request the connector makes.

#### defaultConfig()

Similar to the `defaultHeaders()` method, if defined - this should return an array of any Guzzle configuration options you would like the connector to use in all of its requests.

#### defaultQuery()

If provided, this method should return an array of the query parameters for the connector to use in all of its requests.

#### defaultData()

If provided, this method should return an array of the default form data for the connector to use in all of its requests.

{% hint style="warning" %}
Requires either the **hasJsonBody, HasFormParams** or **HasMultipartBody** traits. [Read more](attaching-data.md). If you are sending XML data, [click here](attaching-data.md#sending-xml).
{% endhint %}

#### boot($request)

If provided, this method can be used to add extra functionality to the connector. You can use methods like `addHeader(), addConfig(), addHandler() and addResponseInterceptor()`. It is also a useful place to add conditional headers/configuration.
